route53에서 도메인을 등록해서 route53에서 관리하는 방법과, 외부에서 도메인을 구입하여 생성한 hosted zone의 네임서버로 변경하는법 2가지가 있다.

예전에 가비아에서 사놓은 도메인으로 실습 진행

route53은 프리티어가 X→ 매달 비용 발생

### 도메인 구입
<img width="2750" height="1124" alt="Image" src="https://github.com/user-attachments/assets/32de3772-1341-49cc-b4f7-76b0cb0a5095" />

작년에 구입해둔 도메인을 사용할것이다. 기간이 얼마 남지 않았고, 가격이 그새 비싸졌기에 조만간 다른 도메인을 사야 할 것 같다..

### 호스팅 영역 생성
<img width="3024" height="1410" alt="Image" src="https://github.com/user-attachments/assets/8be210a4-834b-4d04-b78e-13774651931d" />

route53의 호스팅 영역> 호스틱 영역 생성 창에서 호스팅 영역 구성을 해준다. 도메인이름은 구입한 도메인명을 사용하면 된다.

### 레코드 설정
호스팅 생성을 하면
<img width="2296" height="280" alt="Image" src="https://github.com/user-attachments/assets/491452cd-1b70-4978-b234-b8313373c9b5" />

이렇게 NS 레코드와 SOA레코드가 생성이 되는데, NS 서버를 가비아에 설정해야한다.

<img width="2984" height="988" alt="Image" src="https://github.com/user-attachments/assets/56528cb8-6a99-4b08-b3a1-70a3b3c6a77d" />
1차부터 하나씩 NS를 옮겨 적으면 되는데, 마지막의 .은 지원을 안하는 경우가 있으므로 빼고 적어야 한다.

그다음 A레코드도 생성해줘야 하는데, A레코드는 도메인을 IPv4 주소와 연결해주는 레코드다.

<img width="3014" height="1394" alt="Image" src="https://github.com/user-attachments/assets/adf71796-bc75-47e2-8a74-ef704298f4d2" />
아래 값에다가 ec2의 IP주소를 넣으면 된다.

### 배포 확인
잠시 기다렸다가 주소창에 도메인을 입력해 보면..

<img width="3012" height="1642" alt="Image" src="https://github.com/user-attachments/assets/2a3196e8-12d6-4c2c-bdda-f833088efae2" />
성공적으로 배포가 진행되었다!!
