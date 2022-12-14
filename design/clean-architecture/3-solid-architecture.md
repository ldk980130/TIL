# 설계 원칙 (SOLID와 아키텍처)
## SRP 단일 책임 원칙

모듈이 단 하나의 일만 해야 한다는 의미는 틀렸다. 단 하나의 일만 해야 한다는 원칙은 커다란 함수를 작은 함수들로 리팩터링하는 수준에서 사용된다.

SRP는 다음과 같이 설명할 수 있다.

> 하나의 모듈은 하나의, 오직 하나의 액터에 대해서만 책임져야 한다.
>

모듈의 단순한 정의는 소스 파일이다. 단일 액터를 책임지는 코드를 함께 묶어주는 힘이 바로 응집성이다. 이 원칙을 위반하는 징후를 살펴보면 더 잘 이해할 수 있을 것이다.

### 우발적 중복

급여 애플리케이션의 Employee를 생각해 보자. 세 메서드를 가지고 있다.

- `calculatePay()`
    - 회계팀에게 필요한 메서드로 CFO에 보고하기 위해 사용
- `reportHours()`
    - 인사팀에서 필요한 메서드로 COO에 보고하기 위해 사용
- `save()`
    - DBA에게 필요한 메서드로 CTO에게 보고하기 위해 사용

이 결합으로 인해 CFO 팀에서 결정한 조치가 COO 팀이 의존하는 무언가에 영향을 줄 수 있다. 각 메서드가 공통으로 사용하는 private 메서드를 다른 팀이 건드리면 몇몇 메서드는 정상 동작하지 않게 될지도 모른다. 이는 서로 다른 액터가 의존하는 코드를 너무 가까이 배치했기 때문이다.

### 병합

한 팀에스 Employee 테이블 스키마를 수정하려고 하고, 동시에 한 팀이 reportHours() 메서드의 포맷을 변경하기로 결정했다. 이들 변경사항은 서로 충돌하고 병합이 발생한다.

### 해결책

다른 액터에게 필요한 메서드를 각기 다른 클래스로 이동시키면 된다. 즉 데이터와 메서드를 분리하는 방식을 사용하여 `EmployeeData`라는 클래스를 세 클래스(`PayCalculator`, `HourReporter`, `EmployeeSaver`)가 공유하도록 한다. 세 클래스는 서로의 존재를 몰라야 한다.

개발자가 세 클래스를 추적해야 한다는 단점은 퍼사드 패턴으로 해소할 수도 있다.

## OCP 개방 폐쇄 원칙

소프트웨어 개체 행위는 확장할 수 있어야 하지만, 이 때 개체는 변경해서는 안 된다.

모든 컴포넌트의 의존 관계는 단방향으로 이루어져야 하며, 이들 화살표는 변경으로부터 보호하려는 컴포넌트(도메인 레이어)를 향하도록 그려야 한다. 즉 A의 변경으로부터 B를 보호하려면 반드시 A가 B를 의존해야 한다.

Controller가 Application 레이어에 의존하는 건 보호의 맥락에서도 일치한다. 그렇다면 흔히 DB 레이어의 변경으로부터 도메인을 보호하라고도 말하는 데 Application 레이어는 DB 레이어를 의존한다. 인터페이스를 통해 의존 방향을 역전 시키는 것도 좋은 방법일 수도 있다.

Application 레이어는 비즈니스 규칙을 담당한다. View 등의 저수준 모듈이 Application 등의 고수준 모듈을 의존하여 변경으로부터 보호해야 한다.

## LSP 리스코프 치환 원칙

자료형 S가 자료형 T의 하위형이라면 필요한 프로그램의 속성 변경 없이 자료형 T의 객체를 자료형 S의 객체로 교체할 수 있어야 한다는 원칙이다.

이를 위배하는 전형적인 문제는 직사각형/정사각형 문제가 있다. 직사각형의 가로와 세로는 독립ㅈ거으로 변할 수 있는 반면, 정사각형의 가로와 세로는 함께 변해야 한다. 직사각형의 독립된 가로의 변경을 기대하고 정사각형을 다룬다면 세로도 함께 변할 것이기에 이 둘은 치환될 수 없다.

이를 시스템 아키텍처적인 관점에서 보자면 단순 상속이나 구현의 관계가 아니더라도 다양한 인터페이스에서도 치환이 가능해야 한다. 가령 동일한 REST API에 응답하는 서비스 집단일 수도 있다. 만약 동일한 REST API를 준수해야 하는 여러 비슷한 서비스들에서 한 서비스만 컨벤션을 지키지 않는다면 (destination을 dest로 줄인다거나) REST 서비스들이 치환 가능하지 않다는 사실 때문에 복잡한 코드가 늘어날 것이다.

## ISP 인터페이스 분리 원칙

클라이언트가 자신이 이용하지 않는 메서드에 의존하지 않아야 한다는 원칙이다. A 클래스가 B 인터페이스의 3가지 메서드 중 1가지만을 이용한다고 하면 불필요한 2개의 메서드를 오버라이드 해야할 것이고, 이 2개의 메서드에 변경이 일어나면 A 클래스도 관계가 없지만 변경이 전파될 수도 있다.

이는 아키텍처에서도 마찬가지다. S 시스템 구축에 F라는 프레임워크를 도입하려고 한다. F 프레임워크는 D 데이터베이스를 반드시 사용하도록 만들었다고 가정해 보자. 따라서 S는 F에, 다시 D에게도 의존하게 된다. 이 때 F와 전혀 상관 없는 D만의 새 기능이 포함된다고 가정하면 F를 재배포해야할 수도 있고 따라서 S까지 재배포해야 할지 모른다.

무언가에 의존하면 예상치도 못한 문제에 빠질 수도 있다.

## DIP 의존성 역전 원칙

DIP에서 말하는 ‘유연성이 극대화된 시스템’이란 소스 코드 의존성이 추상에 의존하며 구체에는 의존하지 않는 시스템이다. 엄격한 규칙으로 보자면 자바의 String도 구체 타입이다. 하지만 String 클래스는 매우 안정적이며 변경되는 일은 거의 없다.

이러한 이유로 DJP를 논할 때 OS나 플랫폼 같이 안정성이 보장된 환경에 대해서는 무시하는 편이다. 우리가 의존하지 않도록 피하고자 하는 것은 변동성이 큰 구체적인 요소다. 가령 우리가 열심히 개발하는 중이라 자주 변경될 수밖에 없는 모듈이라던가

### 안정된 추상화 원칙

- 변동성이 큰 구체 클래스를 참조하지 말라
    - 대신 추상 인터페이스를 참조하라
- 변동성이 큰 구체 클래스로부터 파생하지 말라
    - 상속은 아주 신중하게 사용해야 한다.
- 구체 함수를 오버라이드 하지 말라
    - 차라리 추상 함수로 선언하고 구현체들에서 용도에 맞게 구현하라
- 구체적이며 변동성이 크다면 절대로 그 이름을 언급하지 말라

### 팩토리

구체적인 객체는 주의해서 생성해야 하지만 사실상 모든 언어에서 객체를 생성하려면 소스 코드 의존성이 발생하게 된다. 그래서 추상 팩토리를 사용하곤 한다.

`Application` 클래스가 `Service`라는 인터페이스를 의존하고 있지만 `ServiceImpl`이라는 구체 타입을 생성하긴 해야 한다. 그럴 때 `ServiceFactory`라는 인터페이스를 통해 `ServiceImpl`을  `Service` 타입으로 생성하도록 하면 구체 타입은 모른 채 `ServiceImpl`을 생성할 수 있다.
