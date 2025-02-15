# Kotlin RPC
## What is RPC?

RPC(Remote Procedure Call)는 **원격 프로시저 호출**을 의미하며, 네트워크를 통해 원격 시스템에서 함수나 프로시저를 호출할 수 있도록 지원하는 기술입니다. 이 기술은 로컬에서 함수를 호출하는 것처럼 원격지의 함수를 호출할 수 있게 하여, 네트워크 통신의 복잡성을 추상화합니다. 개발자는 네트워크 계층이나 데이터 전송 방식에 대해 신경 쓰지 않고, 마치 로컬 함수처럼 원격 함수를 사용할 수 있습니다

- **장점**
    - 네트워크 통신의 복잡성을 숨기고 개발 생산성을 높일 수 있습니다.
    - 분산 시스템에서 효율적인 통신이 가능합니다.
    - 다양한 언어 및 플랫폼에서 동작할 수 있는 유연성을 제공합니다.
- **단점**
    - 네트워크 상태에 따라 호출 실행 및 반환 시간이 보장되지 않습니다.
    - 보안 문제에 취약할 수 있습니다.
    - RESTful API에 비해 상대적으로 복잡한 설정과 학습 곡선을 요구합니다

### local vs rpc

| 항목 | Local Procedure Call (LPC) | Remote Procedure Call (RPC) |
| --- | --- | --- |
| 호출 위치 | 동일한 프로세스 또는 같은 시스템 내에서 호출됨 | 네트워크를 통해 다른 시스템(원격 서버)에서 호출됨 |
| 통신 방식 | 메모리 내 함수 호출로 직접 실행 | 네트워크를 통한 메시지 기반 통신 사용 |
| 속도 | 매우 빠름 (메모리 접근 수준) | 상대적으로 느림 (네트워크 지연 발생 가능) |
| 복잡성 | 단순하고 추가 설정 필요 없음 | 네트워크 설정, 데이터 직렬화/역직렬화 등 복잡성 존재 |
| 의존성 | 네트워크와 무관 | 네트워크 상태 및 연결에 의존 |
| 오류 처리 | 로컬 오류만 처리 | 네트워크 오류, 타임아웃 등 추가적인 오류 처리 필요 |

### RPC struture

```
Service Stub → RPC Client ←transport→ RPC Server ← Service Implementation
```

- Service Declaration
    - 서비스의 인터페이스를 정의
    - 클라이언트와 서버가 서로 통신하기 위한 메서드, 데이터 타입 등 선언
- Service Stub
    - 클라이언트 측에서 호출되는 Stub
    - 클라이언트는 Stub을 통해 원격 호출을 로컬 함수처럼 사용
- RPC Client
    - 클라이언트 애플리케이션에서 RPC 요청을 보내는 주체
    - Service Stub과 상호작용하여 데이터를 전송 계층으로 전달하고 응답을 처리
- RPC 서버
    - 원격 요청을 처리하는 서버 측 구성 요소
    - 직렬화, 역직렬화 등을 수행
- Service Implementation
    - 서버 측에서 실제 PRC 메서드가 구현된 부분
    - Service Declaration에서 정의된 인터페이스를 구현
- Transport
    - 클라이언트와 서버 간 데이터 전송 계측
    - 다양한 프로토콜이 사용될 수 있다.

### RPC technologies

- gRPC
- Apache Thrift
- Apache Dubbo
- SOAP (Simple Object Access Protocol)
- WCF (Windows Communication Foundation
- JSON-RPC / XML-RPC
- Java RMI (Remote Method Invocation)
- ZeroC; IceRPC
