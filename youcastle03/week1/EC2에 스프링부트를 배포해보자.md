### EC2 생성

EC2 창의 인스턴스 시작을눌러 인스턴스를 생성할 수 있다.

<img width="768" height="352" alt="Image" src="https://github.com/user-attachments/assets/ad012020-e84e-4c9d-a362-2518de34cacc" />


1. 리전 선택: 애플리케이션의 사용자들의 위치와 지리적으로 가까운 리전을 선택하는 것이 좋기에 아시아 태평양(서울)로 지정

<img width="676" height="734" alt="Image" src="https://github.com/user-attachments/assets/57af6eaa-e734-439a-ada5-713b81bb0cf0" />

2. 이름은 사용할 이름으로

<img width="1496" height="216" alt="Image" src="https://github.com/user-attachments/assets/f6e16ac6-3391-49f4-8f90-cd488eddf78c" />

3. OS는 Ubuntu로 진행해보자

<img width="1958" height="1298" alt="Image" src="https://github.com/user-attachments/assets/eef9fc55-1906-463e-a240-7bd60a8b48f7" />

window나 mac OS보다는 가벼운 Ubuntu를 많이 사용한다고한다.

4. 인스턴스 유형
   인스턴스란 AWS에서 제공하는 가상 서버(컴퓨팅 자원)라고 생각하면 된다.
   상황에 맞게 선택하면 되지만 프리티어에 해당하는 t2.micro를 선택했다.

<img width="1884" height="414" alt="Image" src="https://github.com/user-attachments/assets/943cc58c-e2a4-419d-9324-c9af26dbd6e7" />

5. 키페어
   키페어는 비밀번호라 생각하면 된다.

<img width="1212" height="1138" alt="Image" src="https://github.com/user-attachments/assets/9e64c815-609f-4a55-b9a8-1a2a68645862" />

간단하게 이렇게 설정해주자

6. 보안 그룹
   보안그룹이란 AWS에서의 네트워크 보안을 의미한다.
   인바운드 트래픽(외부에서 EC2 인스턴스로), 아웃바운드 트래픽(EC2에서 외부로)에서 어떤 트래픽만 허용할지 설정할 수 있다.

   네트워크 설정에서 **편집**을 누르자. 그리고 인바운드 보안 그룹 규칙에서 ssh는 기본으로 열려있고, HTTP와 HTTPS를 추가하자


<img width="1934" height="1096" alt="Image" src="https://github.com/user-attachments/assets/50925914-33e1-4ed0-b874-a6db1ce98f8f" />

7. 스토리지

<img width="1964" height="802" alt="Image" src="https://github.com/user-attachments/assets/8d5ea84d-a96f-4b14-8e6f-a76251e8adcf" />


스토리지는 파일을 저장하는 공간이다. 프리티어가 가능한 최대인 30으로 설정해보자.

8. 실행
   인스턴스 시작을 누르면 인스턴스 시작이 된다!!

9. 탄력적 IP 설정
   EC2 인스턴스를 생성하면 IP를 할당 받는데, 이 IP는 임시적인 IP라 EC2를 중지시켰다가 다시 실행하면 바뀐다. 이럴때마다 IP가 계속 변하면 당연히 불편하다. 그래서 고정 IP를 할당 하기 위해 탄력적 IP를 설정한다.

<img width="2964" height="884" alt="Image" src="https://github.com/user-attachments/assets/d6b846f4-90cd-4237-a2c9-dabdbce173c6" />

여기서 설정하면 된다. 그냥 기본값 나오는거 생성누르면 된다.

<img width="2914" height="1024" alt="Image" src="https://github.com/user-attachments/assets/36d1ee68-4fe4-45e5-81cc-3fd4e4f067cf" />
여기서 아까 생성한 EC2 인스턴스와 연결하면 된다.

### Spring boot 띄우기

1. ssh나 내부 콘솔에서 접속후 자바 설치

```bash
$ sudo apt update && /
sudo apt install openjdk-21-jdk -y
```

2. 깃허브 프로젝트 클론

```bash
$ git clone ~~
$ cd ~~
```

3. 서버 실행

```bash
$ ./gradlew clean build #
$ cd ~/ec2-spring-boot-sample/build/libs
$ sudo java -jar 프로젝트이름-0.0.1-SNAPSHOT.jar
```

아직 ci-cd를 구성하지 않았기때문에 코드수정이 생길때 마다 추가로 서버를 내리고 다시 명령어를 입력해야하는데(git pull등) 이 작업을 반복하기는 귀찮다.

```bash
#!/bin/sh  
PROJECT_PATH=/home/ubuntu/step2  # 프로젝트가 위치한 루트 경로
PROJECT_NAME=spring-gift-point  # 프로젝트 디렉토리 이름
BUILD_PATH=build/libs  # Gradle 빌드 결과물이 생성되는 하위 경로

JAR_NAME=$(basename $PROJECT_PATH/$PROJECT_NAME/.*jar)

cd $PROJECT_PATH/$PROJECT_NAME  # 프로젝트 디렉토리로 이동

echo ">Git pull"
git pull origin 브랜치  # Git에서 특정 브랜치의 최신 코드를 pull 

echo ">Build 시작"
./gradlew bootJar  # Gradle을 이용해 Spring Boot JAR 빌드 수행

echo ">디렉토리 이동"
cd $PROJECT_PATH  # 다시 상위 디렉토리로 이동 (빌드 결과 복사를 위해)

echo ">Build 파일 복사"
cp $PROJECT_PATH/$PROJECT_NAME/$BUILD_PATH/*.jar $PROJECT_PATH  # 빌드된 .jar 파일을 상위 디렉토리로 복사

echo ">실행중 PID 확인"
CURRENT_PID=$(pgrep -f $JAR_NAME)  # 현재 실행 중인 애플리케이션의 PID 확인

if [ -z $CURRENT_PID ]; then
    echo " >실행중인 애플리케이션이 없어서 바로 실행됩니다..\n"  # 실행 중인 프로세스가 없으면 메시지 출력
else
    echo "실행중인 애플리케이션이 있어서 이를 종료합니다. [PID = $CURRENT_PID]\n"  # 실행 중이면 종료 메시지
    kill -15 $CURRENT_PID  # 프로세스를 종료 (SIGTERM)
    sleep 5  # 5초 대기
fi

echo ">새 애플리케이션 배포"
JAR_NAME=$(ls -tr $PROJECT_PATH/$PROJECT_NAME/$BUILD_PATH | grep *.jar | tail -n 1)  # 가장 최신의 .jar 파일 이름을 가져옴
nohup java -jar $JAR_NAME &  # 백그라운드에서 새로운 애플리케이션 실행

```

다음과 같이 쉘스크립트를 작성하면 서버를 끄고 내리고 하는 작업까지 모드 스크립트 하나로 가능하다.