# 주제: SSH를 사용하여 네트워크 장비 원격 관리하기

## 🌐 학습 목표
1. ssh 접속을 위한 호스트 이름, 도메인 지정 및 rsa 키를 생성한다.
2. 원격 접속 라인(line vty)에서 telnet을 차단하고 ssh 접속만 허용한다.
3. PC CLI 환경에서 ssh 명령어를 통해 스위치로 암호화 원격 접속을 성공시킨다.

---

## 💡 실습 전 알아야 할 개념: SSH를 왜 설정해야 하는지?
네트워크 장비를 관리할 때 매번 장비로 가서 케이블을 꽂아 연결하기는 번거롭기 때문에, 랜선을 통해 원격으로 접속하는 환경을 만들 수 있다.
이 때, 원격 접속 방법은 telnet과 ssh가 있다.

- Telnet: 데이터가 평문으로 전송된다. 따라서 누군가 중간에 패킷을 스니핑하면 관리자의 아이디와 비밀번호가 그대로 노출되어 버린다.
- SSH: Telnet의 취약점에 대응하기 위해 SSH가 등장했다. 이는 모든 데이터를 암호화하여 전송하기 때문에, 계정 정보나 설정 내용을 안전하게 보호할 수 있다.

---
## 🌐 실습 순서
## 1. 토폴로지 구성
- 장비 배치
  - 스위치 1대
  - pc 1대
    - ip 주소: 192.168.10.100
    - 서브넷마스크: 255.255.255.0

  <img width="599" height="268" alt="스크린샷 2026-08-28 233757" src="https://github.com/user-attachments/assets/1f6046b2-2f71-49cd-be2b-40dd8823576a" />

---
## 2. 스위치 관리용 IP 설정
스위치는 원래 IP가 없는 L2 장비이지만, 원격 접속(ssh)을 받아들이기 위해 VLAN1이라는 가상 인터페이스에 IP를 부여한다.

### 1) Switch CLI에 입력
```bash
enable
conf t

interface vlan 1
ip address 192.168.10.254 255.255.255.0
no shutdown
exit
```

### 2) 핑 통신 확인
원격 접속 전, pc0에서 L2로 가는 통신이 정상적으로 이루어지는지 확인한다.
명령어: `ping 192.168.10.254`

<img width="731" height="358" alt="스크린샷 2026-08-28 232513" src="https://github.com/user-attachments/assets/1fb28b15-c5b9-4890-8fc4-380807ce0f7d" />

---
## 3. SSH 보안 설정 및 암호화 키 생성

### Switch CLI에 입력
```bash
Switch(config)# hostname SW1
SW1(config)# ip domain-name lab.local
SW1(config)# crypto key generate rsa

# How many bits in the modulus [512]: 가 뜨면
1024
# 입력하기. 기본값인 512를 쓰면 보안 수준이 너무 낮아서 SSH v2가 켜지지 않음!
```

> ⚠️왜 굳이 호스트 네임을 설정해서 스위치 이름을 바꾸지?
> 
> SSH 암호화 키를 만들 때, 스위치는 내부적으로 [호스트이름].[도메인이름] 형태의 고유한 주소를 조합하여 키를 계산한다.
>
> 여기서 만약 호스트 이름을 바꾸지 않고 그대로 Switch를 사용해버리면,
>
> 다른 스위치들과 고유 식별자가 중복되어 보안 키로서의 고유성이 깨지기 때문에
>
> 꼭! 호스트 이름을 따로 설정해준다.

---
## 4. 원격 접속용 계정 생성

### Switch CLI에 입력
```bash
SW1(config)# username admin privilege 15 secret cisco123
SW1(config)# ip ssh version 2
SW1(config)# line vty 0 15
SW1(config-line)# transport input ssh
SW1(config-line)# login local
SW1(config-line)# exit
```

---
## 5. SSH 암호화 원격 접속하기

### 1) SSH로 원격 접속하기
- pc0의 CLI에 `ssh -l admin 192.168.10.254` 입력

- **결과 확인>> 성공!!**
  
  <img width="370" height="149" alt="스크린샷 2026-08-28 233659" src="https://github.com/user-attachments/assets/cba1fd17-f4c8-4377-9bef-80b6d84e6214" />


### 2) (추가검증) Telnet으로도 접속해보기
- pc0의 CLI에 `telnet 192.168.10.254` 입력

- **결과 확인>> 성공적으로 접속 실패!!**
  
  <img width="429" height="79" alt="스크린샷 2026-08-28 233714" src="https://github.com/user-attachments/assets/a3062664-1663-4653-aff5-e83ff5c753ae" />

  `[Connection to 192.168.10.254 closed by foreign host]`가 나온다면 Telnet 접속이 제대로 차단된 것임!!
