# 아이템 71. 필요 없는 검사 예외 사용은 피하라

### 검사 예외의 특징

- 장점
    - 제대로 활용하면 API와 프로그램의 질을 높일 수 있다.
    - 발생한 문제를 프로그래머가 처리하기에 안전성을 높일 수 있다.
- 단점
    - 과하게 사용하면 쓰기 불편한 API가 된다.
    - try-catch 블록을 쓰거나 더 바깥으로 던져 문제를 전파해야 한다.
    - 검사 예외를 던지는 메서드는 스트림 안에서 직접 사용할 수 없다.

**API를 제대로 사용해도 발생할 수 있는 예외나 의미 있는 조치를 취할 수 있는 경우 외에는 비검사 예외를 사용하는 것이 좋다.**

### 검사 예외 회피

- 적절한 결과 타입을 담은 옵셔널을 반환
    - 단점으로는 예외가 발생한 이유를 알려주는 부가 정보를 담을 수 없다.
    - ㅇㅂ셔널만으로 상황을 처리하기에 충분한 정보를 제공할 수 없다면 검사 예외도 고려하자
- 상태 검사 메서드와 비검사 예외를 던지는 메서들 리팩터링

    ```java
    if (obj.actionPermitted(args)) {
    	obj.action(args);
    } else {
    	// 예외 처리
    }
    ```

    - 단점으로는 외부 요인에 의해 상태가 변할 수 있다면 멀티 스레드 환경에서 조심해야 한다.
