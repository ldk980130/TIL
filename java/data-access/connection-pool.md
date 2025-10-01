# Connection Pool
## Connection이란

> 데이터베이스 커넥션이란 동일한 시스템에 있든 없든 데이터베이스 서버가 소프트웨어와 통신할 수 있도록 하는 컴퓨터 과학의 기능이다. 커넥션은 명령을 전달하고 대답을 수신할 수 있어야 한다.
출처: [https://en.wikipedia.org/wiki/Database_connection](https://en.wikipedia.org/wiki/Database_connection)
>

## JDBC Connection

- JDBC api에서는 `Connection` 객체가 제공된다.
    - Connection은 SQL 구문을 실행시킬 수 있고 그 결과를 받아올 수 있다.
    - 한 애플리케이션은 하나 이상의 커넥션을 가질 수 있고 DB도 하나 이상 있을 수 있다.
- `DirverMangaer.getConnection`으로 `Connection`을 생성할 수 있다.
    - JDBC드라이버를 관리하는 가장 기본적인 방법
    - 커넥션 풀,분산 트랜잭션을 지원하지 않아서 잘 사용하지 않는다
- `Connection`을 연결 및 생성하는 것은 비싼 작업이므로 요청마다 `Connection`을 새로 만든다면 성능 이슈가 발생할 수도 있다.
    1. Database Driver를 사용하여 Connection을 연다.
    2. 데이터를 읽고 쓰기 위한 TCP 소켓을 연다.
    3. 소켓에 데이터를 읽고 쓴다.
    4. Connection을 닫는다.
    5. 소켓을 닫는다.

## Connection Pool

커넥션 풀은 잘 알려진 데이터 엑세스 패턴으로 데이터베이스 커넥션 성능 오버헤드를 줄이기 위한 목적으로 사용된다.

### `Datasource`

- 데이터베이스, 파일 같은 물리적 데이터 소스에 연결할 때 사용하는 인터페이스
- 구현체는 각 vendor에서 제공한다.
- 스프링에서는 `DataSource` 객체를 사용해 데이터베이스 커넥션을 관리한다.
- `DataSource`를 사용해 애플리케이션 코드에서 커넥션 풀과 트랜잭션 관련 코드를 숨길 수 있다.
- 애플리케이션 코드를 수정하지 않고 properties로 DB 연결 설정을 변경할 수 있다.
- 커넥션 풀링 또는 분산 트랜잭션이 가능하다.
- Spring의 JDBC 레이어를 사용할 때 JDNI로부터 data source를 얻을 수 있고 제 3의 곳으로부터 제공되는 커넥션 풀을 설정할 수도 있다.
- 전통적인 선택으로는 Apache Commons DBCP나 C3P0이 있지만 현대에는 JDBC 커넥션 풀의 구현으로 HikariCP를 스프링부트는 선택하고 있다.

> **Supported Connection Pools**
>
>
> Spring Boot uses the following algorithm for choosing a specific implementation:
>
> 1. We prefer [HikariCP](https://github.com/brettwooldridge/HikariCP) for its performance and concurrency. If HikariCP is available, we always choose it.
> 2. Otherwise, if the Tomcat pooling `DataSource` is available, we use it.
> 3. Otherwise, if [Commons DBCP2](https://commons.apache.org/proper/commons-dbcp/) is available, we use it.
> 4. If none of HikariCP, Tomcat, and DBCP2 are available and if Oracle UCP is available, we use it
>
- `spring-boot-starter-jdbc`나 `spring-boot-starter-data-jpa` “starters” 의존성을 추가하면 자동으로 `HikariCP` 의존성이 추가된다.

## HikariCP

HikariCP는 Brett Wooldridge가 개발한 매우 가볍고 빠른 JDBC 커넥션 풀 프레임워크다.

HikariCP는 내부에서 `ConcurrentBag`이라는 구조체로 `Connection`을 관리하는데 `HikariPool.getConnection()` → `ConcurrentBag.borrow()`라는 메서드로 유휴 Connection을 리턴한다.

**default maximumPoolSize = 10**

**HikariCP 동작 원리**

1. Thread-1이 Connection을 요청
2. Thread-1이 이전에 사용했던 Connection이 있는지 확인
3. 존재한다면 리턴하고 없다면 다른 Connection을 Hikari pool 전체에서 찾음
4. 존재한다면 리턴하고 없다면 handoffQueue에서 사용 가능한 Connection을 찾음
5. 있다면 리턴, 없다면 기다리는데 30초가 지나면 Timeout이 발생 (HikariCP default Connection timeout = 30초)

### 자주 쓰는 옵션

- `autoCommit`: 풀에서 반환되는 커넥션의 기본 오토커밋 동작으로 기본 true이며, 프레임워크 트랜잭션을 쓴다면 보통 기본값 유지가 일반적이다.
- `connectionTimeout`: 풀에서 커넥션을 얻기 위해 대기하는 최대 시간으로 최소 250ms, 기본 30초이며, 가용 커넥션 부족 시 이 시간 경과 후 예외가 난다.
- `idleTimeout`: 유휴 커넥션이 풀에 머무는 최대 시간으로 minimumIdle < maximumPoolSize인 경우에만 적용되며 기본 10분이다.
- `keepaliveTime`: DB/네트워크 타임아웃을 방지하기 위한 유휴 커넥션 ping 주기로 maxLifetime보다 작아야 하며, 분 단위 설정이 권장된다.
- `maxLifetime`: 커넥션 최대 수명으로 기본 30분이며, DB나 인프라가 강제 종료하는 한계보다 “몇 초 더 짧게” 설정할 것을 강력 권장한다.
- `connectionTestQuery`: JDBC4 isValid() 지원 드라이버면 설정하지 말 것을 권장하며, 레거시 드라이버에서만 사용한다.
- `minimumIdle`: 유휴 커넥션 최소 수로, 최대 성능/스파이크 대응을 위해 설정을 생략해 고정 크기 풀(maximumPoolSize와 동일)로 운용하는 것을 권장한다.
- `maximumPoolSize`: 풀의 총 커넥션 상한으로, 이 값에 도달하고 유휴가 없으면 getConnection()은 connectionTimeout까지 대기한다.
- `metricRegistry`/`healthCheckRegistry`: Dropwizard Metrics/HealthCheck 인스턴스를 연결해 운영 가시성을 높일 수 있다.
- `poolName`: 로그/JMX에서 식별할 풀 이름 지정으로 운영 관측 시 유용하다.

## JPA와 Connection

1. 데이터베이스 하나만 사용하는 애플리케이션에서는 `EntityManagerFactory`를 하나만 생성한다.
2. 요청이 들어올 때마다 Factory에서 `EntityManager`를 생성한다. (`EntityManager`를 생성하는 비용은 거의 안 든다)
3. 생성된 `EntityManager`는 바로 Connection을 Connection Pool에서 꺼내오지 않고 필요한 시점 (트랜잭션 시작)에 Connection을 획득한다.

## 적정 커넥션 풀 크기?

동시 사용자 요청 수에 따라 다르지만 일반적인 공식이 있긴 있다.

- `Pool Size = Tn x (Cm - 1) + 1`
    - **Tn**: 전체 Thread 개수
    - **Cm**: 하나의 Task에서 동시에 필요한 Connection 수
    - HikariCP wiki에서는 이 공식으로 Dead lock을 피할 수 있다고 한다.
        - 예) max connection pool이 1개인 상황에서 한 task에 connection이 2개가 필요한 상황
        - task가 유일한 connection을 사용하고 아직 작업이 끝나지 않았으니 connection을 반환하지 못하는데 두 번째 connection을 요청 → Dead Lock 발생
- 하지만 위 공식은 최소한의 Pool Size이므로 최적은 아니다. 성능 테스트 등을 통해 위 공식으로 최소 기준을 잡고 조금 더 확장하여 설정하면 좋다고 한다.
    - 예) `Pool Size = Tn x (Cm - 1) + (Tn / 2)` ([우아한 형제들 기술 블로그 참고](https://techblog.woowahan.com/2663/))

---

### 참고

[https://www.cs.princeton.edu/courses/archive/fall97/cs461/jdkdocs/guide/jdbc/getstart/connection.doc.html](https://www.cs.princeton.edu/courses/archive/fall97/cs461/jdkdocs/guide/jdbc/getstart/connection.doc.html)

[https://www.baeldung.com/java-connection-pooling](https://www.baeldung.com/java-connection-pooling)

[https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#jdbc-connections](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#jdbc-connections)

[https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#data.sql.datasource.connection-pool](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#data.sql.datasource.connection-pool)

[https://www.baeldung.com/hikaricp](https://www.baeldung.com/hikaricp)

[https://techblog.woowahan.com/2664/](https://techblog.woowahan.com/2664/)
