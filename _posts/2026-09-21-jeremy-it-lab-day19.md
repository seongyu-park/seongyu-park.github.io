---
title: "DTP/VTP(day19)"
date: 2026-09-21 09:00:00 +0900
categories: [Network, CCNA]
tags: [network, ccna, jeremy-it-lab]
toc: true
math: true
comments: true
---

## day19 - DTP/VTP

1. DTP(Dynamic Trunking Protocol, 동적 트렁킹 프로토콜)
    - 스위치 포트를 사람이 일일이 `switchport mode access` 또는 `switchport mode trunk`로 지정하지 않아도, 대항 장비와 DTP 패킷을 주고 받아 포트의 동작 모드를 자동으로 협상하는 시스코 전용 L2 프로토콜이다.  

    - DTP의 주요 4가지 관리 모드
        1. `access`(`switchport mode access`) : DTP 협상을 비활성화하고 포트를 강제로 Access 포트로 고정한다.
        2. `trunk`(`switchport mode trunk`) : 포트를 강제로 Trunk 포트로 고정하지만, 대항 포트로 DTP 프레임은 계속 송출한다.
        3. `dynamic desirable`(`switchport mode dynamic desirable`) : 상대방에게 먼저 DTP 패킷을 보내며 능동적으로 트렁크 성립을 시도한다.
        4. `dynamic auto`(`switchport mode dynamic auto`) : 상대방이 트렁크를 요청해오면 응하지만, 자신이 먼저 능동적으로 요구하지는 않는 수동적 상태이다. 
    
    - 스위치 장비 세대별 기본 설정값
        - 구형 스위치 : 기본값이 `dynamic desirable`이다.
        - 신형 스위치 : 기본값이 `dynamic auto`이다.  

    | 내 포트 모드 | 상대 포트 모드 | 최종 결정 모드 (Operational Mode) |
    | :--- | :--- | :--- |
    | **Dynamic Desirable** | Trunk / Dynamic Desirable / Dynamic Auto | **Trunk** (트렁크 성립) |
    | **Dynamic Desirable** | Access | **Access** (액세스 작동) |
    | **Dynamic Auto** | Trunk / Dynamic Desirable | **Trunk** (트렁크 성립) |
    | **Dynamic Auto** | Dynamic Auto / Access | **Access** (액세스 작동) |
    | **Trunk** | Access | **Mismatch Error** (통신 불가 / 설정 오류) |

    - DTP 패킷 차단 명령어(`switchport nonegotiate`)
        - 포트가 `trunk`로 고정되어 있어도 DTP 프레임이 계속 출력되므로, 보안상 DTP 패킷 송출을 완전히 차단하려면 `switchport nonegotiate` 명령어를 함께 설정해야 한다. 
        - 실무에서 보안 취약점(VLAN Hopping 공격)을 방지하기 위해 모든 포트의 DTP를 끄고(`switchport mode access` 또는 `switchport nonegotiate`)수동으로 지정하는 것이 절대 권장된다.

2. VTP(VLAN Trunking Protocol, 가상 LAN 트렁킹 프로토콜)
    - 중앙 서버 스위치에서 VLAN을 생성/수정/삭제하면, 네트워크 내의 다른 스위치들이 트렁크 링크를 통해 VLAN 데이터베이스를 자동으로 동기화 받도록 돕는 시스코 전용 프로토콜이다. 

    - VTP의 3가지 동작 모드
        1. Server 모드
            - VLAN의 생성, 수정, 삭제가 가능하다.
            - VLAN 데이터베이스를 NVRAM에 저장하며, 변경 시 리비전 번호(Configuration Revision Number)를 1씩 증가시킨 후 트렁크 포트로 광고(Advertisement)를 전송한다.
            (리비전 번호 : VLAN 설정의 버전 관리용 카운터)
            - 주의 : 자신보다 높은 리비전 번호를 가진 광고를 받으면 동일하게 서버 모드라 할지라도 상대방의 VLAN 데이터베이스로 자신의 설정을 덮어쓴다.
        
        2. Client 모드
            - CLI에서 VLAN을 새로 만들거나 수정/삭제할 수 없다. (명령어 거부)
            - 서버로부터 전송되어 온 리비전 번호가 더 높은 VLAN 정보를 받아 자신의 데이터베이스를 동기화하고, 이 광고를 다른 스위치로 중계(Forward)한다. 
        
        3. Transparent 모드
            - VTP 도메인에 참여하지 않으며, 중앙 서버와 자신의 VLAN 데이터 베이스를 동기화하지 않는다.
            - NVRAM에 자신만의 독립된 VLAN 설정을 수동으로 저장/관리하며(수동 생성/삭제가능), 동일 도메인의 VTP 광고 패킷을 자신은 반영하지 않고 다른 스위치로 통과만 시켜준다.
    
    - 리비전 번호와 Nerwork Wipeout 위험성
        - VTP는 오직 리비전 번호가 가장 높은 VTP 패킷을 최신 정보로 간주한다. 
        - 예를 들어 중고 스위치의 리비전 번호(50)가 기존 운영 망의 리비전 번호보다 높은 상태로 동일한 VTP 도메인에 연결되면, 전체 네트워크 스위치들의 VLAN 정보가 순식간에 날라가버리는 심각한 네트워크 장애(Network Wipeout)가 발생한다. 

        - 리비전 번호를 0으로 리셋하는 2가지 방법 
            1. VTP 도메인 이름을 미사용 이름으로 임시로 변경했다가 돌아오기
            VTP는 도메인 이름이 변경되는 순간 기존의 리비전 카운터를 무효화하고 0으로 즉시 초기화한다.
            2. VTP 모드를 `Transparent`로 변경했다가 다시 원모드로 복귀하기
    
    - VTP 버전 간 차이점
        - VTP v1/v2 : 일반 범위만 동기화 가능(`1~1005`)
        - VTP v3 : 확장 범위 VLAN 동기화 지원 및 승인되지 않는 스위치에 의한 데이터베이스 덮어쓰기 방지 기능 추가

### 주요 어휘

| 영문 표현 | 뜻 / 문맥 | 쓰임새 |
| :--- | :--- | :--- |
| DTP (Dynamic Trunking Protocol) | 동적 트렁킹 프로토콜 | 스위치 포트 간의 모드(Access/Trunk)를 DTP 패킷 교환으로 자동 협상하는 시스코 프로토콜 |
| Dynamic Desirable | 능동적 동적 모드 | 상대방 포트에 DTP 패킷을 적극 송출하여 먼저 트렁크 성립을 유도하는 DTP 관리 모드 |
| Dynamic Auto | 수동적 동적 모드 | 상대방이 트렁크를 요구할 때만 트렁크로 응하고, 먼저 요청하지는 않는 DTP 관리 모드 |
| Non-negotiate (`switchport nonegotiate`) | DTP 협상 금지 명령 | 트렁크 포트에서 DTP 협상 프레임 송출을 강제로 차단하여 보안을 강화하는 명령어 |
| VTP (VLAN Trunking Protocol) | 가상 LAN 트렁킹 프로토콜 | 중앙 VTP 서버의 VLAN 데이터베이스 동기화를 통해 네트워크 전체 VLAN 관리를 자동화하는 프로토콜 |
| Configuration Revision Number | 설정 리비전 번호 | VTP 동기화의 기준이 되는 32비트 정수 값으로, VLAN이 수정될 때마다 1씩 증가함 |
| VTP Server / Client / Transparent | VTP 서버 / 클라이언트 / 트랜스페어런트 | VTP 동기화 및 VLAN 생성/중계 권한을 정의하는 3가지 기본 동작 모드 |
| Network Wipeout | 네트워크 전체 마비 현상 | 리비전 번호가 높은 외부 스위치 유입으로 인해 운영 망 전체의 VLAN이 원치 않게 초기화되는 장애 |
