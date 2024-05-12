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
