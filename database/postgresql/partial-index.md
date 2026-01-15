# Partial Index

## Partial index 정의

- Partial index는 특정 조건(predicate)을 만족하는 행만 인덱싱하는 인덱스이다.
- 인덱스 엔트리는 predicate를 만족하는 행에 대해서만 존재하며, 만족하지 않는 행은 인덱스에 포함되지 않는다.

## 유용한 경우

- "자주 조회하는 데이터가 전체의 일부 조건에 몰려 있는 경우"에 적합하다.
- 공식 문서는 partial index의 주요 동기로 common values(너무 흔한 값)를 인덱싱에서 제외하는 점을 든다.
- 인덱스가 더 작아지면 다음과 같은 장점이 존재한다.
    - 사용하는 쿼리가 더 빨라진다.
    - 모든 행이 인덱스 갱신 대상이 아니므로 업데이트 비용도 줄일 수 있다.

### 사용 사례

1. **Soft Delete 패턴**: `deleted_at IS NULL`인 행만 인덱싱하여 삭제되지 않은 레코드 조회를 최적화한다.
2. **미처리 주문 조회**: 전체 주문 중 `status = 'PENDING'`인 미결제 주문은 소수지만 가장 자주 조회되므로, 해당 행만 인덱싱한다.
3. **활성 사용자 필터링**: `active = true`인 사용자만 인덱싱하여 비활성 사용자를 제외한다.
4. **이벤트 분석**: 전체 이벤트 로그 중 `event_type = 'SIGNUP'`처럼 특정 이벤트만 분석할 때,
5. **웹 서버 로그**: 대부분의 접속이 내부 IP 대역에서 발생하지만, 외부 IP 접속만 분석할 경우 `NOT (ip >= '192.168.0.0' AND ip < '192.168.255.255')`로 내부
   IP를 제외한다.

## 생성 문법

- 생성은`CREATE INDEX ... WHERE predicate`형태로 한다.
- `WHERE`뒤 predicate가 “인덱스에 들어갈 행의 범위”를 결정한다.

```sql
CREATE INDEX users_email_active_ix ON users (email) WHERE deleted_at IS NULL;
```

## predicate 함의(imply)

- partial index는 아무 쿼리에서나 자동으로 쓰이지 않는다. 쿼리 조건이 인덱스 조건과 같거나 더 좁은 범위일 때만 인덱스를 사용한다.
- 예를 들어 `CREATE INDEX orders_active_ix ON orders (created_at) WHERE status = 'ACTIVE';` 인덱스가 있을 때:

| 쿼리 조건 | 인덱스 사용 | 이유 |
|-----------|-------------|------|
| `WHERE status = 'ACTIVE' AND created_at > '2024-01-01'` | O | 인덱스 조건을 포함하면서 더 좁은 범위 |
| `WHERE status = 'ACTIVE'` | O | 인덱스 조건과 동일 |
| `WHERE created_at > '2024-01-01'` | X | `status = 'ACTIVE'` 조건이 없음 |
| `WHERE status IN ('ACTIVE', 'PENDING')` | X | 인덱스 조건보다 넓은 범위 |

- PostgreSQL은 `x < 1`이면 `x < 2`도 만족한다는 단순한 관계는 인식하지만, `status != 'DELETED'`와 `status = 'ACTIVE'`의 관계 같은 복잡한 논리는 인식하지 못한다.
- 따라서 인덱스 predicate를 쿼리 WHERE절에 그대로 포함시키는 것이 가장 확실한 방법이다.

## 부분 유니크(Partial UNIQUE) 패턴

- partial index는 `UNIQUE`와 결합해 "조건을 만족하는 행들끼리만 유니크"를 강제할 수 있다.
- 일반 UNIQUE 제약은 테이블 전체에 적용되지만, partial unique index는 특정 조건의 행에만 유니크를 적용할 수 있다.

### 예시

1. **Soft Delete에서 이메일 유니크**: 삭제된 사용자의 이메일은 재사용 가능하게 하면서, 활성 사용자끼리는 이메일 중복을 방지한다.

```sql
CREATE UNIQUE INDEX users_email_unique_active
ON users (email)
WHERE deleted_at IS NULL;
```

2. **사용자당 활성 구독 하나만 허용**: 한 사용자가 여러 구독 이력을 가질 수 있지만, 활성 상태인 구독은 하나만 존재하도록 강제한다.

```sql
CREATE UNIQUE INDEX subscriptions_one_active_per_user
ON subscriptions (user_id)
WHERE status = 'ACTIVE';
```

3. **대표 주소 하나만 허용**: 사용자가 여러 주소를 등록할 수 있지만, 기본 주소로 설정된 것은 하나만 가능하도록 한다.

```sql
CREATE UNIQUE INDEX addresses_one_primary_per_user
ON addresses (user_id)
WHERE is_primary = true;
```

- 이 패턴을 사용하면 애플리케이션 로직이 아닌 DB 레벨에서 비즈니스 규칙을 강제할 수 있어 데이터 정합성이 보장된다.

## 주의점(운영/설계 관점)

- partial index는 조건이 바뀌거나 데이터 분포가 달라지면(예: predicate를 만족하는 행 비율이 커짐) 기대했던 효과가 약해질 수 있다.
- 또한 predicate-쿼리 매칭이 어긋나면 플래너가 사용하지 못하므로, 애플리케이션 쿼리 규칙을 함께 관리해야 한다.

## 레퍼런스

- https://www.postgresql.org/docs/current/indexes-partial.html
- https://www.postgresql.org/docs/current/indexes-partial.html
- https://www.heap.io/blog/speeding-up-postgresql-queries-with-partial-indexes
- https://atlasgo.io/guides/postgres/partial-indexes

