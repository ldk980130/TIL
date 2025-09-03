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

## **Lambda** expressions and anonymous functions

- function literals - 선언되지 않교 표현식으로 즉시 전달되는 함수
    - 람다 표현식
    - 익명 함수

```kotlin
// max는 함수를 인자로 받기에 고차 함수
max(strings, { a, b -> a.length < b.length })
```

### Lambda expression syntax

- 람다 표현식은 항상 중괄호로 감싸진다.
- 매개변수 선언은 중괄호 안에 들어가며 선택적으로 타입을 명시한다.
- 함수 body는 `→` 뒤에 온다.
- 람다의 반환 타입이 `Unit`이 아닌 경우 람다 본문 내부의 마지막 표현식이 반환 값으로 처리된다.

```kotlin
val sum = { x: Int, y: Int -> x + y }
```

### Passing trailing lambdas

- trailing lambda
    - 함수 마지막 매개변수가 람다인 경우 람다 식이 괄호 밖에 배치되는데 이러한 구문을 뜻한다.
    - 람다가 해당 호출의 유일한 인수인 경우 괄호는 완전히 생략할 수 있다.

```kotlin
run { println("...") }
```

### it: **implicit** name of a single parameter

- 람다 표현식에서 매개변수가 하나만 있는 경우 `→`를 생략하여 암시적으로 `it`으로 매개변수를 표현할 수 있다.

```kotlin
ints.filter { it > 0 } // '(it: Int) -> Boolean'
```

### **Returning a value from a lambda expression**

- 람다에서 qualified return 구문을 통해 명시적으로 값을 반환할 수 있다.
- 그렇지 않으면 마지막 표현식 값이 암시저기으로 반환된다.

```kotlin
// 아래 두 표현은 같다
ints.filter {
    val shouldFilter = it > 0
    shouldFilter
}

ints.filter {
    val shouldFilter = it > 0
    return@filter shouldFilter
}
```

### **Underscore for unused variables**

- 사용하지 않는 매개변수는 아래와 같이 생략 가능하다.

```kotlin
map.forEach { (_, value) -> println("$value!") }
```

### **Destructuring in lambdas**

- 구조 분해 선언(Destructuring Declation)
    - 하나의 객체에서 여러 변수를 한 번에 선언하는 문법
    - ex) `val (name, age) = person`
- 람다 매개변수가 `Pair`나 `Map.Entry` 등일 때 이러한 구조 분해를 이용할 수 있다.

```kotlin
map.mapValues { (key, value) -> "$value!" }
```

### Anonymous functions

- 함수의 반환 타입을 지정할 때 명시적으로 지정해야 하는 경우 익명 함수를 사용할 수 있다.
  - 익명 함수는 표현식 또는 블록일 수 있다.

```kotlin
fun(x: Int, y: Int): Int = x + y

fun(x: Int, y: Int): Int {
    return x + y
}
```

- 람다 식과 익명 함수의 또 다른 차이점은 비지역 반환(Non-local return)의 동작 방식에 있다.
  - 람다 식 내부에서의 `return`은 람다를 감싸고 있는 외부 함수를 종료시킨다.
  - 반면, 익명 함수 내부에서의 `return`은 해당 익명 함수만을 종료시키고 외부 함수에는 영향을 주지 않는다.

```kotlin
// 람다식에서의 non-local return 예시
fun lambdaTest() {
    listOf(1, 2, 3).forEach {
        if (it == 2) return  // lambdaTest 함수 전체를 종료시킴 (non-local return)
        println(it)
    }
    println("이 문장은 출력되지 않음")
}

// 익명 함수에서의 return 예시
fun anonymousFuncTest() {
    listOf(1, 2, 3).forEach(fun(value) {
        if (value == 2) return  // 익명 함수만 종료, forEach는 계속됨 (local return)
        println(value)
    })
    println("이 문장은 정상적으로 출력됨")
} 
```

### **Closures**

- 클로저(Closure)란
  - 자신을 둘러싼 외부 스코프의 변수 를 포착하는 함수를 의미
- 람다와 익명 함수가 바로 외부 스코프에 선언된 변수를 포착할 수 있는데 이 때 클로저가 된다.

```kotlin
var sum = 0
ints.filter { it > 0 }.forEach {
    sum += it // 외부 sum 변수를 캡쳐
}
print(sum)
```

### **Function literals with receiver**

> 함수 리터럴이란 이름 없는 함수 표현 자체를 의미 (람다, 익명 함수)
>

- 리시버가 있는 함수 타입은 특수한 형태의 함수 리터럴로 인스턴스화할 수 있다.
  - ex) `A.(B) → C`
- 리시버 객체 (receiver object)
  - 리시버가 있는 함수 리터럴은 람다에 리시버를 부여할 수 있게 하는 문법이다.
  - 람다 안에서 리시버의 멤버를 `this` 참조로 사용할 수 있게 되는 것
  - 이 동작은 함수 본문 내에서 리시버 멤버에 엑세스 할 수 있는 확장 함수 동작과 비슷하다.

```kotlin
val sum: Int.(Int) -> Int = { other -> plus(other) } // this.plus(other)

val sum = fun Int.(other: Int): Int = this + other // 익명 함수로 표현하면 이와 같다.
```

- 리시버가 있는 함수 리터럴은 type-safe builder에서 자주 쓰인다.

```kotlin
class HTML {
    fun body() { ... }
}

fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()  // 리시버 객체 생성
    html.init() // 파라미터로 전달 받은 Html을 리시버로 가지는 () -> Unit 호출
    return html
}

html { // 리시버가 있는 람다 시작
    body() // html 함수의 init의 매개변수로 body 전달 (Html 클래스의 () -> Unit 타입 멤버 함수)
}
```
