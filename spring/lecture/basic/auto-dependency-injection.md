# 의존 관계 자동 주입

## 다양한 의존 관계 주입 방법

- **생성자 주입**
    - 생성자 호출시점에 딱 1번만 호출되는 것이 보장된다.
    - **불변, 필수** 의존관계에 사용
- **수정자 주입(setter 주입)**
    - **선택, 변경** 가능성이 있는 의존관계에 사용
    - 자바빈 프로퍼티 규약의 수정자 메서드 방식을 사용하는 방법이다.
- **필드 주입**

    ```java
    @Autowired 
    private final MemberRepository memberRepository;
    ```

    - 외부에서 변경이 불가능해서 테스트하기 힘듦
    - DI 프레임워크가 없으면 아무것도 할 수 없다.
    - 실제 코드와 관련 없는 테스트 코드에서 사용하는 경우가 많음
- **일반 메서드 주입**
    - 한번에 여러 필드를 주입 받을 수 있다.
    - 일반적으로 잘 사용하지 않는다

## 옵션 처리

주입할 스프링 빈이 없어도 동작해야 할 때가 있다.

그런데 `@Autowired`만 사용하면 `required`옵션의 기본 값이 `true`로 되어 있어서 자동 주입 대상이 없으면 오류가 발생한다.

```java
//호출 안됨
@Autowired(required = false)
public void setNoBean1(Member member) {
}

//null 호출
@Autowired
public void setNoBean2(@Nullable Member member) {
}

//Optional.empty 호출
@Autowired(required = false)
public void setNoBean3(Optional<Member> member) {
}
```

- `@Autowired(required=false)`: 자동 주입할 대상이 없으면 수정자 메서드 자체가 호출 안 됨
- `org.springframework.lang.@Nullable` : 자동 주입할 대상이 없으면 null이 입력된다. (`@Nullable Member member`)
- `Optional<>` : 자동 주입할 대상이 없으면 `Optional.empty` 가 입력된다.

> **참고**: `@Nullable`, `Optional`은 스프링 전반에 걸쳐서 지원된다. 예를 들어서 생성자 자동 주입에서 특정 필드에만 사용해도 된다.
>

## **생성자 주입을 선택해라!**

- 과거에는 수정자 주입과 필드 주입을 많이 사용했지만, 최근에는 스프링을 포함한 DI 프레임워크 대부분이 생성자 주입을 권장한다
- 순수 자바 코드로 테스트할 때 생성자 주입을 사용하면 주입 데이터를 누락 했을 때 컴파일 오류가 발생한다.
- 생성자 주입을 사용하면 필드에 final 키워드를 사용할 수 있다. 그래서 생성자에서 혹시라도 값이 설정되지 않는 오류를 컴파일 시점에 막아준다.

## 조회 빈이 2개 이상 - 문제

- 스프링 빈 조회에서 학습했듯이 타입으로 조회하면 선택된 빈이 2개 이상일 때 문제가 발생한다.
- `NoUniqueBeanDefinitionException` 오류가 발생한다.

### 해결

- `@Autowired`필드 명 매칭
    1. 타입 매칭
    2. 타입 매칭의 결과가 2개 이상일 때 필드명, 파라미터 명으로 빈 이름 매칭
- `@Qualifier` `@Qualifier`끼리 매칭 빈 이름 매칭
    - `@Qualifier`는 추가 구분자를 붙여주는 방법이다. 주입 시 추가적인 방법을 제공하는 것이지 빈 이름을 변경하는 것은 아니다
    1. `@Qualifier`끼리 매칭 (`@Qualifier("mainDiscountPolicy")`)
    2. 빈 이름 매칭
    3. `NoSuchBeanDefinitionException` 예외 발생
- `@Primary`사용
    - `@Primary`는 우선순위를 정하는 방법이다. `@Autowired`시에 여러 빈이 매칭되면 `@Primary`가 우선권을 가진다.

> 자주 사용하는 메인 스프링 빈은 `@Primary`를 적용해서 조회하는 곳에서 `@Qualifier`지정 없이 편리하게 조회하고, 서브로 사용하는 빈을 획득할 때는 `@Qualifier`를 지정해서 명시적으로 획득 하는 방식으로 사용하면 코드를 깔끔하게 유지할 수 있다. 물론 이때 메인 스프링 빈을 등록할 때 `@Qualifier`를 지정해주는 것은 상관없다.
>

> 스프링은 자동보다는 수동이, 넒은 범위의 선택권보다는 좁은 범위의 선택권이 우선 순 위가 높다. 따라서 `@Primary`보다 `@Qualifier`가 우선권이 높다.
>

**조회한 빈이 모두 필요할 때, List, Map**

```java
@Autowiredpublic 
DiscountService(Map<String, DiscountPolicy> policyMap, 
								List<DiscountPolicy> policies) {
		this.policyMap = policyMap;
		this.policies = policies;
}
```

- `Map<String, DiscountPolicy>` : map의 키에 스프링 빈의 이름을 넣어주고, 그 값으로 `DiscountPolicy` 타입으로 조회한 모든 스프링 빈을 담아준다.
- `List<DiscountPolicy>` : `DiscountPolicy`타입으로 조회한 모든 스프링 빈을 담아준다. 만약 해당하는 타입의 스프링 빈이 없으면, 빈 컬렉션이나 Map을 주입한다.

> 자동, 수동의 올바른 실무 운용 기준: 편리한 자동 기능을 기본으로 사용하자. 직접 등록하는 기술 지원 객체는 수동 등록. 다형성을 적극 활용하는 비즈니스 로직은 수동 등록을 고민해보자.
>
