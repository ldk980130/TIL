# 논리 샤드 vs 물리 샤드와 무중단 리샤딩

- 키를 물리 서버에 직접 해시하면 서버를 늘릴 때 거의 모든 키가 재배치돼 대량 이동·다운타임이 발생
- **논리 샤드(데이터 분할 단위)와 물리 샤드(실제 서버)를 분리**하면 리밸런싱 비용이 급감 → 무중단에 가까운 리샤딩 가능
- DDIA의 "고정 파티션 수(fixed number of partitions)" 리밸런싱 전략과 동일

## 직접 매핑의 문제

- `key → hash % N(서버 수) → 서버` 방식은 N이 바뀌면 나머지 연산 결과가 통째로 달라짐 → 대부분의 키가 다른 서버로 이동해야 함
- 운영 중 이 대량 이동은 비용·다운타임이 커서 사실상 어려움 (= 일반 hash 샤딩에서 리샤딩이 까다로운 이유)

## 논리 샤드 / 물리 샤드 분리

- **논리 샤드**: 샤드 키 해시로 결정되는 논리적 분할 단위. 개수를 크게 잡아 **고정**
- **물리 샤드**: 실제 DB 서버/인스턴스. 논리 샤드 여러 개를 호스팅
- 매핑 단계: `key → 논리 샤드(불변) → 라우팅 테이블 → 물리 서버(가변)`
- 서버 추가 = **논리 샤드 몇 개를 새 서버로 통째 이동**, 키 재해싱 없음
    - 예) Notion 480 논리 / 32 물리(물리당 15개) → 호스트를 32→40→48로 늘릴 때 논리 샤드만 재분배

## 논리 샤드 개수 정하기

- 한 번 정하면 바꾸기 어려우므로 미래 규모를 덮을 만큼 충분히 크게
- **약수가 많은 수**를 골라 물리 서버 수를 유연하게 분배
    - Notion 480은 32·40·48·60·80·96... 으로 고르게 나뉨
    - 2의 거듭제곱(512 등)은 호스트를 두 배(32→64)로만 늘릴 수 있어 경직됨
- 사례: Notion 480, Instagram 4096

## 라우팅 (key → 물리 서버)

- **라우팅 테이블/디렉토리**: 논리 샤드 → 물리 서버 매핑을 별도 보관 (Notion은 앱 레벨, Vitess는 VSchema)
- **ID에 인코딩**: Instagram은 64비트 ID에 샤드 ID를 박아(41 timestamp + 13 shard + 10 sequence) ID만으로 샤드를 결정 → 별도 조회 불필요
- **미들웨어**: Vitess VTGate가 쿼리를 분석해 vindex로 적절한 샤드로 라우팅

## 무중단 리샤딩 플레이북

공통 패턴: **복사 → 동기 유지 → 검증 → 원자적 cutover**

- 1. 새 물리 샤드 provision
- 2. 이동할 논리 샤드를 복사 (기존 샤드는 계속 서빙)
- 3. 동기 유지: 신·구를 일치시킴
    - double-write: 애플리케이션이 구·신에 동시 쓰기 + catch-up 스크립트로 누락분 따라잡기 (Notion)
    - 또는 CDC/VReplication으로 변경분 스트리밍 (Vitess)
- 4. 검증: dark read로 구·신 레코드를 비교(샘플링)해 정합성 확인
- 5. cutover: 라우팅을 신 샤드로 전환
    - Vitess는 단계적 SwitchTraffic — 읽기(RDONLY·REPLICA) 먼저 전환해 건강성 확인 후 쓰기(PRIMARY) 전환 → **쓰기 전환 시 수 초 read-only** 창만 발생
- 6. 구 샤드 정리

현실 사례:
- Notion은 catch-up 때문에 **5분 정비 창**을 사용했고, "<30초로 줄였으면 LB 핫스왑으로 무중단 가능했을 것"이라 회고 → 기법 자체는 무중단 지향
- Vitess는 cutover 시 **수 초 read-only**로 거의 무중단

## 정리

- 장점: 서버 추가 시 키 재해싱 없이 논리 샤드만 이동 → 리밸런싱이 싸고 무중단에 가까움
- 비용
    - 논리 샤드 수를 미리 충분히 잡아야 함 (초과하면 다시 곤란)
    - 라우팅 계층이 critical component가 됨
    - 논리 샤드별 관리 오버헤드
- 참고로 consistent hashing(가상 노드)도 이동량을 줄이는 기법이지만, DB 샤딩 실무에선 "고정 논리 샤드"가 더 흔히 쓰임

## 참고

- [Notion — Herding elephants: sharding Postgres](https://www.notion.com/blog/sharding-postgres-at-notion)
- [Vitess — Resharding](https://vitess.io/docs/user-guides/configuration-advanced/resharding/)
- [Instagram Engineering — Sharding & IDs](https://instagram-engineering.com/sharding-ids-at-instagram-1cf5a71e5a5c)
