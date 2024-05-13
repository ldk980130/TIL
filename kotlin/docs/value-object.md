# Inline value classes

https://kotlinlang.org/docs/inline-classes.html

- 값을 클래스로 래핑하여 도메인에 특화되도록 만드는 것이 유용할 때가 많다.
- 하지만 추가적인 힙 할당으로 런타임 오버헤드가 발생할 수 있다.
    - 원시 타입의 경우 런타임에 크게 최적화될 수 있지만 래퍼는 특별한 처리를 받을 수 없다.
- 이를 해결하기 위해 Kotlin에서 inline class를 도입했다.
    - 값 기반 클래스의 하위 집합으로 ID가 없고 값만 보유할 수 있다.

- inline class를 선언하기 위해선 다음 두 가지가 필요하다.
    - `value` 키워드
    - `JvmInline` 어노테이션

```kotlin
@JvmInline
value class Password(private val s: String)
```

## Members

- 인라인 클래스는 일반 클래스의 일부 기능을 지원한다.
    - 프로퍼티와 함수 선언 가능
    - 초기화 블록과 보조 생성자 선언 가능

```kotlin
@JvmInline
value class Person(private val fullName: String) {
    init {
        require(fullName.isNotEmpty())
    }

    constructor(firstName: String, lastName: String) : this("$firstName $lastName") {
        require(lastName.isNotBlank())
    }

    // 인라인 클래스의 프로퍼티 getter 및 함수는 static 함수로 호출된다.
    
    val length: Int
        get() = fullName.length

    fun greet() {
        println("Hello, $fullName")
    }
}
```

- 인라인 클래스에 없는 것
    - 프로퍼티가 백킹 필드를 가질 수 없다.
    - `lateinit` 불가
    - delegated properties 없음

## Inheritance

- 인라인 클래스가 인터페이스를 구현하는 것은 가능하다.

```kotlin
@JvmInline
value class Name(val s: String) : Printable {
    override fun prettyPrint(): String = "Let's $s!"
}
```

- 하지만 다른 일반 클래스 상속 계층에 참가하는 것은 불가능하다.
    - 항상 `final` 클래스이다.

## Representation

- 코드에서 코틀린 컴파일러는 인라인 클래스에 대한 래퍼를 유지한다.
  - 런타임에선 래퍼 또는 원시 타입으로 표현될 수 있다. (`int`와 `Integer`처럼)
- 코틀린 컴파일러는 코드 최적화를 위해 래퍼 대신 원시 타입을 사용하는 것을 선호한다.
  - 하지만 때로는 래퍼를 유지해야할 수도 있다.

```kotlin
interface I

@JvmInline
value class Foo(val i: Int) : I

fun asInline(f: Foo) {}
fun <T> asGeneric(x: T) {}
fun asInterface(i: I) {}
fun asNullable(i: Foo?) {}

fun <T> id(x: T): T = x

fun main() {
    val f = Foo(42)

    asInline(f)    // unboxed: Foo 자체를 사용
    asGeneric(f)   // boxed: T 타입의 제네릭으로 사용
    asInterface(f) // boxed: 인터페이스 I 타입으로 사용
    asNullable(f)  // boxed: nullable 타입으로 사용, Foo와 차이는?

    // 아래 f는 id 함수의 매개변수에 패싱될 땐 래퍼 타입 이지만
    // 함수 내부에서 리턴될 땐 원시 타입으로 unboxing된다.
    val c = id(f)
}
```

- 인라인 클래스는 원시 타입 또는 래퍼로 표현될 수 있기에 referential equality가 의미가 없어 금지된다.
  - `a === b`
- 인라인 클래스는 제네릭 타입의 파라미터를 가질 수 있다.
  - 이 경우 컴파일러는 이를 `Any?` 또는 타입 파라미터의 상한에 매핑한다.

```kotlin
@JvmInline
value class UserId<T>(val value: T)

fun compute(s: UserId<String>) // fun compute-<hashcode>(s: Any?)
```

### Mangling

- 인라인 클래스는 기본 타입으로 컴파일되기에 예기치 않은 충돌이나 애매한 오류가 발생할 수 있다.

```kotlin
@JvmInline
value class UInt(val x: Int)

// JVM에서 'public final void compute(int x)'로 표현된다
fun compute(x: Int) { }

// 아래도 마찬가지로 JVM에서 'public final void compute(int x)'로 표현된다.
fun compute(x: UInt) { }
```

- 이 문제를 완화하기 위해 인라인 클래스의 함수 이름에는 안정된 해시 코드를 추가하게 된다.
  - `compute-<해시코드>(int x)`

### Calling from Java code

- Java 코드에서 인라인 클래스를 사용하는 함수를 호출할 수 있다.
- 그러기 위해선 직접 맹글링을 비활성화 해야 한다.

```kotlin
@JvmInline
value class UInt(val x: Int)

fun compute(x: Int) { }

@JvmName("computeUInt") // 맹글링 비활성화
fun compute(x: UInt) { }
```

## Inline classes vs type aliases

- 인라인 클래스는 type aliases와 유사해 보인다.
  - 둘 다 새로운 타입을 도입하는 것처럼 보인다.
  - 둘 다 런타임에 기본 유형으로 표현된다.
- 중요한 차이점은 type aliases는 기본 유형과 할당 호환이 가능하지만 인라인 클래스는 그렇지 않다.
- 즉 인라인 클래스는 별칭과 달리 완전히 새로운 타입을 도입하는 것이다.

## Inline classes and delegation

- 인터페이스에서 인라인 클래스의 인라인 값에 대한 위임 구현이 허용된다.

```kotlin
interface MyInterface {
    fun bar()
    fun foo() = "foo"
}

@JvmInline
value class MyInterfaceWrapper(
  val myInterface: MyInterface
) : MyInterface by myInterface

fun main() {
    val myInterfaceImpl = Object : MyInterface {
      override fun bar() { }
    }
    
    val my = MyInterfaceWrapper(myInterfaceImpl)
    
    println(my.foo()) // prints "foo"
}
```
