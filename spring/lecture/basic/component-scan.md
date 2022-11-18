# 컴포넌트 스캔
## 컴포넌트 스캔과 의존 관계 자동 주입

### @ComponentScan

```java
@Configuration
@ComponentScan
public class AutoAppConfig {
}
```
- `@ComponentScan`은 `@Component`가 붙은 모든 클래스를 스프링 빈으로 등록한다.
- 이름 기본 전략: MemberServiceImpl 클래스 memberServiceImpl
- 빈 이름 직접 지정: 만약 스프링 빈의 이름을 직접 지정하고 싶으면`@Component("memberService2")` 이런식으로 이름을 부여하면 된다

### @Autowired 의존 관계 자동 주입
- 생성자에 `@Autowired`를 지정하면, 스프링 컨테이너가 자동으로 해당 스프링 빈을 찾아서 주입한다.
- 이때 기본 조회 전략은 타입이 같은 빈을 찾아서 주입한다.
    - `getBean(MemberRepository.class)`와 동일하다고 이해하면 된다.

## 탐색 위치와 기본 스캔 대상

모든 자바 클래스를 다 컴포넌트 스캔하면 시간이 오래 걸린다. 그래서 꼭 필요한 위치부터 탐색하도록 시작 위치를 지정할 수 있다.

```java
@ComponentScan(
		basePackages = {"hello.core", "hello.service"}
)
```

- `basePackages` : 탐색할 패키지의 시작 위치를 지정한다. 이 패키지를 포함해서 하위 패키지를 모두 탐색한다.
    - `basePackages = {"hello.core", "hello.service"}` 이렇게 여러 시작 위치를 지정할 수도 있다.
- `basePackages`를 지정하지 않으면 기본적으로 `@ComponentScan`이 붙은 클래스 위치 기준 하위 패키지를 모두 탐색한다.

**권장하는 방법**

개인적으로 즐겨 사용하는 방법은 패키지 위치를 지정하지 않고, 설정 정보 클래스의 위치를 프로젝트 최상단에 두는 것이다. 최근 스프링 부트도 이 방법을 기본으로 제공한다.

참고로 스프링부트를 사용하면 스프링부트의 대표 시작 정보인 `@SpringBootApplication`을 이 프로젝트 시작 위치에 두는 것이 관례이다. (이 설정 안에 바로 `@ComponentScan`이 들어 있다.)

## 필터

- `includeFilters`: 컴포넌트 스캔 대상을 추가로 지정한다.
- `excludeFilters`: 컴포넌트 스캔에서 제외할 대상을 지정한다.

```java
includeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyIncludeComponent.class),
excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyExcludeComponent.class)
```

**FilterType 5가지 옵션**

- `ANNOTATION`: 기본값, 애노테이션을 인식해서 동작한다.

  ex) `org.example.SomeAnnotation`

- `ASSIGNABLE_TYPE`: 지정한 타입과 자식 타입을 인식해서 동작한다.

  ex) `org.example.SomeClass`

- `ASPECTJ`: AspectJ 패턴 사용

  ex) `org.example..*Service+`

- `REGEX`: 정규 표현식

  ex) `org\.example\.Default.*`

- `CUSTOM`: TypeFilter 이라는 인터페이스를 구현해서 처리

  ex) `org.example.MyTypeFilte`


**참고**: `@Component`면 충분하기 때문에, `includeFilters`를 사용할 일은 거의 없다. `excludeFilters`는 여러가지 이유로 간혹 사용할 때가 있지만 많지는 않다.

특히 최근 스프링 부트는 컴포넌트 스캔을 기본으로 제공하는데, 개인적으로는 옵션을 변경하면서 사용하기보다는 스프링의 기본 설정에 최대한 맞추어 사용하는 것을 권장하고, 선호하는 편이다

## 중복 등록과 충돌

### 자동 빈 등록 vs 자동 빈 등록

- 컴포넌트 스캔에 의해 자동으로 스프링 빈이 등록되는데, 그 이름이 같은 경우 스프링은 오류를 발생시킨다.
- `ConflictingBeanDefinitionException`예외 발생

### 수동 빈 등록 vs 자동 빈 등록

- 이 경우 수동 빈 등록이 우선권을 가진다.
  (수동 빈이 자동 빈을 오버라이딩 해버린다.)
- 하지만 이러면 개발자의 의도와 달리 잡기 힘든 버그가 날 수도 있기 때문에 최근 스프링 부트는 수동과 자동이 충돌해도 오류가 나도록 한다.

`Consider renaming one of the beans or enabling overriding by setting spring.main.allow-bean-definition-overriding=true`
