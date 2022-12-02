# 아이템1. 생성자 대신 정적 팩터리 메서드를 고려하라
일반적으로 객체를 생성할 때 생성자를 통해 생성한다.
```java
CustomClass customClass = new CustomClass();
```
정적 팩터리 메서드는 객체를 생성할 때 클래스 내부에 구현된 클래스 메서드로 자기 자신 객체를 반환하는 메서드이다. LocalDateTime을 예시로 살펴보자.

```java
LocalDateTime localDateTime = LocalDateTime.of(2022, 2, 24, 10, 26);
```
이처럼 객체를 생성할 때 of와 같은 이름을 가진 메서드로 클래스 메서드로 객체를 생성하는 것을 정적 팩터리 메서드라고 한다. 정적 팩터리 메서드를 사용하면 얻을 수 있는 이점은 다음과 같다.

### 1. 이름을 가질 수 있다.
   단순히 매개변수나 생성자 자체만으로 반환된다면 객체의 특성을 제대로 설명하지 못한다. 하지만 정적 팩터리 메서드를 사용하면 생성하고자 하는 객체의 의미를 잘 전달할 수 있다. 특히 생성자를 오버로딩해야하는 경우 정적 팩터리 메서드를 사용해 적절히 네이밍을 해준다면 특정 경우에 생성될 객체에 대한 묘사가 쉬워진다.
```java
LocalDateTime now = LocalDateTime.now();
LocalDateTime localDateTime = LocalDateTime.of(2021, 11, 11, 20, 25);
```
예시로 `LocalDateTime` 객체는 `now()`를 통해 현재 시간을 가지는 객체를 반환하고 of로 매개변수를 넣어주면 해당하는 년월일시분에 해당하는 객체를 반환해준다. 정적 팩터리 메서드를 통해 생성하고자 하는 객체가 무엇인지 파악이 쉬워지는 것이다.

### 2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.
불변 객체는 미리 인스턴스를 만들어 놓고 그 인스턴스를 사용하면서 불필요한 객체 생성을 막을 수도 있다. 예를 들어 로또와 관련된 애플리케이션을 구현한다고 가정하자. 로또 번호는 1부터 45까지인 숫자들이며 이 숫자들을 반환해야 한다고 했을 때 똑같은 숫자를 생성한다고 새로운 인스턴스가 생길 필요가 없다.

```java
public class LottoNumber {

    // ...

    private final int number;
    private static final Map<Integer, LottoNumber> lottoNumbers = new HashMap<>();;

    static {
        for (int number = 1; number <= 45; number++) {
            lottoNumbers.put(number, new LottoNumber(number));
        }
    }

    private LottoNumber(int number) {
        this.number = number;
    }

    public static LottoNumber getInstance(int number) {
        validateNumberRange(number);
        return lottoNumbers.get(number);
    }
    
    // ...
}
```
그럼 이렇게 1부터 45까지 미리 불변 객체인 `LottoNumber를` 만들어 놓고 정적 팩터리 메서드인 `getInstance()`를 통해 만들어져 있는 인스턴스를 반환할 수 있게 된다.

### 3. 반환 타입의 하위 타입 객체를 반환할 수 있다.
Person 인터페이스와 이를 상속받은 Adult, Child 클래스가 있다고 해보자. 그럼 Person 클래스에서 정적 팩터리 메서드를 사용해 다음과 같이 구현할 수 있다.
```java
public class Person {
	private int age;

	private Person(int age) {
		this.age = age;
	}

	public static Person create(int age) {
		if (age >= 20) {
			return new Adult(age);
		}

		return new Child(age);
	}
}
```
이렇게 하면 나이에 따라 Adult를 반환할지 Child를 반환할지 구현 코드를 공개하지 않고도 객체를 반환할 수 있다. 클라이언트는 실제 구현 클래스가 무엇인지 몰라도 되고 얻은 객체를 상위 객체 또는 인터페이스만으로 다룰 수 있게 된다.

### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
정적 팩터리 메서드를 사용해 메서드를 오버로딩하면서 매개변수 조건에 따라 다른 객체를 반환할 수 있는 것이다. EnumSet 클래스의 경우 매개변수 원소의 수에 따라 64개 이하면 RegularEnumSet의 인스턴스를, 65개 이상이면 JumboEnumSet의 인스턴스를 반환한다.

### 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
정적 팩터리 메서드를 사용하는 클래스가 내부 구현체(객체 생성을 담당하며 분기화 시키는)들을 통제하여 클라이언트가 구현체를 몰라도 되게 만들어 준다. 위의 Person, Adult, Child의 예에서도 Person.create를 작성할 때는 Adult와 Child의 존재를 몰라도 된다.

### 물론 단점도 존재한다.
1. 상속을 하려면 `public`이나 `protected` 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다. 
 기본 생성자 없이 정적 팩터리 메서드만 사용한다면 당연히 하위 클래스를 만들 수 없다. 하지만 이 점은 상속보다는 컴포지션을 사용하도록 유도하고 불변 타입으로 만들려면 이 제약을 지켜야 한다는 점에서 장점으로 볼 수도 있다.

2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
   API 설명에 명확히 드러나지 않기 때문에 다른 사람이 구현한 코드라면 찾기 힘들 수도 있다. 때문에 메서드 이름을 널리 알려진 규약을 따라 짓는 식으로 문제를 완화할 필요가 있다.

### 정적 팩터리 메서드 명명 방식
- `from` - 하나의 매개변수를 받아 해당 타입의 인스턴스를 반환
of - 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환
- `valueOf` - from과 of의 자세한 버전
- `instance`, `getinstance` - 매개변수로 명시한 인스턴스를 반환하지만 (매개변수 없을 수도 있음), 같은 인스턴스임을 보장하지 않음
- `create`, `newInstance` - instance와 같지만, 새로운 인스턴스를 생성해 반환함을 보장
- `getType` - getInstance와 같으나 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용. Type은 반환할 객체의 타입
- `newType` - newInstance와 같으나 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용.
- `type` - getType과 newType의 간결한 버전
