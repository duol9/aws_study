route53에서 도메인을 등록해서 route53에서 관리하는 방법과, 외부에서 도메인을 구입하여 생성한 hosted zone의 네임서버로 변경하는법 2가지가 있다.

예전에 가비아에서 사놓은 도메인으로 실습 진행

route53은 프리티어가 X→ 매달 비용 발생

### 도메인 구입
![image.png](attachment:628695f3-f2cf-404e-8d52-fc40e10797a0:image.png)

작년에 구입해둔 도메인을 사용할것이다. 기간이 얼마 남지 않았고, 가격이 그새 비싸졌기에 조만간 다른 도메인을 사야 할 것 같다..

### 호스팅 영역 생성
![image.png](attachment:7bb29a74-efaf-4817-a3b3-2ff21f7870c3:image.png)

route53의 호스팅 영역> 호스틱 영역 생성 창에서 호스팅 영역 구성을 해준다. 도메인이름은 구입한 도메인명을 사용하면 된다.

### 레코드 설정
호스팅 생성을 하면
![image.png](attachment:4c464900-2c77-4314-8407-2c0898076d2f:image.png)

이렇게 NS 레코드와 SOA레코드가 생성이 되는데, NS 서버를 가비아에 설정해야한다.

![image.png](attachment:d2006d39-fbc7-46bd-85d6-c64f2cf179f3:image.png)

1차부터 하나씩 NS를 옮겨 적으면 되는데, 마지막의 .은 지원을 안하는 경우가 있으므로 빼고 적어야 한다.

그다음 A레코드도 생성해줘야 하는데, A레코드는 도메인을 IPv4 주소와 연결해주는 레코드다.

![image.png](attachment:0d0d9b5f-a2ed-4df2-94d9-2d66a6dab1d6:image.png)

아래 값에다가 ec2의 IP주소를 넣으면 된다.

### 배포 확인
잠시 기다렸다가 주소창에 도메인을 입력해 보면..

![image.png](attachment:32446380-57b1-4d1f-827b-0a03ef664d23:image.png)

성공적으로 배포가 진행되었다!!