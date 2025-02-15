# Kotlin RPC
## What is RPC?

- RPC(Remote Procedure Call)는 **원격 프로시저 호출**을 의미하며, 네트워크를 통해 원격 시스템에서 함수나 프로시저를 호출할 수 있도록 지원하는 기술이다.
- 이 기술은 로컬에서 함수를 호출하는 것처럼 원격지의 함수를 호출할 수 있게 하여, 네트워크 통신의 복잡성을 추상화한다.
- 개발자는 네트워크 계층이나 데이터 전송 방식에 대해 신경 쓰지 않고, 마치 로컬 함수처럼 원격 함수를 사용할 수 있다.
- **장점**
  - 네트워크 통신의 복잡성을 숨기고 개발 생산성을 높일 수 있다.
  - 분산 시스템에서 효율적인 통신이 가능
  - 다양한 언어 및 플랫폼에서 동작할 수 있는 유연성
- **단점**
  - 네트워크 상태에 따라 호출 실행 및 반환 시간이 보장되지 않는다.
  - 보안 문제에 취약할 수 있다.
  - RESTful API에 비해 상대적으로 복잡한 설정과 학습 곡선을 요구한다.

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

## Kotlinx.rpc

- kotlin 언어 기반의 멀티플랫폼 RPC 라이브러리
- kotlin 멀티플랫폼을 지원하여 다양한 플랫폼에서 동일 코드베이스로 RPC를 구현할 수 있다.

## kotlinx.rpc 예제

### Service Declration

- `@Rpc` 어노테이션과 `RemoteService` 인터페이스로 RPC 서비스를 정의한다.
- 매개변수 타입엔 `@Serializable` 어노테이션을 사용한다.

```kotlin
import kotlinx.rpc.RemoteService
import kotlinx.rpc.annotations.Rpc
import kotlinx.serialization.Serializable

@Rpc
interface OrderPlacement : RemoteService {
	suspend fun placeOrder(orderLines: List<OrderLine>): OrderId
	
	@Serializable
	data class OrderLine(val beverageName: String, val quantity: Int)
}
```

### Service Implementation

- 정의된 인터페이스를 구현하고 서버를 구성하는 코드는 아래와 같다.

```kotlin
// OrderPlacement 구현
class NoOpOrderPlacementProcessor(
	override val coroutineContext: CoroutineContext
) : OrderPlacement {
    override suspend fun placeOrder(orderLines: List<OrderPlacement.OrderLine>): OrderId {
        log.info { "Placing order with $orderLines lines" }
        return OrderId(Uuid.random().toString())
    }
}
```

```kotlin
import io.ktor.server.engine.*
import io.ktor.server.netty.*
import io.ktor.server.routing.*
import kotlinx.rpc.krpc.ktor.server.Krpc
import kotlinx.rpc.krpc.ktor.server.rpc
import kotlinx.rpc.krpc.serialization.json.json
import kotlinx.serialization.json.Json

fun main() {
    embeddedServer(Netty, port = SERVER_PORT, host = SERVER_HOST) {
        install(Krpc)
        routing {
            rpc("/order") {
                rpcConfig {
                    serialization { json(Json) }
                }
                registerService<OrderPlacement> { ctx ->
                    NoOpOrderPlacementProcessor(ctx)
                }
            }
        }
    }.start(wait = true)
}
```

- `embeddedServer`를 사용해 Ktor 서버 구성
- `Krpc`는 `kotlinx.rpc`에서 제공하는 Ktor 플러그인으로, RPC 통신을 처리하는 데 필요한 기능을 제공
- **`registerService<T>`**는 특정 서비스 인터페이스(**`OrderPlacement`**)를 구현하는 객체를 등록
- 클라이언트가 /order 경로로 요청을 보내면 서버는 등록된 서비스(**`OrderPlacement`**)의 구현체(**`NoOpOrderPlacementProcessor`**)를 호출하여 요청을 처리

### Service Stup

- 클라이언트 측에서 kotlin.rpc를 사용해 RPC 호출을 수행하는 예제 코드이다.

```kotlin
val httpClient = HttpClient {
	install(WebSockets)
	install(Krpc)
}
val stup = httpClient.rpc {
	url {
		host = "localhost"
		port = 8080
		encodedPath = "/order"
	}
	rpcConfig {
		serialization { json() }
	}
}.withService<OrderPlacement>()

val orderLines = listOf(OrderPlacement.OrderLine(beverageName = "ESPRESSO", quantity = 1))
val orderId = stup.placeOrder(orderLines)
```

- Ktor의 `HttpClient`를 사용해 서버와 통신
- `httpClient.rpc { ... }`는 클라이언트가 서버의 특정 RPC 엔드포인트에 연결하도록 설정
- **`withService<T>()`**
  - `.withService<OrderPlacement>()`는 클라이언트 측에서 사용할 서비스 스텁(stub)을 생성

## Features and Philosophy of kotlinx.rpc

- 순수 Kotlin으로 정의된 서비스 인터페이스와 데이터 모델
  - 서비스 인터페이스와 데이터 모델은 Kotlin 언어만으로 작성된다.
  - 개발자가 Kotlin의 친숙한 문법으로 작업할 수 있다.
- Kotlin 멀티플랫폼 프로젝트와의 완벽한 통합
  - `kotlinx.rpc`는 Kotlin Multiplatform 프로젝트에서 쉽게 사용할 수 있도록 설계되었다.
  - 이를 통해 단일 코드베이스로 Android, iOS, JVM, JS 등 다양한 플랫폼에서 RPC를 구현할 수 있다.
- 코루틴 기반의 비동기 RPC 호출
  - Kotlin 코루틴을 활용해 RPC 호출이 suspend 함수로 구현된다.
  - 이는 비동기 작업을 간단하고 효율적으로 처리할 수 있게 해준다.
- 전송 방식에 독립적 (Transport-agnostic)
  - `kotlinx.rpc`는 HTTP, WebSocket 등 다양한 전송 프로토콜을 지원하며, 사용자는 전송 방식의 세부 사항을 신경 쓰지 않아도 된다.
  - 라이브러리가 전송 프로토콜의 복잡성을 추상화하여 개발자의 편의를 제공
- 경량화 및 고속 처리
  - `kotlinx.serialization`, 바이너리 포맷, WebSocket 등을 활용하여 경량화되고 빠른 RPC 통신을 제공
  - 이로 인해 네트워크 대역폭을 효율적으로 사용하며, 성능이 최적화된다.
- JetBrains의 지원
  - JetBrains가 관리 및 유지보수를 담당하기 때문에 지속적인 업데이트와 개선이 이루어진다.
  - 안정성과 신뢰성을 보장

## Limitations and future challenges

- Kotlin 의존성
  - 클라이언트와 서버 모두 Kotlin을 사용해야 한다는 점이 아이러니하게 제약이 된다.
  - 이는 다른 언어를 사용하는 프로젝트와의 호환성을 제한한다.
- 밀접한 클라이언트-서버 결합
  - 클라이언트와 서버가 동일한 RPC 인터페이스를 공유해야 한다.
  - 때문에 양측의 업데이트가 동기화되어야 하며 이는 유지보수 복잡성을 증가시킬 수 있다.
- 성능 제한
  - 기본 JSON 직렬화와 WebSocket 사용으로 인해 성능이 제한될 수 있다.
  - 고성능 요구 사항이 있는 프로젝트에서는 제약이 될 수 있습니다.
- 새로운 기술
  - 2024년에 출시된 비교적 새로운 기술
  - 아직 실험적이며 정식 릴리스(1.0 버전) 이전 단계이므로 안정성과 성숙도가 부족할 수 있다.
- 도구 생태계 부족
  - 모니터링 및 디버깅 도구가 아직 충분히 개발되지 않았기에 복잡한 문제를 해결하는 데 어려움이 있을 수 있다.

---

References

- https://github.com/Kotlin/kotlinx-rpc/blob/main/README.md
- https://ktor.io/docs/tutorial-first-steps-with-kotlin-rpc.html#client-implementation
- https://github.com/springrunner/learn-kotlinx-libraries-by-examples
- Kotlin Backend Meetup - Kotlin RPC (박용권)
