# Metadata Lock

## MDL(Metadata Lock)이란?

- MySQL에서 테이블, 스키마, 이벤트, 트리거 등 객체의 동시 수정과 데이터 무결성 보장을 위한 내부 잠금 시스템.
- 쿼리 실행 시, 해당 객체의 메타데이터(정의/구조 정보)에 잠금이 자동으로 걸려, DDL/DML 충돌이나 구조 변경 중 데이터 손상을 예방한다.
- 트랜잭션이 종료될 때까지(커밋/롤백), 해당 객체의 메타데이터 락은 유지된다.

## MDL 락 종류 및 특징

**MDL_SHARED**

- SELECT, INSERT, UPDATE, DELETE 등 DML 쿼리 및 트랜잭션 진입 시 획득하는 **공유 락**.
- 여러 커넥션에서 동시에 획득 가능.
- 테이블 데이터를 읽거나 변경하지만, 구조 변경은 발생시키지 않는다.
- 동일 객체에 여러 사용자/세션이 병렬 접근할 수 있어, 동시성에 유리하다.

**MDL_EXCLUSIVE**

- ALTER, DROP, RENAME, CREATE 등 DDL 쿼리 실행 시 필요한 **배타 락**.
- 해당 객체를 단일 세션만 변경 가능.
- 해당 객체에 걸린 모든 공유 락이 해제된 뒤에만 락 획득 가능.
- 구조 변경 혹은 DDL 작업의 원자성과 데이터 무결성을 보장한다.

---

## 락 큐(FIFO) 구조와 대기 현상

- **MDL 락 요청은 "선입선출(FIFO)" 대기열**로 관리된다.
- 이미 DML 쿼리들(MDL_SHARED)이 병렬로 실행 중일 때, DDL(MDL_EXCLUSIVE)이 진입하면 락 대기열에 등록됨.
- **DDL이 대기열에 들어간 후**에는 새로 들어오는 모든 DML(MDL_SHARED) 쿼리들도 반드시 DDL보다 뒤에 줄을 서서 함께 대기한다.
- DDL이 실행되기 전까지, 앞선 DML 트랜잭션이 모두 커밋/종료되어 락이 해제되어야만, DDL이 락을 획득·실행한다.
- 이 구조 때문에 DDL이 해소될 때까지 역순 대기가 쌓이며, 대기시간이 예상외로 길어질 수 있다.

---

## 외래키(FK) 관계와 MDL 확장

- **테이블 간 외래키(FK)로 연결된 경우**, DDL(테이블 구조 변경)을 하면 FK 연관된 모든 부모/자식 테이블에 MDL 락이 확장된다.
- 예시: 자식 테이블 ALTER 혹은 FK 수정 시, 부모/자식 모두 **MDL_EXCLUSIVE** 락 획득.
- 공식문서에서 “Metadata locks are extended, as necessary, to tables related by a foreign key constraint...”로 설명.
- FK 관련 작업(추가/수정/삭제)에만 MDL 락이 연관 객체 전체에 확장되고,
    - 단순 UNIQUE 제약이나 NOT NULL 등은, 그 제약을 추가하는 테이블만 락이 걸린다(연결 테이블 무관).

---

## Waiting for table metadata lock

- 앞서 진입한 트랜잭션이나 세션이 해당 테이블 또는 FK 관련 테이블에 대해 **MDL_SHARED** 락을 오래 보유할 경우, 새로운 DDL(ALTER 등)은 앞선 락이 해제될 때까지 대기한다.
- DDL이 락 큐에 진입한 뒤에도, 신규로 들어오는 DML(MDL_SHARED) 쿼리 또한 무조건 큐 뒤에 배치되어 WAITING 상태에 빠진다.
- **대표 원인**:
    - 커넥션 풀의 connection leak
    - 비정상 종료된 트랜잭션
    - 트랜잭션/세션을 장시간 커밋/종료하지 않음
    - timeout 미설정, 트랜잭션이 길게 유지됨(대량/지연 쿼리)
- 진단 시 `SHOW PROCESSLIST,` `performance_schema.metadata_locks`를 적극 활용하면, 어느 세션이 락을 오래 소유하는지, 어떤 쿼리가 병목을 유발하는지 파악할 수 있다.

---

## 쿼리별 MDL 락 동작 예시

**SELECT, INSERT, UPDATE, DELETE**

- 대상 테이블에 **MDL_SHARED** 락을 획득하고 실행됨.
- 여러 커넥션/세션이 동시에 실행되어도, 모두 MDL_SHARED 락이므로 병목 없이 동시 실행 가능.
- 구조 변경(ALTER, DROP 등 DDL)은 이들이 모두 커밋/종료될 때까지 대기한다.

**ALTER, DROP, RENAME, CREATE**

- MDL_EXCLUSIVE 락을 잡으려 하며, 기존 MDL_SHARED 락(및 앞선 EXCLUSIVE 락) 해제까지 "WAITING FOR TABLE METADATA LOCK" 상태가 됨.
- FK가 있을 경우에는 연관된 모든 테이블(부모/자식)에도 동시에 MDL_EXCLUSIVE가 확장 적용되고, 역시 앞선 락이 모두 풀릴 때까지 대기한다.

---

## 실전 적응 포인트

- 트랜잭션을 빠르게 커밋/종료하고, pool/프로그램상에서 오래 유지되는 커넥션이 있는지 상시 모니터링 필요.
- **DDL은 트래픽이 적은 시간**이나, 트랜잭션/트래픽이 적은 환경에서 실행하는 게 안정적.
- 락 큐(FIFO) 구조와 FK(외래키) 전파 원리 이해가 중요하며, 실제 서비스 병목, 장애 원인이 될 수 있으므로 주기적 모니터링 필수.
- 실무에서는 `SHOW PROCESSLIST`, `performance_schema.metadata_locks`, `slow query log`, application log 등 다양한 진단 도구를 활용해야 한다.

---

## 참고 레퍼런스

- [MySQL 8.4 Reference Manual: Metadata Locking](https://dev.mysql.com/doc/refman/8.4/en/metadata-locking.html)
- [MySQL 8.4 Reference Manual: FOREIGN KEY Constraints](https://dev.mysql.com/doc/refman/8.4/en/create-table-foreign-keys.html)
- [MySQL metadata lock wait queue ordering (Stack Overflow)](https://stackoverflow.com/questions/50714253/what-is-mysql-table-metadata-lock-wait-queue-ordering)
- [Mysql ddl, dml 무한대기 현상](https://hyooi.github.io/%ED%8A%B8%EB%9F%AC%EB%B8%94%EC%8A%88%ED%8C%85/database/2021/11/09/mysql-metadata-lock.html)
