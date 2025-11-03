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

## 호출 검증

### 호출 횟수 검증

```kotlin
// 정확히 3번 호출
verify(exactly = 3) { emailService.sendEmail(any(), any()) }

// 최소 2번 이상
verify(atLeast = 2) { emailService.sendEmail(any(), any()) }

// 최대 4번 이하
verify(atMost = 4) { emailService.sendEmail(any(), any()) }

// 0번 (호출 안 됨)
verify(exactly = 0) { emailService.sendEmail("never@example.com", any()) }
```

### 호출 순서 검증

```kotlin
// ...
every { repository.findUser(any()) } returns User(id = 1, name = "홍길동")
every { cache.put(any(), any()) } just runs

// ...

// 특정 순서대로 호출되었는지 확인
verifySequence {
    repository.findUser(1)
    cache.put(1, any())  // repository 호출 후 cache 호출
}

// verifyOrder는 순서만 확인, 사이에 다른 호출 가능
verifyOrder {
    repository.findUser(1)
    cache.put(1, any())
}
```

## 트러블 슈팅 및 주의사항

### 인라인 함수는 모킹 불가

- 인라인 함수는 컴파일 시점에 호출 코드로 대체 되기 때문에 모킹이 불가능하다.
- 다른 JVM 기반 모킹 프레임워크들은 대부분 마찬가지다.

### private 함수 모킹

- `private` 함수 모킹을 위해선 다음 두 가지가 필요하다.
    - `spyk`로 실제 객체와 함께 사용
    - `recordPrivateCalls = true` 옵션

```kotlin
class Calculator {
    fun add(a: Int, b: Int): Int = internalAdd(a, b)
    
    private fun internalAdd(a: Int, b: Int): Int = a + b
}

@Test
fun `private 함수 모킹`() {
    val calc = spyk(Calculator(), recordPrivateCalls = true)

    every { calc["internalAdd"](any(), any()) } returns 100

    val result = calc.add(5, 3)
    result shouldBe 100

    verify { calc["internalAdd"](5, 3) }
}

```

- 가능은 하지만 `private` 함수에 대한 모킹은 테스트에서 권장하지 않는다.
- `private` 함수를 모킹하거나 검증해야할 정도면 퍼블릭 함수 또는 별도 클래스로의 분리를 추천한다.

### Generic 타입 주의

```kotlin
// 문제: relaxed mock에서 generic 타입 ClassCastException
val service = mockk<GenericService<User>>(relaxed = true)
```

- relaxed = true 옵션은 mock 객체에서 기본값을 반환하도록 해준다.
- 하지만 제네릭 타입이 포함된 경우 MockK가 타입을 제대로 인식하지 못하면 `ClassCastException`이 발생한다.
- 이러한 경우 명시적으로 모킹을 해서 알맞은 타입 객체를 반환하도록 해야 한다.

```kotlin
// 해결: 명시적 stubbing
val service = mockk<GenericService<User>>()
every { service.get() } returns User(id = 1, name = "홍길동")
```

### 생성자 파라미터가 있는 생성자 모킹

- 특정 생성자 인자를 가진 객체를 모킹하고 싶을 때 아래와 같이 할 수 있다.

```kotlin
class OrderProcessor(private val baseDiscount: Double) {
    fun applyDiscount(price: Double): Double = price * (1 - baseDiscount)
}

@Test
fun `특정 생성자만 모킹`() {
    mockkConstructor(OrderProcessor::class)

    // 매개변수 0.1인 생성자만 모킹
    every { 
        constructedWith<OrderProcessor>(EqMatcher(0.1)).applyDiscount(any()) 
    } returns 8000

    val processor = OrderProcessor(0.1)
    processor.applyDiscount(10000) shouldBe 8000
}

```

- `mockkConstructor` - 지정한 클래스의 모든 생성자를 모킹 대상으로 지정
- `constructedWith<Type>(EqMatcher(…)).applyDiscount(…)` - EqMatcher에 전달된 파라미터 값인 경우에만 해당 메서드를 모킹
  - 위 예제 코드에선 0.1로 생성된 `OrderProcessor`에 대해서만 `applyDiscount` 메서드가 8000을 반환한다.

## 참고 자료

- [**공식 문서**](https://mockk.io/)
- [**Github Repository/Issues**](https://github.com/mockk/mockk)
