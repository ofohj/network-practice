# 주제: NAT 및 PAT 설정하기

# 🌐 실습 전 기본 개념
## NAT 설정의 필요성
- IPv4 주소의 고갈 때문에
- IP 주소는 집, 회사 등 내부 네트워크 안에서만 쓰는 **사설 IP**와 전 세계에서 유일한 인터넷 통신용 **공인 IP**가 있다.
- 그런데 사설 IP는 보안상의 이유로 인터넷으로 나갈 수 없다.
- 이처럼 내부에서 사설 IP를 쓰던 PC가 외부 인터넷으로 나갈 때, 라우터가 사설 IP를 통신가능한 공인 IP로 바꿔주는 작업을 NAT 설정이라고 한다.
- 장점: 내부 사설 IP를 외부로부터 숨겨주기 때문에 보안 효과도 있다!

## PAT(Port Address Translation)란?
- NAT의 발전된 형태로, 공인 IP 1개로 수십~수천대의 PC가 동시에 인터넷을 쓸 수 있게 만드는 기술(공유기의 원리)
- NAT의 한계: 기본 NAT는 1:1로 IP를 바꾼다. 즉, PC가 10대 있으면 공인 IP도 10개가 필요하다
- PAT의 등장: IP 뒤에 붙는 포트 번호를 이용해 하나의 공인 IP를 여럿이서 나누어 쓴다.

---

# 🌐 실습 순서
## 1. 장비 배치 및 케이블 연결
장비 구성은 다음과 같다.

<img width="444" height="334" alt="스크린샷 2026-08-24 233503" src="https://github.com/user-attachments/assets/ffa6086d-255d-41c8-b2da-263c03812ea7" />

---

## 2. IP 주소 체계 설계 및 설정

### 1) IP 대역 정리
- 내부 사설망: 192.168.10.0/24
- 라우터 간 공인망 대역: 200.1.1.0/24
- 외부 서버망: 192.168.20.0/24

### 2) 호스트 IP 설정
- PC0 설정
  - IP 주소: 192.168.10.10
  - G/W: 192.168.10.1
- 서버 설정
  - IP 주소: 192.168.20.100
  - G/W: 192.168.20.1

---

## 3. 라우터 IP 설정 및 라우팅

### 1) Router0 설정
```bash
enable
conf t

# 내부 인터페이스(PC0 쪽)
interface gigabitEthernet 0/0/0
ip address 192.168.10.1 255.255.255.0
no shutdown
exit

# 외부 인터페이스(Router1 쪽)
interface gigabitEthernet 0/0/1
ip address 200.1.1.1 255.255.255.0
no shutdown
exit

# 라우팅
ip route 192.168.20.0 255.255.255.0 200.1.1.2
```

### 2) Router1 설정
```bash
enable
conf t

# 외부 인터페이스(Router0 쪽)
interface gigabitEthernet 0/0/1
ip address 200.1.1.2 255.255.255.0
no shutdown
exit

# 서버쪽 인터페이스
interface gigabitEthernet 0/0/0
ip address 192.168.20.1 255.255.255.0
no shutdown
exit

# 라우팅
ip route 192.168.10.0 255.255.255.0 200.1.1.1
```

---

## 4. 통신 점검하기 (ping)

<img width="678" height="494" alt="스크린샷 2026-08-24 234114" src="https://github.com/user-attachments/assets/6b81212d-f926-4afb-98f8-8b2742defa6e" />

<img width="439" height="300" alt="스크린샷 2026-08-24 234118" src="https://github.com/user-attachments/assets/523475c2-8ec0-4cd4-a19f-1d7d8b80a368" />

---

## 5. NAT 설정하기

### 1) Router0 설정

```bash
enable
conf t

# 1. 1:1 매핑 규칙 생성(내부 사설 IP 192.168.10.10 -> 공인 IP 200.1.1.100)
ip nat inside source static 192.168.10.10 200.1.1.100

# 2. 내부 인터페이스 지정
interface gigabitEthernet 0/0/0
ip nat inside
exit

# 3. 외부 인터페이스 지정
interface gigabitEthernet 0/0/1
ip nat outside
end
```

### 2) NAT 동작 검증하기

**(1) 핑 통신 확인(PC0 -> 서버)**

<img width="465" height="182" alt="스크린샷 2026-08-25 000732" src="https://github.com/user-attachments/assets/e747feb7-1db1-4dc4-b644-264b3822923f" />


**(2) NAT 변환 테이블 확인하기**

- 명령어: `show ip nat translations`
- 결과 해석
  - Inside Local: 변환 전 출발지 IP(pc0의 사설 IP)
  - Inside Global: 변환 후 출발지 IP(PC0에서 외부로 나갈 때 바뀐 공인 IP)
  - Outside local/global: 목적지 IP
  
  <img width="590" height="180" alt="스크린샷 2026-08-25 000438" src="https://github.com/user-attachments/assets/def53762-795c-4431-8672-848507b799d8" />

---

## 6. PAT 설정하기

### 1) 기존 NAT 설정 삭제하기
```bash
enable
conf t

# NAT 규칙 삭제
no ip nat inside source static 192.168.10.10 200.1.1.100
```

### 2) PAT 설정
```bash
# 1. NAT 대상이 될 사설 IP 대역 지정
access-list 1 permit 192.168.10.0 0.0.0.255

# 2. PAT 설정
ip nat inside source list 1 interface gigabitEthernet 0/0/1 overload
end
```

### 3) PAT 동작 검증하기

**(1) 핑 통신 확인(PC0 -> 서버)**

<img width="503" height="182" alt="스크린샷 2026-08-25 000915" src="https://github.com/user-attachments/assets/04d5a469-d550-4037-9ddb-7b102bdc3f33" />


**(2) NAT 변환 테이블 확인하기**

<img width="634" height="110" alt="스크린샷 2026-08-25 000943" src="https://github.com/user-attachments/assets/32742f3f-0abb-4955-ac75-90c194e11533" />

---
# 🎉 총정리

| 구분 | NAT | PAT |
| --- | --- | --- |
| 변환 방식 | 1:1 매핑 | N:1 매핑 |
| 공인 IP 개수 | 여러 개 필요 | 1개여도 충분 |
| 포트 사용 여부 | 포트번호 사용 X | 포트 번호를 이용해 패킷 식별 |
| 동시 접속 제한 | 준비된 공인 IP 개수만큼만 접속 가능 | 공인 IP 1대로 여러 대 접속 가 |
| cisco 명령어 | - | 명령어 끝에 overload 붙이기!! |

