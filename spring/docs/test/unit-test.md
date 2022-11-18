# [2. Unit Testing](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#unit-testing)

전통적인 Java EE 개발에서 보다 Dependency Injection은 코드가 컨테이너에 덜 의존적이게 되도록 만들어야 한다.

POJO 코드는 스프링이나 컨테이너 도움 없이 `new` 로 생성하여 Junit 등으로 테스트할 수 있어야 한다.

테스트를 고립시키기 위해 Mock 객체를 사용할 수도 있다.

True unit test는 일반적으로 매우 빠르게 실행되는데 이는 설정할 런타임 인프라가 없기 때문이다.

개발 방법론의 일부로 단위 테스트를 강조하면 개발 생산성을 높일 수 있다.

IoC 컨테이너 기반 테스트는 일반적으로 통합 테스트이지만 특정 시나리오 같은 경우 스프링 프레임워크는 모의 객체와 테스트 지원 클래스를 제공한다.
