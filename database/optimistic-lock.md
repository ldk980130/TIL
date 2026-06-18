# 낙관적 락 (Optimistic Locking)

- 충돌이 드물다고 가정하고 **잠그지 않은 채 진행** → 커밋 시점에 충돌을 *감지*해 재시도/실패시키는 동시성 제어
- 비관적 락(미리 잠금)의 반대. DB 동시성 4갈래 중 하나 ([원자적 쓰기](atomic-write-concurrency.md) 비교표 참고)

## 메커니즘 — version 컬럼

- 행에 `version`(또는 timestamp) 컬럼을 둠
- 읽을 때 version을 기억 → 갱신 시 그 version으로 **조건부 UPDATE**

```sql
UPDATE account SET balance = ?, version = version + 1
WHERE id = ? AND version = :read_version;   -- affected rows = 0 이면 그새 누가 바꿈 → 충돌
```

- DB 레벨 CAS(compare-and-set). affected rows로 충돌을 판정

## JPA @Version

- 엔티티에 `@Version` 필드(`Long`/`int`/timestamp)
- flush/commit 시 Hibernate가 자동으로 `WHERE version = ?`를 붙이고 version을 증가
- 충돌 시 `OptimisticLockException`(Spring의 `OptimisticLockingFailureException`)
- 더티 체킹과 결합돼 별도 코드 없이 동작

## 재시도와 결합

- 낙관적 락은 충돌을 *감지*만 함 → 보통 **충돌 시 트랜잭션을 처음(읽기)부터 재시도**
- AOP나 `@Retryable` 등으로 감싸고 짧은 backoff·최대 횟수를 둠 ([재시도](../design/resilience/retry.md))
- 고경합이면 재시도가 폭증(retry storm) → 비관적 락·큐잉으로 전환하라는 신호

## 비관적 락과 비교

| | 낙관적 락 | 비관적 락 |
|---|---|---|
| 방식 | 진행 후 커밋 시 충돌 감지 | 미리 잠금 |
| happy-path 비용 | 락 0 | 락 획득·대기 |
| 충돌 시 | 재시도 | 대기(블록) |
| 적합 | 저경합 | 고경합 |
| 데드락 | 없음(락 미보유) | 가능 |

## 한계와 함정

- **고경합 retry storm**: 인기 항목 하나에 몰리면 재시도가 폭증해 지연·실패
- **읽고 쓰지 않으면 감지 안 됨**: 조회만 하는 트랜잭션엔 무력
- **write-skew는 못 막음**(가장 중요): 낙관적 락은 *내가 수정하는 그 행*의 version 충돌만 잡음 → 서로 다른 행에 걸친 불변식(겹치는 두 예약 등)은 감지 못 함
- timestamp 버전은 시계 역전 위험 → 단조 증가 정수 version 권장

## write-skew와 충돌 구체화(materializing conflicts)

- 신규 INSERT 2건이 "서로의 미커밋 행을 못 봐서 둘 다 통과"하는 write-skew는 일반 낙관적 락으로 못 막음
- 해결책
    - **충돌 구체화(materializing conflicts)**: 충돌 가능한 **공유 부모 행의 version을 강제로 올려**(force-increment) 원래 없을 write-write 충돌을 인위적으로 만듦 → 한쪽만 커밋, 진 쪽은 재시도 시 걸려 종료
    - **범위 제약**: DB가 PostgreSQL이면 [exclusion constraint](postgresql/exclusion-constraint.md)로 선언적 해결 (MySQL엔 없어 위 방식으로 우회)

## 요약

- 낙관적 락 = "충돌이 드물다"는 베팅 + version CAS + 재시도
- 저경합·데드락 회피에 강하고, 고경합엔 약함(retry storm)
- **행 단위 충돌만** 잡으므로, 교차 불변식은 충돌 구체화나 범위 제약으로 보완

## 참고

- [Hibernate ORM — Optimistic locking](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#locking-optimistic)
- 데이터 중심 애플리케이션 설계(DDIA) 7장 — write skew, materializing conflicts
