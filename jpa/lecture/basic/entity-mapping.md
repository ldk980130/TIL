# 엔티티 매핑
### **`@Entity`**

- `@Entity`가 붙은 클래스는 JPA가 관리, 엔티티라 한다.
- JPA를 사용해서 테이블과 매핑할 클래스는 `@Entity` 필수
- 주의
    - **기본 생성자 필수** (파라미터가 없는 public 또는 protected 생성자)
        - 내부적으로 Reflection 기술을 이용하기 때문
    - final 클래스, enum, interface, inner 클래스 사용X
    - 저장할 필드에 final 사용 X
- 속성: name
    - JPA에서 사용할 엔티티 이름을 지정한다.
    - 기본값: 클래스 이름을 그대로 사용(예: Member)
    - 같은 클래스 이름이 없으면 가급적 기본값을 사용한다.

```java
@Entity(name = "User")
public class Member
```

### **`@Table`**

- `@Table`은 엔티티와 매핑할 테이블 지정
- DB에 테이블 이름과 클래스 이름이 달라야 하는 경우에 사용한다.

```java
@Entity
@Table(name = "MBR")
public class Member
```
**속성**
- `name`: 매핑할 테이블 이름 (엔티티 이름이 기본값)
- `catalog`: 데이터베이스 catalog 매핑
- `schema`: 데이터베이스 schema 매핑
- `uniqueConstraints`: DDL 생성 시에 유니크 제약 조건 생성

## **데이터베이스 스키마 자동 생성**

- DDL을 애플리케이션 실행 시점에 자동 생성
- 테이블 중심 -> 객체 중심
- 데이터베이스 방언을 활용해서 데이터베이스에 맞는 적절한 DDL 생성
- 이렇게 **생성된 DDL은 개발 장비에서만 사용**
- 생성된 DDL은 운영 서버에서는 사용하지 않거나, 적절히 다듬은 후 사용

### **데이터베이스 스키마 자동 생성 - 속성**
`hibernate.hbm2ddl.auto, spring.jpa.hibernate.ddl-auto`
- `create`: drop + create
- `create-drop`: create와 같으나 종료 시점에 테이블 drop
- `update`: 변경 부분만 반영 (운영 DB에서 사용 X)
- `validate`: 엔티티와 테이블이 정상 매핑 되었는지만 확인
- `none`: 사용하지 않음

- update는 변경만 반영하지 지우는 건 반영하지 않는다.
- validate는 정상 매핑이 아니면 오류가 난다.

### **데이터베이스 스키마 자동 생성 - 주의**

- 운영 장비에는 절대 create, create-drop, update 사용하면 안된다.
- 개발 초기 단계는 create 또는 update
- 테스트 서버는 update 또는 validate
- 스테이징과 운영 서버는 validate 또는 none
- 웬만하면 개발할 때 테스트할 때도 validate 정도로만 사용

### **DDL 생성 기능**

- 제약조건 추가: 회원 이름은 필수, 10자 초과X
  - `@Column(nullable = false, length = 10)`
- 유니크 제약조건 추가
  - `@Table(uniqueConstraints = {@UniqueConstraint( name = "NAME_AGE_UNIQUE", columnNames = {"NAME", "AGE"} )})`
  - `@Column(unique = true)`
- DDL 생성 기능은 DDL을 자동 생성할 때만 사용되고 JPA의 실행 로직에는 영향을 주지 않는다.

## **필드와 칼럼 매핑**

```java
@Entity
public class Member {

    @Id
    private Long id;

    @Column(name = "name")
    private String username;

    private Integer age;

    @Enumerated(EnumType.STRING)
    private RoleType roleType;

    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;

    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;

    @Lob
    private String description;
```

- `@Column`: 칼럼 매핑
- `@Temporal`: 날짜 타입 매핑
- `@Enumerated`: enum 타입 매핑
- `@Lob`: BLOB, CLOB 매핑
- `@Transient`: 특정 필드를 칼럼에 매핑하지 않음(매핑 무시)

### @Column
- `name`: 필드와 매핑할 테이블 칼럼 이름
- `insertable`, `updatable`: 등록, 변경 가능 여부
- `nullable`: null 허용 여부
- `unique`: 한 컬럼에 간단히 유니크 제약 조건 걸 때 사용
- `columnDefinition`: 데이터베이스 컬럼 정보를 직접 설정 가능
  - `varchar(100) default 'EMPTY'`
- `length`: 문자 길이 제약 조건, String 타입에만 사용
- `precision`, `scale`: BigDecimal 타입에서 사용(BigInteger도 사용 가능) precision은 소수점 포함 전체 자릿 수를, scale은 소수의 자릿수다. 아주 큰 수나 정밀한 소수를 다루어야 할 때만 사용

- `updatable = false`로 하면 jpa로 uadate 로직을 써도 변경이 반영되지 않음
- `unique`는 이상한 이름이 들어가기 때문에 잘 안 쓰고 이름까지 반영 가능한 `@Table(uniqueConstraints = )`을 주로 쓴다.
- `columnDefinition = “varchar(100) default ‘EMPTY’`

```java
@Column(precision = 19, scale = 2)
private BigDecimal number;
```

### **@Enumerated**

- 자바 enum 타입을 매핑할 때 사용
- 주의! `ORDINAL` 사용X `STRING` 사용할 것
- `ORDINAL`은 enum 순서를 데이터베이스에 저장
- `STRING`은 enum 이름을 데이터베이스에 저장

### **@Temporal**

- 날짜 타입(`java.util.Date`, `java.util.Calendar`)을 매핑할 때 사용
- 참고: `LocalDate`, `LocalDateTime`을 사용할 때는 생략 가능(최신 하이버네이트 지원)

### **@Transient**

- 필드 매핑X
- 데이터베이스에 저장X, 조회X
- 주로 메모리상에서만 임시로 어떤 값을 보관하고 싶을 때 사용

## 기본 키 매핑

### **기본 키 매핑 어노테이션**

```java
@Id @GeneratedValue
private Long id;
```

### **기본 키 매핑 방법**

- 직접 할당: `@Id`만 사용
- 자동 생성(`@GeneratedValue`)
  - `IDENTITY`: 데이터베이스에 위임, MYSQL
  - `SEQUENCE`: 데이터베이스 시퀀스 오브젝트 사용, ORACLE
    - `@SequenceGenerator` 필요
  - `TABLE`: 키 생성용 테이블 사용, 모든 DB에서 사용
    - `@TableGenerator` 필요
  - `AUTO`: DB 방언에 따라 자동 지정, 기본값

### IDNTITY 전략 - 특징

```java
@Id @GeneratedValue(strategy = GenerationType.IDENTITY)
private String id;
```

- 기본 키 생성을 데이터베이스에 위임
- 주로 MySQL, PostgreSQL, SQL Server, DB2에서 사용 (예: MySQL의 AUTO_ INCREMENT)
- JPA는 보통 트랜잭션 커밋 시점에 INSERT SQL 한 번에 실행
- AUTO_ INCREMENT는 데이터베이스에 INSERT SQL을 실행한 이후에 ID 값을 알 수 있음
- IDENTITY 전략은 `em.persist()` 시점에 즉시 INSERT SQL 실행하고 DB에서 식별자를 조회
  - 그런데 한 트랜잭션 안에서 여러 insert 쿼리가 따로 나간다고 해서 유의미한 성능 저하가 일어나는 것도 아니다.

### **SEQUENCE 전략 - 특징**

```java
@Entity
@SequenceGenerator(
        name = "MEMBER_SEQ_GENERATOR", 
        sequenceName = "MEMBER_SEQ",//매핑할 데이터베이스 시퀀스 이름
        initialValue = 1, allocationSize = 1)
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "MEMBER_SEQ_GENERATOR")
    private Long id;
```

- 데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트(예: 오라클 시퀀스)
- 오라클, PostgreSQL, DB2, H2 데이터베이스에서 사용
- 테이블마다 id 생성 전략을 다르게 하고 싶으면 `@sequenceGenerator` 사용
  - 클래스 레벨에 `@sequenceGenerator`가 없으면 하이버네이트가 만드는 기본 시퀀스를 사용한다.
- identity와 다르게 네트워크에서 sequence 값을 가져와서 pk를 설정하고 커밋 시점에 한 번에 insert 쿼리를 날릴 수 있다.
  - `em.persist()` 시점에는 `call next value for MEMBER_SEQ` 호출
  - insert할 때마다 네트워크를 왔다 갔다 해야 하는데 성능 이슈는 없는가? → `allocationSize`로 최적화
  - `allocationSize`만큼 다 쓸 때까지는 `call next …`를 부르지 않아 네트워크를 타지 않는다.
  - 서버가 여러 대일 때 동시성 이슈는 없나? 없다고 한다.
- `updatable = false`로 하면 jpa로 uadate 로직을 써도 변경이 반영되지 않음
- `unique`는 이상한 이름이 들어가기 때문에 잘 안 쓰고 이름까지 반영 가능한 `@Table(uniqueConstraints = )`을 주로 쓴다.
- `columnDefinition = “varchar(100) default ‘EMPTY’`

```java
@Column(precision = 19, scale = 2)
private BigDecimal number;
```

### **@Enumerated**

- 자바 enum 타입을 매핑할 때 사용
- 주의! `ORDINAL` 사용X `STRING` 사용할 것
- `ORDINAL`은 enum 순서를 데이터베이스에 저장
- `STRING`은 enum 이름을 데이터베이스에 저장

### **@Temporal**

- 날짜 타입(`java.util.Date`, `java.util.Calendar`)을 매핑할 때 사용
- 참고: `LocalDate`, `LocalDateTime`을 사용할 때는 생략 가능(최신 하이버네이트 지원)

### **@Transient**

- 필드 매핑X
- 데이터베이스에 저장X, 조회X
- 주로 메모리상에서만 임시로 어떤 값을 보관하고 싶을 때 사용

## 기본 키 매핑

### **기본 키 매핑 어노테이션**

```java
@Id @GeneratedValue
private Long id;
```

### **기본 키 매핑 방법**

- 직접 할당: `@Id`만 사용
- 자동 생성(`@GeneratedValue`)
  - `IDENTITY`: 데이터베이스에 위임, MYSQL
  - `SEQUENCE`: 데이터베이스 시퀀스 오브젝트 사용, ORACLE
    - `@SequenceGenerator` 필요
  - `TABLE`: 키 생성용 테이블 사용, 모든 DB에서 사용
    - `@TableGenerator` 필요
  - `AUTO`: DB 방언에 따라 자동 지정, 기본값

### IDNTITY 전략 - 특징

```java
@Id @GeneratedValue(strategy = GenerationType.IDENTITY)
private String id;
```

- 기본 키 생성을 데이터베이스에 위임
- 주로 MySQL, PostgreSQL, SQL Server, DB2에서 사용 (예: MySQL의 AUTO_ INCREMENT)
- JPA는 보통 트랜잭션 커밋 시점에 INSERT SQL 한 번에 실행
- AUTO_ INCREMENT는 데이터베이스에 INSERT SQL을 실행한 이후에 ID 값을 알 수 있음
- IDENTITY 전략은 `em.persist()` 시점에 즉시 INSERT SQL 실행하고 DB에서 식별자를 조회
  - 그런데 한 트랜잭션 안에서 여러 insert 쿼리가 따로 나간다고 해서 유의미한 성능 저하가 일어나는 것도 아니다.

### **SEQUENCE 전략 - 특징**

```java
@Entity
@SequenceGenerator(
        name = "MEMBER_SEQ_GENERATOR", 
        sequenceName = "MEMBER_SEQ",//매핑할 데이터베이스 시퀀스 이름
        initialValue = 1, allocationSize = 1)
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "MEMBER_SEQ_GENERATOR")
    private Long id;
```

- 데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트(예: 오라클 시퀀스)
- 오라클, PostgreSQL, DB2, H2 데이터베이스에서 사용
- 테이블마다 id 생성 전략을 다르게 하고 싶으면 `@sequenceGenerator` 사용
  - 클래스 레벨에 `@sequenceGenerator`가 없으면 하이버네이트가 만드는 기본 시퀀스를 사용한다.
- identity와 다르게 네트워크에서 sequence 값을 가져와서 pk를 설정하고 커밋 시점에 한 번에 insert 쿼리를 날릴 수 있다.
  - `em.persist()` 시점에는 `call next value for MEMBER_SEQ` 호출
  - insert할 때마다 네트워크를 왔다 갔다 해야 하는데 성능 이슈는 없는가? → `allocationSize`로 최적화
  - `allocationSize`만큼 다 쓸 때까지는 `call next …`를 부르지 않아 네트워크를 타지 않는다.
  - 서버가 여러 대일 때 동시성 이슈는 없나? 없다고 한다.

### **TABLE 전략 - 특징**

- 키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스를 흉내내는 전략
- 장점: 모든 데이터베이스에 적용 가능
- 단점: 별도의 테이블을 쓰기 때문에 성능이 떨어진다.

### **권장하는 식별자 전략**

- 기본 키 제약 조건: null 아님, 유일, 변하면 안된다.
- 미래까지 이 조건을 만족하는 자연키는 찾기 어렵다. 대리키(대체키)를 사용
- 예를 들어 주민등록번호도 기본 키로 적절하기 않다.
- 권장: Long형 + 대체키 + 키 생성 전략 사용
