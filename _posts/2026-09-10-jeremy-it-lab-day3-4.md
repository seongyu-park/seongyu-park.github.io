---
title: "Jeremy's IT Lab Day3~4"
date: 2026-09-11 09:00:00 +0900
categories: [Network, CCNA]
tags: [network, ccna, jeremy-it-lab, CLI, TCP/IP]
toc: true
math: false
comments: true
---

## day3 : How the TCP/IP Model Actully Works

1. 프로토콜과 표준의 필요성
    - 프로토콜 : 네트워크 상에서 장비 간 데이터를 주고받는 규칙의 집합
    - 독접 프로토콜 : 초기 네트워크에서는 IBM, DEC같은 거대 IT 기업들이 자신들의 장비를 더 많이 팔기 위해 독자적인 프로토콜 사용
        - 해당 문제 때문에 소비자는 한 회사의 제품만 구매해야 됬어고 엄청난 비용부담을 초래했다.
    - 개방형 표준 : 제조사에 종속되지 않고 누구나 구현할 수 있는 공통 표준을 제작함으로써 서로 다른 제조사 장비간 원활한 통신 가능해졌다.

2. TCP/IP의 기원과 표준화 기구
    - 역사 : 1969년 미국 국방부 산하 ARPA의 ARPANET에서 출발했다. 
            초기에는 NCP 프로토콜을 썼으나 1974년 빈트 서프(Vint Cerf)와 밥 칸(Bob Kahn)이 개발한 TCP를 거쳐 TCP와 IP로 분리되었고, 1983년 1월 1일 TCP/IP로 완전 전환되었다.
    
    - 주요 표준화 기구 
        - IEEE(Institute of Electrical and Electronics Engineers) : 주로 로컬 영역의 LAN 및 물리/데이터링크 계층 표준을 정의한다. 
            ex: 802.3 : 이더넷, 802.11 : 와이파이
        - IETF(Internet Engineering Task Force) : 인터넷 기반 상위 프로토콜을 정의한다.
            표준 문서를 RFC(Request for Comments)형식으로 자유롭게 공개한다.
            ex: TCP, IP, UDP, HTTP, DNS

3. 5계층 TCP/IP 모델
    각 계층은 하위 서버스를 제공받고, 상위 계층에 서비스를 제공한다. 
    1. Layer 1(Physical Layer,물리 계층)
        - 디지털 데이터를 랜선이나 무선 전파 같은 실제 물리적 신호로 바꾸어 전송한다.
        - UTP 케이블, 광섬유, 안테나 신호
    2. Layer 2(Data Link Layer, 데이터링크 계층)
        - 하나의 로컬 네트워크(LAN) 안에서 장비 간 데이터를 안전하게 전달한다.
        - Ethernet(유선 랜), Wi-Fi(무선 랜), MAC주소, 오류 검출
    3. Layer 3(Network Layer, 네트워크 계층)
        - 네트워크 목적지까지 가는 최적의 경로를 찾아준다. 라우팅, IP주소
        - IP(IPv4, IPv6), 네트워크 상태 점검 ICMP(ping)
    4. Layer 4(Transport Layer)
        - 데이터가 도중에 깨지거나 유실되지 않도록 안전하고 정확하게 이동하도록 관리한다.
        - 포트, TCP, UDP
    5. Layer 5(Application Layer)
        - 사용자 애플리케이션에 네트워크 인터페이스 제공
        - HTTP/HTTPS, DNS

4. 캡슐화(Encapsulation) & 역캡슐화(Decapsulation)
    - 캡슐화(송신) : 상위 계층의 데이터에 각 계층이 필요한 제어정보인 헤더를 덧붙여 하위 계층으로 내려보내는 과정
        (Layer2 에서는 오류 검출영 트레일러(FCS)도 추가됨)
    - 역캡슐화(수신) : 수신 호스트가 물리 신호를 수신한 뒤 , L1부터 상위 계층으로 올라가면서 각 계층의 헤더와 트레일러를 분석, 재거하며 최종 원본 데이터를 추출하는 과정

5. PDU(Protocol Data Unit, 프로토콜 데이터 단위)
    - 네트워크의 각 계층에서 데이터를 처리하고 전달할 때 사용하는 데이터의 기본단위
        - Layer 4 : TCP(Segment, 세그먼트), UDP(Datagram, 데이터그램)
        - Layer 3 : Packet, 패킷
        - Layer 2 : Frame, 프레임
        - Layer 1 : Bits, 비트

6. 계층간 상호작용 
    - 인접 계층간 상호작용(Adjacent-Layer Interaction) : 동일 장비내에서 바로 위/아래 계층끼리 서비스 요청 및 응답을 주고 받는 방식
    - 동일 계층간 상호작용(Same-Layer Interaction) : 송신기기와 수신기기가 동일 계층이 헤더 정보를 통해 논리적으로 대화하는 방식


### 주요 어휘

| 영문 표현 | 뜻 / 문맥 | 쓰임새 |
| :--- | :--- | :--- |
| Protocol Suite | 프로토콜 집합 / 계열 | 네트워크 통신을 완벽하게 지원하는 프로토콜들의 집합 (예: TCP/IP Suite) |
| Vendor-Neutral | 벤더 중립적 (개방형) | 특정 제조사 전용(Proprietary)이 아닌 범용 표준 |
| Encapsulation | 캡슐화 | 송신 호스트가 상위 데이터에 헤더/트레일러를 덧붙이는 작업 |
| Decapsulation | 역캡슐화 | 수신 호스트가 헤더/트레일러를 제거하며 데이터를 꺼내는 작업 |
| PDU (Protocol Data Unit) | 프로토콜 데이터 단위 | 계층별 완성된 전송 단위 (Segment, Packet, Frame) |
| Segment | 세그먼트 | Layer 4(TCP) 헤더가 붙은 데이터 단위 |
| Packet | 패킷 | Layer 3(IP) 헤더가 붙은 데이터 단위 (라우팅 대상) |
| Frame | 프레임 | Layer 2(이더넷 등) 헤더와 트레일러가 붙은 전송 단위 |
| Header / Trailer | 헤더 / 트레일러 | 데이터 앞부분(제어 정보) / 뒷부분(주로 오류 검출 FCS) |
| End-to-End | 종단 간 통신 | 중간 경유 장비와 상관없이 최초 출발지 앱과 최종 목적지 앱 간의 대화 |
| Adjacent-Layer Interaction | 인접 계층 상호작용 | 한 장비 내부에서 위아래 계층 간 서비스를 주고받는 형태 |
| Same-Layer Interaction | 동일 계층 상호작용 | 서로 다른 장비 간 동일 계층끼리 헤더를 통해 통신하는 형태 |


---



## day 4 : Intro to the CLI

1. Cisco IOS 및 CLI vs GUI
    - Cisco IOS (Internetwork Operating System) : 시스코 라우터와 스위치 등의 장비에서 구동되는 전용 네트워크 운영체제
    - CLI (Command-Line Interface) : 텍스트 명령어를 입력하여 장비를 제어하고 구성하는 인터페이스, 실제 시험의 중심 환경
    - GUI (Graphical User Interface) : 마우크 클릭 및 그래픽 환경

2. 콘솔 접속과 통신 기본 설정
    - 네트워크 연결이 되어 있지 않는 새 장비를 최초로 설정할 때는 반드시 콘솔 포트를 통해 직결해야 한다.
    - 콘솔 포트
        - RJ-45 콘솔 포트 : 기존 시스코 장비에서 널리 사용하는 표준 8핀 포트
        - USB mini-B : 최신 시스코 장비에 추가되어 별도의 시리얼 어댑터 없이 USB 케이블로 직접 연결 가능한 포트
    - 롤오버 케이블(Rollover Cable) : 콘솔 포트에 연결하여 초기 설정과 관리를 할 수있게 해주는 특수한 케이블
        - 커넥터 구성 : RJ-45 커넥터와 DB-9 커넥터로 구성되어 있다.
        - 핀파웃 구조 : 양 끝 8가닥 전선의 핀 배열이 완전히 반대로 뒤집혀 결선된다.

    - 최근 노트북이나 PC에 DB-9 구격의 시리얼 포트가 제외되므로 새로운 어댑터가 필요하다.

    - 터미널 에뮬레이터 
        - PC에서 PuTTY 같은 터미널 에뮬레이터 프로그램을 실행하여 연결모드를 Serial로 지정한 뒤, 시스코 장비의 기본 통신 규격에 맞쳐 접속 설정을 구성해야한다. 
        - 직렬 포트 기본 통신 매개변수
            - Speed/Baud Rate(보드레이트) : 9600bps
            - Data Bits : 8 bits
            - Stop Bits : 1 bite
            - Parity : None
            - Flow Control : None

3. 시스코 CLI 모드 계층 구조
    - 시스코 IOS CLI는 권환과 목적에 따라 모드가 나뉘며, 프롬프트 모양으로 현재 모드를 구분한다. 
        
        | 모드 이름 | 프롬프트 표시 | 진입 명령어 | 탈출 명령어 | 주요 역할 및 특징 |
        | :--- | :--- | :--- | :--- | :--- |
        | **User EXEC Mode** (사용자 모드) | `Router>` | 기본 접속 모드 | `exit` | 확인 명령 일부만 실행 가능, 설정 변경 불가 |
        | **Privileged EXEC Mode** (특권 모드 / Enable 모드) | `Router#` | `enable` (줄여서 `en`) | `disable` / `exit` | 장비 재부팅, 설정 저장, 디버깅, 상세 조회(`show`) 수행 |
        | **Global Configuration Mode** (전역 설정 모드) | `Router(config)#` | `configure terminal` (줄여서 `conf t`) | `exit` (이전 모드로) / `end` 또는 `Ctrl+Z` (특권 모드로 직행) | 장비 전체에 영향을 주는 전역 설정(호스트명, 계정 등) 변경 |
        | **Interface Configuration Mode** (인터페이스 설정 모드) | `Router(config-if)#` | `interface <포트번호>` (예: `int g0/0`) | `exit` | 특정 포트/인터페이스의 IP 주소, 속도, 활성화 여부 설정 |
        | **Line Configuration Mode** (라인 설정 모드) | `Router(config-line)#` | `line console 0` 또는 `line vty 0 4` | `exit` | 콘솔 포트나 원격 터미널 접속 보안 및 매개변수 설정 |

4. CLI 편이 기능 및 기본 조작 팁
    - 도움말 기능 (?) : 입력 가능한 전체 명령어 목록이나 특정 명령어 뒤어 올 파라미터 확인
    - 자동 완성 (Tab) : 명령어의 일부 글자만 입력한 후 Tab을 누르면 단어가 자동 완성된다.
    - 축약 명령어 : 중복되지 않는 고유한 글자까지만 치고 바로 Enter만 쳐도 동작한다.
    - do 명령어 : 설정 모드에 머문 상태에서 특권 모드의 명령어를 실행할 때 명령어 앞에 do를 붙인다. 
    - do 명령어 : 이미 적용된 설정을 삭제하거나, 특정 기능을 비활성화하고 기본값으로 되돌릴 때 사용한다.

5. 구성 파일 관리 (Runnig vs Startup Config)
    - runnig-config 
        - 현재 장비가 동작 중인 RAM(비활성 메모리)에 올라와 있는 활성 설정
        - 전원이 꺼지너가 재부팅되면 모두 지워진다.
    - startup-config 
        - 장비 재부팅시 불러올 설정으로 NVRAM(비휘발성 메모리)에 영구 저장되는 설정
    - 설정 저장 명령어
        - copy runnig-config startup-config (축약형 : write, wr)
        - 현재 RAM의 설정을 NVRAM으로 복사하여 영구 보존한다.

6. 기본 장비 보안 설정
    - 호스트명 변경 : (conrig)# hostname <이름>
    - 특권 모드에서 암호 설정(Enable)
        - enable password <암호> : 평문으로 저장되어 설정 파일에서 비밀번호가 그대로 노출된다.(취약)
        - enable secret <암호> : MD5 해시 알고리즘을 통해 자동 암호화되어 저장된다.
    - 비밀번호 일괄 암호화 : (config)# service password-encryption
        - 설정 파일에 평문으로 적힌 모든 패스워드를 가독성이 떨어지도록 변환해주지만, 복호화가 쉬워 취약하다. 
        - Type 7 암호화


### 주요 어휘

| 영문 표현 | 뜻 / 문맥 | 쓰임새 |
| :--- | :--- | :--- |
| CLI (Command-Line Interface) | 명령줄 인터페이스 | 텍스트 기반으로 명령을 입력해 네트워크 장비를 조작하는 기본 인터페이스 |
| IOS (Internetwork Operating System) | 시스코 네트워크 운영체제 | 라우터와 스위치 등 시스코 하드웨어에서 구동되는 코어 OS |
| Console Port | 콘솔 포트 | 장비 최초 구동 및 오프라인 대역 외(Out-of-Band) 관리를 위한 물리 포트 |
| Rollover Cable | 롤오버 케이블 | 한쪽 끝의 핀 배열이 반대쪽과 완전히 반대로 뒤집혀 결선된 시스코 콘솔용 케이블 |
| Baud Rate | 보드 레이트 (전송 속도) | 시리얼 콘솔 연결 시 초당 신호 전송 속도 (시스코 기본값: 9600 bps) |
| User EXEC Mode | 사용자 실행 모드 | 기본 진입 모드로 기호 '>'를 사용하며, 설정 변경 권한이 제한된 모드 |
| Privileged EXEC Mode | 특권 실행 모드 | 기호 '#'를 사용하며, 모든 조회 및 유지보수 작업이 가능한 권한 모드 |
| Global Configuration Mode | 전역 설정 모드 | 프롬프트 '(config)#'로 표시되며, 장비 전반의 설정을 수정하는 최상위 설정 모드 |
| Running-config | 실행 구성 파일 | RAM에 상주하여 장비가 현재 실시간으로 동작 중인 휘발성 설정 데이터 |
| Startup-config | 시작 구성 파일 | NVRAM에 저장되어 장비가 부팅될 때 로드되는 비휘발성 영구 설정 데이터 |
| Enable Secret | 보안 특권 비밀번호 | 해시(암호화)되어 저장되며 일반 enable password보다 우선순위를 갖는 보안 패스워드 |
| Precedence | 우선순위 | 복수의 설정이나 암호가 중복될 때 어떤 것이 우선하여 적용되는지를 나타냄 |
