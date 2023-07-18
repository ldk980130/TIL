# 모니터
### 세마포의 한계

- wait, signal 순서를 바꾸거나 생략하면 상호 배제를 위반하거나 데드락이 발생할 수도 있다.
- wait, signal이 프로그램 전체에 구성 되어 있으면 세마포의 영향이 미치는 곳이 파악하기 어렵다.

## 모니터

- mutual exclusion을 보장
- 조건에 따라 스레드가 대기(waiting) 상태로 전환 기능
- 한 번에 하나의 스레드만 실행 되어야 할 때, 여러 스레드와 협업이 필요할 때 사용될 수 있다.
- 구성 요소
    - **mutex**
    - **condition variables**(s)

### mutex

- 임계 영역에서 상호 배제를 보장하는 장치로서 임계 영역에 진입하려면 mutex lock을 획득해야 한다.
- 락을 취득하지 못한 스레드는 큐에 들어간 후 대기 상태로 전환
- 락을 쥔 스레드가 락을 반환하면 큐에 대기 상태로 있던 스레드 중 하나가 실행
- 임계 영역에 진입을 기다리는 이 큐를 **entry queue**라고 부른다.
    - **상호 베타 큐(Mutual Exclusion Queue)**

### condition variables(s)

- **waiting queue**를 가짐
    - 조건이 충족되길 기다리는 큐
    - **조건 동기 큐(Conditional Synchronization Queue)**
- 주요 동작
    - **wait**
        - 스레드가 자기 자신을 condition variable의 waiting queue에 넣고 대기 상태로 전환
    - **signal**
        - waiting queue에서 대기 중인 스레드 중 하나를 깨움
    - **broadcast**
        - waiting queue에서 대기 중인 스레드를 전부 깨움

## 모니터 코드

```java
acquire(m); // 모니터의 락 취득
while(!p) { //조건 확인
	wait(m, cv); // 조건 충족 안 되면 대기
}
// ...이런 저런 코드들...
signal(cv2); --OR-- broadcast(cv2); // cv2가 cv와 같을 수도 있음
release(m); // 모니터의 락 반환
```

- `acquire()`, `release()`
    - 뮤텍스의 락을 얻고, 해제하는 코드
    - 락을 얻지 못하면 entry queue에서 대기하다가 `release()`가 호출 되면 큐에서 기다리던 스레드가 락을 얻을 수 있게 된다.
- `wait()`
    - 어떤 조건이 맞지 않으면 `wait()`를 호출해서 condition variable을 파라미터 넘겨주게 된다.
    - `wait()`를 호출하면 condition variable이 관리하는 waiting queue에서 대기하게 된다.
    - waiting queue에서 대기할 때 락을 쥐고 있는 것은 비효율이기 때문에 반환하기 위해 `wait()`의 파라미터로 넘겨 주게 된다.
- `signal()`, `broadcast()`
    - waiting queue에서 대기하다가 깨어나서 코드를 수행한 뒤 나의 작업으로 인해 다른 스레드가 원하는 조건이 충족되었을 수도 있다.
    - 하나를 깨워주려면 `signal()`을 호출하고, 모두를 깨워주려면 `broadcast()`를 호출한다.

## 생산자 소비자 문제와 모니터

- Producer가 buffer를 채우고, buffer에 채워진 것들을 처리하는 Consumer가 있다.
- buffer의 크기는 정해져 있기 때문에 buffer 공간이 가득 찼을 때 Producer가 계속 확인을 해야 하는 문제가 생긴다. Consumer도 buffer가 비어 있음에도 반복적으로 buffer가 찼는지 확인을 해야 한다. → **busy waiting**

### 모니터로 해결

- waiting queue에 넣는 조건을 Producer에선 `isFull()`로, Consumer에선 `isEmpty`로 정의한다.
- Producer가 buffer에 값을 넣은 뒤 emptyCV로 `signal()`을 호출하면 `isEmpty()`에 의해 대기하던 Consumer 스레드는 깨어나게 된다.
- Consumer가 buffer에 값을 빼낸 뒤 fullCV로 `signal()`을 호출하게 되면 `isFull()`에 의해 대기하던 Producer 스레드는 깨어나게 된다.

```java
public volatile Buffer buffer;
public Lock lock;
public Boolean fullCV;
public Boolean emptyCV;

void producer() {
	while (true) {
		Task myTask = new Task();
		
		lock.acuire();
		
		while(buffer.isFull() {
			wait(lock, fullCV);
		}
		
		buffer.enqueue(myTask);

		signal(emptyCV);

		lock.release();
	}
}

void consumer() {
	while (true) {
		lock.acuire();
		
		while(buffer.isEmpth() {
			wait(lock, emptyCV);
		}
		
		Task myTask = q.dequeue();

		signal(fullCV);

		lock.release();

		duStuff(myTask);
	}
}
```

## 자바의 모니터

- 자바에서 동기화를 위해 모니터 방색을 채택하고 있다.
- 자바에서 모든 객체는 내부적으로 모니터를 가진다.
- 모니터의 mutual exclusion 기능은 `synchronized` 키워드로 사용한다.
- 자바의 모니터는 condition variable를 하나만 가진다.
- 세 가지 동작
    - **wait**
    - **notify**
    - **notifyAll**

### 자바에서의 생산자 소비자 문제

```java
class BoundedBuffer {

	private final int[] buffer = new int[5];
	private int count = 0;

	public synchrozied void produce(int item) {
		while (count == 5) { wait(); }
		buffer[count++] = item;
		notifyAll();
	}

	public void consume() {
		int item = 0;
		synchronized(this) {
			while (count == 0) { wait(); }
			item = buffer[--count];
			notifyAll();
		}
		System.out.println("Consume : " + item);
	}
}
```

---

[https://youtu.be/Dms1oBmRAlo](https://youtu.be/Dms1oBmRAlo)

[https://steady-coding.tistory.com/557](https://steady-coding.tistory.com/557)
