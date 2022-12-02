# 아이템38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

### 열거 타입의 확장은 불가능?
열거 타입은 활용하기 좋지만 확장할 수 없다는 단점이 있다. 쉽게 말해서 Enum 타입은 상속이 불가능하다. 사실 대부분의 상황에서 열거 타입을 확장하는 것은 좋지 않은 생각이다. 열거 타입은 원소들을 순회할 수 있는데 기반 타입과 확장 타입을 모두 순회하는 방법도 마땅치 않다.

### 열거 타입은 인터페이스를 구현할 수 있다.
열거 타입을 상속받는 것은 불가능하다. 하지만 열거 타입이 인터페이스를 구현하는 것은 가능하다. 상위 인터페이스를 정의하고 이를 확장한 여러 열거 타입으로 유연하게 여러 열거 타입 구현체를 사용할 수 있다.

아래는 [체스 미션](https://github.com/ldk980130/java-chess/tree/step1) 에서 사용한 인터페이스와 열거 타입을 사용한 예시이다.

### 체스 미션에서 활용한 방향 열거 타입
체스 기물인 킹, 퀸, 폰, 룩, 비숍, 나이트는 각각이 이동 방법이 다르다. 구체적으로 들어가면 방향, 거리, 상황에 따른 분기가 다양하지만 여기서는 방향만 생각해보겠다.

- 킹 and 퀸 : 동, 서, 남, 북, 남동, 남서, 북동, 북서
- 블랙 폰 : 남서, 남동, 남
- 화이트 폰 : 북서, 북동, 북
- 룩 : 동, 서, 남, 북
- 비숍 : 남동, 남서, 북동, 북서
- 나이트 : 북북동, 동북동, 동남동, 남남동, 남남서, 서남서, 서북서, 북북서

딱 봐도 복잡하고 다양하다. 공통점도 보이는 것 같다. 일단 방향 자체를 열거 타입으로 정의하고 싶어 진다. 그리고 두 좌표를 받아서 방향을 계산하면 될 것 같다.

```java
public enum Direction implements {

	SOUTH((rowDifference, columnDifference) ->rowDifference > 0 && columnDifference == 0),
	NORTH((rowDifference, columnDifference) ->rowDifference < 0 && columnDifference == 0),
	WEST((rowDifference, columnDifference) ->rowDifference == 0 && columnDifference > 0),
	EAST((rowDifference, columnDifference) ->rowDifference == 0 && columnDifference < 0);
	NORTH_EAST(
		(rowDifference, columnDifference) ->
			rowDifference < 0 && columnDifference < 0 && rowDifference - columnDifference == 0,
	),
	NORTH_WEST(
		(rowDifference, columnDifference) ->
			rowDifference < 0 && columnDifference > 0 && rowDifference + columnDifference == 0,
	),
	SOUTH_EAST(
		(rowDifference, columnDifference) -> rowDifference > 0 && columnDifference < 0
			&& rowDifference + columnDifference == 0,
	SOUTH_WEST(
		(rowDifference, columnDifference) ->
			rowDifference > 0 && columnDifference > 0 &&  rowDifference - columnDifference == 0,

	private final BiPredicate<Integer, Integer> directionPredicate;
    
	DiagonalDirection(
		BiPredicate<Integer, Integer> quadrantPredicate, UnitPosition unitPosition) {
		this.directionPredicate = quadrantPredicate;
		this.unitPosition = unitPosition;
	}
    
	public boolean confirm(Position from, Position to) {
		return this.directionPredicate.test(from.subtractRow(to), from.subtractColumn(to));
	}
}
```
동, 서, 남, 북, 남동, 남서, 북동, 북서는 위와 같이 한  열거 타입으로 표현할 수 있다. 두 좌표를 받아 `BiPredicate`를 사용해 알맞은 방향을 찾을 수 있다. 어떻게 방향을 찾는지는 지금 중요한 게 아니니 신경 쓰지 말자.

하지만 문제가 있다. 일단 방향을 찾는 `Predicate` 로직이 많이 복잡하고 나이트의 방향은 이 로직으로 찾을 수 없다. 그리고 상황에 따라 대각선 방향인지(남동, 남서, 북동, 북서) 판단이 필요한데 그 판단을 하려면 메서드를 if 문으로 작성해야 하는 번거로움이 있다.

그래서 동서남북인 `BaisicDirection`, 대각선 바향인 `DiagonalDirection`, 나이트만의 방향인 `KnightDirection`을 따로 만들기로 했다. 따로 만드려니 타입이 다 달라 사용하는 클라이언트 쪽에서 구분해서 써야 하는 불편함이 있었다. 그래서 `Direction` 인터페이스를 구현하게 하여 클라이언트에선 어떤 확장 열거 타입인지 신경 쓰지 않고 `Direction`만 알고 사용하도록 하였다.

```java
public interface Direction {

	boolean confirm(Position from, Position to);

	boolean isDiagonal();
}
```
```java
public enum BasicDirection implements Direction {}

public enum DiagonalDirection implements Direction {
	@Override
	public boolean isDiagonal() {
		return true;
	}
}

public enum KnightDirection implements Direction {
	NORTH_NORTH_EAST(new UnitPosition(2, 1)),
	EAST_NORTH_EAST(new UnitPosition(1, 2)),
	EAST_SOUTH_EAST(new UnitPosition(-1, 2)),
	SOUTH_SOUTH_EAST(new UnitPosition(-2, 1)),
	SOUTH_SOUTH_WEST(new UnitPosition(-2, -1)),
	WEST_SOUTH_WEST(new UnitPosition(-1, -2)),
	WEST_NORTH_WEST(new UnitPosition(1, -2)),
	NORTH_NORTH_WEST(new UnitPosition(2, -1));

	private final UnitPosition unitPosition;

	KnightDirection(UnitPosition unitPosition) {
		this.unitPosition = unitPosition;
	}

	@Override
	public boolean confirm(Position from, Position to) {
		return from.subtractRow(to) + unitPosition.getUnitRow() == 0
			&& from.subtractColumn(to) + unitPosition.getUnitColumn() == 0;
	}
    // ...
}
```
`BasicDirection`과 `DiagonalDirection`은 앞서 봤던 열거 타입을 둘로 분리한 것이고 `isDiagonal()` 메서드만 `DiagonalDirection`에서 `true`로 오버라이딩 한 부분만 다르다.

`KnightDirection`은 좌표 2개로 방향을 찾는 로직이 달라 다른 두 타입과 구분하여 로직을 작성했다. ~~UnitPosition이 뭔지는 신경쓰지 말자.~~

이제 별도의 도우미 클래스로 각 확장 열거 타입을 하나의 `List<Direction>` 등으로 관리한다면 각 구현체를 몰라도 `Direction`의 여러 구현체들을 추상화하여 사용할 수 있다.

예를 들어 북, 북동, 북서만 취급하는 화이트 폰이 갈 수 있는 방향을 찾아주는 메서드이다.

```java
public Direction find(Position from, Position to) {
	return Stream.of(BasicDirection.NORTH, DiagonalDirection.NORTH_EAST, DiagonalDirection.NORTH_WEST)
		.filter(direction -> direction.confirm(from, to))
		.findAny()
		.orElseThrow(() -> new IllegalArgumentException(INVALID_DIRECTION_PAWN));
}
```

아래는 킹과 퀸의 방향을 찾아주는 메서드이다.
```java
public Direction find(Position from, Position to) {
	List<Direction> directions = new ArrayList<>();
	directions.addAll(Arrays.asList(BasicDirection.values()));
	directions.addAll(Arrays.asList(DiagonalDirection.values()));
	return directions.stream()
		.filter(direction -> direction.confirm(from, to))
		.findAny()
		.orElseThrow(() -> new IllegalArgumentException(INVALID_DIRECTION_ROYAL));
}
```

위의 두 예시의 메서드를 호출하는 곳에선 구체 타입은 모른체 `Direction` 인터페이스만으로 확장 타입들을 사용할 수 있다.
