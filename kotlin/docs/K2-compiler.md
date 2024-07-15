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


---

https://kotlinlang.org/docs/k2-compiler-migration-guide.html
