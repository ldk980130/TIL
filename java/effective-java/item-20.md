# 아이템 20. 추상 클래스보다는 인터페이스를 우선하라

## 인터페이스와 추상 클래스

자바가 제공하는 다중 구현 매커니즘으로 **인터페이스**와 **추상 클래스가** 있다.

### 추상 클래스

- 추상 클래스를 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다.
    - 단일 상속만 지원하는 자바에서 추상 클래스 방식은 새로운 타입을 정의하는데 제약이 생긴다.

### 인터페이스

- 자바 8부터 인터페이스는 디폴트 메서드를 제공한다.
    - `equals()`와 `hashCode()` 같은 `Object`의 메서드는 디폴트 메서드로 제공하면 안 된다.
    - 직접 만들지 않은 인터페이스에는 디폴트 메서드를 추가할 수 없다.
- 인터페이스는 인스턴스 필드와 `public`이 아닌 정적 멤버는 가질 수 없다.
    - `private` 정적 메서드는 예외
- **인터페이스를 구현하는 클래스는 인터페이스가 정의한 규약만 따른다면 다른 어떤 클래스를 상속했든 같은 타입으로 취급된다.**

## 인터페이스의 장점

### 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해 넣을 수 있다.

- 기존 클래스에 인터페이스가 요구하는 메서드를 추가하고 `implements` 구문만 추가하면 된다.
    - ex) `Comparable`, `Iterable`, `AutoCloseable` 인터페이스가 추가됐을 때 수많은 기존 클래스라 이들을 구현한 채 릴리스될 수 있었다.
- 추상 클래스는 계층 구조 상 기존 클래스 위에 새로운 추상 클래스를 추가하기 어렵다.
    - 계층 구조에 큰 혼란을 일으키고 모든 자손이 추상 클래스를 상속해야한다.

### 인터페이스는 믹스인(mixin) 정의에 안성맞춤이다.

- 믹스인이란 클래스가 구현할 수 있는 타입으로, 대상 타입의 주된 기능에 선택적 기능을 혼합한다고 해서 믹스인이라 부른다.
    - ex) `Comparable`은 자신을 구현한 클래스 인스턴스끼리 순서를 정할 수 있다고 선언한 믹스인 인터페이스
- 추상 클래스는 기존 클래스에 덧씌울 수 없기에 믹스인을 넣기 좋은 위치가 없다.

### 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.

- 계층을 엄격히 구분하기 어려운 개념에 인터페이스를 사용하면 유연하게 구조를 만들 수 있다.
    - ex) ‘작곡’도 하고 ‘노래’도 부르는 ‘싱어송라이터’를 표현할 수 있다.

    ```java
    public interface SingerSongwriter extends Singer, Songwriter {
    	// ...
    }
    ```

- `SingerSongwriter` 같은 구조를 클래스로 만들려면 조합 전부를 각각 클래스로 정의한 고도 비만 계층 구조가 만들어질 것이다.
    - 속성이 n개라면 조합의 수는 2의 n승이다.

### 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이다.

- 래퍼 클래스 관용구와 함께 사용하는 경우 기능 추가를 안전하게 할 수 있다.
    - 아이템 18 상속보다는 컴포지션을 활용해라
- 추상 클래스라면 그 타입에 기능을 추가하는 방법은 상속뿐이다.
    - 상속 클래스는 래퍼 클래스보다 활용도가 떨어지고 깨지기 쉽다.

## 인터페이스와 추상 골격 구현 클래스

- 추상 골격 구현(skeletal implementation) 클래스를 사용하면 인터페이스와 추상 클래스의 장점 모두 취할 수 있다.
    1. 인터페이스로는 타입을 정의하고 필요하면 공통적인 디폴트 메서드 몇 개도 함께 제공한다.
    2. 골격 구현 클래스는 나머지 메서드들까지 구현한다.
- 단순히 골격 구현을 확장하는 것만으로 인터페이스를 구현하는 데 필요한 일이 대부분 완료된다.
    - 여러 인터페이스 구현체들 간의 공통점이 있는 가운데 인터페이스 구현체를 만들려면 공통된 메서드들을 모두 중복 구현해서 만들어야 한다.
    - 골격 구현 클래스로 공통된 부분을 추상 클래스로 정의해두면 필요한 부분만 구현해서 쉽게 구현체를 만들 수 있다.

    ```java
    // 골격 구현을 통해 List 인터페이스가 제공하는 수많은 기능 가운데 
    // 단 몇 가지만 확장해서 구현체를 확장할 수 있다. 
    static List<Integer> intArrayAsList(int[] a) {
    	Objects.requireNonNull(a);
    
    	return new AbstractList<>() {
    		@Override
    		public Integer get(int i) {
    			return a[i];
    		}
    
    		@Override 
    		public Integer set(int i, Integer val) {
    			int oldVal = a[i];
    			a[i] = val;
    			return oldVal;
    		}
    
    		@Ovverride
    		public int size() {
    			return a.length;
    		}
    	};
    }
    ```

- 관례상 인터페이스 이름이 `Interface`라면 골격 구현 클래스의 이름은 `AbstractInterface`로 짓는다.
    - ex) `AbstractCollection`, `AbstractSet`, `AbstractList`, `AbstractMap`
- 골격 구현의 작은 변종으로 단순 구현(simple implementation)이 있다.
    - 골격 구현과 같이 상속을 위해 인터페이스를 구현한 것이지만 추상 클래스가 아닌 점이 다르다.
    - 단순 구현은 그대로 써도 되고 필요에 따라 확장해도 된다.
