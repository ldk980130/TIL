# MySql(InnoDB) Lock
## Shared and Exclusive Locks

- **shared lock**은 트랜잭션이 row를 읽기 위해 거는 락
    - 여러 트랜잭션이 동시에 데이터를 읽어도 문제 없고 shared lock끼리는 동시에 접근 가능
- **exclusive lock**은 트랜잭션이 row를 update하거나 delete하기 위해 거는 락
    - exclusive lock을 동시에 걸 순 없고 다른 트랜잭션이 읽는 것조차 막는다.

만약 트랜잭션 t1이 row r1에 shared lock을 걸었다면

- 트랜잭션 t2의 r1에 대한 s lock은 허용된다.
- t2의 x lock은 허용되지 않는다.

만약 t1이 r1에 x lock을 걸었다면 t2는 어떤 lock도 걸지 못한다. t1이 lock을 해제할 때까지 기다릴 수 밖에 없다.

## Intention Locks

- InnoDB는 row 잠금과 table 잠금을 함께 허용하는 다중 단위 락(multiple granularity locking)을 허용한다.
- **Intention locks**은 Talble-level lock으로, row에 대해서 나중에 어떤 row-level lock을 걸 것인지 미리 알려주기 위해 미리 table-level에 걸어두는 lock
    - **Intention shared lock**(IS) - `select … for share`
    - **Intention exclusive lock**(IX) - `select … for update`
- 트랜잭션은 s lock이나 x lock을 획득하기 위해서 IS lock 이나 IX lock을 먼저 획득해야 한다.

다음은 서로 다른 lock들이 서로 충돌하는지 호환 가능한지 요약한 표이다.

|  | X | IX | S | IS |
| --- | --- | --- | --- | --- |
| X | Conflict | Conflict | Conflict | Conflict |
| IX | Conflict | Compatible | Conflict | Compatible |
| S | Conflict | Conflict | Compatible | Compatible |
| IS | Conflict | Compatible | Compatible | Compatible |

- IS, IX lock은 여러 트랜잭션에서 동시에 접근 가능하다.
- 하지만 실제 row-level lock인 s lock, x lock에서 접근 제어를 하게 된다.
- 테이블 자체에 lock을 걸거나 alter table, drop table 등을 실행할 때는 IS, IX를 모두 block하는 table-level lock이 걸린다.
  - ex) row-level의 x lock이 걸려있을 때 테이블 스키마가 변경되서는 안 되기 때문에 IX lock을 table-level에 걸어 두어 alter table의 접근을 막는다.
## Record Locks

- InnoDB에서 **Record lock**은 row가 아닌 index record에 걸리는 lock이다.
  - 예) `select c1 from t where c1 = 10 for update`
  - c1 컬럼에 인덱스를 설정했다고 했을 때 다른 트랜잭션은 t 테이블의 c1이 10인 row들의 데이터에 접근하지 못한다.
- InnoDB는 항상 index record에 lock을 건다. 따로 인덱스를 설정하지 않았더라도 InnoDB는 pk에 clustered index를 만들어 놓는다.

## Gap Locks

- **Gap lock**은 레코드 간의 간격(gap)에 대한 lock을 의미한다.
  - gap이란 index 중 DB에 실제 record가 없는 부분이다.
- 예) id에 인덱스 설정이 되어 있고 현재 전체 데이터베이스에 id=4, id=6밖에 없다고 가정하자.
  - id < 3, 4 < id < 6, 6 < id인 부분들은 현재 값이 없는 index record들이다.
  - 이 세 부분을 gap이라고 하고 gap lock이라 함은 이 세 부분에 대한 접근을 방지한다.
  - 에를 들어 트랜잭션 t1이 `…id between 1 and 10 for update`를 걸었다면 실제 index record가 없는 1 ≤ id < 4, 5 ≤ id < 6, 7 ≤ id < 11에 gap lock이 걸려 저기에 해당하는 id 값으로 insert를 할 수 없다.
  - id = 4, id = 6에 접근할 수 없는 이유는 record lock 때문이다.
- record lock 은 이미 존재하는 row가 변경 되지 않게 보호 하는 반면, gap lock 은 조건에 해당하는 새로운 row 가 추가 되는 걸 방지

## Next-key Locks

- record lock과 gap lock을 함께 사용하는 lock
- 트랜잭션 격리 레벨  REPEATABLE READ에서 phantom read를 방지하게 해줌

## Insert Intention Locks

- insert 실행 시 획득하는 특수한 gap lock
- 이는 서로 다른 트랜잭션이 index gap에 insert하려고 할 때 같은 row에 삽입하는 것이 아니라면 gap lock을 기다릴 필요가 없다는 것을 뜻한다.
- 예) id=4, id=7만 있는 데이터베이스가 있다고 하자.
  - 트랜잭션 t1이 id=5에 insert를 시도하고 트랜잭션 t2가 살짝 늦게 id=6을 insert한다고 가정
  - t1이 5< id <7에 x lock을 걸기 전에 insert intention lock을 건다.
  - t2는 id=6에 걸려 있는 insert intention lock을 보지만 id가 겹치지 않기 때문에 기다리지 않고 insert를 수행한다.

---

[https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)

[https://velog.io/@soyeon207/DB-Lock-총정리-1-InnoDB-의-Lock](https://velog.io/@soyeon207/DB-Lock-%EC%B4%9D%EC%A0%95%EB%A6%AC-1-InnoDB-%EC%9D%98-Lock)

[https://www.youtube.com/watch?v=onBpJRDSZGA&t=723s](https://www.youtube.com/watch?v=onBpJRDSZGA&t=723s)
