---
title: "Jeremy's IT Lab Day1~2"
date: 2026-09-10 09:00:00 +0900
categories: [Network, CCNA]
tags: [network, ccna, jeremy-it-lab]
toc: true
math: false
comments: true
---

# jeremy's IT Labs 정리 

## day1 : Network Devices

1. 1.네트워크란 무엇인가?
    - 정의 : 네트워크는노드들이 자원을 서로 공유할 수 있도록 해주는 디지털 통신망이다.
    - 단순히 2대의 PC를 케이블 하나로 직접 연결하기만 해도 단순한 네트워크가 형성된다.


2. 2.주요 네트워크 노드 및 장비 종류
    1. 엔드 호스트 / 엔드 포인트 : 네트워크의 출발지이지 목적지가 되는 기기들
        - 클라이언트 : 서버가 제공하는 서비스를 이용하는 기기
        - 서버 : 클라이언트에게 기능이나 서비스를 제공하는 기기

    2. 스위치 : 동인란 LAN 내에서 호스트들을 서로 연결한다.
        - 수많은 엔트 호스트를 직접 연결할 수 있도록 포트 수가 많다.
        - ex : Cisco Catalyst 시리즈

    3. 라우터 : 서로 다른 네트워크들을 연결하고 네트워크 간 트래픽을 전달한다.
        - LAN과 인터넷을 연결해 주며, 스위치에 비해 인터페이스 수가 적다.
        - Cisco ISR(integrated services router)시리즈
        
    4. 방화벽 : 설정된 보안 규칙에 따라 네트워크를 드나드는 트래픽을 감시하고 제어(차단/허용)한다.
        - 호스트 기반 방화벽 : PC나 서버 운영체제 내부에서 실행되는 소프트웨어 방화벽
        - 네트워크 방화벽 : 전용 하드웨어 어플라이세서로 네트워크 경계선에 배치
        - 차세대 방화벽(NGFW, next-generation firewall) : 기존 방화벽에서 지능형 패킷/ 애플리케이션 필터링 기능을 결합한 방화벽
        - Cisco ASA, Cisco Firepower

### 주요 어휘

| 영문 표현 | 뜻/문맥 | 쓰임새 |
| :--- | :--- | :--- |
| Forward / Forwarding | 전달하다 / 포워딩 | 패킷이나 프레임을 다음 목적지 인터페이스로 보내는 행위 |
| Drop / Discard | 폐기하다 | 방화벽이나 라우터가 유효하지 않거나 정책상 금지된 패킷을 버리는 것 |
| Permit / Deny (Allow / Block) | 허용하다 / 차단하다 | 방화벽 정책(Rule)이나 ACL(접근 제어 목록)에서 트래픽을 통과시키거나 막을 때 쓰는 표준 용어 |
| Monitor & Control | 모니터링 및 통제 | 방화벽의 기본 정의에 항상 세트로 등장하는 표현 |
| Request / Provide | 요청하다 / 제공하다 | 클라이언트(Request)와 서버(Provide)의 동작을 구분 짓는 기본 동사 |
| Interface / Port | 인터페이스 / 포트 | 장비가 케이블을 통해 네트워크와 물리적·논리적으로 연결되는 접점 |
| Traffic | 트래픽 | 네트워크 상을 이동하는 데이터의 흐름 |
| Appliance | 전용 기기/장비 | 특정 기능(보안, 라우팅 등)만을 위해 독립적으로 제작된 하드웨어 장비 |

---

## day2 : Complete Course

1. 1.스위치 포트와 RJ-45
    - 인터페이스/포트 : 스위치는 PC, 서버 등의 엔드 호스트를 대량으로 연결하기 위해 많은 수의 포트(24~48개)를 가지고 있다.
    - RJ-45 : 구리선(UTP) 기반 이더넷 케이블을 장비에 연결하기 위한 표준 8핀 커넥터 및 포트 규격이며 표준화된 인터페이스이다.

2. 2.데이터 속도 단위 (Bits vs Bytes)
    - Bit : 컴퓨터 데이터의 최소 단위(0,1)
    - Byte : 8개의 비트 묶음 (1byte = 8bite)
    - 네트워크 전속 속도 단위 : 네트워크 속도는 항상 초당 비트 수(bps)로 측정된다.
    - 저장 장치 용량은 byte 단위 사용

3. 3.구리선 이더넷 표준(IEEE 802.3 standards)
    - 모든 구리선의 트위스트 페어(twisted pair) 케이블의 최대 전송 거리는 공통적으로 100m로 제한된다.

| 일반 명칭 (Common Name) | 속도 (Speed) | 비공식 표준명 (Informal Name) | IEEE 공식 표준 | 최대 거리 |
| :--- | :--- | :--- | :--- | :--- |
| **Ethernet** | 10 Mbps | 10BASE-T | IEEE 802.3 | 100 m |
| **Fast Ethernet** | 100 Mbps | 100BASE-TX | IEEE 802.3u | 100 m |
| **Gigabit Ethernet** | 1 Gbps | 1000BASE-T | IEEE 802.3ab | 100 m |
| **10 Gigabit Ethernet** | 10 Gbps | 10GBASE-T | IEEE 802.3an | 100 m |

- T(Twisted Pair) : 꼬인쌍선 구리 케이블을 의미한다.
- X(Block Coding/Extender) : 블록 코딩 방식을 사용하는 규격을 의미한다.
    - 광섬유 표준에서 8B/10B 고속 인코딩을 사용하는 규격에 주로 쓰이지만 구리선 표준에도 해당 방식을 그대로 가져워 고속 전송을 구현할 때도 사용


4. 4.UTP, 핀아웃 규격 및 케이블 종류
    - UTP (Unshielded Twisted Pair) vs STP (Shielded Twisted Pair) : 내부 차폐막(shield)의 유무의 차이로 차폐막 때문에 외부 노이즈을 견디는 능력과 주로 쓰이는 환경이 달라진다.
        - UTP : 서로 꼬여 있는 자체 만으로 어느 정도 전자기 간섭을 상쇄한다. 가격이 저렵하고 작업하기 편리하다. 일반 적인 환경에서도 기가비트 속도를 안정적으로 낼 수 있다.
        - STP : 구리선 주위나 케이블 내부 전체를 알루미늄 은박이나 구리 그물망으로 감싸 외부 전자기 신호를 원천 차단한다. 접치(Groundig)처리가 필수
        
    
    - 핀아웃 표준(T568A/T568B) : 케이블 내부 8가닥 색상선의 배열 순서 표준이다.
    - MDI vs MDI 장비 분류
        - MDI(Medium Dependent Interface)장비 : 장비가 내부적으로 1,2번 핀을 송신(TX)를 사용하고 3,6 핀이 수신(RX)한다.  PC, 라우터, 서버등이 존재한다.
        - MDI-X(MDI Crossover)장비 : 장비들은 반대로 1,2번이 수신(RX)하고 3,6번이 송신(TX)한다. 스위치, 허브 등이 존재한다.

    - 스트레이트 케이블(다이렉트) : 양 끝이 동일한 표준으로 결선된 케이블로 서로 다른 계층의 장비를 연결할 때 사용
    - 크로스오버 케이블 : 한쪽은 T568A 다른쪽은 T568B로 결선된 케이블로 동일한 유형의 장비를 연결할 때 사용한다.

    - Auto MDI-X(Automatic Medium-Dependent Interface Crossover)
        - 포트가 연결된 케이블의 핀 배열을 자동으로 감지하여 내부 회로를 전환하는 기술이다.
        - 현대 스위치는 대부분 Auto MDI-X를 지원하므로 케이블 종류는 상관없이 정상 통신이 가능하다. 

    5. 5.광섬유 케이블(Fiber-Optic Cables)
    - 전기 신호 대신 빛을 이용하여 데이터를 전송하므로 EMI 영향을 받지 않고 휠씬 먼 거리를 전송할 수 있다. 
        - MMF (Multimode Fiber, 멀티모드) : 코어 직경이 크고 저렴한 LED, 레이저 광원을 상요하여 여러 빛 경로가 발생하여 신호 왜곡이 발생하므로 스백미터의 단거리 연결에서 사용됨
        - SMF(Single-Mode Fiber, 싱글모드) : 코어직경이 매우 앏고, 고출력 레이저로 단 하나의 빛 경로만 통과한다. 분산이 거의 없어 장거리 WAN 연결에 필수적이다. 단 케이블 및 송수신 광트랜시버의 가격이 비싸다.
        - SFP(Small Form-factor Pluggable) : 스위치나 라우터에 꽂아 다양한 규격의 광케이블을 연결할 수 있게 해주는 핫스왑 지원 모듈형 트랜시버 인터페이스


### 주요 어휘

| 영문 표현 | 뜻/문맥 | 쓰임새 |
| :--- | :--- | :--- |
| UTP / STP | 비차폐 / 차폐 꼬임선 케이블 | Unshielded/Shielded Twisted Pair. 물리 계층 케이블 분류 |
| Crosstalk | 신호 누화 / 혼선 | 인접 전선 쌍 간의 전자기적 간섭 현상 |
| Straight-Through | 다이렉트(스트레이트) 케이블 | 서로 다른 장비군(Switch-Host) 연결 시의 기본 케이블 |
| Crossover | 크로스 케이블 | 동일 장비군(Switch-Switch, Router-Router, Host-Router) 연결 |
| Auto MDI-X | 자동 MDI/MDI-X 전환 기능 | 케이블 핀아웃을 자동 판별하므로 케이블 종류 오류를 방지함 |
| Single-Mode Fiber (SMF) | 싱글모드 광섬유 | 장거리(Long-distance, several km) 전송 지문에 단골 출제 |
| Multimode Fiber (MMF) | 멀티모드 광섬유 | 건물 내부 및 단거리 백본(Cost-effective, < 500m) 지문에 출제 |
| SFP (Transceiver) | 소형 플러그형 트랜시버 | 스위치의 광 포트 모듈 연결 장치 |
| Bandwidth / Throughput | 대역폭 / 처리량 | 링크가 지원할 수 있는 이론적 최대 속도 / 실제 전송량 |
