---
title: "패킷의 생명(day12)"
date: 2026-09-17 09:00:00 +0900
categories: [Network, CCNA]
tags: [network, ccna, jeremy-it-lab]
toc: true
math: true
comments: true
---

## day 12 : 패킷의 생명

1. 토폴리지 구성
    - 이전 학습 했던 ARP, 이더넷 프레임 캡슐화/역캡슐화, 스위치의 L2 포워딩/MAC 학습, 라우터의 L3 라우팅 테이블 조회를 하나로 엮어 원격지 목적지까지 패킷이 전달되는 전체 과정을 단계별로 완전 추적한다. 
    <img width="1982" height="858" alt="Image" src="https://github.com/user-attachments/assets/8cf1a4b5-4f07-45b6-b1d1-df980f9373aa" />

    - 겅로 시나리오 : PC1 $\rightarrow$ Switch1 $\rightarrow$ R1 $\rightarrow$ R2 $\rightarrow$ R4 $\rightarrow$ Switch4 $\rightarrow$ PC4

    - 장비별 IP 및 MAC 주소 설정
        - PC1: IP 192.168.1.1 (GW: 192.168.1.254), MAC 1111
        - R1: G0/2(LAN) IP 192.168.1.254 / MAC AAAA, G0/0(WAN) IP 192.168.12.1 / MAC BBBB
        - R2: G0/0(WAN) IP 192.168.12.2 / MAC CCCC, G0/1(WAN) IP 192.168.24.2 / MAC DDDD
        - R4: G0/1(WAN) IP 192.168.24.4 / MAC EEEE, G0/2(LAN) IP 192.168.4.254 / MAC FFFE
        - PC4: IP 192.168.4.1 (GW: 192.168.4.254), MAC 4444

2. 단계별 패킷 전달 전 과정 상세 분석
    - 1단계 : PC1 -> R1 (디폴트 게이트웨이로 전달)
        1. L3의 판단 : PC1은 목적지 IP `192.168.4.1`이 자신의 로컬 서브넷(192.168.1.0/24) 외부 임을 인지하고, 패킷을 자신의 디폴트 게이트웨이로 보내기로 결정한다. 
        2. ARP 요청/응답 (R1의 MAC 학습)
            - PC1은 R1의 MAC 주소를 모르므로 ARP Pequest(브로드캐스트, `FFFF.FFFF.FFFF`)를 보낸다. 
            - Switch1은 이를 플러딩하고, 동시에 PC1의 MAC을 자신의 포트에 동적 학습한다.
            - R1은 이를 받아 유니캐스트 ARP Reply로 자신의 MAC을 회신한다. 이때 Switch1은 R1의 MAC을 학습한다.
        3. 3단계 : 프레임 캡슐화 및 송신
            - PC1은 원본 패킷을 L2 프레임으로 캡슐화 한다.
            - `[ L3 Header: Src 192.168.1.1, Dst 192.168.4.1 ]`
            - `[ L2 Header: Src MAC 1111, Dst MAC AAAA ]`
            - Switch1은 목적지 MAC이 등록된 R1의 포트로만 단독 포워딩 한다. 
    
    - 2단계 : R1 -> R2(첫 번째 홉 라우팅)
        1. 역캡슐화 및 라우팅 조회
            - R1은 프레임을 수신하여 L2 헤더를 벗겨내고(역캡슐화)내부 L3 패킷의 목적지 IP를 확인한다. 
            - 라우팅 테이블을 조회하여 최장 일치 경로인 `192.168.4.0/24 via 192.168.12.2`을 찾고, Next-Hop이 R2(`192.168.12.2`)임을 파악한다.
        2. ARP 요청/응답(R2의 MAC 학습)
            - R1은 R2의 MAC을 모르면 브로드캐스트 ARP Request를 보내 R2로부터 MAC을 학습한다. 
        3. 재캡슐화 및 송신
            - IP 헤더의 출발지/목적지IP는 절대 변경되지 않는다.
            - 새로운 L2 이더넷 헤더를 씌운다. `[ L2 Header: Src MAC BBBB (R1 G0/0), Dst MAC CCCC (R2 G0/0) ]
    
    - 3단계 : R2 -> R4(두 번째 홉 라우팅)
        1. R2는 프레임을 받아 L2헤더를 벗겨내고 목적지 IP를 라우팅 테이블에서 조회한다.
        2. Next-Hop이 `192.168.24.4`임을 확인하고, ARP를 통해 R4의 MAC을 획득한다.
        3. 새로운 L2 헤더로 재캡슐화하여 송신한다.
            - `[ L2 Header: Src MAC DDDD (R2 G0/1), Dst MAC EEEE (R4 G0/1) ]`
        
    - 4단계 : R4 -> PC4(최종목적지 직결 전달)
        1. R4는 L2 헤더를 벗겨내고 목적지 IP를 조회한다. `192.168.4.0/24`는 자신의 G0/2 포트에 직접 연결되어 있음을 확인한다.
        2. R4는 ARP를 통해 최종 목적지인 PC4의 MAC을 확습한다.
        3. 최종 프레임을 만들어 Switch를 거쳐 PC4로 전송된다.
            - `[ L2 Header: Src MAC FFFE (R4 G0/2), Dst MAC 4444 (PC4) ]`
            - 패킷이 이동하는 동안 목적지 IP와 목적지 MAC이 동일한 하나의 장비를 가리키는 유일한 순간이다.

3. 패킷 라이프사이클의 핵심 규칙 정리
    1. IP 헤더 불변의 원칙
        - 패킷이 수많은 라우터를 거쳐도 Source IP와 Destination IP는 절대 변경되지 않고 유지된다. (NAT같은 특수 기술 제외)
    
    2. MAC 주소 매 홉 갱신 원칙
        - 라우터를 지날 때마다 이전 L2 헤더는 완전히 버려지고(Decapsulation), 바로 다음 홉 장비를 가리키는 새로운 L2 헤더로 다시 씌워진다.(Re-encapsulation)
        - Source MAC과 Destination MAC은 라우터를 건널 때마다 계속 바뀐다.
    
    3. 스위치 비간섭 원칙
        - 스위치는 L2 프레임의 헤더를 읽어 포워딩하고 자신의 MAC 테이블을 채울 뿐, 프레임을 역캡슐화하거나 자신의 MAC 주소로 헤더를 덮어쓰지 않는다.(무수정 통과)
    
    4. 되돌아오는 응답 트래픽의 간소화
        - PC4가 PC1으로 응답 패킷을 보낼 때는, 모든 장비의 메모리에 이미 ARP 캐시가 채워져 있으므로 브로드 캐스트 ARP 요청과정이 생략되고 즉시 유니캐스트로 고속 포워딩한다. 


### 주요 어휘

| 영문 표현 | 뜻 / 문맥 | 쓰임새 |
| :--- | :--- | :--- |
| Life of a Packet | 패킷의 일생 (전송 수명주기) | 송신지부터 최종 목적지까지 L2/L3 계층을 오가며 패킷이 겪는 캡슐화/포워딩 전 과정 |
| Decapsulation | 역캡슐화 | 수신 장비(라우터)가 상위 데이터를 읽기 위해 바깥쪽 L2 프레임 헤더/트레일러를 벗겨내는 동작 |
| Re-encapsulation | 재캡슐화 | 라우터가 L3 패킷을 다음 홉으로 넘겨주기 위해 새로운 L2 이더넷 헤더를 다시 입히는 동작 |
| Hop-by-Hop | 홉 간 전달 | 패킷이 최종 목적지에 닿기까지 라우터와 라우터 사이를 단계별로 건너가는 방식 |
| Unchanged Source/Destination IP | 불변의 송수신 IP | 라우팅 경로를 거치는 동안 L3 헤더의 원본 출발지 IP와 최종 목적지 IP가 보존되는 원칙 |
| Rewritten MAC Addresses | 갱신되는 MAC 주소 | 라우터를 건널 때마다 L2 프레임의 출발지/목적지 물리 MAC 주소가 새롭게 교체되는 현상 |
| Default Gateway Forwarding | 기본 게이트웨이 전달 | 로컬 서브넷 외부로 향하는 패킷의 L2 목적지 MAC을 첫 번째 라우터(GW)로 설정하여 전달하는 규칙 |
| ARP Caching | ARP 캐싱 | 최초 1회 ARP로 상대 MAC을 알아낸 후 테이블에 저장하여 이후 불필요한 브로드캐스트를 없애는 동작 |

