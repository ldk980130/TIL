관점 지향 프로그래밍 (AOP)는 프로그램 구조에 대한 다른 관점을 제시하며 OOP를 보완한다. OOP에서 핵심 단위가 클래스라면, AOP에서 핵심 단위는 관점(aspect)이다. acpect는 여러 타입과 객체들을 가로지르는 (트랜잭션 관리 같은) 문제의 모듈화를 가능하게 한다. (AOP 문헌에는 이런 문제를 cross-cutting concern, 횡단 관심사라고 부름)

AOP 프레임워크는 스프링의 핵심 구성 중 하나다. 스프링 IoC 컨테이너가 AOP에 의존하지 않는 반면 (원치 않으면 사용할 필요 없다는 의미) AOP는 IoC를 보완하여 매우 유능한 솔루션을 제공한다.

> **Spring AOP with AspectJ pointcuts**
스프링은 schema-based approach나 @AspectJ annotation style을 사용하는 커스텀 aspect를 작성하는 쉽고 강력한 방법을 제공한다.
>

AOP 사용 예

- 선언적 엔터프라이즈 서비스를 제공한다. 이러한 서비스 중 가장 중요한 것이 선언적 트랜잭션 관리이다.
- 사용자가 커스텀 aspect를 구현해 OOP를 보완할 수 있다.

## [5.1 AOP Concepts](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-introduction-defn)

- **Aspect**: 여러 클래스를 아우르는 관심사의 모듈화이다. 트랜잭션 관리가 엔터프라이즈 자바 애플리케이션의 crosscutting 관심사를 보여주는 좋은 예다. 스프링에서 AOP aspect는 일반 클래스(schema-based) 또는 `@Aspect`(`@AspectJ` 스타일) 주석에 의해 구현된다.
- **Join point**: 메서드의 실행이나 예외 처리 같은 프로그램 실행의 한 지점이다. 스프링 AOP에서 join point는 항상 메서드 실행을 나타낸다.
- **Advice**: 특정 join point에서 aspect에 의해 수행되는 작업이다. “around”, “before”, “after” 등의 다른 타입의 advice가 존재한다. (타입에 대해선 나중에 다룸) 스프링을 포함한 많은 AOP 프레임워크에서 advise를 인터셉터로 모델링하고 join point에서 인터셉터 체인을 유지한다.
- **Pointcut**: join point와 매칭되는 predicate이다. advice는 pointcut 표현과 관련이 있으며 pointcut과 일치하는 join point에서 실행된다. (예: 특정 이름을 가진 메서드 실행) pointcut 정규식과 일치하는 join point라는 컨셉은 AOP의 핵심이며 스프링은 AspectJ pointcut 정규식을 기본적으로 사용한다.
- **Introduction**: 타입 대신 추가 메서드 또는 필드를 선언한다. 스프링 AOP를 사용하면 advice가 적용된 객체에 새로운 인터페이스를 도입할 수 있다. 예를 들어 캐싱을 단순화하기 위해 IsModified 인터페이스를 빈이 구현하도록 introduction을 사용할 수 있다.
- **Target object**: 하나 이상의 aspect에 의해 advice가 적용되는 대상 객체이다. advised object라고도 한다. 스프링 AOP는 런타임 프록시를 사용하여 구현되므로 이 객체는 항상 프록시 객체다.
- **AOP proxy**: aspect를 구현하기 위해 AOP가 만드는 객체이다. 스프링 AOP에서 프록시는 JDK 다이나믹 프록시이거나 CGLIB 프록시다.
- **Weaving**: advised object를 만들기 위해 다른 애플리케이션 타입이나 객체의 aspect와 연결한다. (pointcout에 의해 결정된 join point에 advice를 삽입하는 과정) 이는 컴파일 시점, 로드 시점 또는 런타임에 수행될 수 있다. Spring AOP는 다른 순수 자바 AOP와 마찬가지로 런타임에 weaving을 수행한다.

**Advice 타입**

- **Before advice :** join point 전에 실행되는 advice. 하지만 예외를 던지지 않는 한 메서드 실행을 막을 수는 없다.
- **After returning advice:** join point가 일반적으로 완료된 후에 실행되는 advice이다. (메서드가 예외 없이 리턴하는 경우)
- **After throwing advice:** 메서드에 예외가 발생하는 경우 실행되는 advice
- **After (finally) advice:** join point가 종료되는 방법에 상관 없이 실행되는 advice (예외가 발생하든 안하든 실행)
- **Around advice:** 메서드 호출 같은 join point를 둘러싸는 advice. 이는 가장 강력한 advice이며 메서드 호출 전후에 사용자 정의 동작을 수행할 수 있다. 또한 join point 메서드의 실행을 계속하거나 자체 반환 값을 던지든 예외를 던져서 shortcut하기를 선택할 책임도 가지고 있다.

around advice는 가장 일반적인 종류의 advice이다. AspectJ와 마찬가지로 Spring AOP는 모든 범위의 advice 유형을 제공하기 때문에 필요한 동작을 구현할 수 있는 가장 강력한 advice 타입을 사용하는 것이 좋다. 예를 들어 캐시를 메서드의 반환 값으로 업데이트하기만 하면 되는 경우 around advice 보다는 after returning advice를 사용하는 것이 낫다. (동일한 작업을 수행하지만) 가장 구체적인 advice 유형을 사용하면 오류 가능성이 적은 간단한 프로그래밍 모델을 제공할 수 있다.

모든 advice 매개변수는 정적으로 입력되므로 Object 배열이 아닌 적절한 유형의 advice 매개변수로 작업할 수 있다.

point cut과 일치하는 join point 개념은 AOP의 핵심이며 이는 인터셉션만 제공하는 이전 기술과 구별된다. point cut은 advice를 갳체 지향 계층과 독립적으로 존재할 수 있게 한다. 예를 들어 선언적 트랜잭션 관리를 around advice를 사용해 서비스 계층의 모든 비즈니스 작업 등 여러 객체에 걸쳐 있는 일련의 메서드에 적용할 수도 있다.

## [5.3 AOP Proxies](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-introduction-proxies)

스프링 AOP는 기본적으로 AOP 프록시 표준 JDK 동적 프록시를 사용한다. CGLIB 프록시를 사용할 수도 있는데 이는 인터페이스가 아닌 구체 클래스에도 적용된다. 기본적으로 비즈니스 객체가 인터페이스를 구현하지 않는 경우 CGLIB가 사용된다.

## [5.8 Proxying Mechanisms](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-proxying)

스프링에선 인터페이스 유무에 따라 JDK 프록시를 적용할지, CGLIB 프록시를 적용할지 결정한다. 만약 CGLIB 프록시를 사용하도록 강제하고 싶다면 그렇게 할 수도 있지만 다음의 이슈가 있다.

- `final` 메서드는 advise를 적용할 수 없다.
- As of Spring 4.0, the constructor of your proxied object is NOT called twice anymore, since the CGLIB proxy instance is created through Objenesis. Only if your JVM does not allow for constructor bypassing, you might see double invocations and corresponding debug log entries from Spring’s AOP support.

CGLIB를 사용하도록 강제하기 위해선 `proxy-target-class`를 true로 바꾸면 된다.
