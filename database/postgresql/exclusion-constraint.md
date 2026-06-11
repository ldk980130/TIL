# Exclusion Constraint (범위 겹침 방지)

- 두 행이 "특정 키는 같으면서 범위가 겹치는" 상황을 **선언적으로 금지**하는 제약
- 일반 `UNIQUE`는 "값이 같음(점 동등성)"만 막을 수 있어 "구간이 겹침"은 표현 불가 → exclusion constraint가 이 공백을 메움
- 더블부킹·예약 충돌·시간표 중복 방지의 표준 해법
- 세 요소의 조합으로 동작: **Range Type + GiST 인덱스 + `btree_gist` 확장**

## 구성 요소

### Range Type
- 구간을 1급 값으로 표현: `tstzrange`(타임존 포함 시각)·`tsrange`·`daterange`·`int4range` 등
- 경계 표기 — `[)`(끝 제외, 기본)·`[]`·`(]`·`()`
- 주요 연산자: `&&`(겹침)·`@>`(포함)·`<@`(포함됨)·`-|-`(인접)
- 빈 범위(empty)는 무엇과도 겹치지 않음
- `multirange`(PG 14+): `tstzmultirange` 등 불연속 구간 표현

### GiST 인덱스
- B-tree는 동등·순서 질의만 가능 → 범위 겹침(`&&`)을 못 다룸
- GiST(Generalized Search Tree)가 겹침 연산을 지원 → exclusion constraint의 인덱스 기반

### btree_gist 확장
- GiST는 기본적으로 스칼라 동등(`=`)을 지원하지 않음
- `btree_gist`가 스칼라용 GiST 연산자 클래스를 추가 → `room_id WITH =` 와 `during WITH &&` 를 **한 제약에 섞을 수 있게** 함

## 정의와 동작

```sql
CREATE EXTENSION btree_gist;            -- 스칼라 = 를 GiST에서 쓰기 위해

CREATE TABLE reservation (
  room_id int,
  during  tstzrange,
  EXCLUDE USING gist (room_id WITH =, during WITH &&)   -- 같은 방 + 시간 겹침 금지
);

INSERT INTO reservation VALUES (1, '[2026-01-01 10:00, 2026-01-01 11:00)');
INSERT INTO reservation VALUES (1, '[2026-01-01 10:30, 2026-01-01 11:30)');
-- ERROR: conflicting key value violates exclusion constraint (SQLSTATE 23P01)
```

- `EXCLUDE USING gist (A WITH =, B WITH &&)` = "A가 같고 B가 겹치는 행은 공존 불가"
- INSERT/UPDATE 시 GiST 인덱스로 충돌 행을 찾아 있으면 `exclusion_violation`(23P01)으로 거부

## 동시성 — write-skew를 DB가 직렬화

- 두 트랜잭션이 동시에 겹치는 행을 넣으려 하면, GiST 검사가 한쪽을 대기시켰다가 상대가 커밋되면 **exclusion 위반으로 실패**시킴
- 즉 "충돌 없는지 SELECT → INSERT" 사이에 끼어드는 **write-skew를 애플리케이션 코드 없이 제약 하나로** 차단
- 일반 `UNIQUE` + 앱 레벨 충돌 검사로는 못 막는 동시 삽입 레이스를 선언적으로 해결

## 경계 `[)` 의 중요성

- 내장 range는 기본 `[)`(끝 제외) → `[10:00, 11:00)` 과 `[11:00, 12:00)` 은 **겹치지 않음**
- 연속 예약(앞 구간 끝 = 뒤 구간 시작)을 충돌로 오판하지 않으려면 반열림 경계가 적절

## 한계와 주의점

- **용량 1만 강제**: exclusion constraint는 "겹침 0개"만 표현 → 한 키가 N개까지 겹침 허용(용량 N)은 직접 못 함 → 단위별 행 분리나 트리거/집계 검사 필요
- `btree_gist` 확장 설치가 전제 (스칼라 `=`와 범위 `&&` 혼용 시)
- GiST는 근사 후 recheck 구조라 B-tree와 성능 특성이 다름
- `DEFERRABLE INITIALLY DEFERRED`로 커밋 시점 검사 가능 (일괄 갱신 등)

## MySQL에는 없음

- MySQL(InnoDB)엔 exclusion constraint도 GiST도 없어 범위 겹침을 선언적으로 막을 수 없음
- 그래서 MySQL에서는 애플리케이션 레벨(낙관적 락 + 충돌 검사 + 재시도, 또는 공유 자원 행에 비관적 락)로 우회해야 함
- "DB가 PostgreSQL이면 제약 한 줄, MySQL이면 동시성 코드"가 이 기능의 대조점

## 참고

- [PostgreSQL — Range Types (예약 예시 포함)](https://www.postgresql.org/docs/current/rangetypes.html)
- [PostgreSQL — Constraints (Exclusion Constraints)](https://www.postgresql.org/docs/current/ddl-constraints.html)
- [PostgreSQL — btree_gist](https://www.postgresql.org/docs/current/btree-gist.html)
