# k2 compiler

- kotlin 컴파일러가 계속 발전함에 따라 K2라는 새로운 프론트엔드가 등장했다.
    - kotlin 프론트엔드가 완전히 재작성됨
    - 하나의 통합 데이터 구조를 사용
    - 시맨틱 분석, 호출 확인, 타입 추론 등을 담당

![image](https://github.com/user-attachments/assets/0b733dba-d49b-42cc-b992-959b237c551d)

- k2 컴파일러는 새로운 아키텍처와 강화된 데이터 구조를 통해 다음 이점을 제공할 수 있다.
    - 강화된 호출 확인(call resolution)과 타입 추론
        - 더 일관되게 동작하고 코드를 잘 이해한다.
    - 새로운 언어 기능을 위한 syntactic sugar 도입 용이
        - 구문 설탕 (syntactic sugar): 읽는 사람 또는 작성하는 사람이 편하게 디자인 된 문법이라는 뜻
    - 컴파일 시간 단축
    - 향상된 IDE 성능
        - Intellij에서 K2 모드를 활성화하면 안정성과 성능이 향상된다.

> K2 컴파일러는 [Kotlin 2.0.0](https://kotlinlang.org/docs/whatsnew20.html)부터 기본적으로 활성화된다.
>

## Performance improvements

- 아래 성능 향상 결과는 Anki-Android와 Explosed 오픈 소스 프로젝트에서 테스트한 결과다.
- K2 컴파일러를 사용하면
    - 최대 94%의 컴파일 속도 향상을 제공한다.
        - Anki-Android 클린 빌드 시간 57.7초 → 29.7초
    - 초기화 단계가 최대 488% 빨라진다.
        - Anki-Android incremental build 시간 0.126초 → 0.022초
    - 분석 단계에서 최대 376% 빨라진다.
        - Anki-Android incremental build 분석 시간 0.581 → 0.122초

## Language feature improments

- Kotlin K2 컴파일러는 다음 언어 성능을 개선했다.
    - 스마트 캐스팅
    - Kotlin 멀티플랫폼

### Smart casts

- 스마트 캐스팅
    - Kotlin 컴파일러는 특정 경우에 자동으로 객체를 타입으로 형 변환할 수 있다.
    - 명시적 형변환이 필요 없는 것
- Kotlin 2.0.0에서의 K2 컴파일러는 이전보다 더 많은 시나리오에서 스마트 캐스팅을 수행한다.

- **더 넓은 스코프의 지역 변수**
    - 기존에는 `if` 문 내에서 `non-null` 임을 알았다면 `if` 조건문 내부의 지역변수는 스마트 캐스팅 되었다. (`nullable` → `non-null`)
    - 하지만 조건문 외부의 지역 변수는 조건문 내부에서 `non-null`임을 확인해도 스마트 캐스팅할 수 없었다. (`when`이나 `while` 구문도 마찬가지)
    - Kotlin 2.0.0부터는 조건문 블럭을 사용하기 전에 선언한 지역 변수를 컴파일러가 변수에 대해 수집한 모든 정보를 가지고 스마트 캐스팅을 수행한다.

```kotlin
class Cat {
    fun purr() {
        println("Purr purr")
    }
}

fun petAnimal(animal: Any) {
    val isCat = animal is Cat
    if (isCat) {
        // Kotlin 2.0.0에선 컴파일러가 isCat에 접근할 수 있기에
        // animal을 스마트 캐스팅할 수 있다.
        // 때문에 purr() 메서드를 호출할 수 있는 것
        animal.purr() // smart casting!
    }
}

fun main(){
    val kitty = Cat()
    petAnimal(kitty)
    // Purr purr
}
```

- **논리 또는 연산자를 사용한 타입 체크**
    - 객체에 대한 타입 검사를 or(`||`)로 수행하면 가장 가까운 공통 상위 유형으로 스마트 캐스팅을 수행한다.
        - 이전에는 항상 `Any` 유형으로 캐스팅 되었음

```kotlin
interface Status {
    fun signal() {}
}

interface Ok : Status
interface Postponed : Status
interface Declined : Status

fun signalCheck(signalStatus: Any) {
    if (signalStatus is Postponed || signalStatus is Declined) {
        // signalStatus 변수가 두 타입의 공통 상위 타입인 Status로 캐스팅된다.
        signalStatus.signal() // signal()을 호출할 수 있게 된다.
    }
}
```

- **인라인 함수**
    - K2 컴파일러는 인라인 함수를 다르게 처리하여 스마트 캐스팅에 안전한지 여부를 결정할 수 있다.
    - 람다 함수는 내부적으로 호출되기에 컴파일러는 람다 함수가 함수 본문 내에 포함된 변수에 대한 참조를 유출할 수 없음을 알고 있다.
    - 이 정보를 다른 컴파일러 분석과 함께 사용하여 캡쳐된 변수를 스마트 캐스팅하는 것이 안전한지 여부를 결정한다.

```kotlin
interface Processor {
    fun process()
}

inline fun inlineAction(f: () -> Unit) = f()

fun nextProcessor(): Processor? = null

fun runProcessor(): Processor? {
    var processor: Processor? = null
    inlineAction {
        // Kotlin 2.0.0에선 컴파일러가 processor가 지역 변수이고 inlineAction이
        // 인라인 함수임을 알기에 processor가 유출될 수 없음을 안다.
        // 그러므로 processor로 스마트 캐스트하는 것이 안전하다.

        if (processor != null) {
            // null이 아님을 알았다면 safe-call을 할 필요 없이
            // 스마트 캐스팅 되어 함수를 호출할 수 있다.
            processor.process()

            // 이전에는 processor?.process()로 사용해야 했다.
        }

        processor = nextProcessor()
    }

    return processor
}
```

- **함수 타입의 프로퍼티**
    - Kotlin 이전 버전에선 함수 타입 프로퍼티가 스마트 캐스팅되지 않았다.
    - 2.0.0에서 이는 개선되었다.

```kotlin
class Holder(val provider: (() -> Unit)?) {
    fun process() {
        if (provider != null) {
            // 컴파일러가 provider가 null이 아님을 안다.
            provider()

            // 이전 버전에서는 safe-call을 해야 했다.
        }
    }
}
```

- **예외 핸들링**
    - Kotlin 2.0.0에서는 예외 처리를 개선하여 스마트 캐스팅 정보를 `catch` 및 `finally` 블록에 전달할 수 있도록 했다.

```kotlin
fun testString() {
    var stringInput: String? = null
    // 값을 할당함으로써 non-null String 타입으로 스마트 캐스팅
    stringInput = ""
    try {
        // 컴파일러는 null이 아님을 안다.
        println(stringInput.length)
        // 0

        // 컴파일러는 다시 String? 타입으로 인식
        stringInput = null

        // 예외를 발생시키기
        if (2 > 1) throw Exception()
        stringInput = ""
    } catch (exception: Exception) {
        // Kotlin 2.0.0에서 stringInput이 nullable임을 알아서 safe-call 수행
        println(stringInput?.length)
        // null

        // Kotlin 1.9.20에선 safe-call이 필요하지 않다고 말할 것이다.
    }
}
```

- **증감 연산자**
    - 이전 버전에선 컴파일러가 증감 연산자 사용 후 객체 타입이 변경될 수 있다는 사실을 이해하지 못했다.
        - 참조 오류가 발생할 수 있음
    - 2.0.0에선 수정되었다.

```kotlin
interface Rho {
    operator fun inc(): Sigma = TODO()
}

interface Sigma : Rho {
    fun sigma() = Unit
}

interface Tau {
    fun tau() = Unit
}

fun main(input: Rho) {
    var unknownObject: Rho = input

    // unknownObject는 Rho와 Tau 타입이 모두 될 수 있다.
    if (unknownObject is Tau) {

        // Rho 타입으로써 inc 증가 연산자를 호출
        // Kotlin 2.0.0에선 Sigma 타입으로 스마트 캐스팅 된다.
        ++unknownObject

        // 때문에 Sigma 타입의 sigma() 함수를 성공적으로 호출한다.
        unknownObject.sigma()
        
        // In Kotlin 2.0.0에선 컴파일러기 unknownObjet를 Sigma 타입이라
        // 알고 있기에 아래 구문은 컴파일 에러가 발생한다.
        unknownObject.tau()

        // In Kotlin 1.9.20에선 컴파일 에러가 발생하지 않고 런타임에 
        // ClassCastException이 발생한다.
    }
}
```

## Kotlin Multiplatform

K2 컴파일러에는 다음과 같은 영역에서 Kotlin 멀티플랫폼 관련 개선 사항이 있다.

### 공통 소스와 플랫폼 소스 분리

- 이전에는 Kotlin 컴파일러 설계상 컴파일 시 공통 소스와 플랫폼 소스 집합을 분리할 수 없었다.
    - 때문에 공통 코드가 플랫폼 코드에 접근할 수 있어 서로 다른 동작이 발생할 수 있었음
- 또한 공통 코드 일부 컴파일러 설정 및 종속성이 플랫폼 코드로 유출되기도 했다.

```kotlin
// Commmon code
fun foo(x: Any) = println("common foo")

fun exampleFunction() {
  foo(42)
}
```

```kotlin
// Platform code
// JVM
fun foo(x: Int) = println("platform foo")

// JavaScript
// 자바스크립트 플랫폼에는 foo() 함수 오버로드가 없다.
```

- 위 예제에서는 공통 코드가 플랫폼에 따라 동작이 달라지는 것을 보여준다.
    - JVM에선 공통 코드의 `foo`를 호출하면 플랫폼 코드의 `foo`가 호출된다. (`platform foo` 출력)
    - 자바스크립트 플랫폼에선 공통 코드 foo`를` 호출하면 플랫폼 코드엔 해당 함수가 없어 공통 코드의 `foo`가 호출된다. (`common foo` 출력)
- Kotlin 2.0.0에선 공통 코드가 플랫폼 코드에 엑세스할 수 없기 때문에 두 플랫폼 모두 공통 코드의 `foo` 함수를 호출한다. (`common foo` 출력)

- 이 외에더 IntelliJ나 Android Studio 컴파일러 간 동작이 충돌하던 것들이 많이 개선되었다.

### 예측 선언과 실제 선언의 서로 다른 가시성 수준

- Kotlin 2.0.0 이전에는 멀티 플랫폼 프로젝트에서 예상 선언과 실제 선언을 사용하는 경우 가시성 수준이 같아야 했다.
- 2.0.0부터는 서로 다른 가시성 수준도 지원한다.
    - 하지만 실제 선언이 예상 선언보다 더 허용되는 경우만 가능

```kotlin
expect internal class Attribute // internal
actual class Attribute          // public
```


---

https://kotlinlang.org/docs/k2-compiler-migration-guide.html
