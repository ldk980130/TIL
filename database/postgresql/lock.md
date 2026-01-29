# Advisory Lock

## Advisory Lock 정의

- Advisory Lock은 애플리케이션이 정의한 의미를 가진 잠금을 생성할 수 있는 PostgreSQL의 기능이다.
- 일반적인 row lock이나 table lock과 달리 PostgreSQL이 자동으로 강제하지 않으므로, 애플리케이션이 올바르게 사용해야 한다.
- 잠금은 공유 메모리 풀에 저장되며, `max_locks_per_transaction`과 `max_connections` 설정에 따라 크기가 결정된다.

## 세션 레벨 vs 트랜잭션 레벨

Advisory Lock은 두 가지 범위로 획득할 수 있다.

| 특성 | 세션 레벨 | 트랜잭션 레벨 |
|------|---------|-------------|
| 해제 시점 | 명시적 해제 또는 세션 종료 | 트랜잭션 종료 시 자동 해제 |
| 트랜잭션 롤백 | 롤백 후에도 유지됨 | 자동으로 해제됨 |
| 명시적 unlock | 필요 | 불필요 |
| 사용 패턴 | 장기 잠금 | 단기 사용 |
| 스택 가능 | O (획득 횟수만큼 해제 필요) | X |

- 세션 레벨 잠금은 같은 잠금을 여러 번 획득할 수 있으며, 획득한 횟수만큼 해제해야 완전히 해제된다.
- 트랜잭션 레벨 잠금은 명시적으로 해제할 수 없으며 트랜잭션 종료 시 자동 해제된다.

## 배타적 잠금 vs 공유 잠금

| 잠금 유형 | 동일 리소스 공유 잠금 | 동일 리소스 배타적 잠금 |
|----------|---------------------|----------------------|
| 배타적(Exclusive) | 충돌 | 충돌 |
| 공유(Shared) | 허용 | 충돌 |

- 배타적 잠금은 동일 리소스에 대해 다른 모든 잠금과 충돌한다.
- 공유 잠금은 다른 공유 잠금과는 호환되지만, 배타적 잠금과는 충돌한다.

## 함수 목록

### 세션 레벨 잠금 함수

| 함수 | 반환 타입 | 설명 |
|-----|---------|------|
| `pg_advisory_lock(key bigint)` | void | 배타적 잠금 획득 (대기) |
| `pg_advisory_lock(key1 int, key2 int)` | void | 배타적 잠금 획득 (2개 정수 키) |
| `pg_advisory_lock_shared(key bigint)` | void | 공유 잠금 획득 (대기) |
| `pg_try_advisory_lock(key bigint)` | boolean | 배타적 잠금 시도 (대기 없음) |
| `pg_try_advisory_lock_shared(key bigint)` | boolean | 공유 잠금 시도 (대기 없음) |
| `pg_advisory_unlock(key bigint)` | boolean | 배타적 잠금 해제 |
| `pg_advisory_unlock_shared(key bigint)` | boolean | 공유 잠금 해제 |
| `pg_advisory_unlock_all()` | void | 현재 세션의 모든 잠금 해제 |

### 트랜잭션 레벨 잠금 함수

| 함수 | 반환 타입 | 설명 |
|-----|---------|------|
| `pg_advisory_xact_lock(key bigint)` | void | 배타적 잠금 획득 (대기) |
| `pg_advisory_xact_lock(key1 int, key2 int)` | void | 배타적 잠금 획득 (2개 정수 키) |
| `pg_advisory_xact_lock_shared(key bigint)` | void | 공유 잠금 획득 (대기) |
| `pg_try_advisory_xact_lock(key bigint)` | boolean | 배타적 잠금 시도 (대기 없음) |
| `pg_try_advisory_xact_lock_shared(key bigint)` | boolean | 공유 잠금 시도 (대기 없음) |

- `try` 함수는 잠금을 즉시 획득하면 `true`, 획득할 수 없으면 대기 없이 `false`를 반환한다.
- `unlock` 함수는 잠금이 성공적으로 해제되면 `true`, 해당 잠금을 보유하지 않았으면 `false`를 반환한다.

## 사용 예시

### 기본 사용

```sql
-- 배타적 세션 잠금 획득
SELECT pg_advisory_lock(12345);

-- 작업 수행 후 해제
SELECT pg_advisory_unlock(12345);
```

### 트랜잭션 레벨 잠금

```sql
BEGIN;
SELECT pg_advisory_xact_lock(12345);
-- 작업 수행
-- 트랜잭션 종료 시 자동 해제
COMMIT;
```

### try 함수로 논블로킹 처리

```sql
-- 잠금 획득 시도, 실패 시 대기하지 않고 즉시 반환
SELECT pg_try_advisory_lock(12345);
-- true면 잠금 획득 성공, false면 다른 세션이 보유 중
```

## 사용 사례

1. **분산 작업 조율**: 여러 애플리케이션 인스턴스가 동일한 작업을 중복 실행하지 않도록 조율한다.
2. **비관적 잠금(Pessimistic Locking) 구현**: row lock 없이 애플리케이션 레벨에서 동시성을 제어한다.
3. **MVCC 모델이 부적절한 경우**: 테이블 플래그보다 빠르고 테이블 블로트를 방지하면서 잠금을 관리한다.
4. **배치 작업 중복 방지**: 스케줄러로 실행되는 배치 작업이 동시에 여러 번 실행되지 않도록 한다.

## 주의사항

### LIMIT 절과의 상호작용

Advisory Lock 함수를 `LIMIT`과 함께 사용할 때는 주의해야 한다. PostgreSQL의 쿼리 플래너는 함수 호출 순서를 보장하지 않는다.

```sql
-- 위험: LIMIT가 locking 함수 전에 실행된다는 보장이 없음
SELECT pg_advisory_lock(id) FROM foo WHERE id > 12345 LIMIT 100;
```

- 예상하지 못한 행에 대해 잠금이 획득될 수 있다.
- 애플리케이션이 해제하지 못하는 "dangling lock"이 발생할 수 있다.

```sql
-- 안전: 단순 케이스
SELECT pg_advisory_lock(id) FROM foo WHERE id = 12345;

-- 안전: 서브쿼리로 LIMIT 먼저 적용
SELECT pg_advisory_lock(q.id) FROM (
    SELECT id FROM foo WHERE id > 12345 LIMIT 100
) q;
```

### 메모리 제한

- Advisory Lock은 공유 메모리 풀에 저장되므로 무제한으로 사용할 수 없다.
- 일반적으로 수십만 개 정도의 잠금만 가능하며, 메모리가 소진되면 모든 잠금 요청이 거부될 수 있다.

### 잠금 해제 누락

- 세션 레벨 잠금은 명시적으로 해제해야 하며, 해제하지 않으면 세션이 종료될 때까지 유지된다.
- 같은 잠금을 여러 번 획득한 경우 획득 횟수만큼 해제해야 완전히 해제된다.

## 현재 잠금 조회

`pg_locks` 시스템 뷰에서 advisory lock을 조회할 수 있다.

```sql
SELECT * FROM pg_locks WHERE locktype = 'advisory';
```

## 레퍼런스

- https://www.postgresql.org/docs/current/explicit-locking.html#ADVISORY-LOCKS
- https://www.postgresql.org/docs/current/functions-admin.html#FUNCTIONS-ADVISORY-LOCKS
