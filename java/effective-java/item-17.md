# 아이템17. 변경 가능성을 최소화하라

## 불변 클래스
클래스의 인스턴스 내부 값을 수정할 수 없는 클래스를 불변 클래스라고 한다. 인스턴스의 정보는 객체가 파괴되는 순간까지 절대 달라지지 않는다.

### 불변 객체의 특징
- **불변 객체는 단순하다.** 인스턴스 내부가 달라지는 가변 객체와 달리 분변 객체는 생성된 뒤부터 값이 달라지지 않았음을 보장하여 믿고 쓸 수 있다.
- **불변 객체는 스레드 세이프**하고 동기화할 필요가 없다. 여러 스레드가 동시에 사용해도 훼손되지 않는다.
- **불변 객체는 안심하고 공유**할 수 있다. 스레드 세이프한 것과 이어지는 내용인데 값이 달라지지 않기 때문에 재활용할 수 있으면 재활용하면 좋다. 자주 쓰이는 인스턴스는 캐싱하여 같은 인스턴스를 중복 생성하지 않게 해 줄 수도 있다.
- **객체를 만들 때 다른 불변 객체들을 구성 요소로 사용하면 이점이 많다**. 구조가 아무리 복잡하더라도 모든 요소들이 불변이라면 불변식을 유지하기 수월하다.
- **불변 객체는 실패 원자성을 제공**한다. 불변 객체를 사용하는 메서드가 실패하더라도 (예외를 던짐) 해당 객체는 불변이기 때문에 호출 전 상태와 다르지 않다.
- **불변 객체의 상태가 달라질 필요가 있을 때 반드시 독립된 객체를 새로 만들어야 한다**. 이는 단점인데 값의 가짓수가 많다면 비용이 많이 들어갈 것이다.
- **불변 객체끼리는 내부 데이터를 공유할 수 있다**. 예를 들어 `BigInteger` 클래스는 내부에서 값의 부호(`int signum`)와 크기(`int[] msg`)를 가진다. 메서드로 `negate()`를 가지는데 크기가 같고 부호가 반대인 새로운 `BigInteger`를 반환한다. 밑의 메서드를 보면 msg는 배열이라 가변 인스턴스임에도 불구하고 참조를 그대로 전달한다. 내부 인스턴스가 불변이라는 믿음 덕분에 이렇게 내부 데이터를 공유할 수 있는 것이다.
```java
 public BigInteger negate() {
    return new BigInteger(this.mag, -this.signum);
}
```

## 불변 클래스를 만들기 위한 규칙
### 1. 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
대표적으로 `setter()`가 있다. 이처럼 내부 인스턴스에 접근해서 값의 변경을 일으킬만한 메서드를 만들지 않아야 한다. 참조를 가지는 컬렉션이나 배열을 원본 그대로 객체 외부로 보내는 메서드도 쓰면 안 된다.

### 2. 클래스를 확장할 수 없도록 한다.
   변경자를 제공하지 않아도 상속 받은 객체가 변경자를 가진다면 내부 데이터는 달라질 수 있다. 때문에 상속을 막아야 하는데 대표적인 방법은 클래스를 `final`로 선언하거나 모든 생성자를 `private`로 만드는 것이다. 보통 모든 생성자를 `private`로 만들고 정적 팩터리 메서드를 제공하는 방법이 더 유연한 방법이다. 생성자에 접근할 수가 없으니 클래스 확장이 불가능하다.
```java
public class ImmutableClass {

	private final int number;
	private final String sentence;
	private final boolean isImmutable;

	private ImmutableClass(int number, String sentence, boolean isImmutable) {
		this.number = number;
		this.sentence = sentence;
		this.isImmutable = isImmutable;
	}
	
	public static ImmutableClass of(int number, String sentence, boolean isImmutable) {
		return new ImmutableClass(number, sentence, isImmutable);
	}
}
```

### 3. 모든 필드를 final로 선언한다.
클래스 외부뿐만 아니라 클래스 내부에서도 값이 변경되는 것을 시스템이 강제하게 해준다.

### 4. 모든 필드를 private로 선언한다.
변경자 메서드를 제공하지 않는 것과 맥락이 비슷하다. private이 아니라면 클라이언트에서 직접 필드에 접근할 수가 있다. 뿐만 아니라 객체의 캡슐화도 해치게 된다.

(API일 경우 내부 구현을 모르면 내부 표현을 마음대로 바꿔도 사용자 입장에서는 원래 공개 API (public 메서드)만 알면 되지만, 내부 구현이 공개되어 있다면 내부 표현이 바뀔 때마다 변경 사항을 문서화해야할 것이다.)

### 5. 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.
모든 필드가 다 불변일 수도 없을 것이다. 가변 필드가 있을 수도 있는데 클라이언트에서 그 객체의 참조를 절대 얻을 수 없도록 해야 한다. 접근자 메서드가 그 필드를 그대로 반환하는 경우 문제가 된다.

```java
public class ImmutableClass {

	private final List<Integer> mutableList;

	public ImmutableClass(List<Integer> mutableList) {
		this.mutableList = new ArrayList<>(mutableList);;
	}

	public List<Integer> getMutableList() {
		return List.copyOf(mutableList);
	}
}
```
리스트의 경우 `final`을 통해 `mutableList`에 재할당은 불가능하지만 `getMutableList()`를 통해 값을 그대로 전달할 경우 리스트의 내부가 변화할 위험이 있다. [`List.copyOf`를 통해 반환하여 참조도 끊고 변화도 막을 수 있다.](https://does-log.tistory.com/17)  만약 `mutableList`가 외부에서 주입 받은 뒤 내부 요소도 계속 변할 일이 없다고 하면 주입 받은 시점에 `List.copyOf`를 사용하는 것도 방법이지만 가변이라고 했을 땐 새로운 `ArrayList`에 담아 전달하는 것으로 클라이언트 쪽의 리스트와 참조만 끊어준다.

### 정리
클래스는 필요한 경우가 아니라면 불변이어야 한다. 불변으로 만들 수 없더라도 변경할 수 있는 부분을 최소한으로 줄여야 한다. 다른 이유가 없다면 모든 필드는 `private final`이어야 하고 정말 변할 필요가 있는 필드에만 변경 메서드를 열어 둔다. 그 때도 `setXXX()`와 같이 setter로 열어두는 것이 아니라 왜 이 값을 변경하는지 명시하는 적절한 네이밍을 쓰는 것이 좋다.
