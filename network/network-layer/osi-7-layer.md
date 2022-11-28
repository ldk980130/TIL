# OSI 7 계층
**Open Systems Interconnection**

## 개요

컴퓨터 통신망이 확산되면서 다양한 통신망 혼재 상황에서의 기능별 분류가 필요했다.

OSI 7 계층은 네트워크 프로토콜이 통신하는 구조를 7개의 계층으로 분리하여 각 계층 간 상호 작동하는 방식을 정해 놓은 것이다. OSI 계층 모델을 통해 네트워크에서 트래픽의 흐름을 꿰뚫어 볼 수 있으며, 각 계층은 독립 되어 있다. 7단계 중 이상이 생기면 다른 단계의 장비 및 소프트웨어를 건들지 않고 문제가 되는 계층만 고칠 수 있다.

계층 모델 덕분에 분산된 이기종 시스템 간 네트워크 상호 호환을 위한 표준 아키텍처를 정의할 수 있었다. 또한 통신에 관련된 목적을 달성하기 위해 계층 별로 분할 및 분업이 가능해 졌으며 기존 TCP 4계층 모델이 계층 별로 역할이 불명확하여 발생했던 많은 문제들에 대한 해답을 제시한다.

## Physical Layer

- 두 대의 컴퓨터가 통신하려면 0과 1의 나열을 주고 받을 수 있으면 된다. (Bit 단위)
- 물리 계층이란
    - 0과 1의 나열을 아날로그 신호로 바꾸어 전선으로 보내고 (encoding)
    - 아날로그 신호가 오면 0과 1의 나열로 해석하여 (decoding)
    - 물리적으로 연결된 두 대 컴퓨터가 0과 1의 나열을 주고 받을 수 있게 해주는 모듈이다.
- 주소 개념이 없고 연결된 노드 간의 신호를 주고 받는다.
- 물리 계층 모듈은 하드웨어적으로 구현되어 있다.

> 여러 컴퓨터들이 통신하려면 전선으로 연결 되어 있어야 한다. 여러 컴퓨터들이 서로에게 전선을 꼽는 것은 불가능하다. 때문에 컴퓨터들은 라우터들에게 연결 되어 있고, 전 세계 컴퓨터들이 계층 구조로 연결 되어 있는 것을 인터넷이라고 한다.
>

## Data-Link Layer

- 같은 네트워크에 있는 여러 대의 컴퓨터들이 데이터를 주고 받기 위해서 필요한 모듈
- 인접한 노드 간의 신뢰성 있는 데이터 전송을 제어한다. (Frame 단위)
- MAC(Media Access Control) 주소를 통해 목적지를 찾아 간다.
- 신뢰성 있는 전송을 위한 다음의 역할을 수행
    - 흐름 제어(Flow Control)
    - 오류 제어(Error Control)
    - 회선 제어(Line Control)
- Framing 등을 담당

## Network Layer

- 상대의 IP 주소를 알고 있어야 데이터를 전송할 수 있다. (Packet 단위)
- 네트워크 계층은
    - 수많은 네트워크들의 연결로 이루어지는 inter-network 속에서
    - 어딘가에 있는 목적지 컴퓨터로 데이터를 전송하기 위해
    - IP 주소를 이용해서 길을 찾고 (routing 알고리즘)
    - 자신 다음의 라우터에게 데이터를 넘겨주는 일을 한다. (forwarding)
- 운영 체제 커널에 소프트웨어적으로 구현 되어 있다.

## Transport Layer

- 한 컴퓨터에는 여러 프로세스들이 돌아가고 있기 때문에 전달하는 데이터가 어떤 프로세스에게 전달 되어야 하는지 알아야 한다.
- 트랜스포트 계층은
    - Port 번호를 사용하여
    - 도착지 컴퓨터의 최종 도착지인 프로세스에
    - 도달하도록 하는 모듈이다.
- 종단 간 신뢰성 있는 데이터 전송을 담당한다.
    - 분할과 재조합
    - 연결 제어
    - 흐름 제어
    - 오류 제어
    - 혼잡 제어
- 운영 체제 커널에 소프트웨어적으로 구현 되어 있다.

> **OSI 모델 vs TCP/IP 모델**
> 
> 현대의 인터넷은 TCP/IP 모델을 따른다. 이유는 OSI 모델이 시장 점유 싸움에서 졌기 때문이다. TCP/IP는 4계층 모델이며 Application, Transport, Internet, Network 계층으로 이루어져 있다. 업데이트 이후에는 Application, Transport, Network, Data Link, Physical 계층으로 나누어져 있다.
>

> **TCP/IP 소켓 프로그래밍**
> 
> 운영 체제의 Transport 계층에서 제공하는 API를 활용해서 통신 가능한 프로그램을 만드는 것을 TCP/IP 소켓 프로그래밍, 또는 네트워크 프로그래밍이라고 한다. 소켓 프로그래밍 만으로도 클라이언트, 서버 프로그램을 만들 수 있다.
누구든 자신만의 Application 계층 프로토콜을 이를 통해 만들 수 있다.
>

## Session Layer

- Session 계층은 두 장치 간의 Session이라 불리는 communication channel를 만든다.
- Session 계층은 session을 열고 데이터가 전송되고 커뮤니케이션이 끝나면 session을 닫는 책임을 가지고 있다.

## Presentation Layer

- 표현 계층은 애플리케이션 레이어에 보낼 데이터를 준비한다.
- 두 장치가 주고 받는 데이터의 인코딩, 암호화, 압축 등을 담당하여 종단 간 데이터가 올바르게 수신되도록 하는 방법을 정의 한다.
- 애플리케이션 계층에서 전송된 모든 데이터를 가져와 session 계층을 통해 전송할 준비를 한다.

## Application Layer

- email 클라이어느나 웹 브라우저 같은 end-user 소프트웨어에 의해 사용되는 계층이다.
- 대표적인 프로토콜로는 HTTP 프로토콜, DNS 시스템 등이 있다.

---

[https://youtu.be/1pfTxp25MA8](https://youtu.be/1pfTxp25MA8)

[http://wiki.hash.kr/index.php/OSI_7_계층](http://wiki.hash.kr/index.php/OSI_7_%EA%B3%84%EC%B8%B5)

[https://itwiki.kr/w/OSI_7계층#Layer_1_:_물리_계층(Physical_layer)](https://itwiki.kr/w/OSI_7%EA%B3%84%EC%B8%B5#Layer_1_:_%EB%AC%BC%EB%A6%AC_%EA%B3%84%EC%B8%B5(Physical_layer))

[https://www.imperva.com/learn/application-security/osi-model/](https://www.imperva.com/learn/application-security/osi-model/)
