# 주제: Standard-ACL 설정하기

## 🌐 학습목표
- PC0에서 PC1로 가는 통신을 라우터에서 차단하기

---

## 🌐 1. ACL(Access Control List) 핵심 개념
1) Standard ACL(식별번호: 1~99) << 이번 차시 실습
   - 오직 출발지 IP만 보고 차단 및 허용을 결정
   - 목적지 근처 라우터에 규칙을 적용하는 것이 원칙

2) Extended ACL(식별번호: 100~199)
   - 출발지 IP, 목적지 IP, 프로토콜(TCP/UDP/ICMP), 포트번호까지 정밀하게 확인
   - 출발지 근처 라우터에 규칙을 적용하는 것이 원칙
  
3) `Inbound` vs `Outbound`
   - Inbound: 패킷이 라우터 인터페이스 내부로 들어올 때 검사
   - Outbound: 패킷이 라우터 인터페이스 밖으로 나갈 때 검사
  
---

## 🌐 2. 실습: PC0만 차단하기

- PC0의 통신만 차단하고, 나머지 IP는 허용하는 ACL 적용하기

  ▽ 실습 환경

  <img width="446" height="320" alt="스크린샷 2026-08-05 223922" src="https://github.com/user-attachments/assets/e8fceca8-9301-4698-9ef0-991d922c5c51" />


- Router1의 CLI에 다음 코드 입력
  
  ```bash
  enable
  conf t

  # 1. Standard ACL 생성 (번호: 10(1~99 中 임의로 번호 할당))
  access-list 10 deny host 192.168.10.2
  access-list 10 permit any
  
  # 2. 인터페이스에 ACL 적용 (목적지 LAN 포트에 Outbound로 적용-라우터에서 PC1로 패킷을 내보내줘야하기 때문)
  interface gigabitEthernet 0/0/0
  ip access-group 10 out
  exit
  ```
- Router1에 ACL 설정이 잘 됐는지 리스트 확인하기
  ```bash
  show access-lists
  ```
  ▽ 결과
  
  <img width="325" height="54" alt="스크린샷 2026-08-15 112849" src="https://github.com/user-attachments/assets/66379671-c52c-4013-b567-a6c6d7e11129" />


- PC0에서 PC1으로 핑 테스트하기
  
  ▽ 결과

  <img width="642" height="359" alt="image" src="https://github.com/user-attachments/assets/403c0424-8121-4bfd-9d80-966f74881eb1" />

  - 초록: ACL 적용 전(통신 원활)
  - 빨강: ACL 적용 후(Destination host Unreachable이 잘 뜨는 것 확인 완료)
  
