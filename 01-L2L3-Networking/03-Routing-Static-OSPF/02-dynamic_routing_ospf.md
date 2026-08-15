## 5. OSPF 동적 라우팅 실습
### 1) 기존 정적 라우팅 명령어 삭제

- **Router0의 CLI 창에 입력**
```bash
enable
configure terminal

# 기존 정적 라우팅 지우기 (no 붙이기)
no ip route 192.168.20.0 255.255.255.0 10.10.10.2
exit
```

- **Router1의 CLI 창에 입력**
```bash
enable
configure terminal

# 기존 정적 라우팅 지우기 (no 붙이기)
no ip route 192.168.10.0 255.255.255.0 10.10.10.1
exit
```

- **핑 확인하기**(PC0에서 PC1로 ping을 보내면 다시 통신이 끊기는 것을 확인할 수 있음
  <img width="702" height="703" alt="image" src="https://github.com/user-attachments/assets/ce24ff5f-c326-4158-9d2b-ccbd9c45d88a" />

  💡 **궁금한 점:** `Destination host unreachable` vs `Request timed out`
  
  | 에러 메시지 | 원인 |
  | --- | --- |
  | Unreachable | 목적지 도달 불가(목적지로 가는 라우팅 정보가 아예 없을 때 발생) |
  | Timed out | 요청 시간 초과(가는 길은 알지만 목적지 장비가 꺼져있거나, 패킷이 중간에 증발했을 때 발생) |

---

### 2) OSPF 라우팅 설정
라우터가 자기가 연결하고 있는 네트워크 대역들을 이웃 라우터에게 Advertise하도록 설정하기

- **Router0의 CLI 창에 입력**
  : Router0에 직접 연결된 대역 2개(192.168.10.0/24, 10.10.10.0/30)를 OSPF Area 0에 등록
  ```bash
  configure terminal
  
  # OSPF 프로세스 ID 1번 시작
  router ospf 1
  
  # 본사 LAN 대역 광고 (와일드카드 마스크 0.0.0.255 사용)
  # 해석: Router0에 연결된 IP 주소 중 앞 세 자리는 정확히 일치해야 하며, 마지막 자리는 0~255 중 어떤 수든 올 수 있음
  network 192.168.10.0 0.0.0.255 area 0
  
  # WAN 대역 광고 (와일드카드 마스크 0.0.0.3 사용)
  network 10.10.10.0 0.0.0.3 area 0
  
  exit
  ```

- **Router1의 CLI 창에 입력**
  : Router1에 직접 연결된 대역 2개(192.168.20.0/24, 10.10.10.0/30)를 OSPF Area 0에 등록
  
  ```bash
  configure terminal
  
  # OSPF 프로세스 ID 1번 시작
  router ospf 1
  
  # 지사 LAN 대역 광고
  network 192.168.20.0 0.0.0.255 area 0
  
  # WAN 대역 광고
  network 10.10.10.0 0.0.0.3 area 0
  
  exit
  ```
---

### 3) OSPF 동작 확인 및 최종 핑 테스트

- **OSPF Neighbor 및 라우팅 테이블 확인**
  ```bash
  show ip ospf neighbor
  show ip route
  ```
  <img width="628" height="339" alt="image" src="https://github.com/user-attachments/assets/fb4d4e8d-711e-4398-a4ba-9d2339fbb2cc" />

- **핑 테스트**
  
  <img width="711" height="707" alt="image" src="https://github.com/user-attachments/assets/df53e5b4-9b76-4c72-a826-34d82eadd5ac" />

