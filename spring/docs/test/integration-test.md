# [3. Integration Testing](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#integration-testing)

## [3.2.1 Context Management and Caching](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testing-ctx-management)

Spring Test Context Framework는 Spring `ApplicationContext` 인스턴스와 `WebApplicationContext` 인스턴스의 일관된 로드 뿐만 아니라 캐싱도 제공한다.

캐싱은 중요한데 스프링 컨테이너에 의해 컨텍스트가 인스턴스화하는 데 시간이 많이 걸리기 때문이다.

## [3.2.2 Dpendency Injection of Test Fixtures](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testing-fixture-di)

TestContext 프레임워크가 application context를 로드할 때 DI를 사용해서 선택적으로 테스트 클래스의 인스턴스를 구성할 수 있다.

application context에 미리 구성된 bean을 사용해서 테스트 픽스처를 쉽게 세팅할 수 있는 것이다.

다양한 테스트 시나리오에서 컨텍스트를 재사용할 수 있다.

## [3.2.3 Transaction Management](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testing-tx)

테스트에서 흔한 이슈는 실제 DB에 접근하는 것이다.

개발 DB를 테스트에 사용한다면 상태의 변화가 미래 테스트에 영향을 줄 것이다.

데이터 삽입, 수정과 같은 많은 작업은 트랜잭션 밖에서 수행될 수 없다.

TestContext 프레임워크는 각 테스트마다 트랜잭션을 롤백한다.

트랜잭션 지원은 application context의 `PlatformTransactionManager` bean에 의해 제공된다.

만약 트랜잭션을 커밋하고 싶으면  `@Commit` 어노테이션으로 TestContext에게 롤백하지 않게 설정할 수 있다.

## [3.5 Spring TestContext Framework](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-framework)

`org.springframework.test.context` 패키지에 있는 Spring TestContext Framework는 사용 중인 테스트 프레임워크와 무관한 일반적인 어노테이션 기반의 단위, 통합 테스트 지원을 제공한다.

TestContext 프레임워크는  어노테이션 기반 configuration을 통해 오버라이드할 수 있지만 기본값 규칙을 더 중요하게 생각한다.

일반적인 테스트 인프라 외에도 Testcontext Framework는 Junit4 Jupiter (Junit5) 그리고 TestNG에 대한 명시적 지원을 제공한다.

### [3.5.1 Key Abstractions](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-key-abstractions)

프레임워크의 핵심은 `TestContextManager` 클래스와 `TestContext`, `TestExecutionListener` 및 `SmartContextLoader` 인터페이스로 구성된다.

각 테스트 클래스에 대해 `TestContextManager`가 생성된다. (Jupiter 단위의 단일 테스트 클래스 내에서 모든 테스트 메서드를 실행할 경우)

또한 `TestContextManager`는 현재 테스트의 context를 유지하는 `TestContext`를 관리한다.

또한 `TestContextManager`는 테스트가 진행됨에 따라 `TestContext`의 상태를 업데이트 한다.

그리고 의존성 주입, 트랜잭션 관리 등을 통해 실제 테스트 실행을 계측하는 `TestExecutionListener` 구현을 위임한다.

`SmartContextLoader`는 지정된 테스트 클래스에 대한 `ApplicationContext`를 로드한다.

### `TestContext`

`TestContext`는 테스트가 실행되는 컨텍스트를 캡슐화하고 (사용 중인 실제 테스트 프레임워크와는 무관함) 테스트 인스턴스에 대한 컨텍스트 관리 및 캐싱 지원을 제공한다.

`TestContext`는 또한 `SmartContextLoader`에 요청 시 `ApplicationContext`를 로드하도록 위임한다.

### `TestContextManager`

`TestContextManager`는 Spring TestContext Framework의 주요 진입점이며 잘 정의된 테스트 실행 지점에서 등록된 각 `TestExecutionListener`에 대한 단일 `TestContext` 및 시그널링 이벤트를 관리하는 역할을 한다.

잘 정의된 테스트 실행 지점:

- 특정 테스트 프레임워크에서 “before class” 또는 “before all” 메서드에 앞선 지점
- 테스트 인스턴스 post-processing (후 처리)
- 특정 테스트 프레임워크의 “before” 또는 “before each” 메서드 이전
- 테스트 메서드 실행 직전, 테스트 설정 후
- 테스트 메서드 실행 직후, 테스트 해체 전
- 특정 테스트 프레임워크의 “after” 또는 “after each’ 메서드 이후
- 특정 테스트 프레임워크 “after class” 또는 “after all” 메서드 이후

### `TestExecutionListener`

`TestExecutionListener`는 listener가 등록된 `TestContextManager`에 의해 반응하는 테스트 실행 이벤트에 응답하기 위한 API를 정의한다. `[TestExecutionListener` Configuration](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-tel-config) 참조

### `Context Loaders`

`ContextLoader`는 Spring TestContext Framework에 의해 관리되는 통합 테스트를 위한 `ApplicationContext`를 로드하기 위한 전략 인터페이스다.

`ContextLoader` 대신 `SmartcontextLoader`를 구현해야 하는데 이는 component 클래스, active bean definition profiles, test property source, 컨텍스트 계층, 그리고 `WebApplicationContext` 지원을 제공한다.

### [3.5.2 BootStrapping the TestContext Framework](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-bootstrapping)

### ****[3.5.3. `TestExecutionListener` Configuration](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-tel-config)**

Spring은 기본적으로 아래의 `TestExecutionListener` 구현을 제공한다.

- `ServletTestExecutionListener`: `WebApplicationContext`에 대한 Servlet API mock을 구성한다.
- `DirtiesContextBeforeModesTestExecutionListener`: `@DirtiesContext` 어노테이션에 대한 “before” 모드를 핸들링한다.
- `DependencyInjectionTestExecutionListener`: 테스트 인스턴스에 대한 dependency injection을 제공한다.
- `DirtiesContextTestExecutionListener`: `@DirtiesContext` 어노테이션에 대한 “after” 모드를 핸들링한다.
- `TransactionalTestExecutionListener`: 기본적인 롤백과 함께 트랜잭션 테스트 실행을 제공한다.
- `SqlScriptsTestExecutionListener`: `@SQL` 어노테이션을 사용하여 구성된 SQL 스크립트를 실행한다.
- `EventPublishingTestExecutionListener`: 테스트 실행 이벤트를 테스트의 `ApplicationContext`에 게시한다.

### [3.5.6 Context Management](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-ctx-management)

### Context Caching

TestContext Framework가 `ApplicationContext`(또는 `WebApplicationContext`)를 테스트에서 로드하면 그 context는 캐싱되고 동일한 test suite에서 동일한 unique context 설정이 되어 있는 모든 이어진 테스트에서 재사용된다. 캐싱의 작동 방식을 이해하려면 “unique”와 “test suite”가 무엇을 의미하는지 이해하는 것이 중요하다.

`ApplicationContext`는 로드하는 데 사용되는 구성 매개 변수의 조합으로 고유하게 식별할 수 있다. 따라서 고유한 구성 매개 변수는 context가 캐시 되는 키를 생성하는 데 사용된다. TestContext framework는 다음 구성 매개 변수를 사용하여 context 캐시 키를 작성한다.

- `locations` (from `@ContextConfiguration`)
- `classes` (from `@ContextConfiguration`)
- `contextInitializerClasses` (from `@ContextConfiguration`)
- `contextCustomizers` (from `ContextCustomizerFactory`) –
    - 여기에는 `@DynamicPropertySource` 메서드뿐만 아니라 `@MockBean` 및 `@SpyBean`과 같은 스프링 부트의 테스트 지원의 다양한 기능이 포함ehlsek.
- `contextLoader` (from `@ContextConfiguration`)
- `parent` (from `@ContextHierarchy`)
- `activeProfiles` (from `@ActiveProfiles`)
- `propertySourceLocations` (from `@TestPropertySource`)
- `propertySourceProperties` (from `@TestPropertySource`)
- `resourceBasePath` (from `@WebAppConfiguration`)

> **Test suites and forked processes**
Spring TestContext framework는 application context를 정적 캐시에 저장한다. 즉 static 변수에 저장된다는 것을 의미한다. 즉 테스트가 별도의 프로세스에서 실행되는 경우 각 테스트 실행 사이에 정적 캐시가 삭제되어 캐싱 매커니즘이 비활성화된다.

캐싱 매커니즘의 이점을 얻으려면 모든 테스트가 동일한 프로세스 또는 동일한 test suite 내에서 실행되어야 한다. 이는 모든 테스트를 IDE내에서 그룹으로 실행함으로써 달성될 수 있다. 마찬가지로 Ant, Maven, Gradle 등의 빌드 프레임워크로 테스트를 실행할 때 빌드 프레임워크가 테스트 간에 분기되지 않도록 하는 것이 중요하다.
>

컨텍스트 캐시의 크기는 기본 최대 크기 32로 제한된다. 최대 크기에 도달할 때마다 LRU(Latest Recently Used) 제거 정책이 오래된 컨텍스트를 제거하고 닫는 데 사용 된다. 또는 command line이나 빌드 스크립트로 최대 사이즈를 설정할 수 있다. 또는 `SpringProperties` 메커니즘을 통해 동일한 속성을 설정할 수 있다.

`@DirtiesContext`를 사용하여 컨텍스트를 손상 시키고 다시 로드시킬 수도 있다. 그러면 Spring은 동일한 애플리케이션 컨텍스트를 필요로 하는 다음 테스트를 실행하기 전에 캐시에서 컨텍스트를 제거하고 애플리케이션 컨텍스트를 재구성하도록 지시한다.

### ****[3.5.7. Dependency Injection of Test Fixtures](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-fixture-di)****

`DependencyInjectionTestExecutionListener`을 사용하는 경우 (기본값으로) `@ContextConfiguration` 또는 관련 어노테이션을 사용하여 구성한 ApplicationContext의 bean들로부터 의존성들이 테스트 인스턴스에 주입된다.

setter 주입, 필드 주입 혹은 둘 다 사용할 수도 있다.

Junit Jupiter를 사용한다면 생성자 주입을 선택적으로 사용할 수도 있다.

스프링의 어노테이션 기반 의존성 주입 지원과의 일관성을 위해 스프링의 `@Autowired` 어노테이션을 사용할 수 있다.

*필드 주입은 프로덕션 코드에선 권장되지 않지만 테스트 코드에서는 필드 주입이 매우 자연스럽다.*

*차이에 대한 근거는 절대 테스트 클래스를 직접 인스턴스화하지 않는 점에 있다.*

*따라서 테스트 클래스에서 public 생성자나 setter 메서드를 호출할 필요가 없다.*

### ****[3.5.9. Transaction Management](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-tx)****

TestContext framework에서 트랜잭션은 `TransactionalTestExecutionListener`에 의해 관리된다.

테스트 클래스에 `@TestExecutionListeners`를 명시적으로 선언하지 않았어도 기본적으로 `TransactionalTestExecutionListener`의 관리를 받을 수 있다.

하지만 트랜잭션 지원을 받으려면 `@ContextConfiguration`과 함께 로드되는 `ApplicationContext`에 `PlatformTransactionManager`가  빈으로 구성되어 있어야 한다.

또한 스프링의 `@Transactional` 어노테이션을 메서드 혹은 클래스 수준에서 선언해야 한다.

### Test-managed Transactions

Test-managed Trasacntions은 `TransactionalTestExecutionListener`에 의해 관리되거나 `TestTransaction`을 사용해 프로그래밍적으로 관리되는 트랜잭션이다.

이러한 트랜잭션은 스프링이 관리하는 트랜잭션 (테스트용으로 로드된 `ApplicationContext`내에서 스프링이 직접 관리하는 트랜잭션) 또는 application-managed transactions (테스트에 의해 호출된 애플리케이션 코드 내에서 프로그래밍 방식으로 관리되는 트랜잭션)과는 다르다.

Spring-managed와 application-managed 트랜잭션은 일반적으로 test-managed 트랜잭션에 참여한다.

그러나 spring-managed와 application-managed 트랜잭션이 `REQUIRED` 또는 `SUPPORTS`이외의 `propagation` 유형으로 구성된 경우에는 주의해야 한다.

### Enabling and Disabling Transactions

`@Transactional`어노테이션을 테스트 메서드에 추가하면 기본적으로 테스트 완료 후 자동으로 롤백되는 트랜잭션 내에서 테스트가 실행된다.

테스트 클래스에 `@Transactional`이 선언된 경우 해당 클래서 계층 내의 각 테스트 메서드들은 트랜잭션 내에서 실행된다.

`@Transactional`은 Junit Jupiter의 `@BeforeAll`, `@BeforeEach`등의 테스트 라이프 사이클 메서드에서는 지원되지 않는다.

또한 `@Transactional` 어노테이션을 선언해도 `propagation`속성이 `NOT-SUPPORTED`또는 `NEVER`인 경우 트랜잭션 내에 실행되지 않는다.

### ****Transaction Rollback and Commit Behavior****

기본적으로 테스트 트랜잭션은 자동으로 테스트 완료 후 롤백한다. 하지만 `@Commit`과 `@Rollback`어노테이션을 통해 롤백 유무를 설정할 수 있다.
