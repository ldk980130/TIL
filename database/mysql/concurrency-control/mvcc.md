# InnoDB MVCC
## Undo 영역

InnoDB는 multi-version 스토리지 엔진이다. InnoDB는 일관성과 롤백 같은 트랜잭션 기능을 지원하기 위해 변경된 row에 대한 버전 정보를 언두 영역에 보존한다.

## Undo logs

- Insert undo logs
    - 트랜잭션 롤백을 위해서만 쓰인다.
    - 트랜잭션이 커밋되면 바로 삭제된다.
- Update undo logs
    - 롤백 뿐만 아니라 consistency read (일관적인 읽기)를 위해서도 쓰인다.
    - 특정 row에 대해 변경된 후가 아닌 이전 버전의 정보가 필요한 트랜잭션이 모두 사라질 때까지 update undo log는 삭제할 수 없다. (일관적 읽기를 보장하기 위해)
- 만약 트랜잭션을 주기적으로 커밋하지 않는다면 update undo log들이 삭제되지 않아 undo 영역이 커져서 메모리를 차지할 수 있다.

## Delete된 row의 물리적 삭제 시점

InnoDB 엔진에서 `delete` SQL 문에 의해 제거된 row는 물리적으로 바로 삭제되지는 않는다. InnoDB는 이 row와 row의 인덱스 레코드를 `delete` SQL에 의해 생긴 undo log가 사라질 때 물리적으로 제거한다. (이 작업을 purge라고 함)

## InnoDB 버퍼 풀

RDBMS에서 레코드가 insert되거나 update될 때 랜덤 디스크 I/O가 발생한다. 이 때 인덱스가 많다면 자원을 많이 소모해야 하기 때문에 인서트 버퍼를 사용한다.

InnoDB는 변경해야 할 인덱스 페이지가 버퍼 풀에 있으면 바로 update를 수행하지만 버퍼에 없다면 랜덤 디스크 I/O를 즉시 실행하지 않고 버퍼에 저장해 둔 뒤 결과를 반환하며 성능을 향상시킨다.

## MVCC (Multi Version Concurrency Control)

락을 사용하지 않고 일관적 읽기를 제공하는 것을 목적으로 하는 DBMS가 제공하는 동시성 제어 기능이다. InnoDB에서는 이를 언두 로그를 이용해 구현하고 있다.

### 격리 레벨에 따른 특징

- **READ UNCOMMITED**
    - MVCC가 적용되지 않는다.
- **READ COMMITED**
    - 해당 데이터를 읽는 시점기 준으로 가장 최근에 commit된 데이터를 읽는다.
- **REPEATABLE READ**
    - 해당 트랜잭션이 시작하는 시점 기준으로 가장 최근에 commit된 데이터를 읽는다. (DBMS마다 다를 순 있음)
- **Serializable**
    - MVCC로 동작하기 보다는 lock으로 동작한다.
    - 모든 평범한 select 문은 암묵적으로 `select … for share`처럼 동작한다.

## 동시성 제어 예시

1. member에 대한 `insert` 실행
    1. insert undo log가 저장되지만 insert 커밋과 동시에 삭제
    2. insert 버퍼 풀에 insert된 member row가 올라감
    3. 랜덤 디스크 I/O가 발생하여 디스크에도 작업 반영
2. 이후 member에 대한 `update` 실행
    1. 버퍼 풀에 바뀐 결과가 반영
    2. update undo log가 생성되어 바뀌기 전 결과를 저장
    3. 디스크에 언제 랜덤 I/O가 일어날지 모르는 상태이지만 InnoDB는 ACID를 보장하기에 일반적으로 버퍼와 디스크의 상태는 동일할 것
    4. 아직 커밋 전
3. 다른 트랜잭션에서 해당 member row에 대해 `select` 실행

### 격리 수준에 따른 다른 결과

- READ_UNCOMMITED
    - InnoDB 버퍼 풀이나 디스크로부터 변경된 데이터를 읽어 온다.
    - row를 변경한 트랜잭션이 롤백해버리면 정합성이 깨지게 된다.
- 그 이상의 격리 수준
    - 커밋되지 않았기 때문에 변경 전 데이터인 언두 로그의 데이터를 반환한다.

### 참고

[https://dev.mysql.com/doc/refman/8.0/en/innodb-multi-versioning.html](https://dev.mysql.com/doc/refman/8.0/en/innodb-multi-versioning.html)

[https://algopoolja.tistory.com/m/115](https://algopoolja.tistory.com/m/115)

[https://www.youtube.com/watch?v=-kJ3fxqFmqA](https://www.youtube.com/watch?v=-kJ3fxqFmqA)
