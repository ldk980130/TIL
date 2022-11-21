# Lock & 2PL

## 2PL protocol (two-phase locking)

- 트랜잭션에서 모든 locking operation이 최초의 unlock operation 보다 먼저 수행되도록 하는 것
- **Expanding phase** (growing phase)
    - lock을 취득하기만 하고 반환하지 않는 phase
- **Shrinking phase** (contracting phase)
    - lock을 반환만 하고 취득하지는 않는 phase
- 2PL 프로토콜은 Serializability를 보장한다.

## 2PL과 Deadlock

- Serializablity하기 때문에 이상한 결과가 나오지는 않지만 Deadlock 상태가 발생할 수 있다.

## 2PL 종류

- **Conservative 2PL**
  - 모든 lock을 취득한 뒤 트랜잭션 시작
  - deadlock이 발생하지 않는다.
  - 실제로 모든 lock을 모두 얻기 힘든 경우도 있기 때문에 실용적이지 않다.
- **Strict 2PL (S2PL)**
  - strict schedule을 보장하는 2PL
    - 커밋 되지 않은 데이터는 읽지도 쓰지도 않는 schedule
  - recoverability 보장
  - write-lock을 commit/rollback 될 때 반환

- **String Strict 2PL (SS2PL or rigorous 2PL)**
  - strict schedule을 보장하는 2PL
  - recoverability 보장
  - read-lock/write-lock 모두 commit/rollback 될 때 반환
  - lock을 오래 쥐고 있으니 다른 트랜잭션의 기다리는 시간이 길어진다.


### 참고

[https://www.youtube.com/watch?v=0PScmeO3Fig](https://www.youtube.com/watch?v=0PScmeO3Fig)
