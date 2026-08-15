# 주제: Extended ACL 설정하기

## 🌐 학습목표
- PC0만 웹서버(http 80번 포트) 접속을 차단하기
- ping 통신이나 다른 서비스는 정상 통과하도록 허용하기

## 🌐 알아야 할 개념: Extended ACL 설정 방법
- 특징: Extended ACL은 출발지와 가장 가까운 인터페이스에 규칙을 적용하도록 한다.
- 구조: `access-list [100-199] [deny/permit] [protocol] [출발지IP] [wildcard] [목적지IP] [wildcard] eq [포트번호]`
  - ACL 번호는 100~199번 사이
  - eq에는 특정 포트 번호 지정(ex. 80, www 등)
    
---

## 0. (기존에 설정한) Standard ACL 규칙 삭제하기
- 이전 실습(01-standard_acl_260815)에 이어서 진행했기 때문에, 기존에 설정한 규칙은 삭제해야 함
- ACL 규칙을 설정해준 라우터(router1)의 CLI 창에 다음 코드 입력
  ```bash
  configure terminal

  # 1. 10번 ACL 규칙 완전히 삭제
  no access-list 10
  
  # 2. 인터페이스에 걸린 10번 ACL 표지판도 제거
  interface gigabitEthernet 0/0/0
  no ip access-group 10 in
  no ip access-group 10 out
  end

  # 3. 삭제됐는지 acl 목록 확인하기
  show access-lists
  ```

  ▽ 결과
  
  <img width="484" height="247" alt="스크린샷 2026-08-15 142544" src="https://github.com/user-attachments/assets/d6881821-96f2-4645-bb47-2cab3b3f88a8" />

## 1. 서버 연결하기
1) 장비 목록에서 End Devices - Server 선택 후 Switch1과 연결하기
2) 서버 IP 및 게이트웨이 설정하기
   - IP: 192.168.20.100
   - 서브넷 마스크: 255.255.255.0
   - GW: 192.168.20.1

  ▽ 결과

  <img width="495" height="434" alt="스크린샷 2026-08-15 141930" src="https://github.com/user-attachments/assets/5ce303a6-f60a-4701-8a4d-6deb87cd4c0e" />

3) 웹 접속 테스트
   - ACL을 걸기 전, PC0에서 웹사이트가 정상적으로 열리는지 확인
   - Desktop - web browser의 URL 창에 `http://192.168.20.100` 입력
     
   ▽ 결과

   <img width="704" height="702" alt="스크린샷 2026-08-15 142634" src="https://github.com/user-attachments/assets/664beedf-5ef3-4816-a41d-25463dc7f898" />

## 2. Extended ACL 설정하기
- PC0과 가장 가까운 라우터인 Router0의 CLI에서 진행
  ```bash
  enable
  configure terminal
  
  # 1. Extended ACL 100번 생성: PC0이 Server0(192.168.20.100)의 HTTP(80) 포트로 가지 못하게 트래픽 차단
  access-list 100 deny tcp host 192.168.10.10 host 192.168.20.100 eq 80
  
  # 2. 나머지 모든 IP 트래픽은 허용
  Router0(config)# access-list 100 permit ip any any
  
  # 3. PC0이 연결된 라우터 인터페이스(GigabitEthernet 0/0/0)로 이동하여 Inbound로 적용
  Router0(config)# interface gigabitEthernet 0/0/0
  Router0(config-if)# ip access-group 100 in
  Router0(config-if)# end

  ```

## 3. 결과 테스트
1) PC0 -> Server0 웹 접속 테스트(결과: timeout)

   ▽ 결과

   <img width="703" height="703" alt="스크린샷 2026-08-15 142853" src="https://github.com/user-attachments/assets/50d30f85-8593-4cbc-90f3-b5e33b6a961e" />

2) PC0 -> Server0  테스트(결과: 허용(응답))

   ▽ 결과

   <img width="706" height="355" alt="스크린샷 2026-08-15 142914" src="https://github.com/user-attachments/assets/beb13d15-5d4c-4814-ad9c-b76e0fcc885b" />

=> 웹 접근만 차단했기 때문에 핑 테스트는 성공한 모습을 볼 수 있음!
