# Partitioning, Sharding, Replication 개념

## Partitioning

- database table을 더 작은 table들로 나누는 것
- **vertical partitioning**
    - column을 기준으로 테이블을 나누는 방식
- **horizontal partitioning**
    - row를 기준으로 테이블을 나누는 방식

### vertical partitioning

- 문제 상황
    - `ARTICLE`이라는 테이블이 있다. 이 테이블엔 가벼운 데이터를 담고 있는 column들도 있지만 `content`라는 column은 매우 많은 문자열을 담고 있다.
    - 그래서 sql로 조회할 때 `content`를 제외하기 위해 다음과 같이 sql을 작성한다. (`SELECT id, title, … FROM article WHERE …`)
    - 하지만 select문에서 제외시켜도 해당 row를 ssd 같은 디스크에서 읽어올 땐 `content`도 함께 읽어 오고 그 뒤에 필터링을 하기 때문에 IO 성능에 영향을 미친다.
- 이런 상황에 vertical partitioning을 이용할 수 있다.
    - `ARITCLE`과 `ARTICLE_CONTENT` 테이블을 나눠 `ARTICLE_CONTENT`에 content 데이터를 따로 담도록 한다.
    - 이렇게 이미 정규화를 끝낸 테이블이라도 성능에 영향이 있다면 vertical partitioning을 통해 테이블을 더 쪼갤 수 있다.
- 이 외에도 vertical partitioning을 하는 여러 이유가 있다.
    - 민감한 정보에 제한을 더 걸어서 함부로 접근하지 못하게 하기 위한 목적
    - 자주 사용되는 column만 모아 하나의 파티션으로 만들 수도 있다.
- 전체 데이터가 필요하다면 join을 사용한다.

### horizontal partitioning

- vertical과는 다르게 테이블 스키마 변화 없이 수행한다.
- 문제 상황
    - 특정 테이블의 row 수가 엄청 커지는 경우가 있다.
    - 테이블의 크기가 커질수록 인덱스의 크기도 커진다.
    - 즉 테이블에 읽기/쓰기가 있을 때마다 인덱스에서 처리되는 시간도 더 늘어나게 된다.
- 이런 상황에 horizontal partitioning을 이용할 수 있다.
    - 여기서는 **hash 기반**의 방법으로 설명한다.
        - range 기반의 방법 등도 존재한다.
    - db 내부적으로 테이블_0과 테이블_1을 나눴다고 가정하자.
        - 두 테이블의 스키마는 같다.
    - 테이블의 특정 column을 인풋으로 hash function을 작동시켜 테이블_0에 저장할지, 테이블_1에 저장할지 결정할 수 있다.
        - 여기서 어떤 테이블에 저장할지 결정하는 column을 **partition key**라고 한다.
    - 그렇게 데이터들이 두 테이블에 나눠 저장되게 된다.
- partition key를 정할 때 가장 많이 사용될 패턴에 따라 정하는 것이 중요
    - partition key를 조건으로 조회하는 경우에는 어떤 테이블에서 조회해야 하는지 알 수 있기에 효율적이지만, partition key가 아닌 조건으로 조회할 땐 모든 파티션 테이블을 조회해야 한다.
- 데이터가 균등하게 분배될 수 있도록 hash function을 잘 정의하는 것도 중요하다.
- hash 기반으로 파티션을 나누게 되면 이후에 파티션을 추가하기 까다롭다.
    - 파티션이 2개에서 3개로 늘어난다고 하면 이미 기존 2개에 있던 데이터들이 새로운 파티션으로 옮겨야 할 수도 있다. 서비스를 운영하며 이런 작업을 하기에는 굉장히 까다롭다.
- horizontal partitioning에서는 모든 파티션들은 같은 DB 서버에 저장한다.
    - 너무 많은 요청이 오면은 이 DB 서버에 부하가 걸릴 수도 있다.

## Sharding

- 데이터가 너무 커서 단일 데이터베이스에 저장할 수 없거나 성능에 문제가 발생했을 때 필요한 방법이 sharding이다.
- horizontal partitioning처럼 동작한다.
- 각 shard(파티션)가 독립된 DB 서버에 저장된다.
    - Sharding에선 파티션을 shard라고 부른다.
- shard key에 따라 DB 서버의 부하를 분산시킬 수 있다.
    - Sharding에선 partition key를 shard key라고 부른다.
- 데이터 규모가 크고 트래픽이 많이 몰리는 경우에 사용할 수 있다.
- sharding의 장점
    - **확장성** - 디스크 공간 부족 문제 해결
    - **고가용성** - 특정 shard가 다운되어도 나머지 shard는 작업을 계속할 수 있다.
    - **쿼리 응답 시간 단축** - 더 적은 수의 row를 거치기 때문
    - **더 많은 쓰기 대역폭** - 쓰기 작업을 여러 서버에서 병렬로 처리할 수 있다.
    - **스케일 아웃** - 수평 확장에 용이하다.
- sharding의 단점
    - **시스템 복잡성 증가**
    - **데이터 리밸런싱** - 각 shard의 데이터 수 균형이 맞지 않으면 비효율이 발생하기 때문에 re-sharding을 수행해야 한다. 이 작업은 많은 downtime을 유발한다.
    - **여러 shard에서의 join은 비쌈**
    - **모든 db 엔진에서 지원하지 않음**

## Replication

- 여러 DB 서버에 동일한 데이터 복사본을 유지하는 방식
- master DB 서버와 slave DB 서버를 둬서 백엔드 서버와 직접 통신하는 master 서버의 데이터를 변경이 있을 때마다 계속 slave 서버에 복사하게 한다.
- master/slave로 서버를 나누는 이유
    - master 서버에 문제가 있을 시 slave 서버가 백엔드 서버와 직접 통신하도록 대체시킬 수 있다. (고가용성, high availability)
    - 대부분의 서비스는 write보단 read 요청이 더 많기 때문에 master 서버에 write 작업을 시키고 slave 서버에 read 작업을 분산시켜 줄 수 있다.
    - 지역적으로 가까운 DB 서버에 연결하도록 할 수도 있다.
    - 수평 확장에 용이하다.
- replication 단점
    - 동시성 제어가 힘들다.
    - 다른 db 서버와 일관성을 맞추는 복사 작업 때문에 쓰기 성능이 느리다.

---

[https://youtu.be/P7LqaEO-nGU](https://youtu.be/P7LqaEO-nGU)

[https://medium.com/@nishantparmar/distribute-data-replication-partitioning-and-sharding-920a71481c1c](https://medium.com/@nishantparmar/distribute-data-replication-partitioning-and-sharding-920a71481c1c)