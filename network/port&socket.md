# 포트와 소켓

## 포트 (Port)

- 포트는 전송 계층의 일부이며 네트워크 통신을 위해 사용된다.
- 네트워크에서 해당 프로세스를 고유하게 식별하기 위한 프로세스에 할당된 논리적 식별자
- 패킷을 주고 받는 두 네트워크 장치는 포트 번호를 포함하여 통신을 주고 받는다.
    - 하나의 IP 주소 내에 개별적으로 부여된 통신 프로세스

### 일반적인 포트 번호

- 0 ~ 1023 - 잘 알려진 포트
    - 22: ssh
    - 80: HTTP
    - 443: HTTPS
    - 53: DNS
- 1024 ~ 49151 - 등록된 포트
- 49152 ~ 65535 - 동적 포트

## 소켓 (Socket)

- 소켓은 네트워크 상에서 돌아가는 두 개의 프로그램 간 양방향 통신의 하나의 엔트 포인트
    - TCP 레이어에서 데이터가 전달되야하는 애플리케이션을 식별할 수 있게 한다.
- 소켓을 열기 위해선 호스트에 할당된 IP, 포트 번호, 프로토콜이 필요
    - IP와 port, 프로토콜의 조합으로 인터넷 상에서 유니크하게 식별될 수 있고 이 조합을 Socket이라고 한다.
    - 하나라도 다르면 다른 소켓
- Server-Client 구조
    - TCP/UDP 위에서 동작
    - 보내는 쪽과 받는 쪽 모두 소켓을 열어야 한다.
    - 처음 데이터를 보내는 쪽이 client socket, 받는 쪽이 server socket이 된다.
- 프로그래밍 언어나 운영체제에 종속적
    - socket은 표준이 아닌 네트워크 프로그래밍 인터페이스다.
    - 때문에 OS마다, 프로그래밍 언어마다 구현한 라이브러리가 다 다르다.

### Socket Programming

- 개발자가 개발하는 애플리케이션은 시스템 기능을(OS, 네트워크 통신) 함부로 쓸 수 없다.
- 대신 애플리케이션이 네트워크 기능을 사용할 수 있도록 프로그래밍 인터페이스를 제공
- 개발자는 Socket Programming을 통해 네트워크 상의 다른 프로세스와 데이터를 주고 받을 수 있도록 구현한다.

**서버 소켓**

1. 소켓 생성
2. 바인딩 (ip, port번호 설정)
3. `listen()`으로 클라이언트 요청에 대기열을 만들어 몇개의 클라이언트를 대기시킬지 결정
4. `accept()`로 `connect()`를 요청한 클라이언트와 연결
5. 데이터 송수신
6. 소켓 닫기 (`close()`)

**클라이언트 소켓**

1. 소켓 생성
2. `connect()`로 서버에 설정된 ip, port로 연결 시도
3. `accept()`로 클라이언트의 socket descriptor 반환
4. 데이터 송수신
5. 소켓 닫기 (`close()`)

## 프로토콜 표준과는 다른 실제 소켓의 식별

- 프로토콜 표준에서 socket은 <프로토콜, IP, Port>로 유일하게 식별한다.
- UDP에선 맞는 얘기지만 TCP에서 실제 구현은 조금 다르다.

### TCP socket 동작 방식

1. 클라이언트 A에서 서버 소켓으로 연결 요청
2. A와 서버가 3-way handshake를 통해 연결 설정
    1. 서버에서 또 다른 소켓을 생성하여 A와 연결을 맺음
3. 클라이언트 B에서도 서버 소켓으로 연결 요청
4. B와 서버가 3-way handshake를 통해 연결 설정
    1. 서버에서 또 다른 소켓을 생성하여 B와 연결을 맺음

- 위 상황에서 A와 연결된 소켓, B와 연결된 소켓, 연결을 기다리는 소켓 총 3개의 소켓이 존재하지만 모두 프로토콜, IP, Port가 동일하다.
- 서버의 여러 소켓을 식별하는 방법
    - 새로운 커넥션 요청은 listening socket에 전달
    - 이미 커넥션이 성립된 소켓은 <source IP, source Port, 서버 IP, 서버 Port>로 식별한다.
---
https://www.baeldung.com/cs/port-vs-socket

https://on1ystar.github.io/socket%20programming/2021/03/16/socket-1/

https://www.ibm.com/docs/en/i/7.2?topic=programming-how-sockets-work

https://youtu.be/X73Jl2nsqiE

https://youtu.be/WwseO8l8rZc