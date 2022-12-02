# 아이템24. 멤버 클래스는 되도록 static으로 만들라

## 중첩 클래스
중첩 클래스(nested class)란 다른 클래스 안에 정의된 클래스를 말한다. 메서드 밖에서 사용해야 하거나 메서드 안에 정의하기에 너무 길 때 만든다.
```java
public class OuterClass {

	// ...

	class InnerClass {
		// ...
	}
}
```
## 정적 멤버 클래스
static이 붙어 정적으로 선언된 중첩 클래스를 정적 멤버 클래스라고 한다. 바깥 클래스의 private 멤버 변수에도 접근할 수 있다는 점만 제외하고 일반 클래스와 똑같다. 멤버 클래스일 뿐 다른 정적 멤버 변수와 같은 규칙을 적용받는다. 바깥 클래스가 생성되는 것과 상관없이 독립적으로 생성할 수 있다.
```java
public class OuterClass {
	
	static class StaticInnerClass {

	}
}

//생성 시
OuterClass.StaticInnerClass staticInnerClass = new OuterClass.StaticInnerClass();
```
## 비정적 멤버 클래스
정적 멤버 클래스와 달리 static이 붙지 않는 중첩 클래스이다. 그럼 비정적 멤버 변수와 비슷한가 생각하겠지만 멤버 클래스로서 비정적은 의미상 차이가 좀 크다. 비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다. 당연하겠지만 바깥 클래스와 상관없이 독립적으로 생성될 수 없고 바깥 클래스의 인스턴스가 생성되고 그 인스턴스를 통해 생성될 수 있다.
```java
public class OuterClass {

	class NonStaticInnerClass {

	}
}

// 생성시
OuterClass outerClass = new OuterClass();
OuterClass.NonStaticInnerClass nonStaticInnerClass = outerClass.new NonStaticInnerClass();
```
비정적 멤버 클래스의 인스턴스와 바깥 인스턴스 사이 관계는 멤버 클래스가 인스턴스화될 때 확립되며 더 이상 변경할 수 없다.

## 비정적 멤버 클래스 문제점
- 바깥 인스턴스와 비정적 멤버 클래스의 인스턴스의 관계 정보는 비정적 멤버 클래스의 인스턴스 안에 만들어져 메모리를 차지하며 생성시간도 더 걸린다.
- 비정적 멤버 클래스의 인스턴스는 바깥 인스턴스로의 숨은 외부 참조를 갖게 된다. 이 참조를 저장하는 데에도 시간과 공간이 소비되고 참조 때문에 가비지 컬렉션이 더 이상 사용되지 않는 바깥 클래스의 인스턴스를 수거하지 못해 메모리 누수가 생길 수 있다.

## 멤버 클래스는 되도록 static으로
멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 사용하는 것이 좋다. 그러면 숨은 외부 참조를 갖게될 일이 없다.
