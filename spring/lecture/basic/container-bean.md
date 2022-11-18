# 스프링 컨테이너와 스프링 빈
```java
**ApplicationContext applicationContext =new AnnotationConfigApplicationContext(AppConfig.class);**
```

- `ApplicationContext`를 스프링 컨테이너라 한다.
- `ApplicationContext`는 인터페이스이다.
    - 인터페이스이므로 다형성이 적용된다.
    - 구현체 중 하나가 `AnnotationConfigApplicationContext`이다.
- 스프링 컨테이너는 XML을 기반으로 만들 수 있고, 애노테이션 기반의 자바 설정 클래스로 만들 수 있다.
- 자바 설정 클래스를 기반으로 스프링 컨테이너( `ApplicationContext`)를 만들어보자.
    - `new AnnotationConfigApplicationContext(AppConfig.class);`
    - 이 클래스는 ApplicationContext 인터페이스의 구현체이다.

  > 참고: 더 정확히는 스프링 컨테이너를 부를 때 `BeanFactory`, `ApplicationContext`로 구분해서 이야기 한다. 이 부분은 뒤에서 설명하겠다. `BeanFactory`를 직접 사용하는 경우는 거의 없으므로 일반적으로 `ApplicationContext`를 스프링 컨테이너라 한다.
>

## 스프링 컨테이너 생성

### 1. 스프링 컨테이너 생성
- `new AnnotationConfigApplicationContext(AppConfig.class)`
- 스프링 컨테이너를 생성할 때는 구성 정보를 지정해주어야 한다.
- 여기서는 AppConfig.class 를 구성 정보로 지정했다.
- 키 값이 빈 이름이고, value 값이 빈 객체인 빈 저장소에 빈 정보가 저장 된다.

### 2.  스프링 빈 등록
스프링 컨테이너는 파라미터로 넘어온 설정 클래스 정보를 사용해서 스프링 빈을 등록한다.

**빈 이름**

- 이름은 메서드 이름을 사용한다.
- 빈 이름을 직접 부여할 수 도 있다.
  - `@Bean(name="memberService2")`

> **주의: 빈 이름은 항상 다른 이름을 부여해야 한다.** 같은 이름을 부여하면, 다른 빈이 무시되거나, 기존 빈을 덮어버리거나 설정에 따라 오류가 발생한다.

### 3. 스프링 빈 의존 관계 설정 - 완료
- 스프링 컨테이너는 설정 정보를 참고해서 의존 관계를 주입(DI)한다.
- 단순히 자바 코드를 호출하는 것 같지만, 차이가 있다. 이 차이는 뒤에 싱글톤 컨테이너에서 설명한다.

**참고**

- 스프링은 빈을 생성하고, 의존 관계를 주입하는 단계가 나누어져 있다.
- 그런데 이렇게 자바 코드로 스프링 빈을 등록하면 생성자를 호출하면서 의존 관계 주입도 한번에 처리된다.
  - 위 `AppConfig`에서도 보이듯이 `MemberService`를 생성하기 위해 생성자 파라미터에 `memberRepository()` 메서드를 호출하여 의존 관계의 객체를 생성하여 넣어 준다.
- 여기서는 이해를 돕기 위해 개념적으로 나누어 설명했다.

## BeanFactory와 ApplicationContext
### BeanFactory

- 스프링 컨테이너의 최상위 인터페이스다.
- 스프링 빈을 관리하고 조회하는 역할을 담당한다.
- `getBean()` 을 제공한다.
- 지금까지 우리가 사용했던 대부분의 기능은 BeanFactory가 제공하는 기능이다.

### ApplicationContext

- BeanFactory 기능을 모두 상속받아서 제공한다.
- 빈을 관리하고 검색하는 기능을 BeanFactory가 제공해주는데, 그러면 둘의 차이가 뭘까?
- 애플리케이션을 개발할 때는 빈은 관리하고 조회하는 기능은 물론이고, 수 많은 부가 기능이 필요하다.

### ApplicatonContext가 제공하는 부가 기능
- 메시지소스를 활용한 국제화 기능
  - 예를 들어서 한국에서 들어오면 한국어로, 영어권에서 들어오면 영어로 출력
- 환경변수
  - 로컬, 개발, 운영등을 구분해서 처리
  - 운영 디비와 개발 디비 나누기 등
- 애플리케이션 이벤트
  - 이벤트를 발행하고 구독하는 모델을 편리하게 지원
- 편리한 리소스 조회
  - 파일, 클래스 패스, 외부 등에서 리소스를 편리하게 조회
