---
title: "TCP기반 서버-클라이언트 소켓 프로그래밍"
date: 2026-10-08 09:00:00 +0900
categories: [Network, Xpert]
tags: [network, xpert, project]
toc: true
math: true
comments: true
---

# 소켓 프로그래밍
## TCP 기본 개념

1. 연결 지향성 및 신뢰성
    - 연결 지향성 : 데이터를 주고 받기 전 반드시 서버와 클라이언트 간 3-way-handshake(`SYN`->`SYN-ACK`->`ACK`)과정을 거쳐 통신 파이프 라인을 수립한다.
    - 신뢰성 및 순서 보장 : 패킷의 유실을 감지하고, 데이터가 보낸 순서대로 도착하도록 커널 레벨에서 제어한다.
    - 바이트 스트림 : 메시지의 경계가 없어 데이터를 독립된 패킷 단위로 취급하는 UDP와 달리, TCP는 연속된 바이트의 흐름으로 다룬다. 

2. TCP 서버의 소켓 이원화 (`listen`소켓, `accept`소켓)
    - TCP 서버는 하나의 소켓으로 모든 것을 처리하지 않고, 역할에 따라 2가지 종류로 소켓을 나누어 사용한다. 
    
    1. 듣기 전용 소켓(`server_sock`)
        - `listen()`을 호출하여 연결 수신 대기 상태로 전환한다.
        - 직접 데이터를 주고 받지 않고, 접속 요청을 받아들이는 통로 역할만 수행한다.

    2. 1:1 전용 통신 소켓(`client_sock`)
        - `accept()`함수가 대기 큐에서 클라이언트의 접속 요청을 꺼내면서 새롭게 반환하는 소켓 디스크립터이다. 
        - 실제 데이터 송수신은 새로 만들어진 `client_sock`을 통해 1:1로 이루어진다.
        - 기존의 `server_sock`은 다른 클라이언트의 접속을 계속 기다릴 수 있게 된다.
    
    - 이원화의 궁국적인 목적은 동시성과 서비스 가용성의 확보에 존재한다.
        - 만일 단일 소켓으로 진행하는 경우 서버 입장에서 한 클라이언트와 통신 하는 동안 다른 클라이언트가 접속 요청을 보내지 못하고 타임아웃이 발생하게 된다. 
        - 이러한 현상은 TCP의 특성 중 하나인 연결 지향성에서 나타난다. 서버 입장에서 연결된 클라이언트를 구별해야 하기 때문에 1명하고만 통신하면 병목 현상이 발생하게 된다.

## TCP 서버-클라이언트 소켓 작성 - 서버

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
    int server_sock, client_sock;
    struct sockaddr_in server_addr, client_addr;
    socklen_t addr_size;
    char buffer[BUFFER_SIZE];
    int opt = 1;

    // 1. TCP 소켓 생성 (SOCK_STREAM)
    if ((server_sock = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        perror("Socket creation failed");
        exit(EXIT_FAILURE);
    }

    // 포트 즉시 재사용 옵션 설정 (Bind failed 에러 방지)
    setsockopt(server_sock, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));

    // 2. 서버 주소 구조체 설정 및 0 초기화
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;
    server_addr.sin_port = htons(PORT);

    // 3. 소켓과 IP/PORT 바인딩
    if (bind(server_sock, (struct sockaddr *)&server_addr, sizeof(server_addr)) < 0) {
        perror("Bind failed");
        close(server_sock);
        exit(EXIT_FAILURE);
    }

    // 4. 연결 요청 대기 상태 설정 (Backlog Queue 크기: 5)
    if (listen(server_sock, 5) < 0) {
        perror("Listen failed");
        close(server_sock);
        exit(EXIT_FAILURE);
    }

    printf("[TCP Server] Listening on port %d...\n", PORT);

    // 5. 클라이언트 연결 수락 (Accept)
    addr_size = sizeof(client_addr);
    if ((client_sock = accept(server_sock, (struct sockaddr *)&client_addr, &addr_size)) < 0) {
        perror("Accept failed");
        close(server_sock);
        exit(EXIT_FAILURE);
    }

    printf("[Client Connected]: %s:%d\n", 
           inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port));

    // 6. 데이터 송수신 루프 (연결된 client_sock 사용)
    while (1) {
        memset(buffer, 0, BUFFER_SIZE);
        
        // recv() 함수 사용 (TCP는 연결된 상태이므로 주소 구조체 불필요)
        int str_len = recv(client_sock, buffer, BUFFER_SIZE - 1, 0);
        
        // str_len == 0 은 클라이언트가 연결을 정상 종료(close)했음을 의미
        if (str_len <= 0) {
            printf("[Client Disconnected]\n");
            break;
        }

        buffer[str_len] = '\0';
        printf("[Received]: %s", buffer);

        // 받은 메시지 그대로 Echo 전송 (send 사용)
        send(client_sock, buffer, str_len, 0);
    }

    // 7. 소켓 종료 (4-Way Handshake / FIN 패킷 발생)
    close(client_sock);
    close(server_sock);
    return 0;
}
```

### 코드 분석
1. TCP 소켓 생성
    - `SOCK_STREAM` : TCP 프로토콜을 지정한다. 
    - 운영체제 커널에 TCP 통신을 위한 기본 소켓 객체를 생성하고, 이를 가리키는 정수 형태의 듣기 소켓 디스크립터(`server_sock`)를 반환받는다. 

2. 소켓 옵션 재설정
    - `setsockopt(server_sock, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt))`
    - 서버 프로그램이 종료된 직후 재실행될 때 발생하는 `Bind failed: Address already in use`에러를 방지한다.
    - TCP 연결이 종료되면서 OS 커널은 패킷 혼선을 막기 위해 포트를 일정 시간 동안 `TIME_WAIT` 상태로 유지한다.
    - `SO_REUSEADDR` 옵션을 1로 설정하면 `TIME_WAIT`상태인 포트라도 즉시 재바인딩하여 사용할 수 있게 된다.

3. 서버 주소 설정 및 바인딩
    - `INADDR_ANY` : 서버 PC에 존재하는 모든 네트워크 인터페이스로부터 들어오는 패킷을 수신한다.
    - `hton(PORT)` : 호스트 바이트 순서의 포트 번호를 네트워크 바이트 순서로 변환한다.
    - `bind()` : OS 커널에 생성된 `server_sock`과 지정한 IP/PORT 정보를 하나로 결합한다.

4. 연결 대기 상태 전환(`listen`)
    `if (listen(server_sock, 5) < 0){...}`
    - `server_sock`의 역할 변경 : 일반 소켓에서 클라이언트의 접속 요청을 받아들이는 듣기 전용 소켓으로 전환한다.
    - `5`(Backlog Queue 크기) : 3-Way-Handshake를 완료하고 `accept()`를 기다리는 클라이언트들이 줄을 서서 대기할 수 있는 완성된 연결 대기 큐의 최대 크기이다. 

5. 클라이언트 연결 수락(`accept`)
    ```
    addr_size = sizeof(client_addr);
    if ((client_sock = accept(server_sock, (struct sockaddr *)&client_addr, &addr_size)) < 0){...}
    ```
    - 블록킹 동작 : 대기 큐에 접속 용청을 완료한 클라이언트가 들어올 때까지 프로그램 실행을 멈추고 대기한다
    - 소켓 분리 : 클라이언트가 접속하면 커널은 해당 클라이언트와 1:1 통신할 새로운 통신 전용 소켓 디스크립터(client_sock)를 생성하여 반환한다.
        - `server_sock` : 여전히 8080 포트에서 다른 클라이언트 접속을 기다린다
        - `client_sock` : 접속한 클라이언트와 실제 데이터 메시지를 주고 받는 소켓이다.

6. 데이터 송수신 루프(`recv&send`)
    - `recv(client_sock, ...)`: 연결된 client_sock을 통해 클라이언트가 보낸 데이터를 수신 버퍼로 읽어온다.
    - 종료 감지 (`str_len<=0`)
        - 상대방이 close()를 호출하여 연결을 정상 종료하면 TCP 4-Way Handshake가 진행되며, 서버 측 `recv()`는 0을 반환한다.
        - 에러 발생시 -1을 반환한다. 따라서 `str_len<=0`조건으로 루프를 탈출하여 자원을 정리한다.
    - `send(client_sock)` : 수신 받은 문자열 데이터를 클라이언트에게 그대로 되돌려 보내는 에코 동작을 수행한다. 

7. 소켓 자원 정리(close)
    - `close(client_sock)` : 전담 클라이언트와의 연결을 끊기 위해 네트워크 상으로 `FIN` 패킷을 전송하고 4-Way Handshake 종료 절차를 밟는다.
    - `close(server_sock)` : 서버의 듣기 소켓 자원을 OS에 반납하고 8080 포트 점유를 완전히 해제한다. 

전체 흐름 : socket() -> bind() -> listen() ->accept() -> recv()/send() -> close()

---
## TCP 서버-클라이언트 소켓 작성 - 클라이언트
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
    struct sockaddr_in server_addr;
    char buffer[BUFFER_SIZE];

    // 1. TCP 소켓 생성 (SOCK_STREAM)
    if ((client_sock = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        perror("Socket creation failed");
        exit(EXIT_FAILURE);
    }

    // 2. 접속할 서버 주소 설정
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(PORT);
    server_addr.sin_addr.s_addr = inet_addr(SERVER_IP);

    // 3. 서버에 연결 요청 (3-Way Handshake 수행)
    if (connect(client_sock, (struct sockaddr *)&server_addr, sizeof(server_addr)) < 0) {
        perror("Connection Failed");
        close(client_sock);
        exit(EXIT_FAILURE);
    }

    printf("[Connected to TCP Server %s:%d]\n", SERVER_IP, PORT);

    // 4. 데이터 송수신 루프
    while (1) {
        printf("Input message (q to quit): ");
        fgets(buffer, BUFFER_SIZE, stdin);

        if (!strcmp(buffer, "q\n") || !strcmp(buffer, "Q\n")) break;

        // send() 로 서버에 전송
        send(client_sock, buffer, strlen(buffer), 0);

        // recv() 로 서버 Echo 응답 수신
        memset(buffer, 0, BUFFER_SIZE);
        int str_len = recv(client_sock, buffer, BUFFER_SIZE - 1, 0);
        
        if (str_len <= 0) {
            printf("[Server Disconnected]\n");
            break;
        }

        buffer[str_len] = '\0';
        printf("[Server Echo]: %s", buffer);
    }

    // 5. 연결 종료 (FIN 패킷 송신)
    close(client_sock);
    return 0;
}
```

### 코드 분석
1. TCP 소켓 생성
    - 운영체제 커널에 클라이언트 통신용 TCP 소켓 객체를 만들고, 해당 소켓을 제어할 수 있는 정수형 소켓 디스크립터(`client`)를 부여받는다.
    - 클라이언트는 보트 번호를 고정할 필요가 없으므로 `bind()`를 명시하지 않으며, `connect()`호출 시 OS커널이 임의의 포트를 알아서 할당해 준다.

2. 서버 주소 설정
    - 주소 구조체 메모리를 0으로 깨끗하게 초기화 시킨다.
    - 포트 번호를 네트워크 순서인 빅 엔디안으로 변환 시킨다.
    - 문자열 형태의 IP 주소를 32비트 이진 정수 주소로 변환하여 저장한다.

3. 서버에 연결 요청(`connect`)
    - `connect()`함수가 실행되는 그 순간, 클라이언트 OS 커널은 서버로 `SYN`패킷을 발송한다.
    - 이후 서버로부터 `SYN-ACK`를 받고, 클라이언트가 다시 `ACK`를 되돌려보내는 3-Way Handshake 과정이 완벽히 완료되어야 `connect()`함수가 리턴한다.
    - 실패시 에러 ㄹ메시지를 출력하고 소켓 자원을 반납한 뒤 종료한다.

4. 데이터의 송수신 루프
    - `fget(...)`: 사용자로부터 키보드 입력 문자열을 받는다. q, Q 입력시 루프를 탈출한다.
    - `send(client_sock, ...)` : 이미 `connect()`를 통해서 서버와 1:1 통신 파이프라인이 수립되어 있으므로, UDP와 달리 목적지 주소 인자 없이 `send()`만으로 데이터를 전송한다.
    - `recv(client_sock, ...)` : 서버가 응답 패킷을 보내줄 때까지 대기(블록킹)하다가 수신받는다.
    - 서버 종료 감지(`str_len <= 0`) : 서버가 먼저 `close()`를 호출하여 연결을 끊으면 TCP 4-Way Handshacke에 의해 클라이언트 측 recv() 는 0을 반환한다. 이를 통해 서버가 꺼졌음을 감지하고 루프를 탈출한다. 

5. 연결 종료(`close()`)
    - 사용자가 q를 입력하여 루프를 벗어나면 `close(client_sock)`를 호출한다.
    - 서버 측으로 `FIN` 패킷이 발송하여 4-Way Handshake 종료 절차를 수행하고, OS에 소켓 디스크립터 및 메모리 자원을 완전히 반납한다. 

전체 흐름 : socket() -> connect() -> send()/recv() -> close() 
