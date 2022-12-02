# 아이템44. 표준 함수형 인터페이스를 사용하라
자바가 람다를 지원하면서 API를 작성하는 방법이 크게 바뀌었다. 특히 함수형 인터페이스를 사용해 메서드마다 다른 동작을 메서드를 사용하는 클라이언트에서 지정해줄 수 있다. 이런 방식을 동작 파라미터화라고 한다. 이를 이용하면 변화하는 요구사항에 효과적으로 대응할 수도 있고 클래스와 클래스 간의 의존을 맺을 때 의존을 끊어줄 수도 있다.

## 변하는 요구사항에 대응하기
회원들의 리스트에서 어떤 조건에 따라 필터링을 해야 한다고 가정하자.
```java
public class Member {

	private final String name;
	private final int age;

	public Member(String name, int age) {
		this.name = name;
		this.age = age;
	}
 }
```
회원은 이름과 나이를 필드로 가지고 있다. 만약 회원 중에 20살 이상만 선별하고 싶다면 다음과 같이 메서드에 20살 이상을 필터링하는 코드가 명시적으로 들어가야 한다. 다른 조건으로 필터링하고 싶으면 새로운 메서드를 만들 수밖에 없다.
```java
public static void main(String[] args) {
    List<Member> members = List.of(new Member("pobi", 34),
            new Member("does", 24),
            new Member("jason", 27),
            new Member("dongkyu", 30)
    );
    
    List<Member> filteredMember = filterMember(members);
    filteredMember.forEach(System.out::println);
}
    
static List<Member> filterMember(List<Member> membersr) {
    return members.stream()
	.filter(member -> member.getAge() > 20)
	.collect(Collectors.toList());
}
```
여기서 함수형 인터페이스를 전달한다면 큰 유연성을 얻을 수가 있다. 다음과 같이 Member를 받아 boolean을 반환하는 추상 메서드를 가진 함수형 인터페이스를 만들고  filterMember의 파라미터로 넣어준다.

```java
@FunctionalInterface
public interface MemberFilter {
	boolean select(Member member);
}
```

```java
public static void main(String[] args) {
	List<Member> members = List.of(new Member("pobi", 34),
		new Member("does", 24),
		new Member("jason", 27),
		new Member("dongkyu", 30));

	List<Member> filteredMember = filterMember(members, member -> member.getAge() < 30);
	filteredMember.forEach(System.out::println);
}

static List<Member> filterMember(List<Member> members, MemberFilter memberFilter) {
	return members.stream()
		.filter(memberFilter::select)
		.collect(Collectors.toList());
}
```
이렇게 하면 filterMember 메서드를 사용하는 쪽에서 상황에 맞는 람다를 전달해 원하는 필터링을 수행할 수 있다.

## 객체 간 의존 끊기
함수형 인터페이스를 이용하면 의존 관계를 끊어줄 수도 있다. 블랙잭 카드 게임을 구현하는 애플리케이션을 예로 들어보겠다.

### BlackJackGame
BlackJackGame 클래스는 게임을 하는 참가자들을 관리하는 Gamers와 52장의 카드를 가지고 `draw()` 메서드로 한 장씩 뽑아주는 Deck을 필드로 가지고 있다. 게임을 시작하면 모든 게이머에게 카드를 2장씩 줘야 한다. 카드를 주려면 deck를 파라미터로 넘겨야 하는데 그럼 Gamers에서 Deck에 대해 의존을 갖게 된다.
```java
public class BlackJackGame {

	private final Gamers gamers;
	private final Deck deck;

	public BlackJackGame(List<String> names, Deck deck) {
		this.gamers = new Gamers(names);
		this.deck = deck;
	}

	// ...

	private void giveFirstCards() {
		for (int i = 0; i < 2; i++) {
			gamers.giveCardToAllGamers(deck);
		}
	}
    
    	// ...
}
```

### Gamers
```java
public class Gamers {

	private final Dealer dealer;
	private final List<Player> players;

	// ...

	public void giveCardToAllGamers(Deck deck) { // Gamers가 Deck을 알게 됨
		dealer.addCard(deck.draw());
		for (Player player : players) {
			player.addCard(deck.draw()));
		}
	}
    
    	// ...
 }
```
인터페이스를 통해 코드 블록을 전달한다면 의존성을 끊을 수 있다. Gamers의 메서드에서 Deck을 받는 것이 아닌 Supplier를 받는다. Supplier는 인자를 받지 않고 <>에 명시한 타입을 반환하는 표준 함수형 인터페이스이다.
```java
public void giveCardToAllGamers(Supplier<Card> deck) {
	dealer.addCard(deck.get());
	for (Player player : players) {
		player.addCard(deck.get());
	}
}
```
이제 이 메서드를 사용하는 BlackJackGame에서는 다음과 같이 람다로 메서드를 전달하면 된다.
```java
private void giveFirstCards() {
	for (int i = 0; i < 2; i++) {
		gamers.giveCardToAllGamers(Deck::draw);
	}
}
```
함수형 인터페이스를 통해 Gamers는 Deck의 존재를 모른채 똑같은 기능을 할 수 있게 되었다.

## 표준 함수형 인터페이스
Member를 필터링할 때는 직접 만든 함수형 인터페이스를 사용했고, 블랙잭에서는 Supplier라는 표준 함수형 인터페이스를 사용했다. 필요한 용도에 맞는 게 있다면 직접 구현하지 말고 표준 인터페이스를 활용하는 것이 좋다. 그러면 다뤄야할 API 개념 수가 줄어들어 코드를 파악하기 쉬워지고 이미 잘 알려진 표준 함수형 인터페이스들은 디폴트 메서드 등도 잘 정의되어 있어서 상호 운용성도 크게 좋아진다.

### 표준 함수형 인터페이스 종류

| 인터페이스 | 함수 시그니쳐 |
|---|---|
| UnaryOperator<T> | `T apply(T t)` |
| BinaryOprator<T> | `T apply(T t1, T t2)` |
| Predicate<T> | 	`boolean test(T t)` |
| Function<T, R> |	`R apply(T t)` |
| Supplier<T> | `T get()` |
| Consumer<T> | 	`void accept(T t)` |

이 밖에도 위의 인터페이스에서 변형들이 있어서 인자 2개를 받는 `BiPredicate<T, R>`, int나 long, double의 기본 타입용인 `IntPredicate`, `LongPredicate` 등등이 있다. 표준 함수형 인터페이스는 총 43개이며 대부분 위의 기본형에서 변형한 것들이므로 찾아보고 사용하면 좋을 것이다.

### 전용 함수형 인터페이스
그럼에도 직접 만든 인터페이스가 필요할 때도 있을 것이다. 그럴 땐 다음의 고려사항을 생각해봐야 한다.

- 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다.
- 반드시 따라야 하는 규약이 있다.
- 유용한 디폴트 메서드를 제공할 수 있다.

그렇게 해서 전용 함수형 인터페이스를 만들었다면 인터페이스에 `@FunctionalInterface` 애너테이션을 달아야 한다. 이는 프로그래머의 의도를 명시해준다. 의도라 함은
1. 이 인터페이스는 람다용으로 설계된 것이다.
2. 이 인터페이스는 추상 메서드가 하나만 존재해야 한다.
3. 그 결과 유지보수 과정에서 메서드를 추가하지 못하게 막아준다.

### 함수형 인터페이스 사용 시 주의점
서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중 정의해서는 안 된다. 이는 모호함을 줄 뿐이며 올바른 메서드로 사용하기 위해 형변환을 해야 하는 경우가 많이 생긴다.
