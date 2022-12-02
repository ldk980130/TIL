# 아이템26. 로 타입은 사용하지 말라

### 용어 정리
클래스와 인터페이스 선언에 타입 매개변수가 쓰이면, 이를 **제네릭 클래스** 혹은 **제네릭 인터페이스**라 한다.

```java
public interface List<E> extends Collection<E> {
```
List 인터페이스는 E라는 원소 타입 매개변수를 받고 이러한 제네릭 클래스와 인터페이스를 통틀어 **제네릭 타입**이라 한다.

제네릭 타입은 **매개변수화 타입**을 정의한다. 예를 들어 List<Integer>라고 했을 때 Integer가 리스트가 받을 수 있는 타입을 정의한 매개변수화 타입이다.

제네릭 타입에는 **로 타입**이라는 것도 있다. 로 타입이란 타입 매개변수를 사용하지 않은 제네릭 타입을 의미한다.
```java
List list;
```

그렇다면 이 로 타입을 사용하지 말아야 하는 이유는 무엇일까?

### 로 타입은 타입 안정하지 않다.
```java
List rawList = new ArrayList();
rawList.add(1);
rawList.add("1");
rawList.add(1.2);
rawList.add(new BigDecimal(10000000));

Object element = rawList.get(0);
```
로 타입으로 제네릭 타입을 사용하면 온갖 타입의 객체가 들어가도 컴파일 에러가 나지 않는다. 대신 이러한 경고 메시지를 보여주기는 한다.
> Unchecked call to 'add(E)' as a member of raw type 'java.util.List'

로 타입의 문제점은 안에서 값을 꺼내와도 `Object`로 반환되기 때문에 이 값이 무슨 타입인지 알 수가 없다. 사용자가 이 컬렉션을 `Integer` 요소만 넣고 사용할 거라고 생각해도 다른 값이 들어 있어서 형변환을 시도하면 `ClassCastExecption` 예외가 나오게 된다. 즉 런타임에 형변환하는 시점에 돼서야 의도하지 않은 타입이 자료구조 안에 있다는 것을 파악할 수 있는 것이다. 타입 안정하지 않다!

하지만 매개변수화 타입을 명시해준다면 아래와 같이 다른 타입이 들어가게 되면 컴파일러가 잘못된 값이 들어왔다고 알려준다. 컴파일 타임에 오류를 발견할 수 있는 것이다. 그리고 굳이 명시적으로 형변환을 해주지 않아도 원하는 타입으로 바로 가져올 수가 있다.
```java
List<Integer> integers = new ArrayList();
integers.add(1);
integers.add("1"); // 컴파일 에러 발생
```
그렇다면 로 타입으로 쓰는 것과 `Object`를 매개변수화 타입으로 하는 제네릭 타입은 뭐가 다른걸까. 다음과 같이 리스트를 `Object`로 선언하면 로 타입으로 선언한 것처럼 동작한다. 컴파일 에러도 나지 않는다.
```java
List<Object> objects = new ArrayList();
objects.add(1);
objects.add("1");
objects.add(1.2);
objects.add(new BigDecimal(10000000));

Object element = objects.get(0);
```
설명하면 로 타입은 제네릭이 주는 안정성과 표현력을 모두 버리고 아무 타입이나 다 받아들이면서 가지고 있는 타입들에 대해 전혀 신경을 안 쓰는 것이고, 매개변수화 타입을 `Object`로 주는 것은 모든 타입을 허용한다는 의사를 컴파일러에게 명확히 전달한 것이다. 특히 매개변수로 리스트를 받는 상황에서 이 둘의 차이가 명확히 드러난다.

```java
public static void main(String[] args) {
	List<Integer> integers = List.of(1, 2, 3);
	List<String> strings = List.of("a", "b");

	printList(integers);
	printList(strings);
}

static void printList(List list) {
	list.forEach(System.out::println);
}
```
위 코드는 정상 작동한다. 로 타입으로 매개변수를 받으면 어떤 타입의 리스트든 받을 수가 있다. `List<Integer>`나 `List<String>`은 `List`의 하위 타입이기 때문이다. 하지만 로 타입인 매개변수를 `List<Object>`로 바꾸게 되면 컴파일 에러가 일어난다. _`List<Object>`를 매개변수로 받는 메서드에는 오직 `List<Object>`만 받을 수 있다._

어떻게 보면 다양한 타입을 받을 수 있는 로 타입이 좋아보일 수도 있지만 리스트에 의도하지 않은 값이 들어가거나, 형변환을 거치며 조작해야 하는 경우에 어떤 타입 인지도 모를 리스트를 다루게 되면 런타임에 엄청난 에러를 맞닥뜨릴 것이다.

```java
public static void main(String[] args) {
	List<Integer> integers = new ArrayList<>();
	integers.add(1);
		
	List<String> strings = new ArrayList<>();
	strings.add("a");

	addList(integers, strings);

	integers.forEach(System.out::println); // 컴파일에러 x 실행하면 ClassCastExecption 발생
}

static void addList(List list1, List list2) {
	list1.addAll(list2);
}
```
`addList()` 메서드는 두 리스트를 합치는 메서드이다. 로 타입으로 정의했기 때문에 어떤 타입의 리스트든 받을 수 있다. 그래서 `Integer` 리스트에 `String` 리스트를 합치는 기괴한 식이 완성되었다. 컴파일은 되지만 Integer 리스트에 잘못된 타입이 들어갔기 때문에 다음처럼 내부 값을 건드리면 `ClassCastExecption`이 발생한다. 이러한 이유 때문에 로 타입은 쓰면 안 된다.

`List<Object>`를 썼다면 컴파일조차 되지 않기 때문에 이러한 에러를 빠르게 캐치할 수 있다.

### 비한정적 와일드 카드
제네릭을 사용하고 싶지만 실제 타입 매개변수가 무엇인지 신경 쓰고 싶지 않을 때는 로 타입을 사용하지 않고 어떻게 할 수 있을까. 예를 들어 위에서도 사용했던 컬렉션의 모든 요소를 출력하는 메서드는 매개변수 타입이 중요하지 않다.
```java
static void printList(List list) {
	list.forEach(System.out::println);
}
```
이 메서드는 컴파일 에러도 나지 않고 단순히 내부 값을 출력하는 것이기 때문에 로 타입을 써도 런타임에 예외도 발생하지 않는다. 하지만 메서드 내부에서 리스트가 조작된다면 에러는 언제든지 날 수 있는 것도 사실이다. 이런 상황에 비한정적 와일드 카드 타입을 사용할 수 있다.
```java
static void printList(List<?> list) {
	list.forEach(System.out::println);
}
```
매개변수화 타입에 물음포(?)를 넣으면 된다. 로 타입과의 차이점은 간단히 말해서 타입 안정하냐 하지 않냐이다. 로 타입에는 아무 원소나 넣을 수 있지만 와일드 카드 타입을 사용한 자료구조에는 null 이외에 어떤 원소도 넣을 수가 없다. 넣으려고 하면 컴파일 에러가 발생한다.
```java
static void printList(List<?> list) {
    list.add(1); // 컴파일 에러
	list.forEach(System.out::println);
}
```
비한정적 와일드 카드를 사용하면 로 타입을 사용한 것처럼 여러 타입의 제네릭 타입을 받을 수 있으면서 런타임에 예외가 발생하는 것도 막을 수 있다.

### 로 타입을 사용하는 예외 상황
로 타입을 쓰지 말라는 규칙에도 소소한 예외 몇 가지가 있다.

- `class` 리터럴에는 로 타입을 써야 한다. 
  - `List.class`, `String[].class`, `int.class` 허용

  - `List<String>.class`, `List<?>.class` 비허용
  
- `instanceof` 연산자는 비한정적 와일드 카드 타입 이외의 매개변수화 타입에는 적용할 수 없다.
  - 로 타입이든 비한정적 와일드카드 타입이든
  `instanceof`에서 똑같이 동작한다. 
  - 그래서 그냥 로 타입을 쓰는 편이 깔끔하다. 
  - `o instanceof Set` 허용
  - `o instanceof Set<Integer>` 비허용

### 정리
- 로 타입은 런타임에 예외가 날 수 있으니 사용하면 안 된다.
- 매개변수화 타입을 사용하여 컴파일 타임에 오류를 잡아야 한다.
- 매개변수화 타입을 `Object`로 하는 것은 어떤 타입의 객체도 받아들인다는 것이고, 비한정적 와일드 카드 타입은 모종의 타입 객체만 저장할 수 있다.
