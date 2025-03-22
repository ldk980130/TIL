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
    - 여러 스레드가 동시에 실행할 필요가 없다면 lazy()의 매개변수로`LazyThreadSafetyMode.PUBLICATION` 를 전달하면 된다.
- `LazyThreadSafetyMode.NONE`
    - 초기화가 항상 프로퍼티를 사용하는 스레드와 동일 스레드라고 확신하는 경우 `LazyThreadSafetyMode.NONE`을 사용할 수 있다.
    - 스레드 안전 보장 및 관련 오버헤드가 발생하지 않는다.
