# Vitess — MySQL 수평 샤딩

- 애플리케이션 수정 없이 MySQL을 수평 샤딩하는 클러스터링/프록시 시스템
- 앱은 단일 MySQL처럼 **VTGate에 MySQL 프로토콜로 연결** → 샤딩·라우팅·커넥션 풀링·페일오버를 Vitess가 투명하게 처리
- YouTube가 개발, CNCF 졸업 프로젝트. Shopify·PlanetScale·Slack 등이 사용
- 앞서 본 [논리/물리 샤드·라우팅·리샤딩](logical-physical-shard.md)을 실제로 구현하는 대표 도구

## 아키텍처

- **VTGate**: 무상태 쿼리 라우터(프록시). SQL을 파싱해 대상 샤드를 결정, scatter-gather·커넥션 풀링·페일오버 담당. 앱이 연결하는 지점(MySQL 프로토콜)
- **VTTablet**: 각 MySQL 인스턴스 옆의 에이전트. 쿼리 서빙·복제 관리·헬스체크·MySQL 커넥션 풀
- **MySQL**: 실제 백엔드. 샤드 하나 = primary + replica들
- **Topology Service**: etcd/ZooKeeper/Consul. 샤드 분포·노드 상태 등 클러스터 메타데이터 보관
- **vtctld / vtctldclient**: 클러스터 관리 — 리샤딩 등 워크플로 실행

## 핵심 개념

### Keyspace
- 논리적 데이터베이스. **unsharded**(한 MySQL에 직결) 또는 **sharded**(동일 스키마 샤드들로 분할)
- 샤딩은 MySQL 복제와 독립적

### Shard / key range / keyspace_id
- 샤드는 16진 key range로 표현 (2개면 `-80`, `80-`; 분할 시 `80-c0` 식)
- 샤드 키 → vindex → **keyspace_id** 계산 → 그 값이 속한 range의 샤드로 라우팅
- 전체 range를 빈틈없이 덮어야 함 (한 샤드는 start가 비고, 한 샤드는 end가 빔)

### VSchema & Vindex
- **VSchema**: 각 테이블이 어떤 vindex로 샤딩되는지 기술
- **Vindex**: 컬럼 값 → keyspace_id 매핑
    - **Primary vindex**: 샤드 키 매핑(예: `user_id` 해시) — 행의 샤드를 결정
    - **Lookup(secondary) vindex**: 비샤드키 컬럼 → keyspace_id 매핑 테이블 → 그 컬럼으로 조회해도 scatter 없이 라우팅 가능

```json
{
  "sharded": true,
  "vindexes": { "hash": { "type": "hash" } },
  "tables": {
    "user": { "column_vindexes": [{ "column": "user_id", "name": "hash" }] }
  }
}
```

### Sequences
- 샤드 전반에서 `AUTO_INCREMENT`는 ID가 충돌하므로, unsharded 테이블 기반 **sequence**로 단조 증가 ID를 생성해 대체

## 쿼리 라우팅

- **샤드 키 조건 포함**(`WHERE user_id = ?`) → 단일 샤드로 라우팅 (이상적)
- **샤드 키 없음** → **scatter-gather**: 모든 샤드에 fan-out 후 VTGate가 결과 merge (비쌈)
- **lookup vindex**로 비샤드키 조회를 단일/소수 샤드로 좁힐 수 있음
- 따라서 스키마·쿼리를 샤드 키 중심으로 설계해 scatter를 피하는 게 핵심

## 온라인 리샤딩

- `Reshard`(샤드 분할/병합) · `MoveTables`(키스페이스 간 테이블 이동, 예: 모놀리스 → 샤드 키스페이스) 워크플로
- **VReplication**: 소스 샤드 → 타깃 샤드로 데이터 복사 + binlog 변경분을 계속 동기 (소스는 그동안 라이브 서빙)
- **SwitchTraffic**: 단계적 cutover — 읽기(RDONLY·REPLICA)를 먼저 전환해 검증한 뒤 쓰기(PRIMARY) 전환 → primary 전환 시 **수 초 read-only** 창만 발생
- 완료 후 구 샤드 정리

## 장점과 트레이드오프

- 장점
    - 앱 코드 변경 없이(단일 MySQL처럼) 수평 샤딩
    - 거의 무중단 온라인 리샤딩(VReplication + 단계적 cutover)
    - 커넥션 풀링·쿼리 보호·페일오버 내장
- 트레이드오프
    - 운영 복잡도가 큼 (컴포넌트 다수 + topology service, 보통 Kubernetes 위에서 운영)
    - 크로스 샤드 쿼리(scatter-gather)는 여전히 비쌈 → 샤드 키 설계로 단일 샤드 유지가 관건
    - 일부 SQL 제약 — 샤드를 넘는 유니크 제약은 lookup vindex 필요, `AUTO_INCREMENT`는 sequence로 대체

## 참고

- [Vitess 공식 — Sharding](https://vitess.io/docs/reference/features/sharding/)
- [Vitess 공식 — Vindexes](https://vitess.io/docs/reference/features/vindexes/)
- [Vitess 공식 — Resharding](https://vitess.io/docs/user-guides/configuration-advanced/resharding/)
- [Shopify — Vitess로 수평 확장](https://shopify.engineering/horizontally-scaling-the-rails-backend-of-shop-app-with-vitess)
