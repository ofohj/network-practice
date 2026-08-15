# 주제: Inter-VLAN 실습하기

## 💻 실습 목표
- Router-on-a-Stick 환경 구성하기 (하나의 라우터가 단일 케이블을 통해 스위치에 연결되는 방식)
- 서로 다른 VLAN에 있는 PC 간 통신 성공시키기

---

## 1. 네트워크 환경 구축하기

### ▼ 구성
- 스위치, PC 구성은 실습 01과 동일. 라우터만 추가됨

| 장비 | 종류 | 개수 |
| --- | --- | --- |
| **스위치** | 2960 스위치 | 1대 |
| **PC** | PC-PT | 4대(부서별 2대) |
| **라우터** | 4331 라우터 | 1대 |

### ▼ 케이블 연결
| 장비 1 | 포트번호 1 | 장비 2 | 포트번호 2 |
| --- | --- | --- | --- |
| PC0 | fastEthernet 0 | 스위치 0 | fastEthernet 0/1 |
| PC1 | fastEthernet 0 | 스위치 0 | fastEthernet 0/2 |
| PC2 | fastEthernet 0 | 스위치 0 | fastEthernet 0/3 |
| PC3 | fastEthernet 0 | 스위치 0 | fastEthernet 0/4 |
| 스위치 0 | fastEthernet 0/24 | 라우터 0 | GigabitEthernet 0/0/0 |

💡 **스위치 포트 5 말고 24번에 연결시키는 이유**
  - 관례 상, 앞쪽 포트는 사용자 PC나 프린터같은 일반 단말기를 꽂는 용도로 사용
  - 라우터 연결선은 맨 끝 번호인 24번에 지정해두면, 중간에 PC가 추가되더라도 명령어가 꼬이지 않고 한눈에 구분되서 보기 편함

### ▼ 결과

<img width="713" height="425" alt="스크린샷 2026-08-04 234356" src="https://github.com/user-attachments/assets/f0aff5bc-6e7a-41e1-bd8e-05347d970b48" />

---

## 2. 스위치에서 라우터 연결 포트를 Trunk 포트로 변경
- Swich0 클릭 ➔ [CLI] 탭 진입 후 아래 명령어 입력

```bash
# 1) 사용자 모드에서 Privileged EXEC Mode 진입
enable

# 2) Privileged EXEC Mode에서 전역 설정 모드로 진입
configure terminal

# 3) 라우터와 연결된 포트(Fa0/24) 진입 후 Trunk 포트로 변경
interface fastEthernet 0/24
 switchport mode trunk
exit

end
write memory
```

💡 **권한에 따른 터미널 모드**
- Privileged EXEC Mode : 시스템 정보 조회, 장비 상태 **확인** 또는 설정 모드로 진입할 수 있게 됨
- 전역 설정 모드로 진입: ㅜ장비 상태를 **변경**할 수 있음

---

## 3. 라우터 서브인터페이스 및 게이트웨이 설정: Router-on-a-Stick
- 라우터가 스위치에서 들어오는 VLAN 태그를 구분할 수 있도록 가상 서브인터페이스를 만들어 줌
- Router0 클릭 ➔ [CLI] 탭 진입 후 아래 명령어 입력

```bash
enable
configure terminal

# 1) 물리 인터페이스 전원 켜기
interface gigabitEthernet 0/0/0
 no shutdown
exit

# 2) VLAN 10 (영업팀) 전용 서브인터페이스 설정
interface gigabitEthernet 0/0/0.10
 encapsulation dot1Q 10    # 의미: 이 포트는 802.1Q 표준 트렁킹을 사용하고, VLAN 태그가 붙은 패킷만 처리하겠다!
 ip address 192.168.10.1 255.255.255.0    # 의미: VLAN 10 PC들이 사용할 기본 게이트웨이 IP 할당
exit

# 3) VLAN 20 (기술팀) 전용 서브인터페이스 설정
interface gigabitEthernet 0/0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
exit

end
write memory
```

---

## 4. 각 PC에 기본 게이트웨이 지정
- 라우터 IP를 각 PC의 게이트웨이로 지정해주어야 다른 대역으로 패킷 전송 가능
- 각 PC 클릭 ➔ [Desktop] 탭 ➔ [IP Configuration] 이동

<img width="709" height="716" alt="스크린샷 2026-08-04 234847" src="https://github.com/user-attachments/assets/9466324b-f1c3-4eca-9a0d-34516ee2d1b4" />


| PC | VLAN | Gateway |
| --- | --- | --- |
| **PC0** | 10 | 192.168.10.1 |
| **PC1** | 10 | 192.168.10.1 |
| **PC2** | 20 | 192.168.20.1 |
| **PC3** | 20 | 192.168.20.1 |

---

## 5. 최종 검증 (핑 테스트하기)

1) PC0 클릭 ➔ [Command Prompt] 실행
2) 다른 부서로 핑 전송(`ping 192.168.20.10`)

### ▼ 결과

💡 ARP 과정 때문에 `Request timed out.`이 1~2번 뜰 수 있다고 함. 이후 대기 또는 다시 ping [ip 주소]를 입력할 때 `Reply...`가 뜨면 Inter-VLAN 라우팅 성공!

<img width="711" height="720" alt="스크린샷 2026-08-04 234834" src="https://github.com/user-attachments/assets/7926d615-1405-4852-89e3-e91f54faca1b" />

