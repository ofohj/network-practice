# 주제: 정적 및 동적 라우팅 실습하기

## 💻 실습 목표
1. 라우터 2대를 서로 연결하여 네트워크 확장하기
2. 정적 라우팅 설정하기
3. 동적 라우팅(OSPF) 설정으로 전환하기

---

## 1. 네트워크 토폴로지 구성
💡 본사, 지사 총 두 네트워크를 구성하여 각 네트워크의 라우터를 연결해 통신을 가능하게 만든다.
💡 각 장비는 Copper Straight-Through(검은색 실선 케이블) 케이블로 연결하기

- **본사 네트워크**
  - PC0: IP 192.168.10.10 / Gateway 192.168.10.1
  - Switch0 (2960): PC0 ↔ Fa0/1 연결
  - Router0 (4331): Switch0 Fa0/24 ↔ Router0 Gi0/0/0 연결
   
- **지사 네트워크**
  - PC1: IP 192.168.20.10 / Gateway 192.168.20.1
  - Switch1 (2960): PC1 ↔ Fa0/1 연결
  - Router1 (4331): Switch1 Fa0/24 ↔ Router1 Gi0/0/0 연결

- **라우터 대 라우터 연결(WAN)**
  - Router0 Gi0/0/1 ↔ Router1 Gi0/0/1

### ▼ 결과

<img width="446" height="320" alt="스크린샷 2026-08-05 223922" src="https://github.com/user-attachments/assets/f3d76923-d8d1-4186-91ba-aff0ffe5dded" />


---

## 2. 라우터 IP 설정하기(CLI 환경)

- **본사 라우터 IP 설정**
  - Router0 (4331): Switch0 Fa0/24 - Router0 Gi0/0/0 연결
    - Gi0/0/0 IP: 192.168.10.1 / 255.255.255.0 (라우터 0의 Gi0/0/0 포트에 IP 주소와 서브넷 마스크를 다음과 같이 할당하라)
    - 라우터 0의 CLI 창에 아래와 같이 입력
    ```bash
    enable
    configure terminal
  
    # 1. 내부 LAN 인터페이스 설정
    interface gigabitEthernet 0/0/0
     ip address 192.168.10.1 255.255.255.0
     no shutdown
    exit
  
    # 2. 라우터 간 WAN 인터페이스 설정
    interface gigabitEthernet 0/0/1
     ip address 10.10.10.1 255.255.255.252
     no shutdown
    exit
    ```
    
- **지사 라우터 IP 설정**
  - Router1 (4331): Switch1 Fa0/24 ↔ Router1 Gi0/0/0 연결
    - Gi0/0/0 IP: 192.168.20.1 / 255.255.255.0
    - 라우터 1의 CLI 창에 아래와 같이 입력
    ```bash
    enable
    configure terminal
    
    # 1. 내부 LAN 인터페이스 설정 (지사)
    interface gigabitEthernet 0/0/0
     ip address 192.168.20.1 255.255.255.0
     no shutdown
    exit
    
    # 2. 라우터 간 WAN 인터페이스 설정
    interface gigabitEthernet 0/0/1
     ip address 10.10.10.2 255.255.255.252
     no shutdown
    exit
    ```

---

## 3. 정적 라우팅 설정
💡 현재 상태는 각 라우터에 IP만 설정해준 상태
💡 즉, 본사에서 지사 라우터로 가는 길을 모르기 때문에 관리자가 수동으로 길을 알려줌

- **라우터 대 라우터 연결(WAN)**
  - Router0 Gi0/0/1 ↔ Router1 Gi0/0/1
    - Router0 Gi0/0/1 IP: 10.10.10.1 / 255.255.255.252
    - Router1 Gi0/0/1 IP: 10.10.10.2 / 255.255.255.252
   
### Router0
```bash
# ip route [목적지 네트워크 ip] [서브넷마스크] [상대편 라우터 포트의 ip]
ip route 192.168.20.0 255.255.255.0 10.10.10.2
```

### Router1
```bash
# ip route [목적지 네트워크 ip] [서브넷마스크] [상대편 라우터 포트의 ip]
ip route 192.168.10.0 255.255.255.0 10.10.10.1
```
---

## 4. 핑 테스트
```bash
ping 192.168.20.10
```

### ▼ 결과

<img width="443" height="201" alt="스크린샷 2026-08-05 231940" src="https://github.com/user-attachments/assets/d6c8dde0-25fe-46fd-a190-f61f08bc8450" />

>> 여기까지 정적 라우팅 완료!
