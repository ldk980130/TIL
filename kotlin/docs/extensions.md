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
