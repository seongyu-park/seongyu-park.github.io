---
title: "IPv4 헤더"
date: 2026-09-15 09:00:00 +0900
categories: [Network, CCNA]
tags: [network, ccna, jeremy-it-lab]
toc: true
math: true
comments: true
---

## day 10 : IPv4 헤더

1. PDU 게층 복습 및 Layer 3 헤더의 위치
    - 상위 계층 데이터가 4계층 헤더(TCP/UDP)로 캡슐화 되면 세그먼트가 된다. 
    - 3계층 헤더(IPv4/IPv6)가 붙어 캡슐화 된 전체 단위를 패킷이라고 부른다.
    - 2계층 헤더 및 트레일러(이더넷)가 붙으면 프레임이 된다.
    - L3 IPv4 헤더는 패킷을 전세계의 다른 네트워크로 전달하는 라우팅에 필요한 모든 제어 정보가 담겨 있다.

2. IPv4 헤더의 14가지 필드 분석
    - IPv4 헤더는 32 비트(4바이트)단위로 한 줄씩 배치되며, 죄측에서 우측, 상당에서 하단 순으로 해석된다.
    <img width="309" height="262" alt="Image" src="https://github.com/user-attachments/assets/4cee4b2b-d13f-43b1-8c92-e318fa1cc5aa" />

    | 필드 명칭 | 크기 (Bits) | 역할 및 세부 동작 메커니즘 |
    | :--- | :---: | :--- |
    | **Version (버전)** | 4 bits | 사용 중인 IP 프로토콜 버전 명시. IPv4는 항상 **4** (`0100`)로 고정. (IPv6는 `6` = `0110`) |
    | **IHL (Internet Header Length)** | 4 bits | **4바이트(32비트) 단위**로 헤더 전체 길이를 나타냄.<br>• 최소값: **5** (5 × 4B = 20 bytes / 옵션이 없을 때)<br>• 최대값: **15** (15 × 4B = 60 bytes / 옵션이 최대일 때) |
    | **DSCP (DiffServ Code Point)** | 6 bits | **QoS (Quality of Service)**를 위해 사용. 실시간 음성(VoIP)이나 비디오 스트리밍 트래픽에 우선순위 부여. |
    | **ECN (Explicit Congestion Notification)** | 2 bits | 혼잡 발생 시 패킷을 즉시 폐기하지 않고 양단 단말에 망의 혼잡 상태를 알리는 명시적 혼잡 통지. |
    | **Total Length (전체 길이)** | 16 bits (2B) | **[IP 헤더 + L4 페이로드] 전체 패킷의 실제 바이트 수**.<br>• 단위: 1바이트<br>• 최소값: 20바이트<br>• 최대값: 65,535 bytes |
    | **Identification (식별자)** | 16 bits | 패킷이 분할(조각화)될 때, 원본 패킷을 식별하여 재조립할 수 있게 하는 고유 ID 번호. |
    | **Flags (플래그)** | 3 bits | 단편화 제어용 3개 비트:<br>• Bit 0: 항상 0으로 예약<br>• Bit 1: **DF (Don't Fragment)** -> 1이면 단편화 금지, 폐기됨<br>• Bit 2: **MF (More Fragments)** -> 뒤에 조각이 더 있으면 1 |
    | **Fragment Offset (단편 오프셋)** | 13 bits | 원본 패킷 내에서 해당 조각의 상대적 시작 위치. 순서가 뒤바뀌어도 올바르게 재조립 가능. |
    | **TTL (Time to Live)** | 8 bits (1B) | 라우팅 루프 방지용 홉 카운트. 라우터를 통과할 때마다 1씩 차감되며, 0이 되면 패킷 폐기. |
    | **Protocol (프로토콜 번호)** | 8 bits (1B) | IP 헤더 뒤에 캡슐화된 L4 상위 프로토콜 종류:<br>• 1 = ICMP (Ping)<br>• 6 = TCP<br>• 17 = UDP<br>• 89 = OSPF |
    | **Header Checksum (헤더 체크섬)** | 16 bits | IP 헤더 자체의 비트 오류 검출. 일치하지 않으면 패킷 폐기. |
    | **Source IP Address** | 32 bits (4B) | 송신자(출발지)의 IPv4 주소. |
    | **Destination IP Address** | 32 bits (4B) | 최종 수신자(목적지)의 IPv4 주소. |
    | **Options (옵션)** | 가변 (0 ~ 40 Bytes) | 거의 사용되지 않는 선택 필드. IHL 값이 5를 초과하면 옵션이 포함됨을 의미. |

3. IP 단편화(Fragmentation)와 MTU의 관계
    - MTU(Maximum Transminssion Unit)
        - 2계층 이더넷 링크에서 단일 프레임에 담을 수 있는 최대 페이로드 크기는 일반적으로 1500바이트이다.
    
    - 단편화 동작 :
        - IP 패킷(헤더+페이로드)이 경로 상 인터페이스의 MTU(1500B)보다 큰 경우, 라우터는 이를 MTU 이하 크기의 여러 조각(Fragment)으로 쪼갠다.
        - 쪼개진 단편들은 동일한 Identification 값을 공유하며, Fragment Offset과 MF 플래그를 통해 수신 최종 호스트에서 원래대로 재조립(Reassembly)된다.

    - DF(Don't Fragment) 비트 설정 시의 실패
        - 패킷 크기가 MTU(1500B)를 초과했는데 헤더의 DF 비트가 1로 세팅되어 있으면, 라우터는 패킷을 단편화하지 못하므로 즉시 폐기하고 송신자에게 에러를 돌려준다.



### 주요 어휘

| 필드 명칭 | 크기 (Bits) | 역할 및 세부 동작 메커니즘 |
| :--- | :---: | :--- |
| **Version (버전)** | 4 bits | 사용 중인 IP 프로토콜 버전 명시. IPv4는 항상 **`4`** (`0100`)로 고정. (IPv6는 `6` = `0110`, 실험용 Internet Stream Protocol이 `5`를 썼음) |
| **IHL (Internet Header Length)** | 4 bits | **4바이트(32비트) 단위**로 헤더 전체 길이를 나타냄.<br>• 최소값: **`5`** ($5 \times 4\text{B} = \mathbf{20\text{ bytes}}$ / 옵션이 없을 때)<br>• 최대값: **`15`** ($15 \times 4\text{B} = \mathbf{60\text{ bytes}}$ / 옵션이 최대일 때) |
| **DSCP (DiffServ Code Point)** | 6 bits | **QoS (Quality of Service)**를 위해 사용. 지연에 민감한 실시간 음성(VoIP)이나 비디오 스트리밍 트래픽에 우선순위를 부여함. |
| **ECN (Explicit Congestion Notification)** | 2 bits | 혼잡 발생 시 패킷을 즉시 폐기(Drop)하지 않고 양단 단말에 망의 혼잡 상태를 알리는 명시적 혼잡 통지 (선택적 기능). |
| **Total Length (전체 길이)** | 16 bits (2B) | **[IP 헤더 + L4 페이로드] 전체 패킷의 실제 바이트 수**.<br>• 단위: IHL과 달리 **1바이트 단위**<br>• 최소값: 20바이트 (페이로드 없는 순수 IP 헤더)<br>• 최대값: $2^{16} - 1 = \mathbf{65,535\text{ bytes}}$ |
| **Identification (식별자)** | 16 bits | 패킷이 분할(조각화)될 때, 어떤 원본 패킷에 속한 조각인지 수신 측이 식별하여 재조립할 수 있도록 부여하는 고유 ID 번호. (동일 원본의 모든 단편은 같은 값을 가짐) |
| **Flags (플래그)** | 3 bits | 단편화 제어용 3개 비트:<br>• Bit 0: 항상 `0`으로 예약(Reserved)<br>• Bit 1: **DF (Don't Fragment)** $\rightarrow$ `1`이면 "단편화 금지", MTU보다 커도 쪼개지 못해 패킷 폐기됨<br>• Bit 2: **MF (More Fragments)** $\rightarrow$ 뒤에 더 조각이 남아있으면 `1`, 마지막 조각이거나 단편화되지 않았으면 `0` |
| **Fragment Offset (단편 오프셋)** | 13 bits | 원본 unfragmented 패킷 내에서 해당 조각이 위치하는 상대적 시작 위치. 순서가 뒤바뀌어 수신되어도 올바른 순서로 재조립할 수 있게 해 줌. (첫 조각은 0) |
| **TTL (Time to Live)** | 8 bits (1B) | 라우팅 루프(Infinite Loop) 방지용 홉 카운트(Hop Count). 패킷이 라우터를 하나 통과할 때마다 $1$씩 차감되며, **TTL = 0이 되면 라우터가 패킷을 즉시 폐기(Drop)**함. (Cisco 기본 권장 시작값: 64, 최대 255) |
| **Protocol (프로토콜 번호)** | 8 bits (1B) | IP 헤더 바로 뒤에 캡슐화된 L4 상위 프로토콜 종류를 명시:<br>• `1` = ICMP (Ping)<br>• `6` = TCP<br>• `17` = UDP<br>• `89` = OSPF (라우팅 프로토콜) |
| **Header Checksum (헤더 체크섬)** | 16 bits | IP 헤더 자체의 비트 오류만 검출하는 체크섬. 라우터가 수신 후 계산값과 일치하지 않으면 패킷 폐기. (※ 페이로드 내부 데이터 오류는 L4인 TCP/UDP의 Checksum에서 별도로 검출함) |
| **Source IP Address** | 32 bits (4B) | 송신자(출발지)의 IPv4 주소. |
| **Destination IP Address** | 32 bits (4B) | 최종 수신자(목적지)의 IPv4 주소. |
| **Options (옵션)** | 가변 (0 ~ 320 bits / 0 ~ 40 Bytes) | 거의 사용되지 않는 선택 필드. IHL 값이 5를 초과하면 옵션이 포함되어 있음을 의미함. |
