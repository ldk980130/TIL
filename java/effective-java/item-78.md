# 아이템 78. 공유 중인 가변 데이터는 동기화해 사용하라
### synchronized 두 가지 역할

`synchronized` 키워드에는 두 가지 역할이 있다.

- 한 스레드가 변경 중이라서 상태가 일관되지 않은 순간의 객체를 다른 스레드가 접근하지 못하게 막기 (**베타적 실행**)
- 동기화된 메서드나 블록에 들어간 스레드가 같은 락의 보호 하에 수행된 이전 수정의 최종 결과를 보게 함 (**스레드 사이의 안정적인 통신**)

언어 명세상 long, double 외의 변수를 읽고 쓰는 동작은 원자적이기 때문에 여러 스레드가 같은 가변 변수를 읽고 써도 어떤 스레드가 저장한 값을 정상적으로 읽어 올 수는 있다.

하지만 **자바 언어 명세에서는 한 스레드가 저장한 값이 다른 스레드에게  ‘보이는가’까지는 보장하지 않는다.** 즉 스레드 사이의 안정적인 통신이 되지 않는다는 말이다.

### 스레드 간 안정적 통신이 실패하는 경우

```java
public class StopThread {
	private static boolean stopRequested;

	public static void main(String[] args) throws InterruptedException {
			Thread backgroundThread = new Thread(() -> {
				int i = 0;
				while (!stopRequested) 
					i++;
			});
			backgroundThread.start();

			TimeUnit.SECONDS.sleep(1);
			stopRequested = true;
	}
}
```

위 코드는 `stopRequested = true`가 실행되는 1초 뒤에는 프로그램 실행이 끝날 것처럼 보이지만 사실은 끝나지 않는다.

**동기화하지 않으면 메인 스레드가 수정한 값을 백그라운드 스레드가 언제쯤에나 보게 될지 보증할 수 없다.**

### synchronized로 해결

`synchronized` 키워드로 `stopRequested`를 `true`로 바꾸는 메서드와 읽는 메서드를 동기화해주면 1초 뒤에 정상 종료되는 프로그램이 된다.

```java
private static synchronized void requestStop() {
	stopRequested = true;
}

private static synchronized boolean stopRequested() {
	return stopRequested;
}
```

**위 코드처럼 쓰기와 읽기 모두 동기화가 되어야 동작을 보장한다.**

### volatile로 해결

속도가 더 빠른 대안이 존재한다.

```java
public class StopThread {
	private static volatile boolean stopRequested;

	public static void main(String[] args) throws InterruptedException {
		Thread backgroundThread = new Thread(() -> {
			int i = 0;
			while (!stopRequested) 
				i++;
		});
		backgroundThread.start();
	
		TimeUnit.SECONDS.sleep(1);
		stopRequested = true;
    }
}
```

`volatile` 키워드를 사용하면 `synchronized`를 생략해도 된다. `volatile`은 베타적 수행과는 상관 없지만 항상 가장 최근에 기록된 값을 읽게 됨을 보장한다.

### volatile의 문제

```java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
	return nextSerialNumber++;
}
```

위 메서드는 데이터베이스 자동 증가 키값처럼 매번 고유한 값을 반환할 의도로 만들어졌다. `volatile` 덕분에 원자적으로 접근된다. 하지만 이 코드는 동기화 없이는 올바르게 동작하지 않는다.

`++` 연산자는 코드 상으로는 하나지만 실제로 `nextSerialNumber` 필드에 두 번 접근한다. (값을 읽고, 증가한 값을 저장) 만약 두 스레드가 이 사이를 비집고 들어온다면 두 스레드 모두 같은 값을 반환할 것이다. (이런 오류를 **안전 실패(safety failure)**라 한다.)

즉 `volatile`은 스레드 간 안전한 통신은 지원하지만 베타적 실행까지는 지원하지 않는 것이다.

`volatile` 키워드를 제거하고 메서드에 `synchronized`를 붙이면 정상적으로 동작하게 된다. 동시에 호출해도 서로 간섭하지 않고 이전 호출이 변경한 값을 일게 된다.

### AtomicLong으로 해결

`java.util.concurrent.atomic` 패키지의 `AtomicLong`으로도 해결할 수 있다. atomic 패키지에는 락 없이도 스레드 세이프한 프로그래밍을 지원하는 클래스들이 담겨 있다.

```java
private static AtomicLong nextSerialNumber = new AtomicLong();

public static int generateSerialNumber() {
	return nextSerialNumber.getAndIncrement();
}
```

volatile은 동기화의 두 효과 중 통신 쪽만 지원하지만 이 패키지는 베타적 실행까지 지원한다.

### 정리

여러 스레드가 가변 데이터를 공유한다면 그 데이터를 읽고 쓰는 동작은 반드시 동기화해야 한다. 동기화하지 않으면 한 스레드가 수행한 변경을 다른 스레드가 보지 못하거나 안전 실패로 인해 원하지 않는 동작을 야기할 수도 있다. 베타적 실행이 필요 없고 안전한 통신만 필요하다면 volatile 키워드로 동기화할 수 있지만 베타적 실행도 필요하다면 `synchronized`나 `atomic` 클래스들을 사용하자.

하지만 애초에 가변 변수는 스레드 간 공유하지 않게 설계하는 것이 가장 좋은 방법이다.
