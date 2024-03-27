# APPENDIX A. 계약에 의한 설계

### 계약에 의한 설계가 왜 필요한가

- 메서드의 인터페이스만으로는 명령의 부수효과를 명확하게 표현할 수 없다.
    - 의도를 드러내도록 인터페이스를 다듬고 명령과 쿼리를 분리했다 하더라도
- 구현이 단순하다면 그나마 낫지만 구현이 복잡하다면 실행 결과를 예측하기 어려워진다.
- 계약에 의한 설계를 사용하면 협력에 필요한 다양한 제약과 부수효과를 명시적으로 정의, 문서화 가능하다.

## 01 협력과 계약

### 부수효과를 명시적으로

- 객체지향의 핵심은 협력 안에서 객체들이 수행하는 행동
- 메서드의 인터페이스로는 객체 사이의 의사소통 방식은 명확하게 정의할 수 없다.
    - 객체가 수신 가능한 메시지는 정의할 수 있지만 말이다.
- [6장의 명령-쿼리 분리 원칙](https://ldk980130.github.io/TIL/design/object/06.message&interface.html#04-%EB%AA%85%EB%A0%B9-%EC%BF%BC%EB%A6%AC-%EB%B6%84%EB%A6%AC-%EC%9B%90%EC%B9%99)에서 예시로 들었던 일정 관리 프로그램을 살펴보자.
    - 계약에 의한 설계 라이브러리인 Code Contracts를 통해 `IsSatisfied()`가 `true`일 때만 `Reschedule` 메서드를 호출할 수 있다는 사실을 명확히 표현 가능

```csharp
class Event 
{
  public bool IsSatisfied(RecurringSchedule schedule) { ... }
  
  public void Reschedule(RecurringSchedule schedule) 
  {
    Contract.Requires(IsSatisfied(schedule));
    ...
  }
```

- Code Contracts 라이브러리를 통해 제약 조건을 명시적으로 표현하는 것이 가능하다.
- 이러한 계약은 문서화로 끝나는 것이 아니라 조건 만족 여부를 실행 중에 체크 가능하다.

### 계약

- 계약 당사자 간에는 ‘의무’와 ‘계약’이 존재하는데 한쪽의 ‘의무’는 반대쪽의 ‘권리’가 된다.
    - ex) 고객이 업자에게 대금을 지급하는 것은 ‘의무’이고 서비스를 제공 받는 것은 ‘권리’가 된다. 업자는 반대로 서비스를 제공하는 것이 ‘의무’이고 대금을 받는 것이 ‘권리’다.
- 두 계약 당사자 중 한쪽이라도 계약 내용을 위반한다면 계약은 정상적으로 완료되지 않을 것이다.
- 계약은 협력을 명확하게 정의하고 커뮤니케이션할 수 있는 범용적인 아이디어다.

## 02 계약에 의한 설계

- 계약에 의한 설계
    - “인터페이스에 대해 프로그래밍 하라”는 원칙을 확장한 것
    - 협력에 참여하는 객체들이 지켜야 하는 제약을 명시 가능
    - 코드를 분석하지 않고 인터페이스 사용법을 이해 가능
- 의도를 드러내는 인터페이스를 만들면 시그니처만으로도 어느 정도까진 제약 조건을 명시할 수 있다.
    - 하지만 부족하다.

```java
// 가시성, 반환 타입, 메서드 이름
public Reservation reserve(
  Customer customer, // 파라미터 타입과 이름
  int audienceCount
) { ... }
```

- 위 자바 코드로 표현한 인터페이스로는 다양한 제약을 설명할 수는 없다.
    - `customer` 값으로 `null` 전달 불가
    - `audienceCount`로 음수 전달 불가
- 협력하는 클라이언트는 정상적인 상태의 객체와 협력해야 한다.
    - 서버는 자신이 처리 가능한 범위의 값들을 클라이언트가 전달할 것이라고 기대한다.
    - 클라이언트는 서버가 자신이 원하는 값을 반환할 것이라 기대한다.
- 클라이언트와 서버 사이의 기대
    - 사전 조건: 메서드가 호출되기 위해 만족돼야 하는 조건
    - 사후 조건: 메서드 실행 후 클라이언트에게 보장해야 하는 조건
    - 불변식: 항상 참이라 보장되는 서버의 조건
- 자바에선 계약에 의한 설계 개념을 지원하지 않기에 Code Contracts와 C# 예제를 통해 살펴 보자

### 사전 조건

- 사전 조건을 만족시키는 것은 클라이언트의 의무다.
    - 사전 조건이 만족되지 않는 경우는 클라이언트에 버그가 있다는 것
- ex) `Reserve` 메서드의 경우 `customer`가 `null`이 아니여야 하고 `audienceCount`는 1보다 커야한다 등의 조건

```csharp
public Reservation Reserve(Customer customer, int audienceCount)
{
  Contract.Requires(customer != null);
  Contract.Requires(audienceCount >= 1);
  return new Reservation(...);
}
```

- 클라이언트가 사전 조건을 위반하는 경우 `ContractException`이 발생할 것이다.
- 계약에 의한 설계 장점
    - 계약만을 위해 준비된 전용 표기법으로 계약을 명확하게 표현 가능
    - 계약이 메서드 일부로 실행되도록 계약을 강제 가능

### 사후 조건

- 사후 조건을 만족 시키는 것은 서버의 의무이다.
- 사후 조건의 용도
    - 인스턴스 변수 상태가 올바른지 서술하기 위해
    - 메서드에 전달된 파라미터 값이 올바르게 변경 됐는지를 서술하기 위해
    - 반환값이 올바른지 서술하기 위해
- 사후 조건을 정의하는 것이 사전 조건보다 어려운 이유
    - 한 메서드 안에서 return이 여러 번 나올 경우
        - 모든 return 문마다 검증 코드를 추가해야 한다.
    - 실행 전과 실행 후의 값을 비교해야 하는 경우
        - 실행 전의 값이 이미 다른 값으로 변경됐을 수 있기에 비교하기가 어렵다.
- Code Contracts에선 `Ensures` 메서드로 사후 조건을 명시할 수 있다.
    - 아래 예제에서 사후 조건은 `Reservation` 인스턴스가 `null`이어선 안된다는 것이다.

```csharp
public Reservation Reserve(Customer customer, int audienceCount)
{
  Contract.Requires(customer != null);
  Contract.Requires(audienceCount >= 1);
  Contract.Ensures(Contract.Result<Reservation>() != null);
  return new Reservation(...);
}
```

- `Contract.Result<T>` 메서드를 통해 `Reserve` 메서드의 실행 결과에 접근할 수 있다.
    - return 문이 여러개라 하더라도 한 번만 작성하면 된다.
- `Contract.OldValue<T>`를 통해 메서드 실행 전 상태에도 접근할 수 있다.

```csharp
public string Middle(string text)
{
  Contract.Requires(text != null && text.Length >= 2);
  Contract.Ensures(Contract.Result<string>().Length < Contract.OldValue<string>(text).Length);
  text = text.Substring(1, text.Length - 2);
  return text.Trim();
}
```

### 불변식

- 불변식은 인스턴스 생명주기 전반에 걸쳐 지켜져야 하는 규칙을 명세한다.
    - 반면 사전 조건, 사후 조건은 각 메서드마다 달라진다.
- 불변식 특징
    - 클래스의 모든 인스턴스가 생성된 후에 만족돼야 한다.
    - 클라이언트가 호출하는 모든 메서드에 준수돼야 한다.
    - 메서드 실행 전과 실행 후에는 반드시 불변식을 만족하는 상태가 돼야 한다.
- Code Contracts에선 `Contract.Invariant` 메서드로 불변식을 정의할 수 있다.
    - `ContractInvariantMethod` 애트리뷰트가 지정된 메서드를 불변식 체크하는 모든 지점에 자동으로 추가한다.

```csharp
public class Screening
{
  private Movie movie;
  private int sequence;
  private DateTime whenScrenned;
  
  [ContractInvariantMethod]
  private void Invariant() {
    Contract.Invariant(movie != null);
    Contract.Invariant(sequence >= 1);
    Contract.Invariant(whenScreened > DateTime.Now);
  }
}
```
