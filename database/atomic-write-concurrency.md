# 원자적 쓰기로 동시성·멱등 제어

- DB로 동시성을 제어하는 방식은 크게 4갈래: **비관적 락 / 낙관적 락 / 원자적 쓰기 / 범위 제약**
- 이 노트는 그중 **원자적 쓰기** — 별도 락 없이 **단일 SQL 문장의 원자성**으로 race·중복을 제어하는 기법
- 세 가지: ① 원자적 조건부 UPDATE ② UNIQUE 제약 backstop ③ upsert / insert-or-ignore

## 1. 원자적 조건부 UPDATE

```sql
UPDATE stock SET qty = qty - 1 WHERE id = ? AND qty >= 1;
```

- 한 문장 안에서 **조건 검사 + 갱신**이 일어나 row 락으로 원자적 → check-then-act race가 없음
- **affected rows = 0**이면 조건 불충족(품절)으로 판정
- 장점: 애플리케이션 레벨 락·사전 조회 불필요, 가장 단순
- 한계: 같은 행에 요청이 몰리면 row 락이 직렬화돼 **hot row 병목** → 고경합은 다른 갈래(낙관/분산) 고려

## 2. UNIQUE 제약 = race·중복 최종 방어선

- 동시 INSERT를 DB가 유니크 인덱스로 **직렬화** → 한쪽만 성공하고 나머지는 위반
- 용도: 멱등(중복 주문/이벤트 차단), 오버셀 backstop, dedup
- 애플리케이션 락이 뚫려도 최종적으로 막아주는 "last line of defense" → 분산 락 + DB 유니크의 다층 방어가 정석
- PostgreSQL은 `DEFERRABLE INITIALLY DEFERRED`로 커밋 시점 검사 가능
- 위반을 에러가 아니라 정상 흐름으로 흡수하려면 → upsert(아래)

## 3. Upsert / insert-or-ignore

- 충돌(UNIQUE/PK 위반) 시 대체 동작을 지정 → 위반을 "건너뛰기(멱등)" 또는 "갱신(upsert)"으로 처리
- 전제: 충돌을 감지할 UNIQUE 인덱스/제약 필요

### PostgreSQL — ON CONFLICT (9.5+)

```sql
-- insert-or-ignore (멱등): 이미 있으면 건너뜀
INSERT INTO processed(idem_key) VALUES (?) ON CONFLICT (idem_key) DO NOTHING;

-- upsert: 있으면 갱신
INSERT INTO stock(id, qty) VALUES (?, ?) ON CONFLICT (id) DO UPDATE SET qty = EXCLUDED.qty;

-- 누적 upsert
INSERT INTO counter(key, n) VALUES (?, 1)
ON CONFLICT (key) DO UPDATE SET n = counter.n + EXCLUDED.n;
```

- **conflict_target**: `(컬럼)` / `ON CONSTRAINT 이름` / 부분 인덱스 `(컬럼) WHERE ...`. `DO UPDATE`는 target 필수, `DO NOTHING`은 생략 가능
- **EXCLUDED**: 삽입하려던 행. 기존 행은 테이블명으로 → `counter.n + EXCLUDED.n`
- `RETURNING` 가능하나, `DO NOTHING`으로 건너뛴 **기존 행은 반환하지 않음**
- arbiter는 UNIQUE만 — **exclusion 제약은 대상 아님** ([exclusion-constraint](postgresql/exclusion-constraint.md))

### MySQL

- `INSERT IGNORE` — insert-or-ignore. 단 중복키 외 **다른 에러까지 경고로 묵살**하므로 주의
- `INSERT ... ON DUPLICATE KEY UPDATE ...` — upsert. PK·유니크 중 아무거나 걸리면 반응 → **특정 제약 지정 불가**
- `REPLACE INTO` — delete 후 insert. auto_increment·FK·트리거 부수효과로 upsert 용도엔 부적합

### MERGE (PostgreSQL 15+)

- SQL 표준. `WHEN MATCHED`/`WHEN NOT MATCHED`로 복잡한 분기(+DELETE) 가능
- 단 **동시성 안전성은 `ON CONFLICT`가 우위**(MERGE는 동시 INSERT 시 unique 위반 가능) → 단순 동시-안전 upsert는 ON CONFLICT, 복잡 로직은 MERGE

### 동시성

- 문장 단위로 원자적 — 동시 INSERT를 유니크 인덱스가 직렬화(내부적으로 speculative insertion)해 한쪽은 insert, 다른 쪽은 대체 동작
- `DO UPDATE`/`ON DUPLICATE KEY UPDATE`는 갱신 대상 행에 락. REPEATABLE READ 이상에선 동시 갱신 시 직렬화 에러 → 재시도
- 그래서 멱등·중복·race의 backstop으로 활용 (idempotency key 저장, dedup)

## 4갈래 비교

| 갈래 | 기법 | 특징 | 노트 |
|---|---|---|---|
| 비관적 락 | `FOR UPDATE` / `SKIP LOCKED` | 잠그고 진행, 직렬화 안전 | [locking-reads](mysql/concurrency-control/locking-reads.md) |
| 낙관적 락 | `@Version` + 재시도 | 충돌 시 재시도, 고경합 retry storm | [optimistic-lock](optimistic-lock.md) |
| **원자적 쓰기** | 조건부 UPDATE · UNIQUE · upsert | 단일 문장 원자성, 락 최소 | (이 노트) |
| 범위 제약 | exclusion constraint | 구간 겹침 선언적 금지(PG) | [exclusion-constraint](postgresql/exclusion-constraint.md) |

## 참고

- [PostgreSQL — INSERT (ON CONFLICT)](https://www.postgresql.org/docs/current/sql-insert.html#SQL-ON-CONFLICT)
- [PostgreSQL — MERGE](https://www.postgresql.org/docs/current/sql-merge.html)
- [MySQL — INSERT ... ON DUPLICATE KEY UPDATE](https://dev.mysql.com/doc/refman/8.0/en/insert-on-duplicate.html)
