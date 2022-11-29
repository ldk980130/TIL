# 스핀락, 뮤텍스, 세마포
### race condition(경쟁 조건)

여러 프로세스/스레드가 동시에 같은 데이터를 조작할 때 타이밍이나 접근 순서에 따라 결과가 달라질 수 있는 상황

### synchronization(동기화)

여러 프로세스/스레드를 동시에 실행해도 공유 데이터의 일관성을 유지하는 것

### critical section(임계 영역)

공유 데이터의 일관성을 보장하기 위해 하나의 프로세스/스레드만 진입해서 실행 가능한 영역

### mutual exclusion(상호 배제)

여러 프로세스/스레드 사이에서 공유 불가능한 자원의 동시 사용을 피하는 알고리즘으로 임계 영역에 구현된다.

상호 배제를 구현하기 위해 락을 사용하게 된다.

## 스핀락(spinlock)

```java
volatile int lock = 0; //global

void critical() {
	while (test_and_set(&lock) == 1); // lock을 획득할 때까지 기다
	// critical section
	lock = 0;
}

int test_and_set(int* lockPtr) {
	int oldLock = *lockPtr;
	*lockPtr = 1;
	return oldLock;
}
```

- 락을 가질 수 있을 때까지 반복해서 시도
- 하지만 락을 기다리는 동안 CPU를 낭비하게 된다.

> **TestAndSet은 CPU atomic 명령어**
>
> 실행 중간에 간섭 받거나 중단되지 않고 같은 메모리 영역에 대해 동시에 실행되지 않는다.
>

## 뮤텍스(mutex)

- 락을 가질 수 있을 때까지 휴식

```java
class Mutex {
	int value = 1; // 임계 영역에 진입하기 위해 얻어야 하는 값
	int guard = 0;

	void lock() {
		while(test_and_set(&guard));
		if (value == 0) { // 다른 스레드가 락을 먼저 얻은 경우
			// 현재 스레드를 큐에 넣음;
			guard = 0; 
			// 그리고 스레드를 sleep
		} else {
			value = 0; // 락 취득
			guard = 0;
		}
	}

	void unlock() {
		while (test_and_set(&guard));
		if (큐에 하나라도 대기 중이라면) {
			// 그 중 하나를 깨운다;
		} else {
			value = 1; // 락 반환
		}
		guard = 0;
	}
}
```

- 여러 스레드가 `value`라는 값을 얻어 임계 영역에 진입하게 된다.
- `value`도 여러 스레드에서 동시에 값을 바꾸려는 또 하나의 임계 영역이기 때문에 `while(test_and_set(&guard))`를 통해 한 스레드만 `value`를 얻도록 해두었다.
- 이제 `value`를 얻기 위해 기다리는 스레드는 `while`를 돌면서 계속 CPU를 쓰면서 기다리는 것이 아닌 sleep을 하다가 깨어나야 할 때 깨어나게 된다.
- **멀티 코어 환경에서는** 임계 영역에서의 작업이 컨텍스트 스위칭 보다 더 빨리 끝난다면 스핀락이 뮤텍스 보다 더 이점이 있다.

## 세마포(semaphore)

```java
class Semaphore {
	int value = 1; // 임계 영역에 진입하기 위해 얻어야 하는 값
	int guard = 0;

	void wait() {
		while(test_and_set(&guard));
		if (value == 0) { // 다른 스레드가 락을 먼저 얻은 경우
			// 현재 스레드를 큐에 넣음;
			guard = 0; 
			// 그리고 스레드를 sleep
		} else {
			value -= 1;
			guard = 0;
		}
	}

	void signal() {
		while (test_and_set(&guard));
		if (큐에 하나라도 대기 중이라면) {
			// 그 중 하나를 깨워서 준비 시킨다;
		} else {
			value += 1;
		}
		guard = 0;
	}
}
```

- `value`를 증가, 감소 시키는 형태이다. 초기 `value` 값에 따라 임계 영역에 들어오는 프로세스/스레드 수를 조정할 수 있다.
    - `value=1`인 세마포를 바이너리 세마포라고 부르고 그 이상이면 카운팅 세마포라고 한다.
- 세마포는 순서를 정해줄 때 사용할 수 있다.
    - task1 → task2의 순서를 보장하려고 하는 경우
        - task1이 끝나면 signal()을 호출하게 한다.
        - task2를 호출하기 전 wait()를 호출하게 한다.
        - task2를 하려고 wait()를 하기 때문에 task1이 끝나서 signal()을 호출할 때까지는 대기하게 된다.

### 뮤텍스와 바이너리 세마포는 다르다.

- 뮤텍스는 락을 가진 자만 락을 해제할 수 있지만 세마포는 그렇지 않다.
    - 세마포에서는 `wait()`를 하는 스레드와 `siginal()`을 날리는 스레드는 다를 수 있다.
    - 그래서 뮤텍스에서는 누가 락을 해제(`unlock()`)할 지 알 수 있지만 세마포는 누가 `signal()`을 호출할 지 알 수 없다.
- 뮤텍스는 priority inheritance 속성을 가진다. (세마포는 없다.)
- **상호 배제만 필요하다면 뮤텍스를, 작업 간 실행 순서 동기화가 필요하다면 세마포를 권장**

### priority inversion과 priority inheritance

- **priority inversion**이란 높은 우선 순위의 태스크가 낮은 우선 순위의 태스크가 락을 쥐고 있어서 실행되지 못하고 밀리는 상태를 말한다.
    1. task3이 락을 획득하여 작업 수행
    2. task1이 락 획득을 위해 기다림
    3. 락과 관련 없는 task2가 시작 되어 우선 순위 스케줄리에 의해 task3이 밀림
    4. task3를 기다리던 task1도 작업이 밀리면서 우선 순위가 더 낮은 task2에 의해 task1이 대기
    5. 결국 task1이 가장 나중에 수행 됨
- 위와 같은 문제를 해결하기 위해 task1이 락을 얻기 위해 기다리는 task3의 우선 순위를 task1만큼 일시적으로 올려 버리는 기법을 **priority inheritance**라고 한다.
- 락을 누가 해제할 지 모르는 세마포에서는 priority inheritance를 수행할 수 없다.

---

[https://www.youtube.com/watch?v=gTkvX2Awj6g](https://www.youtube.com/watch?v=gTkvX2Awj6g)

[https://j-i-y-u.tistory.com/21](https://j-i-y-u.tistory.com/21)
