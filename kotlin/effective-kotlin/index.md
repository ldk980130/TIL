# 이펙티브 코틀린

## 1부: 좋은 코드
### 1장 안정성
- [아이템 1 가변성을 제한하라](01.mutable.md)
- [아이템 2 변수의 스코프를 최소화하라](02.variable-scope.md)
- [아이템 3 최대한 플랫폼 타입을 사용하지 말라](03.platform-type.md)
- [아이템 4 inferred 타입으로 리턴하지 말라](04.inferred-type.md)
- [아이템 5 예외를 활용해 코드에 제한을 걸어라](05.exception-block.md)
- [아이템 7 결과 부족이 발생할 경우 null과 Failure를 사용하라](07.exception-return.md)
- [아이템 8 적절하게 null을 처리하라](08.null-handling.md)
- [아이템 9 use를 사용하여 리소스를 닫아라](09.close-resource.md)
- [아이템 10 단위 테스트를 만들어라](10.unit-test.md)

### 2장: 가독성
- [아이템 11 가독성을 목표로 설계하라](11.readability.md)
- [아이템 12 연산자 오버로드를 할 때는 의미에 맞게 사용하라](12.operator-overload.md)
- [아이템 13 Unit?을 리턴하지 말라](13.return-unit-nullable.md)
- [아이템 14 변수 타입이 명확하지 않은 경우 확실하게 지정하라](14.type-specify.md)
- [아이템 15 리시버를 명시적으로 참조하라](15.receiver.md)
- [아이템 16 프로퍼티는 동작이 아니라 상태를 나타내야 한다](16.property-status.md)
- [아이템 17 이름 있는 아규먼트를 사용하라](17.named-argument.md)

## 2부: 코드 설계

### 3장: 재사용성
- [아이템 19 knowledge를 반복하여 사용하지 말라](19.knowledge.md)
- [아이템 20 일반적인 알고리즘을 반복해서 구현하지 말라](20.common-algorithm.md)
- [아이템 21 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라](21.property-delegate.md)
- [아이템 22 일반적인 알고리즘을 구현할 때 제네릭을 사용하라](22.generic-method.md)
- [아이템 23 타입 파라미터의 섀도잉을 피하라](23.shadowing.md)
- [아이템 24 제네릭 타입과 variance 한정자를 활용하라](24.variance.md)
- [아이템 25 공통 모듈을 추출해서 여러 플랫폼에서 재사용하라](25.multiplatform.md)

### 4장: 추상화 설계
- [아이템 26 함수 내부의 추상화 레벨을 통일하라](26.abstract-level.md)
- [아이템 27 변화로부터 코드를 보호하려면 추상화를 사용하라](27.code-abstract.md)
- [아이템 28 API 안정성을 확인하라](28.api-safty.md)
- [아이템 29 외부 API를 랩(wrap)해서 사용하라](29.api-wrap.md)
- [아이템 30 요소의 가시성을 최소화하라](30.visibility.md)
- [아이템 31 문서로 규약을 정의하라](31.document-contract.md)
- [아이템 32 추상화 규약을 지켜라](32.abstraction-contract.md)

### 5장: 객체 생성
- [아이템 33 생성자 대신 팩토리 함수를 사용하라](33.factory-method.md)
- [아이템 34 기본 생성자에 이름 있는 옵션 아규먼트를 사용하라](34.default-constructor.md)
- [아이템 35 복잡한 객체를 생성하기 위한 DSL을 정의하라](35.custom-dsl.md)

### 6장: 클래스 설계
- [아이템 36 상속보다는 컴포지션을 사용하라](36.composition.md)
- [아이템 37 데이터 집합 표현에 data 한정자를 사용하라](37.data-class.md)
- [아이템 38 연산 또는 액션을 전달할 때는 인터페이스 대신 함수 타입을 사용하라](38.function-type.md)
- [아이템 39 태그 클래스보다는 클래스 계층을 사용하라](39.avoid-tag-class.md)
- [아이템 43 API의 필수적이지 않는 부분을 확장 함수로 추출하라](43.extension-function.md)
- [아이템 44 멤버 확장 함수의 사용을 피하라](44.avoid-member-extension.md)

## 3부: 효율성

### 7장: 비용 줄이기
- [아이템 45 불필요한 객체 생성을 피하라](45.object-create.md)
- [아이템 46 함수 타입 파라미터를 갖는 함수에 inline 한정자를 붙여라](46.inline-function.md)
- [아이템 47 인라인 클래스의 사용을 고려하라](47.inline-class.md)

### 8장: 효율적인 컬렉션 처리
