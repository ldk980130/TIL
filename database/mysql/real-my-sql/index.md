## Real MySQL

- 3장 사용자 및 권한
  - [3.1 사용자 식별](03-user&account/3.1-user-identification.md)
  - [3.2 사용자 계정 관리](03-user&account/3.2-user-account-management.md)
  - [3.3 비밀번호 관리](03-user&account/3.3-password-management.md)
  - [3.4 권한(Privilege)](03-user&account/3.4-privilege.md)
  - [3.5 역할(Role)](03-user&account/3.5-role.md)
- 4장 아키텍처
  - [4.1 MySQL 엔진 아키텍처](04-architecture/4.1-mysql-engine-architecture.md)
  - 4.2 InnoDB 스토리지 엔진 아키텍처
    - [4.2.1 ~ 4.2.6](04-architecture/4.2-1-innodb-storage-engine.md)
    - [4.2.7 ~ 4.2.8](04-architecture/4.2-2-innodb-buffer-pool.md)
    - [4.2.9 ~ 4.2.10](04-architecture/4.2-3-undo-log-and-change-buffer.md)
    - [4.2.11 ~ 4.2.12](04-architecture/4.2-4-redo-log&log-buffer&adaptive-hash-index.md)
  - [4.3 MyISAM 스토리지 엔진 아키텍처](04-architecture/4.3-myisam-storage-engine.md)
  - [4.4 MySQL 로그 파일](04-architecture/4.4-mysql-log-file.md)
- 5장 트랜잭션과 잠금
  - [5.1 트랜잭션](05-transaction&lock/5.1-transaction.md)
  - [5.2 MySQL 엔진에서의 잠금](05-transaction&lock/5.2-mysql-engine-lock.md)
  - [5.3 InnoDB 스토리지 엔진 잠금](05-transaction&lock/5.3-innodb-storage-engine-lock.md)
  - [5.4 MySQL 격리 수준](05-transaction&lock/5.4-mysql-isolate-level.md)
- 6장 데이터 압축
  - [6.1 페이지 압축](06-data-compression/6.1-page-compression.md)
  - [6.2 테이블 압축](06-data-compression/6.2-table-compression.md)
- 8장 인덱스
  - [8.1 디스크 읽기 방식](08-index/8.1-disk-read.md)
  - [8.2 인덱스란?](08-index/8.2-index.md)
  - 8.3 B-Tree 인덱스
    - [8.3.1 구조 및 특성, 8.3.2 B-Tree 인덱스 키 추가 및 삭제](08-index/8.3-1-b-tree-index.md)
    - [8.3.3 ~ 8.3.5 B-Tree 인덱스 사용과 읽기](08-index/8.3-2-b-tree-index-usage&read.md)
    - [8.3.6 ~ 8.3.7 B-Tree 인덱스 정렬과 효율성](08-index/8.3-3-b-tree-index-sort&efficiency.md)
  - [8.4 R-Tree 인덱스](08-index/8.4-r-tree-index.md)
  - [8.5 전문 검색 인덱스](08-index/8.5-fulltext-search.md)
  - [8.6 함수 기반 인덱스](08-index/8.6-function-based-index.md)
  - [8.7 멀티 벨류 인덱스](08-index/8.7-multi-value-index.md)
  - [8.8 클러스터링 인덱스](08-index/8.8-clustering-index.md)
  - [8.9 유니크 인덱스](08-index/8.9-unique-index.md)
  - [8.10 외래 키](08-index/8.10-foreign-key.md)
- 9장 옵티마이저와 힌트
  - [9.1 개요](09-optimizer&hint/9.1-intro.md)
  - 9.2 기본 데이터 처리
    - [9.2(1) 풀 테이블, 풀 인덱스 스캔과 병렬 처리](09-optimizer&hint/9.2-1-full-table-scan&full-index-scan&parallel.md)
    - [9.2(2) ORDER BY 처리](09-optimizer&hint/9.2-2-orderby.md)
    - [9.2(3) GROUP BY와 DISTINCT
      ](09-optimizer&hint/9.2-3-groupby&distinct.md)
    - [9.2(4) 내부 임시 테이블 활용](09-optimizer&hint/9.2-4-temporary-table.md)
  - 9.3 고급 최적화
    - [9.3(1) MRR과 블록 네스티드 루프 조인](09-optimizer&hint/9.3-1-mrr&block_nested_loop_join.md)
    - [9.3(2) 인덱스 컨디션 푸시 다운과 인덱스 확장](09-optimizer&hint/9.3-2-index_conditioin_push_down&index_extensions.md)
    - [9.3(3) 인덱스 머지](09-optimizer&hint/9.3-3-index-merge.md)
    - [9.3(4) 세미 조인 최적화](09-optimizer&hint/9.3-4-semijoin.md)
    - [9.3(5) 컨디션 팬아웃과 파생 테이블 머지](09-optimizer&hint/9.3-5-condition_fanout&derived_table_merge.md)
    - [9.3(6) 인비저블 인덱스, 스킵 스캔, 인덱스 정렬 선호](09-optimizer&hint/9.3-6-invisibble_index&skip_scan.md)
    - [9.3(7) 해시 조인](09-optimizer&hint/9.3-7-hash_join.md)
    - [9.3(8) 조인 최적화 알고리즘](09-optimizer&hint/9.3-8-join-optimize-algorithm.md)
  - 9.4 쿼리 힌트
    - [9.4(1) 쿼리 힌트 (인덱스 힌트)](09-optimizer&hint/9.4-1-index_hint.md)
    - [9.4(2) 옵티마이저 힌트](09-optimizer&hint/9.4-2-optimizer_hint.md)
- 10장 실행 계획
  - [10.1 통계 정보](10-explain-plan/10.1-stat_information.md) 
  - [10.2 실행 계획 확인](10-explain-plan/10.2-explain_plain_check.md) 
  - 10.3 실행 계획 분석
    - [10.3(1) 실행 계획 분석(id, select_type)](10-explain-plan/10.3-1-id&select_type.md)
    - [10.3(2) 실행 계획 분석(table, partitions)](10-explain-plan/10.3-2-table&partitions.md)
    - [10.3(3) 실행 계획 분석 (type)](10-explain-plan/10.3-3-type.md)
    - [10.3(4) 실행 계획 분석 (possible_keys, key, key_len, ref, rows, filtered)](10-explain-plan/10.3-4-possible_keys&key&key_len&ref&rows,&filtered.md)
    - [10.3(5) 실행 계획 분석 (Extra)](10-explain-plan/10.3-5-extra.md)