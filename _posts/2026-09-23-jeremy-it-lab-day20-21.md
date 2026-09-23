---
title: "Spanning Tree Protocol(STP, day20-21)"
date: 2026-09-23 09:00:00 +0900
categories: [Network, CCNA]
tags: [network, ccna, jeremy-it-lab]
toc: true
math: true
comments: true
---

## day20 : Spanning Tree Protocol - 1

L2 네트워크의 이중화(Redundancy) 구성시 필연적으로 발생하는 루프 현상 및 브로드 캐스트 스톰을 방지하는 IEEE 802.1D 표존 STP(Spanning Tree Protocol)의 기본 원리와 동작 방식 학습

1. 네트워크 이중화(Redundancy)와 L2 루프의 치명적 위험성
    - 이중화의 필요성 : 단일 장애점(SPOF, Single Point of Failure)으로 인한 24/7 서비스 중단을 막기 위해 물리적 회선과 장비를 다중 연결한다.
    (SPOF : 한 장비가 멈추면 전체 시스템이 멈추는 핵심 지점)
    (24/7 서비스 중단 : 24시간 돌아야한는 서버스에서 단일 장비 문제로 서비스가 먹통되는 상황)

    - 스패닝 트리가 없는 L2 이중화의 3대 치명적 문제점
        1. 브로드캐스트 스톰
            - L3 IP 헤더에는 루프 방지용 TTL 필드가 존재하지만, L2 이더넷 헤더에는 TTL 필드가 없다.
            - 스위치가 브로드 캐스트나 Unknown Unicast 프레임을 수신하면 인입 포트를 제외한 전 포트로 플러딩을 한다.
            - 루프 구조에서 프레임이 다수의 스위치 사이를 영구히 맴돌며 기하급수적으로 증포고디어 대역폭과 장비 자원을 마비시킨다.
        
        2. MAC 주소 플래핑(MAC Address Flapping)
            - 동일한 원본 MAC 주소를 가진 프레임이 여러 서로 다른 포트로 끊임없이 인입됨에 따라, 스위치의 MAC 주소 테이블이 포트 정보를 순식간에 수차례 덮어쓰며 진동(Flapping)한다. 

        3. 이중 프레임 수신
            - 목적지 단말이 동일한 프레임을 복수 개 수신하여 통신 에러가 발생한다. 

2. STP(IEEE 802.1D)동작 원리
    - STP는 물리적으로는 이중화 연결을 유지하면서, 논리적으로 특정 포트를 차단 상태로 만들어 루프를 차단한다. 
        - Forwarding 포트 : 정상적인 데이터 프레임 및 BPDUs를 송수신함
        - Blocking 포트 : 일반 데이터 프레임 전송.수신 및 MAC 주소 학습은 모두 차단하며, 오직 STP 제어 메시지인 BPDU(Bridge Protocol Data Unit)만 수신한다. 
        (주 회선 장애시 차단 해제되어 Forwarding으로 전환된다.)

3. BPDU와 브리지 ID
    - 스위치들은 2초마다 `Hello BPDU`를 주고받으며 선거 과정을 진행한다.
    - 브리지 ID(BID) 구조 : (총 8바이트 = 64비트)
        - 전통적 BID : Bridge Priority (16 bits) + MAC Address (48 bits)
        - 현대 802.1t/802.1D-1998 확장 BID
            - Bridge Priority (4 bits): 16비트 중 상위 4비트만 우선순위로 사용된다.(4096 단위)
            - Extended System ID(12 bits) : 해당 VLAN 번호가 자동으로 들어간다.
            - MAC Address (48bits) : 스위치 고유의 물리 MAC 주소
        - 기존 Bridge Priority :`32768`
            - $2^{16} = 65,536$개로 우선 순위로 인하여 딱 중간 값 설정함
        - 우선 순위 변경 단위 : 4비트 자릿값에 의해 반드시 `4096`의 배수 단위로만 수정이 가능하다.

4. STP 포트 역할 결정 3단계 알고리즘
    1. 루트 브리지(Root Bridge) 선출
        - 전체 네트워크에서 단 1대의 스위치만 선출
        - 선출 기준 : 가장 낮은 브리지 ID를 가진 스위치가 선출된다.
            - Bridge Priority가 가장 낮은 스위치
            - Priority가 동일하면 MAC 주소가 가장 낮은 스위치
        - 특징 : 루트 브리지의 모든 활성 포트는 지정포트(Desighated Port, DP)가 되며, 항상 Forwarding 상태이다.
        (DP(지정포트) : 대장 스위치로부터 오는 패킷을 아래쪽으로 전달해 주는 대표 포트)

    2. 루트 포트 선출
        - 루트 브리지를 제외한 나머지 모든 넌-루트 스위치마다 각 1개씩 선출된다.
        - 루트 브리지로 향하는 가장 가까운(비용이 적은) 출구 포트이다.
        - 선출 결정 순서
            - Lowest Root Path Cost : 루트 브리지까지 가는 경로의 누적 스패닝 트리 경로 비용이 가장 낮은 포트
            - Lowest Neighbor BID : 경로 비용이 같으면, 연결된 이웃 스위치의 BID가 더 낮은 쪽으로 연결된 포트
            - Lowest Neighbor Port ID : 이웃 스위치도 같으면,상대방 이웃 포트의 Port ID가 더 작은 쪽과 연결된 내 포트 
        - 속도별 STP 링크 경로비용
            - 10 Mbps: `100`
            - 100 Mbps : `19`
            - 1 Gbps: : `4`
            - 10 Gbps: : `2`
    
    3. 지정 포트(DP) 및 비지정 포트(NDP) 선출
        - 스위치 간의 매 세그먼트마다 1개의 지정 포트가 반드시 존재해야 한다.
        - 두 스위치를 연결하는 선로(세그먼트)에서 데이터를 전송할 대표 포트
        - DP 선출 기준
            - 해당 링크를 낀 양쪽 스위치 중 루트 경로 비용이 더 낮은 스위치 포트가 DP가 된다.
            - 비용이 같으면 BID가 더 낮은 스위치의 포트가 DP가 된다.
        - RP 및 DP로 선택받지 못하고 남은 포트는 비지정 포트가 되어 루프를 방지한다.

4. 정리
    - STP는 네트워크 이중화 문제 발생시 치명적인 브로드캐스트 스톰(Loop)문제를 해결한다.
    - 원리 : BPDU의 신호를 통하여 경로를 설정하고 나머지 포트를 막아 루프를 일시적으로 끊어낸다.
    - 각 스위치(브리지)간의 우선 순위를 정하고 ARP같은 통신이 스위치 내 돌지 않고 소멸하게 된다.



### 주요 어휘

| 영문 표현 | 뜻 / 문맥 | 쓰임새 |
| :--- | :--- | :--- |
| STP (Spanning Tree Protocol) | 스패닝 트리 프로토콜 | L2 이중화 회선 환경에서 차단 포트를 지정하여 L2 루프를 자동 방지하는 표준 프로토콜 |
| Broadcast Storm | 브로드캐스트 스톰 | L2 루프 회선 상에서 브로드캐스트 프레임이 무한 복제 순환하여 네트워크를 마비시키는 현상 |
| MAC Address Flapping | MAC 주소 플래핑 | 루프 패킷으로 인해 동일 MAC 주소의 매핑 포트가 스위치 테이블에서 끊임없이 변동하는 오류 |
| Bridge Protocol Data Unit (BPDU) | 브리지 프로토콜 데이터 단위 | 스위치 간 STP 정보 교환 및 루트 브리지 선출을 위해 2초마다 주고받는 제어 프레임 |
| Bridge ID (BID) | 브리지 식별자 | 4비트 Priority + 12비트 Sys-ID + 48비트 MAC 주소로 구성된 스위치의 고유 식별자 |
| Root Bridge | 루트 브리지 | 스패닝 트리 트리의 중심이 되는 기준 스위치 (가장 낮은 BID 소유) |
| Root Port (RP) | 루트 포트 | 넌-루트 스위치에서 루트 브리지 방향으로 가는 최단 경로(최저 Cost) 포트 |
| Designated Port (DP) | 지정 포트 | 각 물리 링크 세그먼트 상에서 BPDU를 송출하는 포트 (루트 브리지 포트는 전원 DP) |
| Non-Designated Port (NDP) | 비지정 포트 (차단 포트) | RP나 DP로 선택되지 못해 루프 방지를 위해 데이터 통신이 차단(Blocking)된 포트 |
| Root Path Cost | 루트 경로 비용 | 루트 브리지까지 가는 경로 상의 송출(Outgoing) 포트 속도별 Cost를 합산한 값 |


---


## day21 : Spanning Tree Protocol - 2

1. 스패닝 트리 4가지 포트 상태 & 타이머
    - STP 포트는 루프 방지와 정상 데이터 전송을 위해 총 4가지 핵심 상태를 가진다.
    1. 포트의 4가지 상태
        - Blocking(차단상태) : Non-Designated 포트의 기본 상태. 일반 데이터 프레임 전송/수신 불가, MAC 주소 학습 불가. 오직 BPDU 메시지만 수신한다.(BPDU 송출도 하지 않는다.)

        - Listening(경청상태) : 토폴로지 변경 시 Blocking에서 Forwarding으로 넘어가기 위한 첫 번째 수동적 이행 상태. 일반 데이터 프레임 전송/수신 불가, MAC 주소 학습 불가, BPDU 송수신만 수행하며 Forward Delay 타이머(기본 15초)

        - Learnig(학습상태) : Listening 단계를 거친 포트의 두번째 수동적 이행 상태로 일반 데이터 프레임 전송/수신은 여전히 불가능하지만, 포트로 들어오는 프레임의 source MAC 주소를 읽어 MAC 주소 테이블을 미리 구축하기 시작한다.
        Forward Delay(기본 15초) 동안 유지된다.

        - Forwarding(전송 상태) : Root 포트 및 Designated 포트의 안정화된 정상 동작 상태로 일반 데이터 프레임 송수신, MAC 주소 학습, BPDU 송수신이 모두 정상 수행된다. 

        - (참고) Disabled : 관리자가 명령어로 포트를 비활성화 한 상태
    
    2. STP 3대 핵심 타이머
        - Hello Timer(기본 2초) : 루트 브리지가 Hello BPDU를 생성하여 송출하는 주기
        - Forward Delay Timer(기본 15초) : Listening 및 Learning 각 상태에 머무는 시간으로 따라서 Blocking에서 Forwarding까지 총 30초의 대기 시간이 소요된다. 
        - MAX Age Timer(기본 20초) : 이웃으로부터 BPDU 수신이 끊겼을 때, 기존 STP 토폴로지 정보를 유지하며 대기하는 최대 시간으로 BPDU가 도착하지 않고 20초가 지나면 포트는 상태 재계산을 시작한다.
        - 장애 복구 소요 시간 : 물리 회선 장애 발생시 MAX Age(20초) + Forward Delay 2회(30초) = 최대 50초간 통신 장애(대기 시간)가 발생한다. 
    
2. BPDU(Bridge Protocol Data Unit) 헤더 구조
    - 스위치 간 통신을 위해 전달되는 제어 프레임의 세부 필드이다. 
        - 목적지 MAC 주소
            - 스위치가 BPDU 패킷을 보낼 때 나타내는 L2 목적지 주소로 일반 PC로 가는 패킷이 아닌 스위치 장비들 끼리만 주고 받는 멀티캐스트 주소를 사용한다. 
            - Cisco PVST+ : `01:00:0C:CC:CC:CD`
            - IEEE 표준 802.1D(Classic STP): `01:80:C2:00:00:00`
            (PVST+ : 시스코 전용 PVST+환경에서 VLAN 별 BPDU를 주고받기 위해 시스코가 독자적으로 정의한 멀티캐스트 주소이다.)
        - 핵심 필드 구성 : Root BID(우선순위+VLAN ID+MAC), Root Path Cost, Sender BID, Sender Port ID(Port Priority + Port Number), Timers(Message Age, Max Age, Hello, Forward Delay)

3. STP 보조 보호 기능
    1. PortFast(포트패스트)
        - PC, 서버, 프린터 등 단말 장치가 연결된 액세스 포트에 설정
        - 30초간의 Listening/Learning 대기 단계를 즉시 건너뛰고 포트 연결 직후 즉시 Forwarding 상태로 전환시킨다. 
        - CLI 설정 문법
            - 인터페이스 개별 적용 : `SW1(config-if)# spanning-tree portfast`
            - 전역 일괄 적용 (모든 Access 포트 대상) : `SW1(config)# spanning-tree portfast default`
    
    2. BPDU Guard(BPDU 가드)
        - PostFast가 설정된 단말 포트에 사용자가 임의로 스위치를 연결하여 루프를 만드는 사고를 방지한다. 
        - 해당 포트로 BPDU 패킷이 수신이 되면 즉시 포트를 비활성화(`err-disable`/`shutdown`)시켜 루프 생성을 원천 차단한다.
        - 복구 방법 : 문제 원인 제거 후 포트에 들어가 `shutdown` -> `no shutdown` 실행
        - CLI 설정 문법
            - 인터페이스 개별 적용 : `SW1(config-if)# spanning-tree bpduguard enable`
            - 전역 일괄 적용 (PortFast 활성화 포트 대상) : `SW1(config)# spanning-tree portfast bpduguard default`
    
    3. 기타 보호 기능(Root Guard/Loop Guard)
        - Root Guard : 해당 포트로 외부에서 더 우수한(더 낮은 BID) BPDU가 들어와도 루트 브리지 지위를 빼앗기지 앟도록 해당 포트를 차단(`root-inconsistent`)한다.
        - Loop Guard : 단방향 링크 장애로 BPDU 수신이 끊겼을 때, 차단 포트가 멋대로 Forwarding으로 풀려 루프가 발생하는 것을 방지한다. 
    
    4. 시스코 스위 STP 기본 설정 및 부하 분산(Load Balancing)
        1. 루트 브리지 수동 설정
            - 특정 스위치를 특정 VLAN의 루트 브리지로 직접 지정하는 명령어이다.
            - `SW1(config)# spanning-tree vlan 10 root primary`(우선 순위를 자동으로 하향 조정)
            - `SW1(config)# spanning-tree vlan 10 root secondary`(우선 순위를 자동으로 하향 조정)
            - 직접 수동 지정 : `SW1(config)# spanning-tree vlan 10 priority 4096`

        2. PVST+ 기반 STP 부하 분산(STP Load Balancing)
            - 단일 루트 브리지 사용시 일부 트렁크 링크가 항상 차단되어 대역폭이 낭비된다. 
            - VLAN별로 서로 다른 스위치를 루트 브리지로 지정하는 것이다. 따라서 서로 차단 되는 포트가 서로 달라진다. 따라서 물리 회선의 대역폭을 효율적으로 분산 이동할 수 있다. 

### 주요 어휘

| 영문 표현 | 뜻 / 문맥 | 쓰임새 |
| :--- | :--- | :--- |
| Listening State | 경청 상태 (15초) | BPDU 수신/송신만 수행하며 토폴로지 변경을 감지하는 첫 번째 STP 전환 상태 |
| Learning State | 학습 상태 (15초) | 데이터 전송은 차단되나, 인입 프레임으로 MAC 주소 테이블을 미리 구축하는 전환 상태 |
| Forward Delay Timer | 전송 지연 타이머 | Listening 및 Learning 상태에 머무는 각각의 대기 시간 (기본값 15초) |
| Max Age Timer | 최대 대기 타이머 | 이웃 BPDU 수신이 끊겼을 때 기존 상태를 유지하며 기다리는 최대 시간 (기본값 20초) |
| PortFast | 포트패스트 | Access 포트의 30초 STP 대기 시간을 건너뛰고 즉시 Forwarding으로 전환하는 기능 |
| BPDU Guard | BPDU 가드 | PortFast 포트로 BPDU 수신 시 즉시 포트를 차단(err-disable)하여 루프를 방지하는 기능 |
| Primary / Secondary Root | 주 / 보조 루트 브리지 | 특정 VLAN에 대해 1순위 및 2순위 루트 브리지 역할을 하도록 우선순위를 자동 계산 설정하는 기능 |
| STP Load Balancing | STP 부하 분산 | VLAN별로 루트 브리지를 다르게 지정하여 차단 포트를 분산시킴으로써 대역폭 활용을 극대화하는 기법 |
