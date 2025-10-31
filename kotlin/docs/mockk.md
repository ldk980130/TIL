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

### Spy와 부분 모킹

- 실제 구현과 Mock을 혼합하여 사용할 수 있다.

```kotlin
// ...
    @Test
    fun `이벤트 필터링이 정상 작동해야 한다`() {
		    val eventRepository = mockk<EventRepository>()
		    
        // 실제 EventService 인스턴스 생성, private 호출 기록
        val eventService = spyk(EventService(eventRepository), recordPrivateCalls = true)

        val events = listOf(
            Event(id = 1, status = "ACTIVE", createdAt = LocalDateTime.now()),
            Event(id = 2, status = "INACTIVE", createdAt = LocalDateTime.now().minusDays(1)),
            Event(id = 3, status = "ACTIVE", createdAt = LocalDateTime.now().minusDays(2))
        )

        every { eventRepository.findAll() } returns events

        // 실제 filterActiveEvents() 메서드 호출
        val result = eventService.getActiveEvents()

        // private 메서드 호출 검증
        verify { eventService["filterByStatus"](events, "ACTIVE") }
        result.size shouldBe 2
    }
}

```

### 인자 값 캡쳐링 (Capturing)

- `CapturingSlot`로 호출된 인자값을 추출할 수 있다.

```kotlin
// ...
    @Test
    fun `전송된 이메일 내용을 검증해야 한다`() {
        val emailSlot = slot<Email>()

        every { emailRepository.save(capture(emailSlot)) } returns Email(id = 1, to = "", subject = "")

        // When
        emailService.sendWelcomeEmail("newuser@example.com")

        // Then - 실제로 저장된 Email 객체 내용 검증
        val capturedEmail = emailSlot.captured
        capturedEmail.to shouldBe "newuser@example.com"
        capturedEmail.subject shouldContain "환영"
        capturedEmail.body shouldContain "가입을 환영합니다"
    }
}

```

- `MutableList`로 여러 호출을 기록할 수도 있다.

```kotlin
@Test
fun `여러 번의 로깅 호출을 모두 기록해야 한다`() {
    val logMessages = mutableListOf<String>()
    val logger = mockk<Logger>()

    every { logger.info(capture(logMessages)) } just runs

    // When
    val userService = UserService(logger)
    userService.registerUser("john@example.com")
    userService.registerUser("jane@example.com")

    // Then
    logMessages.size shouldBe 2
    logMessages[0] shouldContain "john@example.com"
    logMessages[1] shouldContain "jane@example.com"
}

```

### 확장함수 및 정적함수 모킹

```kotlin
// 모듈 레벨 확장함수 (String.kt 파일에 정의)
fun String.isValidEmail(): Boolean {
    return this.contains("@")
}

// 테스트
@Test
fun `확장함수 모킹`() {
    val user = User("test@example.com")
    
    // 모듈 전체를 mock 대상으로 지정
    mockkStatic("com.example.StringKt")
    
    every { user.email.isValidEmail() } returns false
    
    val result = user.email.isValidEmail()
    result shouldBe false
    
    unmockkStatic("com.example.StringKt")
}

```

```kotlin
// Utils.kt
fun generateOrderId(): String = "ORD-${System.currentTimeMillis()}"

// 테스트
@Test
fun `탑레벨 함수 모킹`() {
    mockkStatic(::generateOrderId)
    
    every { generateOrderId() } returns "ORD-MOCK-001"
    
    val orderId = generateOrderId()
    orderId shouldBe "ORD-MOCK-001"
    
    unmockkStatic(::generateOrderId)
}

```
