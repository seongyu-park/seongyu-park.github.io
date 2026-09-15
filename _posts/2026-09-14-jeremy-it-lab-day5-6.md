---
title: "이더넷 LAN 스위칭(day5~6)"
date: 2026-09-14 09:00:00 +0900
categories: [Network, CCNA]
tags: [network, ccna, jeremy-it-lab]
toc: true
math: false
comments: true
---

## day 5 : 이더넷 LAN 스위칭 - 1

1. 계층 복습 및 LAN의 경계
    - Layer 1(물리 계층) : 매체의 물리적 측성을 정의하고 디지털 비트를 전기/광/무선 신호로 변환한다.
    - Layer 2(데이터 링크 계층) : 노드간 신뢰성 있는 전송, 물리적 주소(MAC 주소)기반 통신, 오류 검출 수행
    - 스위치는 Layer 1과 Layer 2에서 동작하는 장비이다. 

    - LAN(Local Area Network)의 분리 기준
        - 스위치는 LAN을 확장할 뿐 LAN을 분리하지 못한다. 
        여러대의 스위치에 연결되어 있어도 하나의 LAN이다. 
        - 라우터의 인터페이스와 서로다른 LAN의 경계가 된다. 

2. 이더넷 프레임 구조 분석
    - 이더넷 헤더 (22바이트) + 페이로드 (IP 패킷) + 이더넷 트레일러 (4 바이트)로 구성되며, 오버헤드는 26바이트이다.
    <img width="508" height="145" alt="Image" src="https://github.com/user-attachments/assets/c78423d2-97c2-45c0-9be7-4cd138fd1edf" />
    
    - Preamble(프리앰블)-7byte : **10101010** 패턴이 7회 반복되면서 수신 장치가 클록 신호를 통기화할 수 있도록 준비시킨다
    - SFD(Start Frame Delimiter)-1byte : 비트패턴이 10101011로 끝나며, 프리앰블이 끝났고 다음 바이트부터 실제 프레임이 시작됨을 알린다.
    - Destination MAC Address-6byte : 프레임을 수신할 목적지 장비의 하드웨어 물리주소이다.
    - Source MAC Address-6byte : 프레임을 최초 송신한 장비의 하드웨어 물리주소이다.
    - Type/Length-2byte : 
        - <= 1500 : 캡슐화된 페이로드 데이터의 바이트 길이
        - >= 1536 : 상위 계층 프로토콜 종류 (2048:IPv4, 34528:IPv6)
    - FCS(Frame Check Sequence)-4byte : 트레일러에 위치하며, CRC(Cyclic Redundancy Check)알고리즘을 수행하며 전송 중 발생한 비트 왜곡/ 오류를 검출한다.
    (트레일러 : 네트워크 캡슐화 과정에서 데이터(페이로드)의 맨 뒷부분에 덧붙여지는 제어 정보)

3. MAC(Media Access Control) 주소의 구조
    - 길이 : 6바이트이며, 16진수(hexadecimal) 12자리로 표기된다. 
    - BIA(Burned-In Address) 제조장시 NIC(랜카드) 하드웨어 칩에 영구적으로 각인되어 전세계에서 유일한 식별 주소이다.
    - 주소 구성
        - 앞쪽 3바이트 : OUI(Organizationally Unique Identifier) : IEEE가 제조사에 고유하게 배정하는 벤더 식별 번호
        - 뒤쪽 3바이트 : 해당 제조사가 장비마다 개별적으로 부여하는 고유 시리얼 번호

4. 스위치의 동적 학습 및 프레임 포워딩 과정
    - 스위치가 프레임을 수신했을 때 수행하는 동작이 크게 3개로 구분된다.
    1. 출발지 MAC 주소 학습 (Learning)
        - 스위치는 인입된 프레임의 출발지 주소를 확인한다.
        - 해호스트(PC)가 프레임을 보낼 때 헤더에 적어 보낸 '호스트 자신의 MAC 주소(Source MAC)'를 스위치가 읽고, '이 호스트를 찾아가려면 프레임이 들어온 내 포트 쪽으로 보내면 된다'고 MAC 주소 테이블에 매핑(기록)하는 것
        - 에이징 타임 (Aging Time) : 5분동안 해당 MAC 주소로부터 트래픽이 없으면 테이블에서 자동으로 삭제된다. 
    
    2. 목적지 주소 조회 및 전달 (Fowarding vs Flooding)
        - 스위치는 프레임의 목적지 MAC를 읽고 자신의 MAC 테이블을 조회한다.
        - Known Unicast : 목적지 MAC 주소가 테이블에 이미 등룍되있는 경우 해당 포트로 전달(Forwarding) 한다. 
        - Unknown Unicast : 목적지 MAC 주소가 테이블에 없는 경우, 프레임임이 들어온 수신 포트를 제외한 동일 LAM의 모든 포트를 복제하여 전송(Flooding)한다.
            브로드캐스트 : 하나의 송신 호스트가 동일한 네트워크에 연결된 모든 수신 장치에 동시에 데이터를 전송

    3. 수신 호스트의 반응
        - 플러딩된 프레임을 받은 장비 중 자신의 MAC 주소와 일치하지 않는 장비는 프레임을 즉시 폐기(Drop)한다.
        - 목적지 장비는 프레임을 수신하여 상위 계층으로 올리며, 이후 응답 프레임을 보낼 때 스위치는 목적지 장비의 MAC 주소 또한 자신의 테이블에 학습한다.


### 주요 어휘

| 영문 표현 | 뜻 / 문맥 | 쓰임새 |
| :--- | :--- | :--- |
| MAC Address | 매체 접근 제어 주소 | Layer 2 통신에 사용되는 48비트(6바이트) 물리적 하드웨어 주소 |
| BIA (Burned-In Address) | 각인 주소 (물리 주소) | 제조 공정 시 랜카드(NIC) ROM에 물리적으로 영구 기록된 MAC 주소의 별칭 |
| OUI (Organizationally Unique Identifier) | 제조사 고유 식별자 | MAC 주소의 앞쪽 24비트(3바이트)로, IEEE가 제조 업체에 배정하는 고유 코드 |
| Preamble | 프리앰블 | 이더넷 프레임 맨 앞에 위치하여 수신측 클록(Clock)을 동기화시키는 7바이트 필드 |
| SFD (Start Frame Delimiter) | 시작 프레임 구분자 | 비트 패턴 10101011로 실제 프레임 데이터의 시작을 알리는 1바이트 필드 |
| EtherType | 이더타입 | 프레임 내부에 캡슐화된 상위 Layer 3 프로토콜(IPv4, IPv6 등)의 종류를 나타내는 필드 |
| FCS (Frame Check Sequence) | 프레임 체크 시퀀스 | 프레임 트레일러의 4바이트 필드로, 전송 중 에러 검출용 CRC 계산값이 들어감 |
| CRC (Cyclic Redundancy Check) | 순환 중복 검사 | 데이터 전송 중 비트 오류 발생 여부를 감지하기 위해 수행되는 수학적 알고리즘 |
| Unicast | 유니캐스트 | 단일 송신자가 지정된 단일 수신자(1:1)에게 데이터를 전송하는 방식 |
| Unknown Unicast | 미확인 유니캐스트 | 프레임 목적지는 특정 단일 주소이나, 스위치의 MAC 테이블에 등록되어 있지 않은 프레임 |
| Flooding | 플러딩 (역류/살포) | 스위치가 목적지를 모를 때 프레임을 수신 포트를 제외한 모든 활성 포트로 뿌리는 동작 |
| Aging Time | 에이징 타임 (만료 시간) | 동적으로 학습된 MAC 주소가 갱신 없이 테이블에 머무를 수 있는 유효 시간(기본 5분) |


---


## day 6 : 이더넷 LAN 스위칭 - 2

1. 이더넷 프레임 크기 제한과 패딩 (Padding)
    - 헤더와 트레일러 크기
        - Preamble(7byte) + SFD(1byte)는 물리적 동기화용 신호이므로 순수 이더넷 헤더 범위에서 제외하기도 한다.
        - 헤더(14byte) = 목적지 주소(6byte) + 소스 주소(6byte) + type(2byte)
        - 트레일러(4byte)
    - 최소 프레임 크기
        - 이더넷 표준에서 하나의 프레임은 최소 64바이트 이상이여야 한다.
        - 최소 크기인 64바이트에서 오버헤드 18바이트를 빼면 최소 페이로드의 크기는 46바이트이다.
    - 패딩
        - 상위 계층 데이터의 크기가 46미만인 경우 46바이트를 맞추기 위헤 데이터 끝에 0으로 채워진 빈 바이트를 덧붙여 전송 
        - 프레임의 최소 단위가 64바이트인 이유는 CSMA/CD 충돌 감지 방식에서 데이터 전송 중 발생한 충돌을 확실하게 감지하기 위해서이다.

2. ARP(Address Resolutin Protocol, 주소 결정 프로토콜)
    - 호스트가 통신할 때 사용자는 ip 주소만 지정하지만, 로컬 L2 스위치 망을 건너가려면 상대방의 MAC 주소를 반드시 알아야 한다. 이를 해결하기 위해서 ARP를 사용한다. 
    - 정의 : 이미 알고 있는 상대방의 IP 주소를 바탕으로, 아직 모르는 상대방의 MAC 주소를 알아내는 프로토콜
    - EtherType : ARP 프레임의 Type 필드 값은 0x0806이다.

    - ARP의 2단계 동작 과정
        - ARP Request(요청)
            - 송신 장비가 특정 ip를 쓰는 자입가 누구인가? 해당 ip의 장비에게 당신의 MAC을 알려달라고 묻는다.
            - 목적지 MAC 주소를 모르기 때문에 브로드 캐스트로 프레임을 만들어 전송한다. 목적지 주소 값을 (FFFF.FFFF.FFFF)로 설정
            - 스위치는 브로드캐스트 프레임을 수신 포트를 제외한 모든 활성 포트로 전송한다.
            - 해당 IP를 쓰지 않는 호스트들은 프레임을 열어보고 목적지 IP가 맞지 않아 폐기(Drop)한다.
        
        - ARP Reply(응답)
            - 해당 IP 주인인 목적지 호스트만 응답한다.
            - ARP Request 메시지 안에 송신자의 MAC가 이미 들어 있었기 때문에, 목적지 호스트는 송신자에게 유니캐스트로 자신의 MAC 주소를 담아 1:1로 직접 응답한다. 
        
        - ARP 테이블
            - 장비는 학습한 [IP 주소 - MAC 주소]쌍을 메모리에 기록해두고 재사용한다.
            - 확인 명령어
                - Windows/Linux/macOS : `arp -a`
                - Cisco IOS(특권모드) : `show arp`

3. Ping과 ICMP의 동작 원리
    - Ping(Packet Internet Groper) : 원격 호스트와의 통신 가능 여부 및 왕복 지연 시간(RTT)을 측정하는 유틸리티
    - ICMP(Internet Control Message Protocol) : 네트워크 장치들이 서로 통신 상태를 확인하고, 문제가 생겼을 때 에러를 알려주기 위해 사용하는 L3 프로토콜
    - 동작 메시지
        - ICMP Echo Request : 송신 장비가 목적지로 보내는 유니캐스트 요청
        - ICMP Echo Reply : 요청을 받은 목적지가 되돌려주는 유니캐스트 응답
    - 시스코 라우터 Ping의 기본 특징
        - 기본적으로 100바이트 크기의 ICMP 패킷을 5회 전송한다.
        - ! = 성공, . = 타임아웃 실패
        - 첫번째 핑이 실패하는 이유 : 첫 핑을 보내기 전에 상대방의 MAC 주소를 몰라 먼저 수행되는 ARP 브로드캐스트/응답과정에서 지연이 발생하여 첫번째 패킷이 만료되기 때문이다.

4. 시스코 스위치 MAC 주소 테이블 관리 CLI
    - 테이블 확인 : Switch# show mac address-table
    - 테이블 초기화/삭제 명령어(clear) 
        - 동적으로 학습된 모든 MAC 주소 삭제 : `Switch# clear mac address-table dynamic`
        - 특정 MAC 주소 하나만 지정하여 삭제 : `Switch# clear mac address-table dynamic address <MAC주소>`
        - 특정 인터페이스(포트)에 학습된 MAC 주소들만 삭제 : `Switch# clear mac address-table dynamic interface <인터페이스ID>`


### 주요 어휘

| 영문 표현 | 뜻 / 문맥 | 쓰임새 |
| :--- | :--- | :--- |
| Minimum Payload | 최소 페이로드 | 이더넷 프레임이 규격상 만족해야 하는 최소 데이터 크기(46 Bytes) |
| Padding | 패딩 (덧채움) | 페이로드가 46바이트 미만일 때 최소 프레임 길이를 맞추기 위해 붙이는 0의 나열 |
| ARP (Address Resolution Protocol) | 주소 결정 프로토콜 | Layer 3(IP) 주소를 기반으로 Layer 2(MAC) 주소를 조회/해결하는 프로토콜 |
| ARP Request | ARP 요청 메시지 | 목적지 IP의 MAC 주소를 묻기 위해 로컬 LAN 전체로 브로드캐스트하는 메시지 |
| ARP Reply | ARP 응답 메시지 | 요청자의 MAC을 확인하고 자신의 MAC을 1:1로 알려주는 유니캐스트 메시지 |
| Broadcast MAC | 브로드캐스트 MAC 주소 | 로컬 네트워크의 모든 단말을 지정하는 특수 MAC 주소 (FFFF.FFFF.FFFF) |
| ARP Table / Cache | ARP 테이블 / 캐시 | 호스트가 통신을 위해 IP와 매핑된 MAC 주소를 기억해 두는 메모리 테이블 |
| Ping | 핑 (네트워크 유틸리티) | ICMP 프로토콜을 사용해 종단 간 도달 가능성과 왕복 시간을 점검하는 도구 |
| ICMP Echo Request / Reply | ICMP 에코 요청 / 응답 | 핑 테스트 시 송수신되는 질의 패킷과 그에 대한 회신 패킷 |
| Round-Trip Time (RTT) | 왕복 시간 | 데이터 패킷이 목적지로 떠나 응답을 받아 돌아올 때까지 걸리는 전체 시간 |
| Packet Capture | 패킷 캡처 | 와이어샤크(Wireshark) 등으로 네트워크 회선을 오가는 트래픽을 가로채 분석하는 작업 |
| Aging | 에이징 (노후화 삭제) | 스위치 MAC 테이블에 일정 시간(5분) 동안 통신이 없는 주소를 자동 삭제하는 메커니즘 |
