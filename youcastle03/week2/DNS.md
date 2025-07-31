https://www.youtube.com/watch?v=pEtbC6dYaiA&t=1229s 본 유튜브 영상을 정리한 내용입니다.

## 1. DNS란 무엇인가?

- **Domain Name System (DNS)**
    
    사람이 읽기 쉬운 도메인(예: `www.example.com`)을 컴퓨터가 이해하는 IP 주소(예: `192.0.2.1`)로 변환
    
- **왜 필요한가?**
    - 숫자(IP)만으로는 외우기 어려움
    - 웹·메일·FTP 등 모든 인터넷 서비스의 기본

---

## 2. DNS의 핵심 개념

- **도메인(Domain)**
    - APEX 도메인: `example.com` (가장 상위, 추가 문자열 없음)
    - 서브도메인: `youcastle.example.com` 등
- **레코드(DNS Record)**
    - A 레코드: IPv4
    - AAAA 레코드: IPv6
    - MX 레코드: 메일 서버
    - CNAME, TXT 등 다수
- **Zone & Zone File**
    - Domain Zone: 한 도메인(및 서브도메인)의 레코드 집합
    - Zone File: Zone 정보를 담은 텍스트 파일
- **Name Server (NS)**
    - Authoritative NS: 원본 Zone File 보유
    - Non-Authoritative(NS 캐시): 조회된 결과 저장
- **DNS Resolver**
    - 클라이언트 요청을 받아 순차 조회 수행
    - 주로 ISP나 클라우드 서비스 제공

---

## 3. DNS의 규모와 중요성

- 2024년 기준 전 세계 도메인 약 **3억 6천만 개**
- 거의 모든 HTTPS 통신에서 매번 활용
- 초고가용성·저지연 구조 설계 필수

---

## 4. 계층적 구조 (Hierarchy)

```
. (Root)
 └─ .kr / .com / .org (TLD)
     └─ example.com (Second-level)
         └─ youcastle.example.com (Subdomain)

```

- **루트(Root)**: 최상위, 13개의 주체(A~M) 관리
- **TLD (Top-Level Domain)**
    - 일반(gTLD): `.com`, `.net`, `.org`
    - 국가(ccTLD): `.kr`, `.jp`, `.uk`
    - 신규(New gTLD): `.app`, `.tech`, `.xyz`
    - 관리: 각 Registry(예: Verisign, KISA)

---

## 5. DNS 조회 과정 (예: `youcastle.example.kr`)

1. **클라이언트 → Resolver**
    
    “youcastle.example.kr의 IP가 뭐야?”
    
2. **Resolver → Root NS**
    
    Root Hints 파일로 루트 서버 목록 확인
    
3. **Resolver → TLD NS**
    
    `.kr`을 관리하는 KISA(NS)로 요청
    
4. **Resolver → Authoritative NS**
    
    `example.kr` Zone을 가진 NS로 요청
    
5. **Resolver → 최종 레코드 조회**
    
    `youcastle` 서브도메인 A 레코드 반환 → 클라이언트에 전달
    

---

## 6. 도메인 등록 절차

1. **Domain Registrar**
    - ICANN 인증, Registry에 등록 권한 보유
    - 예: 가비아, GoDaddy, Cafe24, AWS Route 53
2. **NS(Server) 설정**
    - 자체 NS 운영 or DNS Hosting 서비스 이용
    - 등록 시 TLD Registry에 NS 서버 주소 등록
3. **유지·관리**
    - 레코드 추가·수정: Zone File 편집 or 관리 콘솔 사용
    - TTL, 보안(DNSSEC) 옵션 설정

