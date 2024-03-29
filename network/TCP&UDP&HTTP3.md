# TCP와 UDP (HTTP/3)
- 네트워크 계층 중 전송 계층(Transport Layer)에서는 송신자와 수신자를 연결하는 통신 서비스를 제공하는 계층이다.
    - 데이터 전달을 담당
- 전송 계층에서 사용하는 프로토콜에는 TCP와 UDP 등이 존재한다.

## TCP (Transmission Control Protocol)

- **연결형 서비스를 지원하는 전송계층 프로토콜**
- 인터넷 환경(http)에서 기본으로 사용
- 초기화(3way-hand-shaking)과 종료(4way-hand-shaking) 필요
- **멀티캐스팅이나 브로드캐스팅을 지원하지 않는다.**
- TCP 특징
    - 흐름 제어(송수신 측의 데이터 처리 속도차이 해결)
    - 혼잡 제어(송신측의 전달과 네트워크 데이터 처리 속도 해결)
    - 오류 제어(오류 검출과 재전송)
    - UDP 보다 속도가 느리지만 높은 신뢰성 보장
    - 전이중, 점대점 방식
        - 전이중 : 전송이 양방향으로 동시에 일어날 수 있다.
        - 점대점 : **각 연결이 정확히 2개의 종단점을 가지고 있다.**
- TCP 동작 과정
    - 소켓 생성 → 3 way handshake → 데이터 송수신 → 4 way handshake

### 3-way handshake

- TCP 통신을 위해 네트워크 연결 설정을 하는 과정
- 양쪽 모두 데이터 전송 준비가 끝났다는 것을 보장

**[STEP 1]**

A클라이언트는 B서버에 접속을 요청하는 SYN 패킷을 보낸다. 이때 A클라이언트는 SYN 을 보내고 SYN/ACK 응답을 기다리는SYN_SENT 상태가 되는 것이다.

**[STEP 2]**

B서버는 SYN요청을 받고 A클라이언트에게 요청을 수락한다는 ACK 와 SYN flag 가 설정된 패킷을 발송하고 A가 다시 ACK으로 응답하기를 기다린다. 이때 B서버는 SYN_RECEIVED 상태가 된다.

**[STEP 3]**

A클라이언트는 B서버에게 ACK을 보내고 이후로부터는 연결이 이루어지고 데이터가 오가게 되는것이다. 이때의 B서버 상태가 ESTABLISHED 이다.

- 위와 같은 방식으로 통신하는것이 신뢰성 있는 연결을 맺어 준다는 TCP의 3 Way handshake 방식이다.

### 4-way handshake

- TCP 연결을 해제하는 과정
1. 클라이언트가 서버로 연결을 종료하겠다는 FIN 패킷 전송
2. 서버는 클라이언트에게 우선적으로 ACK 패킷 전송
3. 서버는 자신의 통신의 끝날 때까지 기다리고 끝나면 클라이언트에게 FIN 패킷 전송
4. 클라이언트는 확인했다는 의미로 ACK 패킷을 서버에게 전송
5. 서버가 보내는 FIN 보다 서버가 보내는 데이터가 늦게 보내질 경우를 대비해 클라이언트는 일정 시간 동안 소켓을 닫지 않고 잉여 패킷을 기다림(time wait)
6. 이후에 연결 종료
- Client가 데이터 전송을 마쳤다고 하더라도 Server는 아직 보낼 데이터가 남아있을 수 있기 때문에 일단 ACK만 먼저 보내고, 데이터를 모두 전송한 후에 자신도 FIN 메시지를 보내는 것

## UDP (User Datagram Protocol)

- **비연결성 서비스를 지원하는 전송 프로토콜**
- 전송을 위한 논리적인 경로가 없다.
- 데이터그램 방색을 사용
    - 패킷 간 순서가 존재하지 않는 독립적인 패킷을 사용
    - 목적지만 정해져 있다면 중간 경로 신경쓰지 않기에 종단 간 연결 설정을 하지 않음 (handshake 과정이 필요 없음)
- UDP 특징
    - 전송 순서를 보장하지 않음
    - 수신 여부를 확인하지 않음
    - UDP 헤더의 CheckSum 필드를 통해 최소한의 오류만 검출
    - 신뢰성이 낮지만 TCP보다 속도가 빠르다 (스트리밍 서비스에 이용)
    - 커스터마이징이 용이
        - TCP는 신뢰성 확보를 위한 여러 과정들이 필수이기 때문에 커스터마이징하기 어렵다.

### HTTP3는 UDP를 사용

- HTTP/3는 기존 HTTP/1과 2와는 다르게 UDP 기반 프로토콜 QUIC를 사용
    - QUIC - Quick UDP Internet Connection
- TCP에서 발생하는 레이턴시를 극복하기 위함
    - 3 way, 4 way handshake와 신뢰성 보장으로 인한 과정 때문에 TCP는 느리다.
    - TCP의 이런 동작들은 필수 과정이기에 바꿀 수 없다.
    - HTTP/2에서는 handshake를 최소화하는 방법을 사용
    - handshake 과정을 없애고 신뢰성을 확보하는 방법이 필요
- UDP를 사용하여 TCP에선 할 수 없었던 레이턴시를 제거하고 TCP와 비슷한 수준으로 신뢰성을 확보하는 것이 가능하다.
    - UDP는 데이터 전송을 제외한 기능이 정의되지 않은 프로토콜이기에 개발자가 구현하기 나름으로 TCP와 비슷한 수준의 기능을 할 수도 있다.
- HTTP3에서 나아진 점
    - 연결 설정 시 레이턴시 감소
        - 연결 설정 소요 시간이 반 정도밖에 안 됨 (3RTT → 1RTT, Round Trip Time)
        - 첫 번째 handshake 시 연결 설정에 필요한 정보와 함께 데이터도 전송
        - 한 번 연결에 성공하면 서버는 그 설정을 캐싱하여 그 다음에는 0RTT만에 통신이 가능
    - 패킷 손실 감지에 걸리는 시간 단축
        - 기존 TCP에선 수신측에서 패킷 순시에 대한 ack를 보내지 않으면 송신측이 기다렸다가 패킷을 재전송한다.
        - 자세한 내용은 [QUIC Loss Detection and Congestion Control](https://datatracker.ietf.org/doc/draft-ietf-quic-recovery/?include_text=1)의 `3.1 Relevant Differences Between QUIC and TCP`챕터 참고
    - 멀티 플렉싱 지원
        - 단일 연결에서 여러 개의 데이터 흐름(스트림)을 섞이지 않게 보내는 방식
        - 기존 HTTP/1의 경우에는 하나의 스트림만 사용했기에 손실 시 전체가 막힌다. (HTTP/2에서 멀티플렉싱 지원)
        - 특정 패킷이 손실되어도 다른 스트림은 멀쩡하게 굴릴 수 있다.
    - 클라이언트 IP가 바뀌어도 연결 유지
        - Connection ID를 사용하여 서버와 연결을 생성하기에 IP와 무관
        - IP가 바뀌어서 발생하는 handshake 과정 생략 가능

---

참고

https://mangkyu.tistory.com/15

https://youtu.be/ad4AO1shXsY

https://evan-moon.github.io/2019/10/08/what-is-http3/
