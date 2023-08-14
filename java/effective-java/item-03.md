# 아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라
## 싱글턴이란

- 인스턴스를 오직 하나만 생성할 수 있는 클래스
- ex) 함수 같은 무상태 객체나 설계상 유일해야 하는 시스템 컴포넌트
- 싱글턴 클래스는 이를 사용하는 클라이언트를 테스트하기가 어려워진다.
    - 싱글턴 인스턴스를 mock 구현으로 대체할 수 없기 때문이다.

## 싱글턴을 만드는 방법

### private 기본 생성자와 static 멤버

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() {}
}
```

- private 생성자는 `INSTANCE`를 초기화할 때 딱 한 번 호출된다.
- 클라이언트에서 리플렉션 API(`AccessibliObject.setAccessible()`)를 통해 private 생성자를 호출하지 않는 이상 싱글턴이 보장된다.
    - 두 번째 객체가 생성되려 할 때 예외를 던지게 하면 된다.
- 이 방식의 장점
    - 이 방법은 해당 클래스가 싱글턴임이 API에 명백히 드러난다.
    - 싱글턴 코드가 간결하다.

### 정적 팩터리 메서드로 public static 멤버 제공하기

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() {}

    public static Elvis getInstance() { return INSTANCE; }
}
```

- `Elvis.getInstance()`는 항상 같은 객체 참조를 반환하므로 싱글턴이 보장된다.
    - 리플렉션을 통한 예외는 똑같이 적용된다.
- 이 방식의 장점
    - 싱글턴이 아니게 변경하고 싶을 때 변경이 유연하다.
        - `getInstance()` 내부 구현만 변경하면 된다.
    - 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
    - 정적 팩터리 메서드 참조를 supplier로 사용할 수 있다.
        - `Elvis::getInstance`를 `Supplier<Elvis>`로 사용

### 직렬화 문제

- 앞선 두 방법으로 만든 싱글턴 클래스를 직렬화하려면 `Serializable`을 구현하는 것만으로는 부족하다.
    - 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어지기 때문
- 모든 인스턴스 필드를 일시적(`transient`)라 선언하고 `readResolve()` 메서드를 제공해야 한다.

### 열거 타입 싱글턴

```java
public enum Elvis {
    INSTANCE;
}
```

- public 필드 방식과 비슷하지만 더 간결하고 추가 노력 없이 직렬화할 수 있다.
- 리플렉션 공격에서도 제 2의 객체가 생성되는 것을 막아준다.
- 단 Enum 클래스 외에 클래스를 상속해야 한다면 사용할 수 없다.
    - 인터페이스는 구현할 수 있다.
