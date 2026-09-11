# 주제: STP 설정하기

## 🌐 학습 목표
- STP 설정 필요성 공부하기
- STP 설정 실습 진행하기

---
## 🌐 STP(Spanning Tree Protocol)에 대한 공부 내용
### 1. STP의 필요성(간단ver)
- 루핑을 방지하기 위해 설정해야한다

### 2. 루핑이란
- 장비와 장비 사이를 오갈 수 있는 통로가 2개 이상인 경우, 프레임이 그 자리를 뱅뱅 도는 현상

### 3. 루핑 발생장소
- 2계층(브릿지 또는 스위치)
- 2계층에서는 프레임이 영생하기 때문에 루핑이 발생한다.
- 3계층의 경우 TTL이 있어 시간이 지나면 소멸되어 이러한 위험성이 적다.

💡**그럼 2계층에도 TTL 만들면 되지않냐? 싶었지만?**
- TTL 값을 계산하려면 그만큼 처리속도가 길어짐.
- 2계층의 목표는 **LAN 내부에서 최대한 빠르고 단순하게 프레임을 던져주는 것**이기 때문에, TTL같은 안전장치가 없음

💡**초기의 2계층엔 루프가 없어야 정상이었다.**
- 사실 초기 LAN은 단순한 형태였기 때문에 루핑이 없었음.
- 그러나 네트워크 규모가 커지고 장비 이중화가 필요해지면서 2계층에도 루프가 발생하기 시작.
- 이미 세계적으로 2계층에서 TTL이 없는 이더넷 표준 규격을 사용하고 있었기 때문에, 이제와서 TTL을 추가하는 것은 불가능하다고 판단
- 이에 대한 해결방안으로 STP 개발

### 4. 2계층의 비상구, STP란
- 스위치 간 물리적 이중화 구성으로 인해 다중 경로가 생성될 때, 특정 포트를 논리적으로 차단하여 브로드캐스트 스톰 및 루핑을 방지하는 L2 프로토콜
- 메인 링크에 장애가 발생하면 차단되어 있던 우회 포트를 자동으로 열어 네트워크 통신을 즉시 복구한다.

---
## 🌐 실습 순서
## 1. 토폴로지 구성하기
- 스위치 3대
- 각 포트를 cross-over cable로 연결(같은 계층 장비니까 다이렉트 케이블 아니고 크로스 케이블로!)

▼ 결과: 아무리 기다려도 모든 아이콘이 초록색으로 변하지 않음

<img width="416" height="323" alt="스크린샷 2026-09-11 232549" src="https://github.com/user-attachments/assets/2475a0f9-47ce-47cf-a7fb-5266a182f82c" />

▶ 이유: 자동으로 STP가 적용되어 한쪽 케이블이 블락되어 있었던 것!(Switch2의 CLI에서 Gi0/1의 Sts 확인)

<img width="572" height="264" alt="스크린샷 2026-09-11 231820" src="https://github.com/user-attachments/assets/caf0dac1-0a19-4b5f-b7b5-2f2898915b26" />

STP 설정을 하려고 시작한 실습인데, 연결하자마자 자동으로 설정되어버렸다.
대신 
- 루트 브릿지(대장 스위치) 우선순위 변경
- 고속 STP로 변경
하는 실습을 진행하려고 한다.

---
## 2. 루트 브릿지 우선순위 변경하기
- 기존 루트 브릿지: Switch0 -> 변경 루트 브릿지: Switch2

### 1) 루트 브릿지로 설정하고자 하는 Switch2의 CLI 창에서 아래 명령어 입력
```bash
enable
conf t
spanning-tree vlan 1 priority 4096
end
```
### 2) 결과 확인
- **기존(Switch2 쪽이 막힌 상황)**

  <img width="416" height="323" alt="스크린샷 2026-09-11 232549" src="https://github.com/user-attachments/assets/5f7a8999-d0e7-4179-85b8-9ebb135fff40" />

- **변경 후(Switch0쪽이 막히고 Switch2 활성화)**

  <img width="1264" height="807" alt="스크린샷 2026-09-11 233031" src="https://github.com/user-attachments/assets/dc19909d-544a-4406-9121-bce302af14c8" />

---
## 3. 고속 STP로 변경하기
- 구형 STP는 작동하는데 30초가 걸린다.
- 대신 1~2초만에 반응하는 RSTP(Rapid STP)로 변경하기!

### 1) 모든 스위치에 RSTP 명령어 입력하기
- STP 모드를 설정하는 것은 장비 각각의 고유한 설정이기 때문에 스위치마다 명령어를 입력해줘야 함

```bash
enable
conf t

spanning-tree mode rapid-pvst
```

### 2) 바뀐 모드 확인하기
- 각 스위치의 CLI에서 `show spanning tree` 입력하여 변경된 모드를 확인한다.

  <img width="727" height="882" alt="스크린샷 2026-09-11 233419" src="https://github.com/user-attachments/assets/7ae47100-39f3-4210-bdbb-d9b5c984018e" />

---
## 4. 간단한 STP 작동 테스트
- 일부러 선 하나를 끊어서, 우회로가 자동으로 복구되는지 테스트해보았다.

- **기존**
  
  <img width="454" height="296" alt="스크린샷 2026-09-11 233531" src="https://github.com/user-attachments/assets/0c57ed16-e5bd-4326-8375-bbcf1c01e382" />

- **선 끊은 후(즉시 우회로가 열리는 모습 확인)**

  <img width="512" height="316" alt="스크린샷 2026-09-11 233551" src="https://github.com/user-attachments/assets/1f07374a-5791-4dff-81ce-84a58a5bc901" />

---
## 🌐 배운점
- STP는 따로 설정할 필요 없이 자동으로 작동된다
- 하지만 우회로가 열릴때까지 30초 정도의 시간이 필요해서, 실무에서는 RSTP를 사용하는 게 적합해 보인다.
