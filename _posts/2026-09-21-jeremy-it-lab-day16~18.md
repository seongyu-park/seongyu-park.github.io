---
title: "VLANs(Day16~18)"
date: 2026-09-21 09:00:00 +0900
categories: [Network, CCNA]
tags: [network, ccna, jeremy-it-lab]
toc: true
math: true
comments: true
---

## day 16 - VLANs(가상 LAN) - 1

1. LAN의 기술적 정의와 브로드캐스트 도메인
    - LAN의 엔지니어링 정의
        - 단순히 " 한 건물/사무실에 모여 있는 장비의 집합을 넘어 기술적으로 단일 브로드캐스트 도메인을 의미한다.
    
    - 브로드캐스트 도메인
        - 네트워크 내의 임의의 한 장비가 브로드캐스트 프레임(목적지 MAC이 `FFFF.FFFF.FFFF`인 프레임)을 송출했을 때, 그 프레임을 수신하게 되는 모든 장비들의 논리적/물리적 범위 
    
    - 스위치 vs 라우터의 브로드캐스트 처리
        - 스위치 : 브로드캐스트 및 알 수 없는 유닠캐스트를 수신 포트를 제외한 모든 포트로 플러딩하므로, 동일 스위치에 연결된 장비들은 기본적으로 동일한 브로드캐스트 도메인에 속한다. 
        - 라우터 : 브로드캐스트 프레임을 수신하여 처리할 뿐, 다른 인터페이스로 절대 포워딩하지 않고 차단한다.

        - 따라서 각 라우터 인터페이스마다 독립된 하나의 브로드캐스트 도메인(하나의 LAN)이 형성된다. (라우터와 라우터 사이를 1:1로 잇는 WAN/P2P 회선 역시 하나의 독립된 브로드캐스트 도메인이다.)

2. 단일 서브넷/LAN 구조의 문제점과 VLAN의 필요성
    - 하나의 물리적 스위치에 엔지니어링팀, 영업팀, 인사팀 등 서로 다른 부서가 함께 물려 있는 경우 발생하는 문제점
    
    1. 네트워크 성능 저하
        - 특정 부서의 PC가 보낸 브로드캐스트(ARP)가 스위치 전체 포트를 살포되어, 무관한 다른 부서 단말들의 CPU와 대역폭을 낭비시킨다.
    
    2. 보안 취약점
        - 동일 L2 브로드캐스트 도메인 내 장비들은 라우터를 거치지 않고 스위치를 통해 직접 통신할 수 있으므로, 라우터나 방화벽에 설정한 보안 접근 제어 정책(ACL 등)이 무력화된다.
        (ACL : 네트워크 상에서 트래픽을 제어하기 위해 라우터 등에 적용하는 출입 통제 규칙이다.)

    3. L3 서브넷 분할만으로는 불완전한 이유
        - 각 부서를 서로 다른 IP 서브넷으로 쪼개더라도, L2 스위치는 IP를 보지 않고 오직 MAC 주소만 확인한다. 
        - 따라서 특정 서브넷 브로드캐스트 프레임이 스위치로 들어오면 스위치는 IP 서브넷 경계를 무시하고 모든 부서의 포트로 플러딩해 버린다. 
    
    4. VLAN(Virtual LAN)의 해결책
        - 고가의 물리적 스위치를 부서마다 따로 구매할 필요 없이, 단일 스위치 내부를 논리적으로 쪼개어 여러 개의 독립된 L2 브로드캐스트 도메인으로 격리시키는 기술이다.

3. VLAN의 핵시 동작 특성
    <img width="1812" height="982" alt="Image" src="https://github.com/user-attachments/assets/cc932fa7-13a4-42d8-979a-b7e92202136f" />
    - 포트 단위 격리 : 스위치 포트 단위로 특정 VLAN을 할당한다.
    
    - L2 브로드 캐스트 차단 : VLAN10 포트에서 발생한 브로드캐스트 및 플러딩 프레임은 오직 동일한 VLAN 10에 속한 포트로만 전달되어, VLAN 20이나 30으로는 절대 넘어가지 않는다.

    - VLAN 간 직접 통신 불가(L3 라우터 필수)
        - 스위치는 서로 다른 VLAN 간에 프레임을 직접 넘겨주지 못한다. 
        - 서로 다른 VLAN에 속한 장비끼리 통신하려면 반드시 라우터를 거쳐서 라우팅을 수행해야 한다. 

4. 스위치 포트 유형 : 액서스 포트(Access Port)
    - 정의 : 단 하나의 단일 VLAN에만 속하는 스위치 포트이다.
    - 용도 : PC, 서버, 프린트 등 네트워크 끝단 단말과 연결하는 포트이며, 단말에게 네트워크 접근 권한을 제공하므로 Access 포트라고 부른다. 

5. 시스코 기본 VLAN 및 CLI 설정 명령어
    - 스위치 기본 내장 VLAN 확인
        - VLAN 1(name : default) : 스위치의 모든 포트가 기본적으로 소속되어 있는 기본 VLAN 이다.
        - VLAN 1002 ~ 1005 : 과거 구형 기술 호환을 위해 시스템에 예약된 VLAN이다.
        - VLAN1, VLAN1002~1005는 절대 삭제하거나 수정할 수 없다.
    
    - 포트를 Access 모드로 지정하고 VLAN 할당
    ```
    SW1(config)# interface range g1/0 - 3
    SW1(config-if-range)# switchport mode access
    SW1(config-if-range)# switchport access vlan 10
    ```
    - g1/0-3 포트를 엑세스 포트로 지정하고 그 포트들을 vlan10에 속하게 한다.
    - 자동 생성 기능: 만약 vlan10이 스위치 데이터베이스에 아직 생성되어 있지 않는 상태에서 인터페이스에 할당하면 시스템이 `% Access VLAN does not exist. Creating vlan 10` 메시지를 띄우며 vlan 10을 자동으로 생성한다.

    - VLAN 수동 생성 및 이름 부여
    ```
    SW1(config)# vlan 10
    SW1(config-vlan)# name Engineering
    SW1(config)# vlan 20
    SW1(config-vlan)# name HR
    SW1(config)# vlan 30
    SW1(config-vlan)# name Sales
    ```
    - 설정한 vlan의 이름을 변경하는 명령어이다.

    - 설정 확인 명령어
        - `show vlan brief` : 생성된 vlan 번호, 이름, 상태, 그리고 각 vlan에 매핑된 포트 목록을 요약 점검한다.
    

    ### 퀴즈
    1. 아래 네트웨크 다이어그램에는 몇 개의 브로드캐스트 도메인이 존재하는가?
        <img width="1492" height="874" alt="Image" src="https://github.com/user-attachments/assets/3e085a20-1137-4ee9-80e7-891d1ffd6a09" />
        - 브로드캐스트 도메인은 해당 브로드캐스트가 울리는 자리로 라우터와 라우터를 연결하는 선 자체가 하나의 독립된 브로드 캐스트 도메인이 된다.
        따라서 각 스위치들을 연결하는 브로드캐스트 도메인 5 + 라우터 사이 링크 1개를 더하면 총 브로드캐스트 도메인이 6개가 된다.
    
    2. VLAN이 구성된 아래 네트워크 다이어그램에는 몇 개의 브로드 캐스트 도메인이 존재하는가?
        <img width="1682" height="822" alt="Image" src="https://github.com/user-attachments/assets/c2b67b03-b538-4bcf-8983-cfea869f6fe1" />
        - VLAN을 사용하면 스위치 내부에 분리된 브로드캐스트 도메인이 형성된다. 따라서 라우터 사이의 브로드캐스트 도메인 1에 분리된 VLAN을 더하면 총 5개가 존재하게 된다.
    
    3. VLAN 20에 속한 PC 3이 브로드캐스트 메시지를 송출했다면, 스위치 및 라우터를 포함하여 총 몇 대의 장비가 이 브로드 캐스트를 수신하는가?
        <img width="1644" height="846" alt="Image" src="https://github.com/user-attachments/assets/539ef9a0-1189-46ab-b5bc-9fd69cada334" />
        - 프레임을 최초 수신한 스위치 1대 + 스위치의 VLAN 20포트와 연결된 라우터 1대 + 스위치 VLAN 20 포트에 연결된 다른 PC 1대 총 3대이다. 

    4. 시스코 스위치에서 VLAN 10, 20, 30을 새로 생성했다. 이후 `show vlan brief` 명령어를 실행했을 때 출력되는 전체 vlan 항목의 개수는 몇 개인가?
        - 해설 : vlan 10, 20, 30으로 3개를 쓰고 기본 vlan 1 1에 구형 레거지 예약한 vlan 1002~1005까지 4개로 총 1+4+3으로 총 8개의 vlan을 조회할 수 있다. 


### 주요 어휘

| 영문 표현 | 뜻 / 문맥 | 쓰임새 |
| :--- | :--- | :--- |
| VLAN (Virtual LAN) | 가상 로컬 영역 네트워크 | 물리적 스위치 하나를 논리적으로 쪼개어 여러 개의 독립된 L2 브로드캐스트 도메인으로 분할 |
| Broadcast Domain | 브로드캐스트 도메인 | 특정 장비가 송출한 브로드캐스트 프레임(Destination MAC: All Fs)이 도달하는 네트워크 영역 |
| Access Port | 액세스 포트 | 단 하나의 VLAN에만 속하여 PC, 서버 등 엔드 호스트와 직접 연결되는 스위치 포트 |
| Trunk Port | 트렁크 포트 | 스위치 간 혹은 스위치-라우터 간에 여러 VLAN의 트래픽을 단일 링크로 다중 전송하는 포트 |
| Inter-VLAN Routing | VLAN 간 라우팅 | 서로 격리된 서로 다른 VLAN 간에 패킷을 전달하기 위해 L3 라우터를 경유하여 처리하는 라우팅 |
| Default VLAN (VLAN 1) | 기본 VLAN | 시스코 스위치 전 포트가 공장 초기화 상태에서 기본적으로 소속되어 있는 삭제 불가 VLAN |
| Reserved VLANs (1002-1005) | 시스템 예약 VLAN | 레거시 토큰링(Token Ring) 및 FDDI 용도로 영구 예약되어 수정이나 삭제가 불가능한 VLAN |
| Flooding | 플러딩 (살포) | 브로드캐스트나 대상 MAC을 모를 때 인입 포트를 제외한 동일 VLAN의 전 포트로 프레임을 복제 전송 |


---


## day17 : VLANs - 2

1. 트렁크(Trunk Port)의 정의와 필요성
    - 엑세스 포트(Access Port)
        - 단 하나의 VLAN 트래픽만 전송 가능하므로, 스위치 간이나 스위치 간이나 스위치-라우터 간에 여러 VLAN을 연결하려면 VLAN 개수만큼 물리적 케이블과 포트를 낭비해야 한다. 
    - 트렁크 포트
        - 단일 물리적 인터페이스를 통해 여러 VLAN의 트래픽을 동시에 다중 전송을 하는 포트이다.
        - 트렁크 링크를 통과하는 프레임에는 어느 VLAN에 속한 데이터인지 식별하기 위한 VLAN 태그가 부착된다. 
    
2. 트렁킹 프로토콜 및 IEEE 802.1Q 헤더 구조
    - 트렁킹 표준 규격으로 과거 시스코 전용 ISL(Inter-Switch Link)이 있었으나 현대에는 완전히 사장되었으며, 산업 표준인 IEEE 802.1Q(Dot1Q)가 표준으로 사용된다.

    - 802.1Q 태그 삽입 위치
        - 기존 이더넷 프레임의 Source MAC 주소와 Type/Length 필드 사이에 정확히 4바이트 크기로 끼워넣는다.

    - 802.1Q 태그의 4가지 세부 필드 구조
        1. TPID(tag protocol identifier, 16 bits/2 bytes)
            - 항상 `0x8100` 16진수 값으로 고정되어 있다. 
            - 프레임을 수신한 스위치가 이 자리에 `0x8100`이 들어있는 것을 보고 "이 프레임은 802.1Q 태그가 붙은 트렁크 프레임이다"라고 즉시 판별한다.
        
        2. PCP(Priority Code Point, 3bit)
            - 2계층 Qos인 CoS(Class of Service)를 구현하는 필드로, 트래픽 혼잡 시. 음성/비디오 등의 우선순위를 지정한다. 
        
        3. DEI(Drop Eligible Indicator, 1bit)
            - 네트워크 혼잡 시 우선적으로 버려져도 무방한 패킷인지 여부를 표시하는 플래그 비트이다. 
        
        4. VID(VLAN ID, 12 bits)
            - 해당 프레임이 어느 VLAN에 속하는지를 직접 나타내는 가장 핵심적인 필드이다. 
            - $2^{12} = 4,096$ 개 VLAN ID를 표현할 수 있다.
            - 양 끝의 0과 4095는 시스템 용으로 예약되어 있어, 실제 사용 가능한 VLAN 번호는 1~1094이다. 
            - 표준 범위 : `1 ~ 1005` : 가장 흔하게 사용하는 표준 VLAN 대역
            - 확장 범위 : `1006 ~ 4096` : ISP나 수많은 가상 네트워크 격리가 필요한 초대형 데이터센터 등에서 특수한 목적으로 사용된다.

3. 네이티브 VLAN (Native VLAN)의 개념과 불일치 장애
    - 개념
        - 802.1Q 트렁크 링크에서 유일하게 태그를 붙이지 않고 그대로 전송하는 특수 VLA이다. 
        - 스위치는트렁크 포트에서 태그가 없는 순수 이더넷 프레임을 수신하면, 이를 자신이 설정된 네이티브 VLAN 프레임으로 간주한다. 
        - 기본 설정 값은 `VLAN 1`이다.
        - 태그 기능을 아예 모르는 구형 허브나 스위치 끼리 서로 상태를 주고 받는 관리용 트래픽인 상황일 때 사용한다.
    
    - 네이티브 VLAN 불일치 위험성
        - 만일 switch1 트렁크 포트는 native vlan이 `vlan 10`이고 switch2 트렁크 포트는 `vlan 30`으로 엇갈려 있는 상황
            - switch1이 보낸 `vlan 10` 트래픽이 태그 없이 날라간다.
            - switch2는 태그가 없으므로 이를 자신의 native vlan인 `vlan 30` 트래픽으로 오인하여 `vlan 30` 포트로 잘못 배달하거나 폐기한다.

    - 따라서 양단 스위치의 Native VLAN 설정 번호는 반드시 일치 시켜야 한다.
        - 일반적으로 보안 강화를 위해서 native VLAN을 일반 사용자가 쓰지 않는 더미 번호로 변경하는 것이 권장된다. 

4. 스위치 트렁크 포트 CLI 설정 및 검증
    ```
    SW1(config-if)# switchport trunk encapsulation dot1q   ! (ISL/802.1Q 동시 지원 구형/L3 스위치 모델에서 선행 필수)
    SW1(config-if)# switchport mode trunk
    ```
    - 주의 캡슐화가 `auto`인 상태인 스위치에서 곧바로 `switchport mode trunk`를 입력 시 명령어가 거부 됨으로 먼저 `switchport trunk encapsulation`를 선언해야 한다. 

    - 허용 VLAN 목록 제어
        - 기본값은 `1~4094` 전체 허용 상태이다. 보안과 브로드캐스트 대역폭 절약을 위해 필요한 VLAN만 통과하도록 필터링한다.
        - `switchport trunk allowed vlan 10,30` : 목록을 VLAN 10과 30으로 덮어쓴다.
        - `switchport trunk allowed vlan add 20` : 기존 목록에 20을 추가한다.
        - `switchport trunk allowed vlan remove 20` : 목록에서 20을 제거한다.
        - `switchport trunk allowed vlan all` : 전체 VLAN을 허용한다.(기본값 복원)
        - `switchport trunk allowed vlan except 1-5` : 지정 번호를 제외한 나머지 허용
        - `switchport trunk allowed vlan none` : 모든 VLAN 차단
    
    - 네이티브 VLAN 변경
        ```
        SW1(config-if)# switchport trunk native vlan 1001
        ```
    
    - 점검 명렁어
        - `show interfaces trunk` : 트렁크로 동작 중인 포트, 모드, 인탭슐레이션, 네이티브 VLAN, 허용된 VLAN 목록 그리고 스위치에 실제 생성되어 활성화된 VLAN 목록을 완벽히 점검한다.
        - 주의 : `show vlan brief` 명령어는 Access 포트만 표시하며, 트렁크로 설정된 포트는 이 테이블에서 완전히 사라지므로 반드시 `show interfaces trunk`로 확인해야 한다. 

5. ROAS(Router-on-a-stick)
    - 정의 
        - 라우터의 단 1개의 물리적 인터페이스와 스위치의 트렁크 포트를 단일 링크로 연결한 뒤, 라우터 내부에서 이를 여러 개의 논리적 서브 인터페이스로 분할하여 서로 다른 VLAN간 라우팅을 처리하는 표준 구성 기법이다. 
        - 토폴로지 상에서 라우터가 스위치 위에 막대기 처럼 서 있는 형태여서 붙여진 명칭이다. 
    
    - 시스코 ROAS 설정 문법
        1. 물리 인터페이스 활성화(물리 포트 자체에는 IP를 부여하지 않음)
            ```
            R1(config)# interface g0/0
            R1(config-if)# no shutdown
            ```
        
        2. 서브 인터페이스 진입 및 8021.Q 태그 매핑, IP 부여
            ```
            R1(config)# interface g0/0.10
            R1(config-subif)# encapsulation dot1q 10
            R1(config-subif)# ip address 192.168.1.62 255.255.255.192

            R1(config)# interface g0/0.20
            R1(config-subif)# encapsulation dot1q 20
            R1(config-subif)# ip address 192.168.1.126 255.255.255.192

            R1(config)# interface g0/0.30
            R1(config-subif)# encapsulation dot1q 30
            R1(config-subif)# ip address 192.168.1.190 255.255.255.192
            ```

            - 주의 : `encapsulation dot1q <VLAN-ID>` 명령어를 입력하기 전에는 `ip address` 명령어를 넣을 수 없다. 
            서브 인터페이스 번호와 VLAN ID가 기술적으로 반드시 일치할 필요는 없으나, 실무 관리상 동일하게 일치시키는 것이 절대 권장 표준이다.
            - 라우팅 동작 : 라우터가 `g0/0.10`으로 나가는 패킷을 송출할 때 자동으로 `VLAN 10`태그를 붙여 스위치로 전달하고, 스위치로부터 `VLAN 10`태그가 붙여 들어온 패킷은 `g0/0.10`서브 인터페이스로 수신된 것으로 인지하여 처리한다. 

### 퀴즈 
    1. 인터페이스에 switchport trunk allowed vlan add 10을 설정했으나, show interfaces trunk의 'Vlans allowed and active in management domain' 항목에 VLAN 10이 나타나지 않는다. 그 원인은 무엇인가?
        - Vlans allowed and active in management domain 항목은 단순 설정상의 허용 목록이 아니라, 현재 활성화되어 실제 데이터 전송이 가능한 상태의 VLAN만 보여준다. 따라서 포트 허용 목록에 vlan 10을 등록하고 vlan 10이란 명령어를 통하여 스위치 데이터 베이스에 vlan10을 등록(생성) 해야 정상 활성화 된다.


### 주요 어휘

| 영문 표현 | 뜻 / 문맥 | 쓰임새 |
| :--- | :--- | :--- |
| Trunk Port | 트렁크 포트 | 단일 물리 포트를 통해 여러 VLAN의 태깅된 트래픽을 동시에 운반하는 포트 |
| IEEE 802.1Q (Dot1Q) | 도트원큐 (표준 트렁킹) | 이더넷 프레임에 4바이트 VLAN 식별 태그를 삽입하는 국제 전기전자표준 규약 |
| Tagged / Untagged Port | 태그 / 언태그 포트 | 802.1Q 태그가 프레임에 부착되는 트렁크 포트와 부착되지 않는 액세스 포트의 별칭 |
| TPID (Tag Protocol Identifier) | 태그 프로토콜 식별자 | 802.1Q 태그임을 명시하는 16비트 필드로 항상 16진수 '0x8100' 값을 가짐 |
| VID (VLAN ID) | VLAN 식별자 | 프레임의 소속 VLAN 번호(1 ~ 4094)를 지정하는 802.1Q 태그 내부의 12비트 필드 |
| Native VLAN | 네이티브 VLAN | 802.1Q 트렁크 상에서 예외적으로 태그 없이(Untagged) 전송되는 특수 VLAN (기본 VLAN 1) |
| Native VLAN Mismatch | 네이티브 VLAN 불일치 | 트렁크 양단의 Native VLAN 번호 설정이 달라 트래픽이 엉뚱한 VLAN으로 누수/차단되는 오류 |
| Router-on-a-Stick (ROAS) | 라우터 온 어 스틱 | 라우터 물리 포트 하나를 논리 서브 인터페이스로 쪼개어 트렁크와 연동하는 VLAN 간 라우팅 기법 |
| Subinterface | 서브 인터페이스 | 물리적 라우터 포트 아래에 'g0/0.10' 형식으로 생성되는 가상의 논리적 인터페이스 단위 |
| Allowed VLAN List | 트렁크 허용 VLAN 목록 | 스위치 트렁크 포트를 통과할 수 있는 VLAN의 범위를 수동으로 제한하는 필터 목록 |


---


## day18 : VLANs - 3

1. 라우터에서의 네이티브 VLAN(Native VLAN) 설정 두가지 방법
    - 네이티브 VLAN의 프레임은 태깅되지 않아 헤더 크기가 줄어들므로 전송 효율이 향상되는 장점이 있다. 라우터에서 이를 동작하게 만드는 설정법이 2가지 존재한다.

    - 방법 1: 서브 인터페이스에 `native` 옵션 부여
    ```
    R1(config)# interface g0/0.10
    R1(config-subif)# encapsulation dot1q 10 native
    R1(config-subif)# ip address 192.168.1.62 255.255.255.192
    ```
    
    - 리우터는 `g0/0.10` 서브 인터페이스로 들어오는 태그 없는 프레임을 VLAN 10 트래픽으로 간주하여 처리하며, 해당 서브 인터페이스로 내보내는 프레임 역시 802.1Q 태그를 붙이지 않고 전송한다. 

    - 방법 2: 물리 인터페이스 자체에 직접 IP 할당
    ```
    R1(config)# no interface g0/0.10         ! 기존 서브 인터페이스 삭제
    R1(config)# interface g0/0
    R1(config-if)# ip address 192.168.1.62 255.255.255.192  ! 물리 포트에 직접 부여
    ```

    - 물리 포트는 태그를 읽지 못하는 일반 포트라서 별도의 `encapsulation`명령어를 넣지 않아도, 스위치에서 태그 없이 넘어오는 패킷은 그냥 물리 인터페이스가 직속으로 받아서 처리한다. 
    - 라우터의 물리 포트는 태그 없는 프레임을 기본 수신하므로, 자연스럽게 native vlan의 역할을 수행하게 된다.

2. 와이어 샤크 캡처를 통한 802.1Q 태그 검증
    - 태깅된 프레임(Tagged Frame, VLAN 20)
        <img width="1960" height="604" alt="Image" src="https://github.com/user-attachments/assets/ebc6c723-dab8-4356-a54d-f804bc93c4a5" />
        - Ethernet Header 내의 Source MAC qkfh enldp 802.1Q Virtual LAN 필드가 삽입되어 있다.
        - TPIC : `0x8100` (802.1Q태그임을 식별)
        - PCR : 0(우선순위 없음)
        - DEI : 0(혼잡 시 드롭 대상 아님)
        - VID(VLAN ID) : 20

    - 태그 없는 프레임(Untagged Frame, Native VLAN 10)
        <img width="1956" height="358" alt="Image" src="https://github.com/user-attachments/assets/e0c62b3b-7748-43f4-9c3e-cdfad6cafac6" />
        - R1이 Native VLAN 10으로 지정된 단말로 패킷을 되돌려줄 때, 와이어샤크 캡처 화면의 이더넷 헤더에는 802.1Q 태그 필드 자체가 완전히 존재하지 않는다. 

3. L3 스위치(Multi-layer Switch)기반 Inter-VLAN 라우팅
    - 라우팅에 의존하던 기존 ROSA 방식의 트렁크 병목 현상을 극복하기 위해 , L3 지원 스위치 내부에서 고속 라우팅을 직접 처리하는 방식이다. 

    1. SVI(Switch Virtual Interface, 스위치 가상 인터페이스)
        - L3 스위치 내부 소프트웨어 상 특정 VLAN 번호마다 1:1로 생성하는 가상의 L3 인터페이스이다.
        - 각 서브넷 PC들의 디폴트 게이트웨이 IP 주소로 작동한다. 
        ```
        SW2(config)# ip routing                 ! (★필수) 스위치 전체 L3 라우팅 기능 활성화
        SW2(config)# interface vlan 10
        SW2(config-if)# ip address 192.168.1.62 255.255.255.192
        SW2(config-if)# no shutdown            ! (★필수) SVI는 기본적으로 shutdown 상태임
        ```
    
    2. 라우티드 포트
        - L3 스위치의 특정 물리적 포트에서 L2 스위칭 기능을 끄고, 라우터 포트처럼 직접 IP 주소를 부여하여 점대점 L3 링크로 동작시키는 포트이다. 
        ```
        SW2(config)# interface g0/1
        SW2(config-if)# no switchport           ! L2 스위치 포트 속성을 제거하고 L3 포트로 전환
        SW2(config-if)# ip address 192.168.1.193 255.255.255.252
        ```

4. SVI 상태가 `Up/Up`이 되기 위한 4가지 절대 조건
    - SVI 인터페이스를 생성하고 `no shutdown`을 했음에도 불구하고 생태가 `Up/Up`이 되지 않고 `Down/Down`으로 상주하는 원인과 해결조건이다. 

    1. 스위치 데이터베이스에 해당 VLAN번호가 실제로 생성되어 존재해야 한다.
    2. 해당 VLAN에 속한 최소 1개 이상의 물리 엑세스 포트가 `Up/Up` 상태이거나, 해당 VLAN을 허용하는 트렁크 포트가 `Up/Up` 상태로 존재해야 함
    3. VLAN wkcprk `shutdown`상태가 아니여야 함
    4. SVI 인터페이스 자체에 `no shutdown`이 적용되어 있어야 함


### 주요 어휘

| 영문 표현 | 뜻 / 문맥 | 쓰임새 |
| :--- | :--- | :--- |
| Multi-layer Switch (Layer 3 Switch) | 다계층 스위치 (L3 스위치) | L2 스위칭 기능과 L3 IP 라우팅 기능을 단일 하드웨어에서 동시에 수행하는 장비 |
| SVI (Switch Virtual Interface) | 스위치 가상 인터페이스 | L3 스위치 내부에서 특정 VLAN의 게이트웨이 역할을 수행하도록 IP를 부여하는 논리 포트 |
| Routed Port | 라우티드 포트 | 'no switchport' 명령을 통해 L2 스위치 포트를 라우터 물리 포트처럼 L3 전용으로 전환한 포트 |
| IP Routing Command (`ip routing`) | IP 라우팅 활성화 명령어 | L3 스위치에서 자체 라우팅 테이블을 생성하고 패킷 교환을 시작하도록 제어하는 글로벌 명령어 |
| Untagged Frame | 태그 없는 프레임 | 802.1Q 헤더가 부착되지 않은 일반 이더넷 프레임으로, Native VLAN 전송 시 발생 |
| SVI Up/Up Conditions | SVI 활성화 조건 | SVI가 정상 동작하기 위해 VLAN 존재 및 관련 포트 활성화 등 4가지 필수 요건 |
