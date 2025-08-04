# Scope function

## 코틀린 스코프 함수 (Scope Functions)

- 코틀린의 스코프 함수(scope functions)는 객체의 컨텍스트 내에서 코드 블록을 실행하는 함수이다.
    - 대표적으로 `let`, `run`, `with`, `apply`, `also` 함수가 있다.
- 스코프 함수를 사용하는 이유는 다음과 같다.
    - 객체의 프로퍼티에 접근하여 초기화하거나 조작할 때 반복되는 객체 이름 작성 줄이기
    - 임시로 코드를 한정하는 블록을 만들기 위해
    - 안전한 호출과 null 체크를 간결하게 처리하기 위해
    - 어떤 결과를 만들어내기 위해 객체와 람다를 조합
- 스코프 함수는 새로운 기능이 있는건 아니지만 코드를 더 간결하고 가독성 있게 만들 수 있다.

## 주요 스코프 함수

| Function | Object reference | Return value | Is extension function |
| --- | --- | --- | --- |
| [`let`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/let.html) | `it` | 람다 결과 | O |
| [`run`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/run.html) | `this` | 람다 결과 | O |
| [`run`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/run.html) | - | 람다 결과 | No: called without the context object |
| [`with`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/with.html) | `this` | 람다 결과 | No: takes the context object as an argument. |
| [`apply`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/apply.html) | `this` | Context object | O |
| [`also`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/also.html) | `it` | Context object | O |

- 다음은 각 스코프 함수 사용에 대한 간단한 가이드이다.
    - 람다를 널이 아닌 객체에 실행할 때는 `let`
    - 표현식을 지역 변수로 도입할 때도 `let`
    - 객체를 구성할 때는 `apply`
    - 객체를 구성하고 결과를 계산할 때는 `run`
    - 표현식이 필요한 곳에서 문장을 실행할 때는 non-extension `run`
    - 추가 효과를 줄 때는 `also`
    - 객체 위에서 함수 호출을 그룹화할 때는 `with`
- 스코프 함수 사용 사례가 겹쳐서 어떤 함수를 쓸지는 프로젝트나 팀의 컨벤션에 따라 결정한다.
- 스코프 함수는 코드를 간결하게 만들지만, 과도하게 쓰면 가독성이 떨어지고 오류를 일으킬 수 있다.
- 스코프 함수를 중첩하거나 체이닝할 때는 `this`나 `it`의 값이 헷갈릴 수 있어서 조심해야 한다.

## Distinctions

- 각 스코프 함수는 두 가지 주요 차이점이 존재한다.
  - 컨텍스트 객체 접근 방식 (`this` 또는 `it`)
  - Return Value

### 컨텍스트 접근 방식

- `this`
  - `run`, `with`, `apply`는 수신 객체로 context object를 참조한다.
  - 람다 내부에서 해당 객체 멤버에 접근 가능하므로 코드가 짧아진다.
  - `this`를 생략 가능하지만 외부 객체와 혼동될 수 있어 객체 멤버에 집중된 작업에 적합하다.

```kotlin
val adam = Person("Adam").apply { 
    age = 20 
    city = "London"
}
```

- `it`
  - `let`, `also`는 람다 인자로 context object를 참조하며 it으로 접근한다.
  - 객체 멤버를 사용할 땐 항상 `it.`을 명시해야 한다.
  - 함수 호출 시 인자로 주로 사용하거나 람다 블록 내에 여러 변수를 사용할 때 적합하다.

```kotlin
fun getRandomInt(): Int {
    return Random.nextInt(100).also {
        writeToLog("getRandomInt() generated value $it")
    }
}
```

### Return value

- Context Object 반환
  - `apply`, `also`는 객체 자기 자신을 반환한다.

```kotlin
// 반환된 객체를 이어서 체이닝 할 수 있다.
val numberList = mutableListOf<Double>()
numberList.also { println("Populating the list") }
    .apply {
        add(2.71)
        add(3.14)
        add(1.0)
    }
    .also { println("Sorting the list") }
    .sort()
    
// 함수에서 해당 객체를 직접 반환할 때 유용하다.
fun getRandomInt(): Int {
    return Random.nextInt(100).also {
        writeToLog("getRandomInt() generated value $it")
    }
}
```

- Lambda 결과 반환
  - `let`, `run`, `with`는 람다 블록의 결과 값을 반환한다.
  - 결과를 변수로 할당하거나 결과를 바탕으로 연산을 이어갈 때 적합

```kotlin
val numbers = mutableListOf("one", "two", "three")

// 결과를 변수로 할당하거나 결과를 바탕으로 연산을 이어갈 때 적합
val countEndsWithE = numbers.run { 
    add("four")
    add("five")
    count { it.endsWith("e") }
}

// 반환값이 필요 없다면 단순 임시 스코프를 만들기 위해 사용할 수도 있다.
with(numbers) {
    val firstItem = first()
    val lastItem = last()        
    println("First item: $firstItem, last item: $lastItem")
}
```

## Functions

### let

- Context Object: 람다 인자 (`it`)
- 반환 값: 람다 결과
- 용도
  - 호출 체인 결과에 여러 동작을 적용할 때
  - null 체크 후 안전하게 작업할 때
  - 임시로 변수 범위를 제한하고 싶을 때

```kotlin
val numbers = mutableListOf("one", "two", "three", "four", "five")

// 호출 체인의 결과에 여러 동작을 수행할 때
val resultList = numbers.map { it.length }.filter { it > 3 }

// null 체크 후 안전하게 사용
val firstOrNull = numbers.firstOrNull()?.let {
    println("첫 번째 원소: $it")
}

// 임시 변수 범위 제한
val result = numbers.map { it.length }.filter { it > 3 }.let { lengths ->
    lengths.sum()
}
```

### with

- Context Object:람다 수신자 (`this`)
- 반환 값:람다 결과
- 용도:
  - 객체의 여러 멤버를 연속적으로 사용할 때
  - 반환값이 필요하지 않을 때

```kotlin
val builder = StringBuilder()
with(builder) {
    append("Hello, ")
    append("Kotlin!")
    toString()
}
```

### run

- Context Object:람다 수신자 (`this`)
- 반환 값:람다 결과
- 용도:
  - 객체 초기화와 연산을 동시에 할 때
  - 객체를 확장함수로 바로 사용하고 싶을 때

```kotlin
val message = StringBuilder().run {
    append("Kotlin Scope Functions. ")
    append("run 예제")
    toString()
}

```

### apply

- Context Object:람다 수신자 (`this`)
- 반환 값:객체 자신
- 용도:
  - 객체 설정(초기화) 및 연쇄적 설정에 적합
  - 여러 프로퍼티를 설정한 뒤 객체를 그대로 반환하고 싶을 때

```kotlin
val person = Person().apply {
    name = "홍길동"
    age = 30
}
```

### also

- Context Object:람다 인자 (`it`)
- 반환 값:객체 자신
- 용도:
  - 부수 효과(디버깅·로깅 등)를 추가하고 싶을 때
  - 체인을 유지하면서 작업을 삽입할 때

```kotlin
val numbers = mutableListOf("one", "two", "three")
numbers
    .also { println("초기 값: $it") }
    .add("four")
```

### takeIf / takeUnless

- Context Object:람다 인자 (`it`)
- 반환 값:객체 자신 또는 null
- 용도:
  - 조건을 만족할 때만 객체를 반환하거나, 그렇지 않으면 null 반환
  - 조건부로 체인을 이어가고 싶을 때

```kotlin
val number = 10
val even = number.takeIf { it % 2 == 0 }   // 10
val odd = number.takeUnless { it % 2 == 0 } // null
```
