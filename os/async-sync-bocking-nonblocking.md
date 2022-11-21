# Asynchronous, Synchronous, Blocking, Non-blocking
## Blocking vs Non-blocking

- 다른 주체가 작업할 때 자신의 **제어권이** 있는지 없는지로 확인할 수 있다.
- 내가 직접 제어할 수 없는 대상을 상대하는 방법
- 대상이 제한적임
    - IO
    - 멀티 스레드 동기화

### Blocking

함수가 다른 함수를 호출했을 때 **호출된 함수**가 작업을 끝낼 때까지 **호출한 함수**에게 제어권을 돌려주지 않는 것

### Non-blocking

함수가 다른 함수를 호출했을 때 **호출된 함수**가 **호출한 함수**에게 제어권을 바로 돌려주어 **호출한 함수**가 다른 일을 할 수 있게 하는 것

## Synchronous vs Asynchronous

- 결과를 돌려주었을 때 **순서**와 **결과**에 관심이 있는지 아닌지로 판단할 수 있다.
- 또는 A와 B가 있을 때 시작 시간 또는 종료 시간이 일치하면 동기라고 한다.
    - A, B 스레드가 동시에 작업을 시작할 때
    - A가 B를 호출했을 때 B의 결과 리턴 시간과 A가 결과를 받는 시간이 일치할 때

### Synchronous

함수가 다른 함수를 호출했을 때 **호출된 함수**의 **결과**를 **호출한 함수**가 신경 쓰고 받아 처리할 수도 있는 것

### Asynchronous

함수가 다른 함수를 호출했을 때 **호출된 함수**의 **결과**를 **호출한 함수**가 전혀 신경 쓰지 않는 것

## 4가지 조합의 경우

### Blocking/Sync

함수가 다른 함수를 호출했을 때 **제어권**을 **호출된 함수**에게 넘겨주어 기다린다. **호출된 함수**가 **결과**를 리턴하면 그 **결과**를 받아 작업을 마저 끝낸다.

**예)** I/O 작업의 경우 입력이 들어올 때까지 프로그램 실행이 멈춘다.

### Non-Blocking/Sync

함수가 다른 함수를 호출하고 **제어권**을 다시 가져와 자신의 일을 하지만, **호출된 함수**가 **결과**를 리턴했는지 계속 확인하다가 **결과**를 리턴하면 **결과**를 받아 처리한다.

**코드 예)**

```java
ExecutorService es = Executors.newFixedThreadPool(10);
Future<String> future = es.submit(() -> "Hello World");

// some operations

while (!futre.isDone()) {
    log.info("Task Completed");
}
String result = future.get();
// 결과 처리
```

**예)**

```java
ExecutorService es = Executors.newCachedThreadPool();
Future<?> res = es.submit(() -> "Hello Async");

// ...다른 작업...

while (!res.isDone()) {
    Object result = res.get();
}
```

위 코드는 async 호출로 제어권을 다시 가져오는 Non

1. 일단 Non-Blocking/Async로 다른 작업 호출 (`es.submit()`)
2. 제어권을 다시 가져와 작업을 하다가 (Non-Blocking의 특성)
3. 호출했던 비동기 작업의 결과를 기다림 (`!future.isDone()`) (Sync의 특성)
4. 결과를 받고 처리 (`futre.get()`)

### Blocking/Async

함수가 **호출된 함수**에게 **제어권**을 넘기고 기다리지만 **결과**를 신경쓰지 않는다.

비효율적인 조합이라 직접적으로 사용하지는 않지만 의도치 않게 동작할 때가 있다고 한다.

**예)**

대표적인 예로는 Node.js와 MySQL을 함께 사용하는 경우이다. Node.js는 비동기로 작업하려 하지만 MySQL 드라이버가 Blocking 방식으로 동작하므로 어쩔 수 없이 Asynchronous Blocking 방식으로 동작하게 된다.

**예)**

```java
ExecutorService es = Executors.newCachedThreadPool();
es.submit(() -> "Hello Async").get();
```

위 코드는 다른 스레드로 작업을 진행 시켜 (`submit()`) async이지만 리턴되는 `Future` 객체의 `get()` 메서드로 결과를 받아오기 기다리면셔 blocking으로 동작한다.

### Non-Blocking/Async

다른 함수를 **호출한 함수와** **호출된 함수**가 서로가 **제어권**을 가지고 작업을 진행하고 **호출된 함수**의 **결과**를 **호출한 함수**가 신경 쓰지 않는다.

---

### 참고

[https://www.youtube.com/watch?v=oEIoqGd-Sns](https://www.youtube.com/watch?v=oEIoqGd-Sns)

[https://www.youtube.com/watch?v=IdpkfygWIMk](https://www.youtube.com/watch?v=IdpkfygWIMk)

[https://youtu.be/HKlUvCv9hvA](https://youtu.be/HKlUvCv9hvA)

[https://velog.io/@codemcd/Sync-VS-Async-Blocking-VS-Non-Blocking-sak6d01fhx](https://velog.io/@codemcd/Sync-VS-Async-Blocking-VS-Non-Blocking-sak6d01fhx)
