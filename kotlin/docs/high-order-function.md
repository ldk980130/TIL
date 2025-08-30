# Higher-order functions and lambdas

- kotlin 함수는 일급 함수로 취급된다.
    - 변수와 데이터 구조에 저장 가능
    - 상위 함수에 인수로 전달하거나 반환 가능
    - 함수가 아닌 다른 값에 대해 가능한 모든 연산을 함수에 수행 가능
- 이를 용이하게 하기 위해 kotlikn은 람다 표현식을 제공한다.

## Higher-order functions

- 고차 함수(higher-order functions)란 함수를 매개변수로 받거나 반환하는 함수이다.

```kotlin
fun <T, R> Collection<T>.fold(
    initial: R,
    combine: (acc: R, nextElement: T) -> R
): R {
    // ...
}
```

- 위 `fold` 함수를 호출하려면 함수 타입의 인스턴스를 전달해야 하는데 이 때 람다 표현식이 널리 사용된다.

```kotlin
val joinedToString = items.fold("Elements:", { acc, i -> acc + " " + i })
```

## Function types

- `(Int) → String`과 같은 함수 타입엔 특수 표기법이 존재한다.
    - 모든 함수 타입은 괄호로 묶인 파라미터 타입 목록과 반환 타입을 가진다.
        - ex) `(A, B) -> C`
        - 반환 타입이 없어도 Unit을 명시해야 한다.
    - 함수 타입엔 선택적으로 리시버 타입을 붙일 수 있다.
        - ex) `A.(B) → C`
        - 리시버가 있는 함수 리터럴은 이러한 타입에서 자주 활용된다.
    - 지연 함수(suspending functions)는 `suspend` 수식어를 통해 포함된다.
        - ex) `suspend () → Unit`

### **Instantiating a function type**

- 함수 리터럴 코드 블록.
    - 람다 표현식: `{ a, b -> a + b }`.
    - 익명 함수: `fun(s: String): Int { return s.toIntOrNull() ?: 0 }`
    - 리시버가 있는 함수 리터럴(Function literals with receiver)도 함수 타입(리시버 포함)의 값으로 사용할 수 있다.
- 이미 선언된 함수, 프로퍼티, 생성자에 대한 참조(callable reference)
    - 함수 참조: `::isOdd`, `String::toInt`
    - 프로퍼티 참조: `List::size`
    - 생성자 참조: `::Regex`
    - 바운드 참조(bound callable reference)를 사용하면 특정 인스턴스의 멤버 함수도 참조할 수 있다. 예: `foo::toString`
- 함수 타입 인터페이스를 구현한 클래스를 인스턴스화

```kotlin
class IntTransformer: (Int) -> Int {
    override operator fun invoke(x: Int): Int = TODO()
}

val intFunction: (Int) -> Int = IntTransformer()
```

- 컴파일러가 정보만 충분하다면 변수의 함수 타입을 자동으로 추론한다.
    - ex) `val a = { i: Int -> i + 1 } // (Int) -> Int로 추론`
- 리시버가 있는 함수 타입과 리시버가 없는 함수 타입 값은 서로 대입하거나 전달할 수 있다.
    - 즉, `(A, B) -> C` 타입의 값을 `A.(B) -> C` 타입 자리에 써도 되고, 그 반대도 가능하다.
    - 다만 리시버가 없는 함수 타입이 기본적으로 추론되기 때문에 [확장 함수](https://kotlinlang.org/docs/extensions.html) 참조를 사용할 때에는 타입을 명확하게 지정해야 할 때가 있다.

### **Invoking a function type instance**

- 함수 타입을 호출할 때 `invoke` 연산자 또는 바로 괄호를 통해 호출 가능하다.
    - ex) `f.invoke(x)` 또는 `f(x)`

### **Inline functions**

- 때론 고차 함수에 유연한 제어 흐름을 제공하는 [인라인 함수](https://kotlinlang.org/docs/inline-functions.html)를 사용하는 것이 유리할 수 있다.
