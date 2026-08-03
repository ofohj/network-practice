# 주제: VLAN 구성하기

## 💻 실습 목표
- L2 스위치 환경에서 VLAN을 이용해 부서별(영업팀, 기술팀) 네트워크 가상 분리하기
- 동일한 스위치 내 서로 다른 VLAN 간 통신 차단 확인하기 (L2 보안 및 브로드캐스트 대역 분리)

## 알아야 할 개념: Trunking
- **개념**: 하나의 물리적 랜선을 통해 여러개의 VLAN 데이터 패킷을 동시에 전송하는 기술
- **활용예시(문제상황)**: VLAN 10 전용 랜선 1개, VLAN 20 전용 랜선 1개 총 2개의 선만 연결할 때는 굳이 필요하지 않음. 하지만 만약 랜선이 50개라면 포트 낭비가 발생함. 
- **활용예시(해결상황)**: 이를 해결하기 위해 트렁크 포트 존재. 1개의 랜선을 꽂고 트렁크 포트로 설정하면 VLAN이 많아져도 선 하나로 전부 통신시킬 수 있음.
---

## 01. 네트워크 환경 구축하기

### ▼ 구성
| 장비 | 종류 | 개수 |
| --- | --- | --- |
| **스위치** | 2960 스위치 | 1대 |
| **PC** | PC-PT | 4대(부서별 2대) |

### ▼ 케이블 연결
| PC | 스위치 | 포트번호 |
| --- | --- | --- |
| **PC0** | 0 | fastEthernet 0/1 |
| **PC1** | 0 | fastEthernet 0/2 |
| **PC2** | 0 | fastEthernet 0/3 |
| **PC3** | 0 | fastEthernet 0/4 |

### ▼ 결과
<img width="607" height="401" alt="스크린샷 2026-08-04 002505" src="https://github.com/user-attachments/assets/b713c683-e051-445e-bafd-052323c690ad" />

---

## 02. PC별 IP 주소 설정하기
- 각 PC를 클릭한 뒤 [Desktop] ➔ [IP Configuration]에서 아래와 같이 IP와 서브넷 마스크를 입력하기

### ▼ 결과

| <img width="500" height="500" alt="스크린샷 2026-08-04 002526" src="https://github.com/user-attachments/assets/1d64966f-a68e-4c0c-94b1-79e57b17d331" />| <img width="500" height="500" alt="스크린샷 2026-08-04 002611" src="https://github.com/user-attachments/assets/533cfe42-19ba-4354-a3fc-a8a238c37bbe" /> |
| --- | --- |

---

## 03. 스위치에서 VLAN 생성 및 포트 할당
- Switch0을 클릭한 뒤 [CLI] 탭으로 이동해서 Enter를 친 후 아래 명령어를 순서대로 입력하기

```bash
# 1. 전역 설정 모드 진입
enable
configure terminal

# 2. VLAN 10 (영업팀) 생성
vlan 10
 name Sales
exit

# 3. VLAN 20 (기술팀) 생성
vlan 20
 name Tech
exit

# 4. PC0, PC1 포트(Fa0/1 ~ Fa0/2)를 VLAN 10에 할당
interface range fastEthernet 0/1 - 2
 switchport mode access
 switchport access vlan 10
exit

# 5. PC2, PC3 포트(Fa0/3 ~ Fa0/4)를 VLAN 20에 할당
interface range fastEthernet 0/3 - 4
 switchport mode access
 switchport access vlan 20
exit

# 6. 설정 저장
end
write memory
```

### ▼ 결과

<img width="780" height="721" alt="스크린샷 2026-08-04 003000" src="https://github.com/user-attachments/assets/025ff800-e9c1-4283-9686-3fe0404d3a10" />

---

## 04. 네트워크 환경 검사
- Ping 테스트를 통해 네트워크 설정이 잘 됐는지 검사하기
- 영업팀 PC0에서 같은 영업팀인 PC1으로 핑을 전송했을 때 통신이 성공하는지
- 영업팀 PC0에서 다른 기술팀인 PC2로 핑을 전송했을 때 통신이 실패하는지

<img width="724" height="728" alt="스크린샷 2026-08-04 003207" src="https://github.com/user-attachments/assets/a05d6569-15e4-4d63-8147-238c0be912b2" />

