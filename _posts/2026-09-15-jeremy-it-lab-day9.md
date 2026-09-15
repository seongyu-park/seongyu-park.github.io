---
title: "스위치 인터페이스"
date: 2026-09-15 09:00:00 +0900
categories: [Network, CCNA]
tags: [network, ccna, jeremy-it-lab]
toc: true
math: true
comments: true
---

## day9 : 스위치 인터페이스

1. 스위치 포트 vs 라우터 포트의 기본 동작 차이
    - 기본 활성화 상태 (Default Administrative State)
        - 라우터 인터페이스 : 보안상 기본적으로 `shutdown` 명령어가 적용되어 있어 케이블을 꽂아도 `administratively down / down` 상태로 비활성화되어 있다.
        - 스위치 인터페이스 기본적으로 `no shutdown` 상태이다. 케이블을 꽂고 반대편 장비가 켜져 있으면 별도 설정 없이 즉시 `up/up` 상태가 된다.
        - 스위치 포트에서 케이블이 뽑혀 있으면 `down / down` 상태가 되며 관리자가 인위적으로 끈 `administratively down / down` 상태와 다르다. 
        (1계층 / 2계층)
    
    - IP 주소 할당 여부
        - 일반적인 L2 스위치 포트는 프레임을 스위칭하는 물리 포트이므로 개별 포트에 IP 주소를 부여하지 않는다. (`unassigned` 유지)
    
2. 전이중(Full Duplex) vs 반이중(Half Duplex) 통신 방식
    - Half Duplex(반이중)
        - 장비가 송신과 수신을 동시에 수행할 수 없음, 단방향 통신
        - 수신 중에는 송신을 멈추고 대기해야 한다.
        - 허브에 연결된 모든 기기는 반이중 방식으로 동작해야 한다. 
        허브 내부에는 주소 판별 칩셋이 없어 전이중 방식을 시도하면 내부 통로에서 신호가 뒤엉켜 깨지는 데이터 충돌(Collision)이 발생한다.
    
    - Full Duplex(전이중)
        - 송신과 수신을 동시에 수행 가능하다.
        - 대기할 필요가 없어 충돌이 발생하지 않으며 현대 스위치 네트워크의 표준 방식이다.

3. 허브(Hub)와 충돌 도메인(Collision Domain), 그리고 CSMA/CD
    - 허브의 특성
        - L1(물리계층) 장비(Repeater)로, 한 포트로 들어온 전기 신호를 모든 포트로 단순 복제(Flooding)한다. 브로드캐스트
        - 두 기기가 동시에 신호를 보내면 전기 신호가 겹치며 충돌이 발생한다.
        - 허브에 물린 모든 장비는 하나의 단일 충돌 도메인에 묶이게 된다.
        즉 여러기기가 동시에 통신하면 내부 통로에 충돌이 일어나 충돌 영역이 하나로 묶이는 것을 의미한다.
    
    - CSMA/CD(Carrier Sense Multiple Access with Collision Detection)
        - 반이중 이더넷 환경에서 충돌을 회피하고 처리하기 위한 제어 방식
        - Carrier Sense : 회선에 다른 신호가 흐르는지 먼저 귀를 기울려 확인한다.
        - Multiple Access : 아무 신호도 감지되지 않을 때 여러 단말이 회선에 접근하여 전송한다.
        - Collision Detection : 전송 중 충돌이 감지되면 재민 신호(Jamming Signal)를 회선 전체에 뿌려 충돌 사실을 알린다.
        - 모든 장비는 각자 임의의 무작위 시간(Random Backoff Time)동안 대기한 후 전송을 재시도 한다.
    
    - 스위치(Switch)의 개선점
        - 스위치는 L2 장비로 버퍼링과 MAC 주소 기반 포워딩을 수행하므로 각 포트마다 독립된 충돌 도메인을 생성한다. 
        (주소기반 포워딩 : 데이터 패킷을 받았을 때 해당 패킷의 목적지 주소을 보고 인터페이스를 결정하는 동작)
        - 전이중으로 동작하여 신호 충돌이 본질적으로 발생하지 않는다.

4. 속도 및 듀플렉스 자동 협상(Auto-Negotiation)
    - 동작 원리 
        - 기본 설정 `speed auto`, `duplex auto`이다.
        - 두 장비가 링크로 연결 될때 바로 데이터를 주고받지 않고 먼저 FLP(Fast Link Pulse) 전기 신호를 통해 자신의 지원 능력(10/100/1000 Mbps, Half/Full)을 광고하고 둘다 지원하는 최상의 조건으로 자동 합의 된다.
    
    - 상대방 장비의 Auto-negotiation이 꺼져있는 경우의 기본 판별 규칙
        - 속도 : 상대방이 쏘아 보내는 전기적 펄스를 감지하여 상대 속도에 맞춘다. 
        만일 감지에 실패하면 가장 낮은 속도인 10Mbps 적용한다.

        - 듀플렉스 : 장비의 전이중, 반이중 통식 방식을 결정하는 통신방식이다.
            - 감지된 속도가 10 Mbps, 100Mbps인 경우 -> 안전을 위해 Half Duplex로 설정
            과거에는 10/100Mbps 시절 허브 같은 반이중 장비가 많았기에 충돌 방지를 위해서이다.
            - 감지된 속도가 1000 Mbps 이상인 경우 -> Full Duplex로 설정
        
        - 듀플렉스 불일치 (Duplex Mismatch)의 위험성
            - 상대방은 Full인데 스위치가 Half로 결정되면 Duplex Mismatch가 발생한다.
            - Full 측은 상대방 수신 대기를 고려하지 않고 데이터를 마구 전송하여 충돌이 다발하고 심각한 네트워크 성능 저하와 패킷 유실이 발생한다.
    
5. 스위치 포트 설정 CLI
    - 수동 속도 / 듀플렉스 설정
    ```
    SW1(config-if)# speed 100            ! (10, 100, 1000, auto 중 선택)
    SW1(config-if)# duplex full          ! (half, full, auto 중 선택)
    SW1(config-if)# description ### Link to R1 ###
    ```

    - 일괄 범위 설정 (`interface range`)
        - 사용하지 않는 포트들을 보안상 일괄적으로 일괄 셧다운할 때 사용한다.
        - 연속된 범위 : `interface range fastEthernet 0/5 - 12`
        - 불연속 범위(콤마 사용) : `interface range f0/5 - 6, f0/9 - 12`
        - 범위 모드에서 `shutdown`을 치면 지정된 모든 포트가 동시에 차단된다.

6. 인터페이스 상태 및 오류 카운터 확인 명령어
    - `show interfaces status`
        - 스위치 전용 필수 확인 명령어로 각 포트의 정보를 테이블 형태로 확인한다. 
            - Port
            - Name(설명 라벨)
            - Status(connected/notconnect/disabled)
            - Vlan, Duplex(a-full, full)
            - Speed(a-100, 100)
            - Type
        - 참고 : `a-`가 붙어 있으면 자동 협상(Auto-negotiation)으로 결정되었음을 의미한다.
    
    - `show interfaces <인터페이스명>` (오류 카운터 분석)
        - Runts : 이더넷 최소 크기 규격인 64바이트 미만으로 깨져서 들어온 프레임 수
        - Giants : 이더넷 기본 최대 크기 규격인 1518바이트를 초과한 프레임 수
        - CRC : 프레임 트레일러의 FCS 체크 계산값이 맞지 않아 변조/손상된 불량 프레임 수
        - Frame : 형식이 깨지거나 올바르지 않은 프레임 수
        - Input errors : 위 수신 오류들의 총합
        - Output errors : 스위치가 송출하려 했으나 에러로 실패한 프레임 수


### 주요 어휘

| 영문 표현 | 뜻 / 문맥 | 쓰임새 |
| :--- | :--- | :--- |
| Half Duplex | 반이중 통신 | 한 번에 송신 또는 수신 중 하나만 수행 가능한 단방향 교대 통신 방식 |
| Full Duplex | 전이중 통신 | 송신과 수신을 동시에 수행할 수 있어 충돌이 없는 양방향 통신 방식 |
| Collision Domain | 충돌 도메인 | 전기적 신호가 겹쳐 데이터 충돌이 발생할 수 있는 물리적 네트워크 영역 |
| Hub | 허브 (리피터) | 1계층 장비로 들어온 신호를 모든 포트로 재생 살포하여 전체가 하나의 충돌 도메인을 형성 |
| CSMA/CD | 반이중 충돌 감지 프로토콜 | Carrier Sense Multiple Access with Collision Detection. 반이중 매체 접근 및 충돌 제어 규약 |
| Jamming Signal | 재밍 신호 | CSMA/CD에서 충돌 발생 시 모든 노드에 충돌 사실을 알리기 위해 방출하는 신호 |
| Auto-Negotiation | 자동 협상 기능 | 연결된 양측 장비가 최적의 전송 속도(Speed)와 듀플렉스(Duplex)를 자동으로 절충하는 기능 |
| Duplex Mismatch | 듀플렉스 불일치 | 링크 양단이 서로 다른 듀플렉스(한쪽 Full, 다른 쪽 Half)로 설정되어 충돌이 다발하는 장애 |
| Interface Range | 인터페이스 일괄 지정 명령 | 여러 포트를 한 번에 묶어 'shutdown' 등의 명령을 동시에 적용하는 CLI 문법 |
| Runts | 런트 (과소 프레임) | 이더넷 표준 최소 크기인 64바이트 미만으로 인입된 비정상 프레임 카운터 |
| Giants | 자이언트 (과대 프레임) | 표준 최대 크기인 1518바이트를 초과하여 인입된 비정상 프레임 카운터 |
| CRC Error | 순환 중복 검사 오류 | 프레임 끝부분 트레일러(FCS)의 계산값이 맞지 않아 데이터가 손상되었음을 나타내는 카운터 |

