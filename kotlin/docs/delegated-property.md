# Delegated properties

## 코틀린의 Delegated Properties

- 코틀린엔 프로퍼티의 getter와 setter를 다른 객체에게 위임할 수 있는 강력한 기능을 제공한다.
- 반복적으로 구현해야 하는 공통 패턴을 재사용 가능한 형태로 추출할 수 있게 해준다.
- 기본적인 문법은 다음과 같다.

```kotlin
// val/var <프로퍼티명>: <타입> by <표현식>
```

- `by` 뒤의 표현식이 위임 객체(delegate)가 된다.
    - 이 객체는 프로퍼티의 `get()`, `set()`을 자신의 `getValue()`와 `setValue()` 메서드에 위임한다.
    - 위임 객체는 인터페이스를 구현할 필요는 없지만 `getValue()`와 `setValue()`(var의 경우)를 제공해야 한다.

## 표준 위임 객체

### 지연 초기화 프로퍼티 (Lazy)

- 지연 초기화 프로퍼티를 구현하는 위임 객체
- `lazy()` 함수는 람다를 받아 `Lazy<T>` 인스턴스를 반환한다.
- 첫 번째 `get()` 호출 시 람다가 실행되고 결과가 저장되어 이후에는 저장된 결과를 반환한다.

```kotlin
val lazyValue: String by lazy {
    println("계산 중...")
    "Hello"
}
```

- `LazyThreadSafetyMode.PUBLICATION`
    - 기본적으로 `lazy` 프로퍼티의 평가는 동기화 되어 하나의 스레드에서만 값이 계산되지만 모든 스레드에서 같은 값을 볼 수 있다.
    - 여러 스레드가 동시에 실행할 필요가 없다면 lazy()의 매개변수로`LazyThreadSafetyMode.PUBLICATION`를 전달하면 된다.
- `LazyThreadSafetyMode.NONE`
    - 초기화가 항상 프로퍼티를 사용하는 스레드와 동일 스레드라고 확신하는 경우 `LazyThreadSafetyMode.NONE`을 사용할 수 있다.
    - 스레드 안전 보장 및 관련 오버헤드가 발생하지 않는다.

### Observable Properties

- `Delegates.observable()`은 초기값과 변경 핸들러를 인자로 받는다.
- 프로퍼티에 값이 할당될 때마다 핸들러가 호출된다.

```kotlin
var name: String by Delegates.observable("초기값") { 
    prop, old, new -> println("$old -> $new") 
}
```

- 할당을 가로채서 거부하고 싶다면 대신 `vetoable()`을 사용할 수 있다.

### 다른 프로퍼티에 위임

- 프로퍼티는 다른 프로퍼티에 getter와 setter를 위임할 수 있다.

```kotlin
class Example {
    var newName: String = "Kotlin"
    
    @Deprecated("이 프로퍼티는 곧 삭제될 예정입니다")
    var oldName: String by ::newName
}
```

### 맵에 프로퍼티 저장

- 프로퍼티 값을 맵에 저장하는 것은 JSON 파싱 같은 동적 작업에서 자주 사용된다.

```kotlin
class User(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int by map
}

val user = User(mapOf(
    "name" to "John Doe",
    "age" to 25
))

println(user.name) // Prints "John Doe"
println(user.age)  // Prints 25
```

### 로컬 위임 프로퍼티

- 함수나 코드 블록 내에서도 로컬 변수를 위임 프로퍼티로 선언할 수 있다.

```kotlin
fun example(computeFoo: () -> Foo) {
    val memoizedFoo by lazy(computeFoo)

    if (someCondition && memoizedFoo.isValid()) { // 여기서 접근할 때 할당
        memoizedFoo.doSomething() 
    }
}
```

### 위임 객체 요구사항

- 읽기 전용 프로퍼티(`val`)의 경우 위임 객체는 다음 파라미터를 가진 `getValue()` 연산자 함수를 제공해야 한다.
  - `thisRef`: 프로퍼티 소유자와 같은 타입 또는 상위 타입
  - `property: KProperty<*>` 타입 또는 상위 타입
  - `getValue`는 같은 타입 또는 서브 타입을 반환해야 한다.
  - 변경 가능 프로퍼티(`var`)의 경우 추가로 `setValue()`를 제공해야 한다.
- 이러한 함수들은 위임 클래스의 멤버 함수 또는 확장 함수로 구현할 수 있다.
  - 모두 `operator` 키워드로 표현되어야 한다.

```kotlin
class Resource

class Owner {
    val valResource: Resource by ResourceDelegate()
}

class ResourceDelegate {
    operator fun getValue(thisRef: Owner, property: KProperty<*>): Resource {
        return Resource()
    }
}
```

- 코틀린 표준 라이브러리는 `ReadOnlyProperty`와 `ReadWriteProperty` 인터페이스를 통해 위임 객체를 더 쉽게 구현할 수 있게 해준다.

```kotlin
val readOnlyResource: Resource by resourceDelegate()
var readWriteResource: Resource by resourceDelegate()

fun resourceDelegate(): ReadWriteProperty<Any?, Resource> =
    object : ReadWriteProperty<Any?, Resource> {
        var resource = Resource()
        override fun getValue(thisRef: Any?, property: KProperty<*>) = resource
        override fun setValue(thisRef: Any?, property: KProperty<*>, value: Resource) {
            resource = value
        }
    }
```

- 코틀린의 위임된 프로퍼티는 코드 재사용성을 높이고 보일러플레이트 코드를 줄이는 강력한 도구이다.
- 지연 초기화, 관찰 가능한 변경, 맵 기반 저장 등 다양한 패턴을 간결하게 구현할 수 있다.
