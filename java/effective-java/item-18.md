# 아이템18. 상속보다는 컴포지션을 사용하라
상속은 코드 중복을 제거하는 강력한 수단이지만 잘못 사용하면 오류를 내기 쉬운 소프트웨어를 만들게 된다.

## 상속의 문제점
### 상속은 캡슐화를 깨뜨린다
하위 클래스가 가지는 상위 클래스에 대한 강한 의존성 때문에 상위 클래스의 구현에 따라 하위 클래스 동작이 의도와는 다르게 동작할 수도 있다. 게다가 상위 클래스의 변화가 하위 클래스까지 전파되어 변화에 맞춰 수정도 계속해주어야 한다.



예를 들어 `HashSet`을 상속 받는다고 가정하자. 상속받은 하위 클래스는 처음 생성된 이후 몇 개의 원소가 더해졌는지 횟수를 알고 싶어서 횟수를 알 수 있는 기능을 추가했다.

```java
public class InstrumentedHashSet<E> extends HashSet<E> {

	private int addCount = 0;

	@Override
	public boolean add(E e) {
		addCount++;
		return super.add(e);
	}

	@Override
	public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return super.addAll(c);
	}

	public int getAddCount() {
		return addCount;
	}
}
```
멤벼변수 `addCount`를 `add()`를 호출할 때는 1씩, `adAll()`을 호출할 때는 컬렉션의 크기만큼 더해줌으로써 원소가 추가된 횟수를 저장했다. 이제 아래의 코드를 실행하면 3이 나와야 할 것이다.
```java
InstrumentedHashSet<Integer> set = new InstrumentedHashSet<>();
set.addAll(List.of(1, 2, 3));
System.out.println(set.getAddCount());
```
히지만 결과는 6이 나온다. 이는 `HashSet`의 `addAll()` 메서드가 내부에서 `add()`를 사용하기 때문이다. `addAll`에서 3이 더해지고 `add`에서 또 1씩 3번 더해져서 6이 나온 것이다. 다르게 구현해서 제대로 동작할 수는 있겠지만 만약 `HashSet`의 구현이 변경되면 또 의도하지 않은 동작을 할 수도 있고 변경을 계속 주시하고 수정해야 할 것이다.

객체지향에선 객체의 내부를 캡슐화하여 내부 구현에 신경쓰지 않고 설계하도록 한다. 하지만 상속을 하는 순간 상위 클래스의 내부 구현에 신경을 쓰고 하위 클래스를 확장해야 한다. 어떻게 하면 상위 클래스의 내부를 신경 쓰지 않고 기능을 확장할 수 있을까? 답은 컴포지션에 있다.

## 상속 대신 컴포지션
컴포지션이란 재사용하고 싶은 클래스를 새로운 클래스가 상속하는 것이 아니라 필드로 가지는 방법이다. 새로운 클래스는 재사용하려는 클래스에 대응하는 메서드를 호출해 그 결과를 반환하는데 이러한 방식을 전달(forwarding)이라고 한다.

```java
public class InstrumentedSet<E> {

	private int addCount = 0;
	private final Set<E> set;

	public InstrumentedSet(Set<E> set) {
		this.set = set;
	}

	public boolean add(E e) {
		addCount++;
		return set.add(e);
	}

	public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return set.addAll(c);
	}

	public int getAddCount() {
		return addCount;
	}
}
```
위의 코드를 보면 `Set` 인터페이스를 필드로 가지고 생성자에서 구현체를 받고 있다. 그리고 정의된 `add`와 `addAll` 메서드는 재정의한 메서드가 아니라 이 클래스에서 새로 만든 메서드이기 때문에 `addAll`이 호출되어도`add`와 전혀 무관하게 동작한다. 이렇게 하면 필드로 가지는 `Set` 구현체의 내부 구현에 대해 전혀 신경 쓸 필요가 없고 Set의 `add`와 `addAll`이 해주는 일에 대해서만 집중할 수 있다. 캡슐화가 지켜지는 것이다. 

```java
InstrumentedSet<Integer> set = new InstrumentedSet<>(new HashSet<>());
set.addAll(List.of(1, 2, 3));
```
이제 위처럼 호출해도 3이라는 의도한 결과가 나온다. Set 구현체가 변경되든 확장되든 add와 addAll의 기능만 잘 동작한다면 이 새로운 클래스는 변경하지 않아도 된다.

### 정리
확신이 안선다면 컴포지션을 사용해야 한다. 그럼에도 상속을 사용하기로 결정했다면 다음 사항들 또한 고려해야 한다.

- 하위 클래스와 상위 클래스가 정말 is-a 관계인가?
- 확장하려는 클래스의 API에 아무런 결함이 없는가?
- 결함이 있다면 이 결함이 새로운 클래스에 전파되어도 괜찮은가?
- 컴포지션을 사용한다면 결함을 숨기는 새로운 API를 설계할 수 있을 것이다.
