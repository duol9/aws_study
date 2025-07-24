1. vpc

<img width="1896" height="1022" alt="Image" src="https://github.com/user-attachments/assets/bd5eb7b2-3d88-4805-9672-d3ea41ecae12" />

VPC는 AWS 클라우드 내에서 사용자가 정의하는 격리된 가상 네트워크 환경을 제공하기 때문에, EC2 인스턴스를 생성하면 반드시 이 VPC 안에 속하게 됨.

**서브넷(Subnet) 생성 :** VPC를 여러 서브넷으로 나누어 퍼블릭 서브넷(인터넷 연결 가능)과 프라이빗 서브넷(인터넷 연결 불가)으로 구성 가능

프라이빗 서브넷 → rdb, 애플리케이션 서버 등이 존재

**라우팅 테이블(Route Table):** 트래픽 컨트롤러 역할을 수행, 네트워크 트래픽이 어디로 이동할지 규칙을 정의

2. 라우팅 테이블 설정

<img width="1994" height="902" alt="Image" src="https://github.com/user-attachments/assets/3e7e32a9-ba82-4472-83c3-f489c1e5fef2" />

3. 보안그룹 설정

인바운드 설정

인바운드 : 외부에서 네트워크 내부로 들어오는 트래픽을 의미 (수신)

<img width="2048" height="553" alt="Image" src="https://github.com/user-attachments/assets/20e5092f-ecc4-431b-9fbf-156cf18f904d" />

아웃바운드 설정

아웃바운드: 네트워크 내부에서 외부로 나가는 트래픽 (송신)

<img width="2048" height="286" alt="Image" src="https://github.com/user-attachments/assets/90b74f5d-8dba-4820-9020-eee86d9f9fc8" />

지금은 모든 IP를 허용 중이지만, 진짜로 구성할 땐 보안을 위해 특정 IP만 허용을 해야 함. 

4. EC2 생성
<img width="962" height="370" alt="Image" src="https://github.com/user-attachments/assets/9f481784-14b6-4b33-9b75-6b00ff3fbd28" />
    

5. Elastic IP 설정 
EC2 서버가 재시동 될 때마다 IP가 바뀌는 걸 방지해주는, IP 주소 고정시키는 친구.
