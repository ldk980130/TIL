# 아키텍처와 경계
## 경계: 선 긋기

- 소프트웨어 아키텍처는 선을 긋는 기술이다. 이들 경계 중 일부는 프로젝트 아주 초기에 그어지기도 하고 어떤 경계는 아주 나중에 그어지기도 한다.
- 아키텍트의 목표는 필요한 시스템을 구성하는 데 드는 인적 자원을 최소화하는 것이다. 인적 자원의 효율을 떨어뜨리는 요인은 결함이고 특히 너무 일찍 내려진 결정에 따른 결함이 있다.
- 프레임워크, 데이터베이스, 웹 서버, 라이브러리, 의존성 주입에 대한 결정 등이 여기에 포함된다. 좋은 아키텍처는 이러한 부수적 결정을 연기할 수 있고, 결정이나 변경의 영향이 크지 않다.

### 업무 규칙과 데이터베이스

- 데이터베이스는 업무 규칙이 간접적으로 사용할 수 있는 도구다.
- 업무 규칙은 스키마, 쿼리 등의 세부 사항에 대해 알면 안 된다.
- 업무 규칙과 데이터베이스 구현체 사이에 인터페이스를 둠으로써 데이터베이스 결정을 연기시킬 수 있다.

### 업무 규칙과 입출력

- GUI 애플리케이션을 만들 때 GUI 자체가 시스템이라고 착각하기도 한다.
- 하지만 화면에 전혀 출력되지 않더라도 시스템은 동작될 수 있고 그래야만 한다.
- 입출력은 GUI든 웹이든 얼마든지 교체할 수 있도록 설계되어야 한다.

### 플러그인 아키텍처

- 업무 규칙에 세부 사항에 대한 플러그인을 붙임으로써 확장 가능하며 유지 보수가 쉬운 구조로 만들어야 한다.
- 이렇게 되려면 핵심 업무 규칙은 입/출력이나 인프라에 대한 세부 사항으로부터 잘 분리되어야 한다.
- 시스템을 플러그인 아키텍처로 배치함으로써 변경이 전파될 수 없는 방화벽을 생성할 수 있다.
- 업무 규칙과 세부 사항들은 다른 시점에 다른 속도로 변경되기 때문에 이 역시 단일 책임 원칙의 관점으로 경계가 반드시 필요하다.
- 의존성 역전 원칙 + 안정된 추상화 원칙의 응용
