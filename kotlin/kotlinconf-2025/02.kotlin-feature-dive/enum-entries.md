# EnumEntries

## values의 문제점

- 코틀린에서 Enum을 사용하면서 정의된 상수들의 목록을 가져오는 코드를 작성해 본 경험이 있을 것이다.

```kotlin
enum class ShowCategory {
	SliceOfLife,
	Isekai,
	Mecha,
	Sports,
	Shonen,
}
```

- 자바에 익숙하다면 열거형 목록을 가져오기 위해 `values()`를 사용하려할 것이다.
    - 하지만 IDE의 경고를 확인할 수 있다.

> 'Enum.values()' is recommended to be replaced by 'Enum.entries' since 1.9
>

- `values`가 더 이상 권장되지 않는 이유는 다음과 같다.
    - `values()`는 상수들의 배열(을 반환한다. 배열은 원칙적으로 내부 값이 변경될 수 있는 자료구조이기 때문에, 호출할 때마다 새로운 배열이 할당되고 복제되는 구조다. 이로 인해 불필요한 메모리 할당과 객체 생성을 유발하여 비효율적이다.
    - 코틀린에서는 불변(immutable) 컬렉션 사용이 일반적이라 `values()`는 mutable 배열을 반환하므로, 값을 리스트나 기타 컬렉션으로 따로 변환해야 하는 번거로움이 있다.
- 실제로 스프링의 `HttpStatus.resolve`가 내부적으로 `values`를 사용하여 실제 벤치마크 성능에 영향을 줬다는 사례도 존재한다.

## Entries의 등장

```kotlin
val entires: EnumEntries<ShowCategory> = ShowCategory.entires
```

- `entries`는 enum의 새로 추가된 프로퍼티로 `EnumEntires<E>` 객체를 반환한다.
    - entries는 불변 리스트(`EnumEntries<E>`)를 미리 생성해두고, 호출 시에도 이 동일 객체를 "재사용"한다.
    - 즉, 매번 새로 할당하는 비용 없이, 동일한 리스트 인스턴스를 반환한다.
- `EnumEntries<E>`
    - 변경 불가능한(immutable) `sealed` 인터페이스로 `List`를 상속받는다.

```kotlin
/**

[List] 인터페이스의 특수한 불변 구현체로,

지정된 enum 타입 [E]의 모든 enum 엔트리를 포함합니다.

[EnumEntries]는 소스 코드에 선언된 순서대로 모든 enum 엔트리를 포함하며,

각 엔트리는 [Enum.ordinal] 값과 일관된 순서를 가집니다.

이 인터페이스의 인스턴스는 EnumClass.entries 프로퍼티나 [enumEntries] 함수를 통해 얻을 수 있습니다.

구현 참고
contains 및 indexOf와 같은 모든 기본 연산은 상수 시간( O(1) )에 실행되며,

일반적인 ArrayList<E>보다 더 빠르게 동작할 수 있습니다.
*/
@SinceKotlin("1.9")
@WasExperimental(ExperimentalStdlibApi::class)
public sealed interface EnumEntries<E : Enum<E>> : List<E>
```

- 그런데 그냥 리스트가 아닌 굳이 새로운 인터페이스(`EnumEntries`)를 사용한 이유
    - 단순 `List`는 모든 `enum` 엔트리들의 목록이라는 의도를 잘 표현하지 못한다.
    - 만약 `enum` 상수 리스트를 통해 모든 가능한 값을 다루는 유틸리티를 작성하고 싶을 때 단순 리스트로는 그 작업을 하기 매우 번거롭다.
        - 복잡한 사전 검증이 필요할 지도 모른다.
- `sealed` 클래스인 이유
    - 하위 타입의 범위를 명확히 제한하여, 코틀린 컴파일러가 `EnumEntries`의 모든 구현을 컴파일 타임에 완전히 알 수 있게 보장한다.
    - 이를 통해 추후에 외부(사용자)가 임의로 `EnumEntries`를 구현해서 혼란을 유발하거나, 라이브러리 설계 의도와 다르게 확장하는 것을 막아 API 안전성 및 일관성을 확보한다.
