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
