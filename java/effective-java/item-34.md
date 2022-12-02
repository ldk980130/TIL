# 아이템34. int 상수 대신 열거 타입을 사용하라

### 자바가 열거 타입을 지원하기 전
로또의 등수를 표현한다고 가정해보자. 열거 타입을 쓸 수 없다면 다음과 같이 표현할 수 있을 것이다.
```java
public static final int LOTTO_RANK_FIRST = 1;
public static final int LOTTO_RANK_SECOND = 2;
public static final int LOTTO_RANK_THRID = 3;
public static final int LOTTO_RANK_FOURTH = 4;
public static final int LOTTO_RANK_FIFTH = 5;
public static final int LOTTO_RANK_NOTHING = 0;
```
1등은 정수 1로, 5등은 5로, 꽝은 0으로 정수를 열거하여 작성했다. 이렇게 정수 열거 패턴을 사용하면 단점이 많다.

- **타입 안전을 보장할 방법이 없다.** 그냥 정수 값이기 때문에 등수를 매개변수로 받는 메서드에선 1이 들어오든 100이 들어오든 상관이 없는 것이다.
- **이름 공간(namespace)를 지원하지 않는다**. 때문에 LOTTO_RANK_XXX와 같이 접두어를 사용해서 이름 충돌을 방지했다. 로또 등수가 아닌 다른 개념의 FIRST, SECOND 등과 충돌하는 것을 막을 수 없기 때문이다.
- **정수 열거 패턴을 사용한 프로그램은 깨지기 쉽다**. 그냥 상수를 나열한 것이라서 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일 해야 한다. 실행이 되더라도 엉뚱하게 동작할 것이다. (NOTHING을 0이 아니라 -1로 표시한다고 변경되면 NOTHING을 사용하던 곳에선 숫자 0으로 인식하고 있었기 때문에 프로그램 동작이 엉뚱해진다.)
- **정수 상수는 문자로 출력하기가 까다롭다**. 출력하면 그냥 정수가 출력되기 때문에 의미가 와닿지 않는다.
- **상수와 연관된 데이터를 처리하기 곤란하다**. 1등은 로또 번호 6개를 맞춰야 하고 2등은 5개를 맞추고 보너스 번호도 맞춰야 한다. 이런 상수와 관련된 로직을 다른 곳에서 처리해줘야 한다.
- 이 외에도 같은 열거 그룹에 속한 상수를 순회하는 것도 어렵고 상수가 몇 개인지도 세보지 않으면 알 수 없다.

## 열거 타입(enum type)
이제 로또 등수에 열거 타입을 적용하여 정수 열거 패턴의 단점을 어떻게 극복하는지 살펴보자.
```java
public enum LottoRank {

	FIFTH(3, 5_000),
	FOURTH(4, 50_000),
	THIRD(5, 1_500_000),
	SECOND(5, 30_000_000),
	FIRST(6, 2_000_000_000),
	NOTHING(0, 0);

	private final int matchNumber;
	private final int prize;

	LottoRank(int matchNumber, int prize) {
		this.matchNumber = matchNumber;
		this.prize = prize;
	}
}
```
- **열거 타입은 컴파일 타임 타입 안정성을 제공**한다. 매개변수로 LottoRank 열거 타입을 받는 메서드가 있다면 그 메서드는 LottoRank 타입밖에 받지 못한다. 애초에 열거 타입 자체가 클래스이기 때문에 다른 타입을 넘기면 당연히 컴파일 에러가 날 것이다.
- **이름 공간을 지원**한다. LottoRank의 LottoRank.FIRST는 다른 열거 타입인 XXX.FIRST와 다르게 인식되기 때문에 이름이 같은 상수도 공존할 수 있다.
- **열거 타입에서 상수에 변경이나 삭제가 일어나도 해당 상수를 참조하지 않는 클라이언트에는 영향이 없다**. 상수의 변경은 열거 타입이 정의한대로 적용될 것이며, 삭제의 경우에는 적절한 컴파일 에러를 발견할 수 있다.
- **toString 메서드는 출력하기에 적합한 문자열을 내어준다**.
- **상수 각각을 특정 데이터와 연결지을 수 있다**. 위의 LottoRank에서는 각 등수에 몇 개의 번호가 맞아야 하는지와 상금까지 연관 지어 저장해주고 있다. 이제 클라이언트에선 LottoRank.FIRST를 사용하면 맞아야 하는 번호 개수와 상금을 연관 지을 수 있다.
- 열거 타입의 `values()` 클래스 메서드를 사용하면 각 상수를 선언 순서대로 담은 배열을 반환한다. 때문에 **각 상수를 순회할 수도, 몇 개인지 알기도 쉽다**.
- **상수와 연관된 데이터로 적절한 메서드를 사용할 수 있다**. 열거 타입이 강력한 이유 중 하나이다. 밑의 메서드를 보면 맞는 번호 개수와 보너스 번호가 맞는지 안맞는지를 매개변수로 받아 적절한 등수를 반환해주고 있다. 정수 열거 패턴에선 상상도 할 수 없는 일이다.
```java
public static LottoRank valueOf(int matchNumber, boolean bonus) {
	if (matchNumber == SECOND.matchNumber) {
		return checkSecondOrThird(bonus);
	}

	return Arrays.stream(LottoRank.values())
		.filter(rank -> rank.matchNumber == matchNumber)
		.findFirst()
		.orElse(NOTHING);
}

private static LottoRank checkSecondOrThird(boolean bonus) {
	if (bonus) {
		return LottoRank.SECOND;
	}
	return LottoRank.THIRD;
}
```
즉 열거 타입은 단순한 상수 모음이 아니라 메서드를 추가하여 고차원의 추상 개념 하나를 완벽히 표현해낼 수도 있다.

### 상수마다 다른 동작 구현하기
상수마다 동작이 달라져야 하는 상황도 있을 것이다. 예를 들어 알바생에게 근로 수당을 챙겨주는 상황이라고 해보자. 주간 근무라면 '일한 시간 X 임금'만큼 주면 되지만, 야간 근무를 했을 경우 야간 수당으로 1.5배를 더 줘야 한다. 열거 타입으로 DAY_SHIFT와 NIGHT_SHIFT가 있을 때 다른 급여 산출 기준이 필요하다. 열거 타입을 사용하여 다음과 같이 구현할 수 있다.

```java
public enum PayrollDay {

	DAY_SHIFT(((time, wage) -> time * wage)),
	NIGHT_SHIFT((time, wage) -> time * wage * 1.5);

	private final PayType payType;

	PayrollDay(PayType payType) {
		this.payType = payType;
	}

	public double pay(int time, int wage) {
		return payType.pay(time, wage);
	}

	interface PayType {
		double pay(int time, int wage);
	}
}
```
직접 구현한 내부 인터페이스를 필드로 가지며 생성자에 인터페이스 구현체를 람다로 넣어주었다. 이렇게 하면 주간 근무와 야간 근무의 급여 산출이 달라질 수 있다.
```java
double dayShiftPay = PayrollDay.DAY_SHIFT.pay(60, 10000); // 60_000
double nightShiftPay = PayrollDay.NIGHT_SHIFT.pay(60, 10000); // 90_000
```
