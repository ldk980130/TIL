# 아이템 27 비검사 경고를 제거하라

## 비검사 경고

- 제네릭을 사용하면 수많은 컴파일러 경고를 보게 된다.
    - 비검사 형변환 경고
    - 비검사 메서드 호출 경고
    - 비검사 매개변수화 가변인수 타입 경고
    - 비검사 변환 경고 등
- 비검사 경고 예시

    ```java
    Set<Lark> exaltation = new HashSet();
    
    /** 컴파일러가 경고를 보낸다.
    Venery.java:4: warning: [unchecked] unchecked conversion
        Set<Lark> exaltation = new HashSet();
                                 ^
      required: Set<Lark>
      found: HashSet
    */
    ```

    - 위와 같은 경우는 자바 7부터 지원하는 `<>` 연산자로 해결할 수 있다.
        - `Set<Lark> exaltation = new HashSet<>();`
- 모든 비검사 경고를 제거하면 그 코드는 타입 안정성이 보장된다.
    - 런타임에 `ClassCastException`이 발생할 일이 없게 된다.

## 경고 숨기기

- `@SuppressWarnings(”unchecked”)`
    - 경고를 제거할 수 없지만 타입 안전하다고 확신할 수 있다면 경고를 숨길 수 있다.
    - 단 타입 안전함을 검증하지 않고 사용하면 런타임 예외를 마주칠 수도 있다.
- 안전하다고 검증된 비검사 경고를 그대로 두면 진짜 문제를 알리는 경고를 놓칠 수도 있다.
    - 수많은 거짓 경고에 심각한 경고를 놓칠 수 있기 때문
- `@SuppressWarnings(”unchecked”)` 범위는 최대한 좁히는 것이 좋다.
    - 넓은 범위로 경고를 숨기게 되면 다른 경고도 놓치게 된다.
    - 메서드 단위, 나아가 지역 변수에만 붙일 수도 있다.
    - `@SuppressWarnings(”unchecked”) T[] result = …`
- 경고를 숨기면 이 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다.
    - 다른 사람이 코드를 이해하는데 도움이 된다.
    - 다른 사람이 그 코드를 잘못 수정하는 일을 줄인다.
