# Locking Reads (FOR UPDATE / SKIP LOCKED)

- 일반 `SELECT`는 잠금 없는 consistent read라 스냅샷만 읽고 다른 트랜잭션의 수정을 막지 못함
- "읽은 값을 근거로 갱신"(check-then-act)을 안전하게 하려면 읽는 행에 잠금을 거는 **locking read**가 필요
- InnoDB는 `FOR SHARE` / `FOR UPDATE`와 수식어 `NOWAIT` / `SKIP LOCKED`(8.0+)를 제공

## Locking Read 종류

### SELECT ... FOR SHARE
- 읽는 행에 공유(S) 락 — 다른 트랜잭션은 읽을 수 있으나 수정·`FOR UPDATE` 불가
- 여러 트랜잭션이 동시에 `FOR SHARE` 보유 가능
- 구버전 문법은 `LOCK IN SHARE MODE`

### SELECT ... FOR UPDATE
- 읽는 행과 인덱스 항목에 배타(X) 락 — `UPDATE`처럼 취급
- 다른 트랜잭션의 수정·`FOR SHARE`·`FOR UPDATE`를 모두 차단

```sql
SELECT * FROM account WHERE id = 1 FOR UPDATE;  -- 이 행을 잠그고 최신값 읽기
```

## Consistent Read vs Current Read

- **Consistent read**(일반 `SELECT`): MVCC 스냅샷을 읽음, 잠금 없음
- **Current read**(`FOR UPDATE`/`FOR SHARE`/`UPDATE`/`DELETE`): **최신 커밋 버전**을 읽고 잠금
- 그래서 REPEATABLE READ에서도 locking read는 스냅샷이 아니라 최신값을 본다는 점이 일반 `SELECT`와 다름

## NOWAIT / SKIP LOCKED

### NOWAIT
- 잠그려는 행이 이미 잠겨 있으면 **대기하지 않고 즉시 에러** 반환
- "지금 못 잠그면 실패"가 필요한 경로에 사용

### SKIP LOCKED
- 다른 트랜잭션이 잠근 행은 **건너뛰고 잠글 수 있는 행만** 반환 (절대 대기하지 않음)
- 건너뛰므로 결과가 비결정적(일관된 스냅샷이 아님) → 대신 경합 없이 "사용 가능한 행만" 집는 데 최적

```sql
SELECT id FROM jobs
WHERE status = 'ready'
ORDER BY id
LIMIT 10
FOR UPDATE SKIP LOCKED;   -- 안 잠긴 작업 10개만 원자적으로 선점
```

## 잠금 범위와 갭 락 (InnoDB)

- 기본 격리수준 REPEATABLE READ에서 검색·스캔에 **next-key 락**(레코드 락 + 직전 갭 락)을 걸어 phantom 방지
- 잠금 범위는 인덱스에 의존
    - **유니크 인덱스 + 동등 검색**: 찾은 레코드 하나만 잠금 (갭 락 없음)
    - **비유니크 인덱스·범위 검색**: 스캔한 범위에 갭/next-key 락 → 그 갭으로의 INSERT 차단
- READ COMMITTED에선 갭 락이 대부분 비활성(FK·중복키 검사 용도만) → 매칭 안 된 행의 락도 곧 해제
- 따라서 `SKIP LOCKED`로 정확히 "원하는 행만" 잠그려면 인덱스를 태우거나 READ COMMITTED를 쓰는 게 유리

## SKIP LOCKED로 작업 큐 (DB-as-queue)

- 여러 워커가 같은 테이블에서 `FOR UPDATE SKIP LOCKED LIMIT n`으로 **서로 겹치지 않는 행 집합을 비차단으로 선점**
- 흐름: 짧은 트랜잭션으로 claim(상태를 processing으로) → 락 밖에서 처리 → 완료 마킹
- 별도 메시지 큐 없이 DB만으로 워커 분산·work-stealing 구현 가능 (단 폴링 부하·락 churn이 DB에 집중되는 한계)

## MySQL vs PostgreSQL

- 문법(`FOR UPDATE`/`FOR SHARE` + `NOWAIT`/`SKIP LOCKED`)과 `SKIP LOCKED`의 기본 의미는 거의 동일하나, 내부 동작은 다름

| 구분 | MySQL (InnoDB) | PostgreSQL |
|---|---|---|
| 기본 격리수준 | REPEATABLE READ | READ COMMITTED |
| 갭·next-key 락 | 있음 (RR에서 phantom 방지) | 없음 (직렬화는 SERIALIZABLE의 SSI predicate lock) |
| locking read 충돌 시 | current read로 최신 읽고 **블록(대기)** | RR/SERIALIZABLE이면 **직렬화 에러→재시도**, RC면 대기 후 최신 재평가 |
| MVCC 저장 | undo log로 이전 버전 재구성 | heap에 다중 버전 보관(+ VACUUM 정리) |
| 행 잠금 모드 | FOR UPDATE / FOR SHARE (2종) | FOR UPDATE / FOR NO KEY UPDATE / FOR SHARE / FOR KEY SHARE (4종) |
| SKIP LOCKED / NOWAIT | 8.0+ | 9.5+ |

- **갭 락 유무**가 가장 큰 차이: MySQL은 RR에서 범위·비유니크 검색 시 갭까지 잠가 `SKIP LOCKED`가 갭 락과 얽힘. PostgreSQL은 갭 락 자체가 없어 매칭된 행만 잠금
- **충돌 처리 철학**: 같은 RR이라도 MySQL은 "최신 읽고 블록", PostgreSQL은 "스냅샷 이후 변경된 행이면 직렬화 실패 에러" → 한쪽은 대기, 한쪽은 재시도를 전제
- PostgreSQL은 FK 의미에 맞춘 `FOR KEY SHARE`/`FOR NO KEY UPDATE`로 잠금 모드를 더 세분화

## 참고

- [MySQL — Locking Reads](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-reads.html)
- [MySQL — InnoDB Locking (record/gap/next-key)](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)
- [PostgreSQL — Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html)
- [PostgreSQL — Row-Level Locks](https://www.postgresql.org/docs/current/explicit-locking.html#LOCKING-ROWS)
