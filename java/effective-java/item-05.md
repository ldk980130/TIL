# 아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

- 많은 클래스가 하나 이상의 자원에 의존한다.
- 예를 들어 맞춤법 검사기(`SpellChecker`)가 사전(`dictionary`)를 의존할 때 `SpellChecker`를 정적 유틸 클래스나 싱글턴으로 사용하면 문제가 생긴다.

```java
// 정적 유틸 클래스를 잘못 사용한 예
public class SpellChecker {
    private static final Lexicon dictionary = ...;

    private SpellChecker() {}

    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```

```java
public class SpellChecker {
    private final Lexicon dictionary = ...;

    private SpellChecker() {}
    public static SpellChecker INSTANCE = new SpellChecker(...);

    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```

- 위 두 코드의 단점
    - 다른 사전을 사용할 수 없어 유연하지 않다.
    - 테스트하기 어렵다. (의존 클래스를 mock으로 대체 불가)

## 의존 객체 주입을 사용하자

- `dictionary` 필드에서 `final`을 제거하고 다른 사전으로 교체하는 메서드(`setter`)를 추가할 수도 있지만 오류를 내기 쉽고 멀티스레드 환경에서 쓸 수 없다.
- 사용하는 자원에 따라 동작이 달라지는 클래스는 정적 유틸 클래스나 싱글턴 방식이 적합하지 않다.
- 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 의존 객체 주입 방식을 사용해야 한다.

```java
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```

- 의존성 주입 패턴의 장점
    - 의존성 주입 패턴은 불변을 보장한다.
    - 유연성과 테스트 용이성을 개선해준다.
- 의존성 주입의 단점
    - 의존성이 많아지면 코드를 어지럽게 만들기도 한다.
    - 스프링 같은 의존성 주입 프레임워크를 사용하면 이를 해소할 수 있다.
