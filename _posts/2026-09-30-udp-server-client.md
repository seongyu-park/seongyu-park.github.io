---
title: "UDP기반 서버-클라이언트 소켓 프로그래밍"
date: 2026-09-30 09:00:00 +0900
categories: [Network, Xpert]
tags: [network, xpert, project]
toc: true
math: true
comments: true
---

# 소켓 프로그래밍
## 기본 개념

1. 소켓이란?
    - 정의 : 네트워크 상 서로 다른 두 프로그램이 데이터를 주고받을 수 있도록 운영체제가 제공하는 양방향 통신 인터페이스이다. 
    - 구성 요소 : IP주소(L3) + 포트 번호(L4)

2. 소켓의 종류
    1. Stream Socket - TCP 기반
        - 연결 지향성, 신뢰성 있는 데이터 전송, 순서 보장
        - 데이터를 보내기 전 상대방과 연결 상태를 먼저 확인하는 3-way Handshake를 수행한다.
        - 주로 웹(HTTP) + 파일 전송(FTP), 이메일, 채팅 등 데이터가 손실되면 안되는 프로그램이 주로 사용한다.
    2. Datagram Socket - UDP기반
        - 비연결형, 빠른 전송 속도, 신뢰성 미보장
        - 연결 확인 과정 없이 일단 데이터를 상대방에게 전송하는 방식이다.
        - 주로 실시간 스트리밍, 온라인 게임, DNS 등 속도가 중요한 프로그램 등이 사용한다.
    3. RAW 소켓
        - 가공 되지 않는 소켓
        - 특정 프로토콜 용의 전송 계층 포맷팅 없이 인터넷 프로토콜 패킷을 직접적으로 주고 받게 해주는 소켓
        - 헤더 정보들에 대해 프로그래머가 직접 제어 가능하다.

## UDP 서버-클라이언트 소켓 작성 - 서버
UDP 서버 코드
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>

#define PORT 8080
#define BUFFER_SIZE 1024

int main() {
    int server_sock;
    char buffer[BUFFER_SIZE];
    struct sockaddr_in server_addr, client_addr;
    socklen_t addr_size;

    // 1. UDP 소켓 생성 (SOCK_DGRAM)
    if ((server_sock = socket(AF_INET, SOCK_DGRAM, 0)) < 0) {
        perror("Socket creation failed");
        exit(EXIT_FAILURE);
    }

    memset(&server_addr, 0, sizeof(server_addr));
    memset(&client_addr, 0, sizeof(client_addr));

    // 2. 서버 주소 설정
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY; // 모든 IP로부터 수신
    server_addr.sin_port = htons(PORT);

    // 3. 소켓과 주소 바인딩
    if (bind(server_sock, (const struct sockaddr *)&server_addr, sizeof(server_addr)) < 0) {
        perror("Bind failed");
        close(server_sock);
        exit(EXIT_FAILURE);
    }

    printf("[UDP Server] Waiting for messages on port %d...\n", PORT);

    while (1) {
        addr_size = sizeof(client_addr);
        
        // 4. 클라이언트로부터 데이터 수신 (수신자 주소 정보가 client_addr에 저장됨)
        int str_len = recvfrom(server_sock, buffer, BUFFER_SIZE - 1, 0, (struct sockaddr *)&client_addr, &addr_size);
        if (str_len < 0) continue;

        buffer[str_len] = '\0';
        printf("[Received from %s:%d]: %s\n", inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port), buffer);

        // 5. 받은 메시지를 그대로 클라이언트에게 다시 전송 (Echo)
        sendto(server_sock, buffer, str_len, 0, (struct sockaddr *)&client_addr, addr_size);
    }

    close(server_sock);
    return 0;
}
```

### 코드 분석
1. 헤더 분석
    - `<unistd.h>` : 리눅스/유닉스 운영체제 시스템 호출 API를 제공하는 헤더 파일
    - `<sys/socket.h>` : 소켓을 생성하고 데이터를 주고받는 리눅스 소켓 통신 핵심 API
        - 소켓 관련 함수 : socket(), bind(), listen(), accept(), recvfrom(), sendto() 등
        - 소켓 관련 상수 : AF_INET (IPv4), SOCK_DGRAM (UDP), SOCK_STREAM (TCP) 등
    - `<arpa/inet.h>` : ip주소 및 포트 번호의 엔디안 변환과 문자열-숫자 주소 변환 함수 제공
        - 주소 구체화 : struct sockaddr_in, struct in_addr
        - 바이트 순서 변환 : htons(), ntos()
        - IP 주소 변환 : inet_addr(), inet_ntoa(), inet_pton()

2.  포트넘버, 버퍼사이즈
    - `#define PORT 8080`
    - `#define BUFFER_SIZE 1024`
    - 포트 번호와 버퍼 사이즈를 코드 상단에 적어두는 이유는 유지 보수를 위함이다. 주로 해당 숫자는 코드 중간 중간 숫자의 형태로 작성되는데 이는 의미를 알기 어려운 매직넘버가 될 수 있기 때문이다. 또한 다양한 함수에 주로 사용되는 숫자임으로 일괄적으로 수치를 변경하기도 간편하다. 
    

3. UDP 소켓 생성(`socket()`)
    ```
    if ((server_sock = socket(AF_INET, SOCK_DGRAM, 0)) < 0) {
    perror("Socket creation failed");
    exit(EXIT_FAILURE);
    }
    ```
    - `socket(AF_INET, SOCK_DGRAM, 0)`
        - `AF_INET` : ipv4 형식의 ip 주소를 사용하는 통신을 수행
        - `SOCK_DGRAM` : UDP 프로토콜 사용
        - `0` : 소켓에서 사용할 구체적 프로토콜 지정으로 0으로 입력시 1,2번 인자의 조합에 맞는 기본 프로토콜을 운영체제가 자동으로 선택해준다.
    - `server_sock = socket(...);`
        - 리눅스/유닉스 시스템에서 모든 입출력 장치를 파일로 취급하여 관리한다. 
        - `socket()`함수를 통하여 리눅스 커널이 메모리에 소켓 객체를 하나 만들고 다른 함수가 호출할때 `server_sock` 인자를 넘겨 줌으로써 커널에 지시할 수 있다. 
    -  `if (... < 0)` : 예외 처리 조건 문
        - 함수는 시스템 자원이 부족하거나 권한이 없어서 소켓 생성에 실패시 -1을 반환한다. 
        - `perror` : 사람이 읽기 쉬운 표준 오류 메시지로 출력하는 함수
        - 소켓 생성 실패시 네트워크 통신 진행이 불가능하므로, 프로그램을 안전하게 즉시 종료한다. 

4. 쓰레기 값 제거
    ```
    memset(&server_addr, 0, sizeof(server_addr));
    memset(&client_addr, 0, sizeof(client_addr));
    ```
    - C언어에서 지역 변수로 구조체(`struct sockaddr_in server_addr;`)를 선언하면 메모리 공간에 이전에 사용하다 만 쓰레기 값이 그대로 남아있게 된다.
    - `memset(초기화할_메모리_주소, 채울_값, 초기화할_바이트_크기);`
    - 쓰레기 값이 남아 있는 상태에서 bind(), sendto() 함수 사용시 운영체제가 주소 정보를 잘못 해석하여 소켓 통신 에러가 발생할 수 있다.

5. 서버 주소 구조체 설정 및 바인딩
    ```
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY; // 가상머신의 모든 NIC/IP 수신 허용
    server_addr.sin_port = htons(PORT);       // 호스트 바이트 순서 -> 네트워크 바이트 순서 변환

    if (bind(server_sock, (const struct sockaddr *)&server_addr, sizeof(server_addr)) < 0) {
        perror("Bind failed");
        close(server_sock);
        exit(EXIT_FAILURE);
    }
    ```
    - 주소 구조체 또한 ipv4라고 사용할 것을 명시 된 후 구조체안 실제 32비트 ip주소가 들어가는 자리에(`sin_addr.s_addr`) `INADDR_ANY`를 설정한다.
    `INADDR_ANY` : 컴퓨터에 존재하는 모든 네트워크 카드(ip주소)로 부터 들어오는 패킷을 다 받아들이겠다.
    - `server_addr.sin_port = htons(PORT);` : 해당 포트의 바이트오더링을 네트워크 표준인 빅엔디안으로 변경
    - bind() : 소켓번호(server_sock)와 ip/port 정보가 담긴 구조체(server_addr)를 운영체제 커널에 묶어서 등록하는 역할을 한다. 
        - `struct sockaddr *` : bind 함수의 2번째 인자로 범용 주소 구조체 포인터 타입을 요구한다.
    - 예외 처리 : 해당 포트를 다른 프로그램이 이미 사용중이면 -1을 반환한다.

6. 메시지 수신 및 데이터 처리
    - `client_addr`구조체의 메모리 크기를 구해 addr_size 변수에 저장한다. 
    - `int str_len = recvfrom(server_sock, buffer, BUFFER_SIZE - 1, 0, (struct sockaddr *)&client_addr, &addr_size);`
        - recvfrom()은 udp 통신의 핵심적인 데이터 수신 함수이다. 
        - server_sock: 수신 대기할 서버의 UDP 소켓 번호이다.
        - buffer: 수신한 데이터 패킷을 저장할 메모리 공간(배열)이다.
        - BUFFER_SIZE - 1: 버퍼 오버플로우를 방지하기 위해 최대 수신 크기를 제한으로 마지막 1바이트는 문자열 끝의 나타내는 NELL 문자를 위해 남겨둔다.
        - 0 : 수신 옵션 플래그ㄹ, 기본 동작을 위해 0으로 설정한다.
        - (struct sockaddr *)&client_addr : 가장 핵심적인 부분으로, TCP와 달리 UDP는 연결이 없기 때문에 패킷을 보낸 클라이언트의 IP와 포트번호 정보가 해당 client_addr 구조체에 자동으로 기록된다. 
        - &addr_size : 구조체의 크기가 담긴 변수의 주솟값이다.
        - 리턴값(str_len) : 실제 수신된 데이터의 바이트 수를 반환한다. 
        - 소켓 버퍼에 데이터가 없으면, 데이터를 기다리면서 계속 블록킹 상태가 된다.

7. 에코 전송 및 정보 출력
    - 수신한 데이터 배열 바로 다음 자리 문자열의 끝을 알려주는 널 문자 삽입
    - `printf("[Received from %s:%d]: %s\n", inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port), buffer);`
        - `inet_ntoa(client_addr.sin_addr)` : recvfrom을 통해 기록된 클라이언트의 32비트 이진 ip 비트를 사람이 읽을 수 있는 문자열의 형태로 변경해준다.
        - `ntohs(client_addr.sin_port)`: 네트워크 엔디안 방식으로 정렬되어 들어온 포트번호를 컴퓨터 가독 형태의 10진수 숫자로 바꾸어준다.
        - `buffer` : 완성된 수신 메시지 문자열이다. 

8. 패킷 소멸
    - UDP 환경에서는 비연결형 프로토콜이기에 연결을 종료할 때 아무런 패킷을 전송하지 않는다.
    - 소켓 디스크립터를 운영체제에 반납하고 연관된 자원도 반납한다. 
    (소켓 디스크립터 운영체제가 현재 생성되어 있는 소켓을 식별하고 관리하기 위해 부여된 정수 값 번호표)

---
## UDP 서버-클라이언트 소켓 작성 - 클라이언트
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>

#define SERVER_IP "127.0.0.1"
#define PORT 8080
#define BUFFER_SIZE 1024

int main() {
    int client_sock;
    char buffer[BUFFER_SIZE];
    struct sockaddr_in server_addr;
    socklen_t addr_size;

    // 1. UDP 소켓 생성 (SOCK_DGRAM)
    if ((client_sock = socket(AF_INET, SOCK_DGRAM, 0)) < 0) {
        perror("Socket creation failed");
        exit(EXIT_FAILURE);
    }

    memset(&server_addr, 0, sizeof(server_addr));

    // 2. 목적지(서버) 주소 설정
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(PORT);
    server_addr.sin_addr.s_addr = inet_addr(SERVER_IP);

    while (1) {
        printf("Input message (q to quit): ");
        fgets(buffer, BUFFER_SIZE, stdin);

        // 'q' 입력 시 종료
        if (!strcmp(buffer, "q\n") || !strcmp(buffer, "Q\n")) break;

        // 3. 서버로 메시지 전송
        sendto(client_sock, buffer, strlen(buffer), 0, (const struct sockaddr *)&server_addr, sizeof(server_addr));

        addr_size = sizeof(server_addr);
        
        // 4. 서버 응답 수신
        int str_len = recvfrom(client_sock, buffer, BUFFER_SIZE - 1, 0, (struct sockaddr *)&server_addr, &addr_size);
        buffer[str_len] = '\0';

        printf("[Server Echo]: %s", buffer);
    }

    close(client_sock);
    return 0;
}
```

### 코드 분석
1. 헤더 파일 및 서버 정보 설정
    - `SERVER_IP "127.0.0.1"` : 접속할 서버의 ip로 `127.0.0.1`은 루프백 ip로 자기 자신울 가리킨다.
    - `PORT 8080` : 메시지를 전송할 서버의 포트로 서버 코드에서 바잉딩한 포트 번호와 동일해야 통신이 가능하다.

2. UDP 소켓 생성
    - 서버와 동일 하게 `AF_INET`(IPv4) 및 `SOCK_DGRAM`(UDP) 방식을 지정하여 클라이언트 측 UDP 소켓을 만든다.
    - 성공 시 반환되는 정수 형태의 소켓 디스크립터(파일 디스크립터)번호가 `client_sock` 변수에 저장된다.
    - 서버의 경우 누구나 찾아올 수 있는 포트를 고정(`bind`)해야 하지만, 클라이언트는 자신이 패킷을 처음 보낼 때 운영체제가 남는 임의의 포트를 자동으로 할당해주기에 별도의 bind 과정이 필요 없다.

3. 서버(목적지) 주소 구조체 설정
    - `memset` : 구조체의 메모리를 0으로 초기화하여 쓰레기 값을 지원다.
    - `htons(POST)` : 포트 번호 `8080`을 네트워크 바이트 순서인 빅엔디안으로 변환하여 설정한다.
    - `inet_addr(SERVER_IP)` : 점으로 구분된 IP 문자열을 네트워크 바이트 순서인 32비트 이진 정수 값으로 변환하여 주소 구조체에 할당한다.

4. 메시지 입력 및 전송 루프
    - `fgets(buffer, BUFFER_SIZE, stdin)` : 키보드의 표준 입력인 stdin으로 문자열을 입력 받아 버퍼에 저장한다. 
    - `strcmp()`: 사용자가 q, Q를 입력시 `break`를 실행하며 무한 루프를 탈출하고 클라이언트를 종료한다.
    - `sendto(...)`
        - `client_sock` : 데이터를 보낼 클라이언트의 소켓 디스크립터이다.
        - `buffer` : 입력받은 텍스트 메시지가 저장된 메모리이다.
        - `strlen(buffer)` : 실제 전송할 데이터의 바이트 길이
        - `const struct sockaddr *)&server_addr` : 목적지의 ip와 port 정보가 채워진 구조체 주소값이다.

5. 서버 응답 수신 및 출력
    - `recvform()`: 서버가 다시 반호나해 준 응답 패킷이 도착할 때까지 대기하다가 데이터를 다시 받아온다.
    - `buffer[str_len] = '\0'` : 수신받은 바이트 끝 널 문자를 삽입하여 안전한 C 문자열 형태로 바꾼다.

6. 클라이언트 종료 및 자원 정리
    - q 입력하여 루프를 벗어나면 `close(client_sock)`를 호출하여 클라이언트 소켓 자원을 os에 반납하고 프로그램을 정상 종료한다. 

## 결과
<img width="2580" height="320" alt="Image" src="https://github.com/user-attachments/assets/b0498563-669b-452a-a4cf-47a1f317b31f" />
실제 구축한 UDP 서버-클라이언트 클라이언트의 터미널 출력 결과이다. 
서버를 먼저 가동 시키고 클라이언트가 보낸 메시지를 서버가 잘 수신한다는 것을 확인 할 수 있다.

<img width="2246" height="850" alt="Image" src="https://github.com/user-attachments/assets/d7ddc598-8c1b-4ce7-9953-eeba0eb803be" />
해당 사진은 루프백 주소를 활용한 udp 방식의 패킷을 분석한 것이다.
데이터 그램(udp) 프로토콜을 이용하여 8080번 포트에 해당 메시지를 수신받은 것을 확인할 수 있다. 
작성한 코드는 테이터 암호화 로직을 않았기 때문에 평문에 그대로 내가 전송한 데이터를 확인 할 수 있다. 

### 느낀점
이번 실습에서 비연결현 프로토콜인 UDP 방식에서의 클라이언트-서버 소켓 프로그래밍을 해보았다. 소켓 프로그래밍의 소켓API의 수명 주기를 이해하고 시스템에서 자원과 메모리를 어떻식으로 관리하는가 학습하였다. 
해당 코드는 간단한 서버-클라이언트에 대한 코드로 암호화 로직을 사용하지 않아 평문이 그대로 패킷에 노출되어 암호화의 중요성에 대해 와이어 샤크를 이용한 분석으로 확인하였다.
