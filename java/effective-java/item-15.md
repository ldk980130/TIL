# 아이템 15 클래스와 멤버의 접근 권한을 최소화하라

## 캡슐화

- 잘 설계된 컴포넌트는 클래스 내부 데이터와 내부 구현 정보를 외부 컴포넌트에게 드러내지 않는다.
- 오직 API를 통해서만 다른 컴포넌트와 소통한다.
- 소프트웨어 설계의 근간이 되는 원리
- 컴포넌트를 서로 독립시키면 얻을 수 있는 이점이 많다.

### 정보 은닉의 장점

- 시스템 개발 속도를 높인다.
    - 여러 컴포넌트 병렬 개발 가능
- 시스템 관리 비용을 낮춘다.
    - 각 캄포넌트를 빨리 파악 가능
    - 컴포넌트 교체 비용도 적음
- 성능 최적화에 도움이 된다.
    - 다른 컴포넌트에 영향을 주지 않고 해당 컴포넌트만 최적화 가능
- 소프트웨어 재사용성을 높인다.
    - 외부에 의존하지 않고 독자적으로 동작하는 컴포넌트라면 낯선 환경에서도 유용하게 쓰일 가능성이 크다.
- 큰 시스템을 제작하는 난이도를 낮춰준다.
    - 시스템 전체가 완성되지 않더라도 개별 컴포넌트의 동작을 검증할 수 있다.

## 접근 제어

- 기본 원칙은 모든 클래스와 멤버의 접근성을 가능한 좁혀야 한다.
- 자바는 정보 은닉을 위한 다양한 장치를 제공한다.

### 톱레벨 클래스와 인터페이스

- 톱레벨 클래스와 인터페이스에는 `package-private`과 `public` 접근 제한자를 선언할 수 있다.
    - `package-private`은 해당 패키지 내에서만 사용 가능
    - 패키지 외부에서 쓸 일이 없다면 `package-private`를 사용하자
- 한 클래스에서만 사용하는 `package-private` 톱레벨 클래스나 인터페이스는 이를 사용하는 클래스 안에 `private static`으로 중첩시켜보자.

### 멤버 변수

- 클래스의 공개 API를 세심히 설계한 후 그 외의 모든 멤버는 `private`으로 만들자.
    - `final`이 아닌 인스턴스 필드를 `public`으로 선언하면 필드 값을 제한할 수 없게 된다.
    - 즉 불변식을 보장할 수 없고 스레드 안전하지 않게 된다.
    - `final`을 붙이더라도 내부 구현을 바꾸는 리팩터링을 할 때 필드가 외부 컴포넌트에 노출된 코드가 있다면 코드 변경이 어려어진다.
- 같은 패키지 내 다른 클래스가 접근해야 한다면 `package-private`
    - `private`, `package-private`은 공개 API에 영향을 주지 않는다.
- 상위 클래스의 메서드를 재정의할 때는 상위 클래스에서보다 접근 수준을 좁게 설정할 수 없다.
    - 리스코프 치환 원칙을 지키기 위해
    - 상위 클래스의 인스턴스는 하위 클래스 인스턴스로 대체할 수 있어야하기에 어기면 컴파일 에러가 발생한다.

### 테스트와 접근 제어

- 코드를 테스트하려는 목적으로 클래스, 인터페이스, 멤버 접근 범위를 넓히려 할 때가 있다.
- `public` 클래스의 `private` 멤버를 `package-private`까지 푸는 것은 괜찮지만 그 이상은 안 된다.

### 배열과 접근 제어

- 배열을 `public`으로 하거나 그대로 반환하는 접근자 메서드를 제공해서는 안 된다.
    - 길이가 0이 아닌 배열은 모두 변경 가능하기 때문
    - 외부에서 배열을 수정할 우려가 생긴다.
- `public` 배열을 `private`으로 만들고 불변 리스트를 추가하는 방법이 있다.

    ```java
    rivate static final Thing[] PRIVATE_VALUES = {...};
    
    public static final Thing[] values() {
        return Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
    }
    ```

- 반환할 때 배열을 복사하는 방법도 있다. (방어적 복사)

    ```java
    private static final Thing[] PRIVATE_VALUES = {...};
    
    public static final Thing[] values() {
        return PRIVATE_VALUES.clone();
    }
    ```


## 모듈과 접근 제어

- 자바 9에서 모듈 시스템이 도입되어 두 가지 암묵적 접근 수준이 추가 되었다.
- 모듈은 패키지들의 묶음이다.
- 모듈은 자신의 패키지 중 공개할 것들을 선언할 수 있다.
    - `public`, `protected`여도 공개하지 않았다면 외부 모듈에서 접근할 수 없다.
- 두 가지 암묵적 접근 수준이란 외부 모듈에서는 사용할 수 없는 `public`과 `protected`이다.
- 모듈의 JAR파일을 자신의 모듈 경로가 아닌 애플리케이션 클래스패스에 두면  문제가 생긴다.
    - 그 모듈 안의 모든 패키지는 마치 모듈이 없는 것처럼 행동하여 모듈 공개 여부와 상관없이 `public` 클래스는 어디서든 접근 가능해진다.
