# Classes (constructor)

https://kotlinlang.org/docs/classes.html

- 코틀린에서 클래스는 `class` 키워드로 선언할 수 있다.

```kotlin
class Person { /*...*/ }
```

- 클래스 선언은 이름, 헤더, 중괄호로 둘러싼 바디로 구성된다.
    - 헤더, 본문은 선택 사항이며 바디가 없으면 중괄호도 생략할 수 있다.

```kotlin
class Empty
```

## Constructors

- 코틀린은 주 생성자(primary constructor)와 하나 이상의 부 생성자(secondary constructors)를 가질 수 있다.
    - 주 생성자는 클래스 헤더에 선언한다.

```kotlin
class Person(firstName: String)
```

- 주 생성자는 클래스 헤더에서 클래스 인스턴스와 해당 프로퍼티를 초기화한다.
    - 클래스 헤더엔 실행 가능한 코드를 포함할 수 없다.
- `init` block을 사용하면 객체 생성 동안 일부 코드를 실행시킬 수 있다.

```kotlin
class InitOrderDemo(name: String) {
    val firstProperty = "First property: $name".also(::println)
    
    init {
        println("First initializer block that prints $name")
    }
    
    val secondProperty = "Second property: ${name.length}".also(::println)
    
    init {
        println("Second initializer block that prints ${name.length}")
    }
}
```

- 주 생성자의 파라미터는 `init` block에서 사용할 수 있다.
    - 클래스 바디 선언된 프로퍼티 초기화에도 사용 가능

```kotlin
class Customer(name: String) {
    val customerKey = name.uppercase()
}
```

- 생성자에서 기본값을 프로퍼티에 설정할 수 있다.

```kotlin
class Person(val firstName: String, val lastName: String, var isEmployed: Boolean = true)
```

- 클래스 속성을 선언할 때 trailing comma를 사용할 수 있다.
    - 변경된 값에만 초점을 맞추기에 버전 관리 Diff를 깔끔하게 한다.
    - 요소를 쉽게 추가하고 순서를 변경 가능

```kotlin
class Person(
    val firstName: String,
    val lastName: String,
    var age: Int, // trailing comma
) { /*...*/ }
```

- 생성자에 어노테이션이나 접근 제어자가 있는 경우 `constructor` 키워드는 필수이다.

```kotlin
class Customer public @Inject constructor(name: String) { /*...*/ }
```

## Secondary constructors

- 클래스는 `constructor` 키워드가 필수인 부 생성자를 선언할 수도 있다.

```kotlin
class Person(val pets: MutableList<Pet> = mutableListOf())

class Pet {
    constructor(owner: Person) {
        owner.pets.add(this) // adds this pet to the list of its owner's pets
    }
}
```

- 클래스에 주 생성자가 있는 경우 각 부 생성자는 직접 또는 다른 부 생성자를 통해 간접적으로 주 생성자에게 위임해야 한다.

```kotlin
class Person(val name: String) {
    val children: MutableList<Person> = mutableListOf()
    
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}
```

- `init` block 코드는 사실상 주 생성자의 일부가 된다.
    - 주 생성자에 대한 위임은 부 생성자의 첫 번째 문에 접근하는 순간 발생
    - 때문에 모든 `init` block과 프로퍼티 초기화는 부 생성자 바디보다 먼저 실행 된다.
- 주 생성자가 없더라도 위임은 묵시적으로 실행되기에 `init` block은 호출된다.

```kotlin
class Constructors {
    init {
        println("Init block") // 먼저 실행
    }

    constructor(i: Int) {
        println("Constructor $i") // init block 후에 실행
    }
}
```

- 추상 클래스가 아닌 클래스가 주 생성자나 부 생성자를 선언하지 않는다면 인수가 없는 `public` 기본 생성자가 생성된다.
    - 기본 생성자를 원하지 않는다면 접근 제어자 + `constructor` 키워드를 사용해라

```kotlin
class DontCreateMe private constructor() { /*...*/ }
```

> JVM에선 모든 주 생성자의 매개변수에 기본값이 설정되어 있으면 컴파일러는 매개 변수 없는 기본 생성자를 추가로 생성한다.
>

```kotlin
class Customer(val customerName: String = "")
```
