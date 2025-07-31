# GitHub Actions와 AWS(EC2, ECR)로 Spring Boot CI/CD 파이프라인 구축하기

## 파이프라인 구축 과정

### **1. AWS 인프라 구축**

우선 애플리케이션이 실행될 클라우드 환경을 구성한다.

1. **IAM**
    
    파이프라인의 보안을 위해 **최소 권한 원칙**을 적용한 IAM 엔티티를 생성해야 한다.
    
    - **IAM 사용자 (for GitHub Actions):** GitHub Actions Runner가 ECR에 이미지를 푸시할 때 사용할 전용 사용자. `AmazonEC2ContainerRegistryFullAccess` 정책을 부여하고 **액세스 키**를 발급한다.
    - **IAM 역할 (for EC2):** EC2 인스턴스가 ECR에서 이미지를 풀(pull)할 때 사용할 역할. 신뢰 엔티티를 `EC2`로, 권한은 `AmazonEC2ContainerRegistryReadOnly`로 설정한다.
2. **ECR: Docker 이미지 레지스트리**
    1. **ECR vs Docker Hub**
        
        Docker Hub와 ECR은 모두 컨테이너 이미지를 저장하는 레지스트리지만, AWS 환경에서는 ECR을 사용하는 것이 몇 가지 명확한 이점을 제공한다.
        
        - **보안 및 권한 관리:** ECR은 AWS IAM과 통합되어, 역할을 기반으로 한 세분화된 접근 제어가 가능하다. 이는 단순 계정 기반의 Docker Hub보다 높은 수준의 보안을 제공한다.
        - **성능:** EC2 인스턴스에서 ECR의 이미지를 전송할 때 AWS 내부 네트워크를 사용하므로 더 빠르다.
        - **AWS 생태계와의 통합:** 향후 ECS, EKS 등 다른 AWS 컨테이너 서비스로 확장할 경우, ECR을 사용하는 것이 유연한 아키텍처 구성에 유리하다.
        
        이러한 이유로, AWS 기반의 파이프라인을 구축하는 이번 과제에서는 ECR을 사용하게 되었다.
        
    2. 빌드된 Docker 이미지를 저장할 프라이빗 레지스트리를 생성한다. 리포지토리 이름은 관리의 일관성을 위해 프로젝트명과 동일하게 지정한다.
3. **EC2: 애플리케이션 서버**
    1. **인스턴스 생성:** `Amazon Linux 2`, `t2.micro` 타입으로 인스턴스를 생성하고 SSH 접속용 **키 페어**를 발급한다.
    2. **보안 그룹:** 인바운드 규칙으로 `SSH(22)`와 애플리케이션 포트인 `TCP(8080)`을 허용한다.
    3. **서버 설정:** 인스턴스에 접속하여 Docker를 설치하고, 위에서 생성한 IAM 역할을 인스턴스에 연결한다.
        
        ```bash
        # Amazon Linux 기준
        sudo yum update -y
        sudo yum install -y docker
        sudo service docker start
        sudo usermod -a -G docker ec2-user # 매번 sudo 없이 docker 명령어 사용하기 위함
        ```
        

---

### **2. 프로젝트 및 GitHub 설정**

GitHub 리포지토리에 자동화를 위한 구성을 추가한다.

1. **Dockerfile**
    
    Dockerfile은 애플리케이션의 실행 환경과 빌드 결과물을 하나의 이미지로 패키징하는 설계도 역할을 하는 파일이다. 아래와 같이 작성해서 루트 디렉토리에 저장한다.
    
    ```docker
    # 베이스 이미지로 Java 17 버전을 사용
    FROM openjdk:17-jdk-slim
    
    # jar 파일이 생성될 경로를 변수로 지정
    ARG JAR_FILE=build/libs/*.jar
    
    # jar 파일을 이미지 안의 app.jar로 복사
    COPY ${JAR_FILE} app.jar
    
    # 애플리케이션 실행 명령어
    ENTRYPOINT ["java","-jar","/app.jar"]
    ```
    
2. **GitHub Secrets**
    
    AWS 자격 증명이나 SSH 키와 같은 정보는 GitHub Secrets를 통해 안전하게 관리해야 한다.
    
    | **Secret 이름** | **값** | **설명** |
    | --- | --- | --- |
    | `AWS_ACCESS_KEY_ID` | 1단계에서 저장해 둔 IAM 사용자의 액세스 키 ID | AWS 인증용 |
    | `AWS_SECRET_ACCESS_KEY` | 1단계에서 저장해 둔 IAM 사용자의 비밀 액세스 키 | AWS 인증용 |
    | `AWS_REGION` | ap-northeast-2 (또는 본인이 사용하는 AWS 리전) | AWS 리전 정보 |
    | `AWS_ACCOUNT_ID` | AWS 계정 ID (12자리 숫자) | ECR 접속용 |
    | `EC2_HOST` | 1단계에서 생성한 EC2 인스턴스의 퍼블릭 IPv4 주소 | 배포할 서버 주소 |
    | `EC2_USERNAME` | ec2-user (Amazon Linux) 또는 ubuntu (Ubuntu) | EC2 접속 계정 |
    | `SSH_PRIVATE_KEY` | 1단계에서 다운로드한 .pem 키 파일의 내용 전체 | EC2 서버에 SSH로 접속하기 위한 개인 키 |

### **3. GitHub Actions 워크플로우: 자동화 파이프라인 정의**

`.github/workflows/deploy.yml`: CI/CD의 모든 로직을 담고 있는 핵심 파일.

```yaml
name: Spring Boot CI-CD with AWS
on:
  push:
    branches: [ "main" ]
env:
  AWS_REGION: ${{ secrets.AWS_REGION }}
  ECR_REPOSITORY: spring-cicd-practice
jobs:
  build:
    name: CI
    runs-on: ubuntu-latest
    outputs:
      image_tag: ${{ steps.build-image.outputs.image_tag }}
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      - run: chmod +x gradlew
      - run: ./gradlew build
      - uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      - id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1
      - id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          echo "image_tag=${{ env.IMAGE_TAG }}" >> $GITHUB_OUTPUT
  deploy:
    name: CD
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            ECR_URI="${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ env.AWS_REGION }}.amazonaws.com"
            IMAGE_NAME="${ECR_URI}/${{ env.ECR_REPOSITORY }}:${{ needs.build.outputs.image_tag }}"
            aws ecr get-login-password --region ${{ env.AWS_REGION }} | docker login --username AWS --password-stdin ${ECR_URI}
            docker pull ${IMAGE_NAME}
            if [ $(docker ps -q --filter "name=spring-app") ]; then
              docker stop spring-app
              docker rm spring-app
            fi
            docker run -d --name spring-app -p 8080:8080 ${IMAGE_NAME}
```

1. **CI Job (build): 빌드 및 패키징**
    
    `build` Job은 소스 코드를 실행 가능한 Docker 이미지로 변환하는 역할을 수행한다. 코드 체크아웃부터 빌드, ECR 로그인, 이미지 푸시까지의 과정을 담당하며, 후속 `deploy` Job에서 사용할 `image_tag`를 `outputs`으로 출력한다.
    
2. **CD Job (deploy): 배포 자동화**
    
    `deploy` Job은 `needs: build`를 통해 `build` Job의 성공을 전제로 실행된다. `build` Job에서 전달받은 `image_tag`와 Secrets 정보를 조합하여 ECR의 전체 이미지 URI를 구성한다. 이후 SSH를 통해 EC2 서버에 접속하여 ECR 로그인, 이미지 풀, 기존 컨테이너 교체, 새 컨테이너 실행까지의 배포 스크립트를 수행한다.
