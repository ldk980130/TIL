## MockK란

- Kotlin 문법 최적화: Null-safety, 확장함수, 코루틴 완벽 지원
- 엄격한 기본 동작: 기본적으로 strict mock으로 시작 (불필요한 stub 방지)
- 풍부한 타입 지원: 클래스, 오브젝트, 인터페이스, 생성자, 익스텐션 함수, 탑레벨 함수 등 거의 모든 타입 모킹 가능
- Spring 통합: JUnit 5 확장, 애너테이션 기반 의존성 주입 지원
- 고급 기능: Spy(부분 모킹), 캡처링, 계층적 모킹, 동적 호출 등

## 다양한 stubbing 기법

### 단순 반환값 지정

```kotlin
val userRepository = mockk<UserRepository>()

// 특정 입력에 대한 반환
every { userRepository.findById(1) } returns User(id = 1, name = "홍길동")

// 모든 입력에 대해 동일 반환
every { userRepository.findAll() } returns listOf(
    User(id = 1, name = "홍길동"),
    User(id = 2, name = "김영희")
)
```

### Matcher 활용

```kotlin
val orderService = mockk<OrderService>()

// any() - 모든 값 허용
every { orderService.calculateShippingFee(any()) } returns 5000

// 범위 지정 (more, less 등)
every { 
    orderService.applyDiscount(more(100000)) 
} returns 0.15  // 100,000원 이상 구매 시 15% 할인

// or() - 여러 값 중 하나
every { 
    orderService.isEligibleForPromotion(or(1, 2, 3)) 
} returns true  // ID가 1, 2, 3인 고객만 프로모션 대상

```

### 콜백 활용 - answers

```kotlin
val userService = mockk<UserService>()

every { userService.authenticateUser(any(), any()) } answers {
    val email = firstArg<String>()
    val password = secondArg<String>()
    
    if (email == "admin@example.com" && password == "correct") {
        AuthToken(token = "jwt-token-123", userId = 1)
    } else {
        throw AuthenticationException("인증 실패")
    }
}

val token = userService.authenticateUser("admin@example.com", "correct")
// AuthToken(token = "jwt-token-123", userId = 1)

```

### 시나리오 검증

```kotlin
val apiClient = mockk<ApiClient>()

every { apiClient.retry() } returns Unit andThen {
    throw TimeoutException("타임아웃")
} andThen {
    Unit  // 세 번째 시도 성공
}

apiClient.retry()  // 성공
apiClient.retry()  // 실패
apiClient.retry()  // 성공

```
