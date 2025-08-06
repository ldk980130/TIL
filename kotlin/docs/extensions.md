# Extensions

- kotlin에선 상속이나 데코레이터 없이 클래스나 인터페이스를 확장할 수 있는 기능을 제공한다.
    - 타사 라이브러리 클래스나 인터페이스에 새 함수 정의 가능
    - 마치 클래스의 멤버 함수처럼 사용할 수 있다.

## Extension functions

- 확장 함수 정의 시 수신 객체 타입(receiver type)을 지정하여 이 타입이 함수가 확장되는 대상이 된다.

```kotlin
// MutableList<Int>가 수신 객체 타입이 된다.

fun <T> MutableList<T>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // 'this' corresponds to the list
    this[index1] = this[index2]
    this[index2] = tmp
}
```

## Extensions are resolved statically

- 확장 함수는 실제 클래스를 수정하지 않는다.
- 확장 함수는 정적으로 디스패치된다.
    - 런타임이 아닌 컴파일 타임에 확장 함수의 호출 여부가 결정된다.
    - 때문에 컴파일 시점에 수신자 타입에 따라 어떤 함수가 호출될지 알 수 있다.
- 아래 예제 코드를 살펴보자
    - 다형성에 의하면 구현체 타입인 Rectangle 구현에 따라 “Rectangle”이 호출되었을 것이다.
    - 하지만 확장 함수는 선언 타입에 바인딩되어 실행되기에 “Shape”가 호출된 것

```kotlin
open class Shape
class Rectangle : Shape()

fun Shape.getName() = "Shape"
fun Rectangle.getName() = "Rectangle"

fun main() {
	val shape: Shape = Rectangle()
	println(shape.getName()) // 결과는 "Shape"
}
```

- 멤버 함수와 확장 함수 중 같은 이름이 있다면 멤버 함수가 우선 호출된다.

```kotlin
class Shape {
    fun getName() = "멤버 함수 Shape"
}

fun Shape.getName() = "확장 함수 Shape"

val s = Shape()
println(s.getName()) // 결과는 "멤버 함수 Shape"
```

- 하지만 멤버 함수의 메서드를 오버로딩하는 것은 가능하다.

```kotlin
class Example {
    fun printFunctionType() { println("Class method") }
}

fun Example.printFunctionType(i: Int) { println("Extension function #$i") }

Example().printFunctionType(1) // "Extension function 1"
```

## Nullable receiver

- nullable 수신 객체 타입으로 확장 함수를 만들 수 있다.
    - null인 경우에도 객체 변수에서 확장 함수를 호출 가능하다.
- 컴파일 오류 방지를 위해 함수 본문에서 null 체킹을 하는 것이 좋다.

```kotlin
fun Any?.toString(): String {
    if (this == null) return "null"
    return toString()
}
```

## Extension properties

- 확장 프로퍼티도 지원한다.

```kotlin
val <T> List<T>.lastIndex: Int
    get() = size - 1
```

- 확장 프로퍼티는 실제로 클래스 멤버가 아니기에 백킹 필드를 가질 수 없다.
    - 때문에 확장 프로퍼티엔 초기화가 허용되지 않는다.
    - 확장 프로퍼티는 오직 명시적인 getter/setter를 제공해야만 정의할 수 있다.

## Companion object extensions

- Companion 객체에도 확장 함수 및 프로퍼티를 정의할 수 있다.
- 클래스 이름을 한정자로만 호출할 수 있다.

```kotlin
class MyClass {
    companion object { }
}

fun MyClass.Companion.printCompanion() { println("companion") }

fun main() {
    MyClass.printCompanion()
}
```

## Declaring extensions as members

- 클래스 내부에서 다른 클래스의 확장 함수를 선언할 수 있다.
- 멤버 확장 함수는 클래스 내부에서만 사용할 수 있다.

```kotlin
class Host(val hostname: String) {
    fun printHostname() { print(hostname) }
}

class Connection(val host: Host, val port: Int) {
    fun printPort() { print(port) }

    fun Host.printConnectionString() {
        printHostname() // Host의 멤버 호출
        print(":")
        printPort()     // Connection의 멤버 호출
    }

    fun connect() {
        host.printConnectionString()
    }
}

fun main() {
    Connection(Host("kotl.in"), 443).connect()
    // 결과: kotl.in:443
    // Host("kotl.in").printConnectionString(443) // 오류: 확장 함수는 Connection 안에서만 호출 가능
}

```

- 멤버 확장 함수 내부에선 한정자 없이 접근 가능한 여러 암시적 리시버가 존재하게 된다.
  - dispatch receiver - 확장 함수가 선언된 클래스의 인스턴스
  - extension receiver - 확장 함수의 리시버 타입
- 만약 두 리시버에 동일한 이름의 멤버가 있다면 extension receiver의 멤버가 우선 호출된다.
  - `this@{className}` 형태로 디스패치 리시버 멤버에 접근할 수는 있다.

```kotlin
class Connection {
    fun Host.getConnectionString() {
        toString()         // Host.toString() 호출
        this@Connection.toString()  // Connection.toString() 호출
    }
}
```

- 멤버 확장 함수를 open으로 선언하여 서브 클래스에서 오버라이드할 수도 있다.
  - 디스패치 리시버 타입에 대해선 가상 디스패치(런타임 결정)가 일어난다.
  - 확장 리시버 타입에 대해선 정적 대스패치(컴파일 타임 결정)가 일어난다.

```kotlin
open class Base
class Derived : Base()

open class BaseCaller {
    open fun Base.printFunctionInfo() {
        println("Base extension function in BaseCaller")
    }

    open fun Derived.printFunctionInfo() {
        println("Derived extension function in BaseCaller")
    }

    fun call(b: Base) {
        b.printFunctionInfo()
    }
}

class DerivedCaller : BaseCaller() {
    override fun Base.printFunctionInfo() {
        println("Base extension function in DerivedCaller")
    }

    override fun Derived.printFunctionInfo() {
        println("Derived extension function in DerivedCaller")
    }
}

fun main() {
    BaseCaller().call(Base()) // "Base extension function in BaseCaller"
    DerivedCaller().call(Base()) // "Base extension function in DerivedCaller"
    DerivedCaller().call(Derived()) // "Base extension function in DerivedCaller" (여기가 포인트)
}

```

- 위 예제에서 `BaseCaller`와 `DerivedCaller`는 디스패치 리시버, `Base`, `Derived`는 확장 리시버이다.
  - `DerivedCaller().call(Derived())`에서 호출되는 함수는`DerivedCaller`에서 오버라이드한`Base.printFunctionInfo()`이다. ("Base extension function in DerivedCaller")
  - 실제 객체가`Derived`가 전달되었지만 `call(b: Base)`에서`b`의 타입이`Base`로 정해져 있으므로, 확장 리시버 타입(`Base`) 기준으로만 확장 함수가 호출된다.
  - 즉 디스패치 리시버(`DerivedCaller`)는 virtual하게 선택되지만, 확장 리시버(`Base`/`Derived`)는 static하게 선택된다.

## Note on visibility

- 확장 함수/프로퍼티는 일반 함수/프로퍼티와 동일한 가시성 제한자를 사용한다.
  - 파일 최상위에 선언된 확장 함수는 같은 파일 내의 다른 private top-level 선언에 접근 가능하다.
- 하지만 확장 함수가 대상 타입 외부에 선언되어 있다면 그 클래스의 private/protected 멤버에는 접근할 수 없다.
- 즉 결국은 확장 함수는 외부 함수로 동작하기에 내부 구현을 숨기거나 캡슐화를 깰 수 없다.
