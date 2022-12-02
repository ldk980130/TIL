# 아이템6. 불필요한 객체 생성을 피하라
똑같은 객체를 매번 생성하기보다 하나를 재사용하는 편이 나을 때가 많다. 특히 불변 객체는 생성할 필요 없이 언제든 재사용할 수 있다.

### 예시 1 String
```java
String newString = new String("string");

String string1 = "string";
String string2 = "string";
```
첫 줄처럼 문자열을 생성하는 사람은 없을 것이나 예시를 들기 위해 가져왔다. 문자열(String)은 프로그램이 실행되는 동안 엄청 많이 생성되는 객체이다. String은 String pool에서 관리되기 때문에 단순히 2,3번째 줄처럼 생성하면 String pool에서 인스턴스를 가져오기 때문에 불필요한 생성은 하지 않는다. 하지만 new로 생성하게 되면 새로운 인스턴스로 생성하기 때문에 밑의 "string"과는 다른 참조를 갖게 된다. 따라서 동등성은 보장하겠지만 동일성은 보장하지 않는다.
- `newString.equals(string1) // true`
- `newString == string1 // false`
- `string1 == string2 // true`
- `string1.equals(string2) // true`
> 여담으로 문자열끼리 +를 사용해서 더하는 연산을 반복하면 계속 새로운 String 객체를 생성하게 된다. (메모리 할당과 해제가 계속 발생) 문자열을 많이 더해야 하는 상황에서는 StringBuilder를 사용하자.

### 예시2. String.matches
다음은 주어진 문자열이 유효한 로마 숫자인지를 정규표현식으로 확인하는 메서드이다.
```java
static boolean isRomanNumeral(String s) {
	return s.matches("^(?=.)M*(C[MD]|D?C{0,3})" 
		+ "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```
이 방법도 불필요한 인스턴스를 계속 생성한다. 메서드 내부에서 많드는 정규표현식용 Pattern 인스턴스는 생성 비용도 높은데 한 번 쓰고 나면 바로 가비지 컬렉션의 대상이 된다. 성능을 개선하려면 Partern 인스턴스를 클래스 변수로 초기화하고 재사용하는 방법이 있다.

```java
private static final Pattern ROMAN = Pattern.compile(
	"^(?=.)M*(C[MD]|D?C{0,3})"
		+ "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

static boolean isRomanNumeral(String s) {
	return ROMAN.matcher(s).matches();
}
```
이러면 이 메서드가 자주 호출되어도 Pattern 인스턴스는 계속 재사용할 수 있다.

### 예시3. 오토박싱
오토박싱이란 프로그래머가 기본 타입과 박싱 된 기본 타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술이다. 기본 타입과 박싱 타입은 사용할 때 의미적으로 차이가 없어 보이지만 성능에선 큰 차이가 난다.
```java
private static long sum() {
	Long sum = 0L;
	for (long i = 0; i < Integer.MAX_VALUE; i++) {
		sum += i;
	}
	return sum;
}
```
Integer 타입의 범위 안에서 모든 수를 더하는 메서드이다. 결과를 담는 변수 sum은 박싱 타입인 Long으로 선언되어 있고 반복문 안에 i는 기본 타입인 long으로 되어 있다. 이렇게 되면 더해질 때마다 오토박생이 일어나 Long 타입 인스턴스가 불필요하게 엄청 생성된다. 박싱 된 기본 타입보다는 기본 타입을 사용하여 오토박싱이 성능에 영향을 미치지 않도록 해야 한다.

### 예시4. 로또 번호
[로또에 관련된 애플리케이션](https://github.com/ldk980130/java-lotto/tree/step1) 을 만든다고 가정하자. '로또 번호'라는 불변 객체가 필요하게 된다. 이 '로또 번호'는 1부터 45까지의 범위의 45개가 있으며, 이 45개 말고는 더 생성되지 않을 것이다. 하나하나가 불변이고 같은 숫자가 여러 로또에서 반복돼서 쓰일 것이기 때문에 같은 '로또 번호 1'이라는 인스턴스가 2개 이상 생길 필요가 없다. 따라서 위에서 `Pattern`을 캐싱했던 것처럼 '로또 번호' 클래스에서 45개의 인스턴스를 미리 캐싱하고 사용하면 45개를 초과하는 인스턴스는 더 이상 생기지 않을 것이다.
```java
public class LottoNumber {

    private final int number;
    private static final Map<Integer, LottoNumber> lottoNumbers = new HashMap<>();;

    static {
        for (int number = MINIMUM_LOTTO_NUMBER; number <= MAXIMUM_LOTTO_NUMBER; number++) {
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

    private static void validateNumberRange(int number) {
        if (number < MINIMUM_LOTTO_NUMBER || number > MAXIMUM_LOTTO_NUMBER) {
            throw new IllegalArgumentException(INVALID_LOTTO_NUMBER_RANGE);
        }
    }
  }
```

### 정리
하지만 무조건 재사용할 수 있는 인스턴스는 재사용하고 객체 생성을 피하라는 얘기는 아니다. 상황에 따라 새로운 인스턴스를 생성해야만 할 때도 있다. (애초에 요즘 JVM의 가비지 컬렉터는 상당히 잘 최적화되어 있다!) 방어적 복사가 필요한 상황에는 객체를 재사용했을 때 성능이 문제가 아닌 더 심각한 버그를 가져올 수가 있다.
