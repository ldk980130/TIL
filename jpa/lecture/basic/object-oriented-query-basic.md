# 객체지향 쿼리 언어 - 기본 문법
## 소개

### **JPA는 다양한 쿼리 방법을 지원**

- JPQL
- JPA Criteria
- QueryDSL
- 네이티브 SQL
- JDBC API 직접 사용, `MyBatis`, `SpringJdbcTemplate` 함께 사용
    - 대부분 JPQL로 해결이 되지만 가끔 DB 종속적인 쿼리가 필요할 때도 있다.

### **JPQL**

- JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어 제공
- SQL과 문법 유사, SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 지원
- JPQL은 엔티티 객체를 대상으로 쿼리
- SQL은 데이터베이스 테이블을 대상으로 쿼리
- 테이블이 아닌 객체를 대상으로 검색하는 객체 지향 쿼리
- SQL을 추상화해서 특정 데이터베이스 SQL에 의존X
- JPQL을 한마디로 정의하면 객체 지향 SQL

> 단순 JPQL은 동적 쿼리를 만들기 어렵다.
>

### **Criteria 소개**

- 문자가 아닌 자바 코드로 JPQL을 작성할 수 있음
- JPQL 빌더 역할
- JPA 공식 기능
- 단점: 너무 복잡하고 실용성이 없다.
- Criteria 대신에 **QueryDSL** 사용 권장

### **QueryDSL 소개**
``` java
JPAFactoryQuery query = new JPAQueryFactory(em);
QMember m = QMember.member;
List<Member> list = query.selectFrom(m)
    .where(m.age.gt(18))
    .orderBy(m.name.desc())
    .fetch();
```
- 문자가 아닌 자바 코드로 JPQL을 작성할 수 있음
- JPQL 빌더 역할
- 컴파일 시점에 문법 오류를 찾을 수 있음
- 동적 쿼리 작성 편리함
- 단순하고 쉬움
- 실무 사용 권장

### **네이티브 SQL 소개**

- JPA가 제공하는 SQL을 직접 사용하는 기능
- JPQL로 해결할 수 없는 특정 데이터베이스에 의존적인 기능
- 예) 오라클 CONNECT BY, 특정 DB만 사용하는 SQL 힌트

```java
String sql = "SELECT ID, AGE, TEAM_ID, NAME FROM MEMBER WHERE NAME = 'kim'"; 

List<Member> resultList = em.createNativeQuery(sql, Member.class)
		.getResultList();
```

### **JDBC 직접 사용, SpringJdbcTemplate 등**

- JPA를 사용하면서 JDBC 커넥션을 직접 사용하거나, 스프링 JdbcTemplate, 마이바티스 등을 함께 사용 가능
- 단 영속성 컨텍스트를 적절한 시점에 강제로 플러시 필요
- 예) JPA를 우회해서 SQL을 실행하기 직전에 영속성 컨텍스트 수동 플러시

# **JPQL(Java Persistence Query Language)**

## **JPQL - 기본 문법과 기능**

### **JPQL 소개**

- JPQL은 객체지향 쿼리 언어다. 따라서 테이블을 대상으로 쿼리하는 것이 아니라 엔티티 객체를 대상으로 쿼리한다.
- JPQL은 SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않는다.
- JPQL은 결국 SQL로 변환된다.

### **JPQL 문법**

- select_문 :: =
    - select_절
    - from_절
    - [where_절]
    - [groupby_절]
    - [having_절]
    - [orderby_절]
- update_문 :: = update_절 [where_절]
- delete_문 :: = delete_절 [where_절]

- `select m from Member as m where m.age > 18`
- 엔티티와 속성은 대소문자 구분O (Member, age)
    - 자바 코드 상으로 똑같이 써야 함
- JPQL 키워드는 대소문자 구분X (SELECT, FROM, where)
- 엔티티 이름 사용, 테이블 이름이 아님(Member)
- 별칭은 필수(m) (as는 생략가능)

### **집합과 정렬**

**select**

**COUNT**(m), //회원수

**SUM**(m.age), //나이 합

**AVG**(m.age), //평균 나이

**MAX**(m.age), //최대 나이

**MIN**(m.age) //최소 나이

**from** Member m

- GROUP BY, HAVING
- ORDER BY

### **TypeQuery, Query**

- `TypeQuery`: 반환 타입이 명확할 때 사용

```java
TypedQuery<Member> query1 = em.createQuery("select m from Member m", Member.class);
```

- `Query`: 반환 타입이 명확하지 않을 때 사용

```java
Query query2 = 	em.createQuery("select m.userName, m.age from Member m");
```

### **결과 조회 API**

- `query.getResultList()`: 결과가 하나 이상일 때, 리스트 반환
    - 결과가 없으면 빈 리스트 반환

```java
List<Member> members = em.createQuery("select m from Member m", Member.class)
  	 .getResultList();
```

- `query.getSingleResult()`: 결과가 정확히 하나, 단일 객체 반환
    - 결과가 없으면: `javax.persistence.NoResultException`
    - 둘 이상이면: `javax.persistence.NonUniqueResultException`

```java
Member member = em.createQuery("select m from Member m", Member.class)
   .getSingleResult();
```

### **파라미터 바인딩 - 이름 기준, 위치 기준**

```java
em.createQuery("SELECT m FROM Member m where m.userName = :username")
   .setParameter("username", "member1")
   .getSingleResult()
```

- m.userName의 userName은 Member 객체 필드 이름이고, :username은 여기서 설정한 파라미터로 setParameter의 인자 값으로 쓰인다. (username이 member1인 것)
- 위치 기반은 쓰지 마라 (순서가 밀릴 수 있음)

## **프로젝션**

- SELECT 절에 조회할 대상을 지정하는 것
- 프로젝션 대상: 엔티티, 임베디드 타입, 스칼라 타입(숫자, 문자 등 기본 데이터 타입)
- `SELECT m FROM Member m` -> 엔티티 프로젝션
    - 조회된 엔티티 모두 영속성 컨텍스트에서 관리된다. (변경 감지 등 사용 가능)
- `SELECT m.team FROM Member m` -> 엔티티 프로젝션
    - 이렇게 하면 sql은 Member와 Team 테이블을 조인해서 team 정보를 가져온다. Jpql 상에선 조인을 사용하지 않았는데 sql은 조인이 실행되는 것이다. 조인은 성능상 문제가 되는 부분이 많을 수도 있기 때문에 jpql 상에서도 조인을 명시해 주는 것이 좋다. (실행 sql은 똑같지만 조인을 사용한다는 것을 확실히 알 수 있다.)
    - `select t from Member m join m.team t`
- `SELECT m.address FROM Member m` -> 임베디드 타입 프로젝션
- `SELECT m.username, m.age FROM Member m` -> 스칼라 타입 프로젝션
- DISTINCT로 중복 제거

## **프로젝션 - 여러 값 조회**

- `SELECT m.username, m.age FROM Member m`

1. Query 타입으로 조회

2. Object[] 타입으로 조회

```java
List resultList = em.createQuery("select m.userName, m.age from Member m")
   	.getResultList();

Object o = resultList.get(0);
Object[] result = (Object[])o;
System.out.println("userName = " + result[0]);
System.out.println("age = " + result[1]);
```

1. new 명령어로 조회
- 단순 값을 DTO로 바로 조회 DTO를 만들어야 함

```java
@Setter @Getter
@AllArgsConstructor
public class MemberDto {

   private String userName;
   private int age;
}
```

- DTO의 패키지 명을 포함한 전체 클래스 명 입력
- 순서와 타입이 일치하는 생성자 필요

```java
List<MemberDto> resultList = em.createQuery(
      "select new hellojpa.jpql.MemberDto(m.userName, m.age) from Member m", 
			MemberDto.class)
   .getResultList();
```

## 페이징 API

- JPA는 페이징을 다음 두 API로 추상화
- `setFirstResult(int startPosition)` : 조회 시작 위치 (0부터 시작)
- `setMaxResults(int maxResult)` : 조회할 데이터 수

```java
//페이징 쿼리
String jpql = "select m from Member m order by m.age desc";
List<Member> resultList = em.createQuery(jpql, Member.class)
   .setFirstResult(10)
   .setMaxResults(20)
   .getResultList();
```

```java
select
        member0_.id as id1_0_,
        member0_.age as age2_0_,
        member0_.team_id as team_id4_0_,
        member0_.user_name as user_nam3_0_ 
    from
        member member0_ 
    order by
        member0_.age desc limit ? offset ?
```

## **조인**

- 내부 조인: Member와 연관된 Team이 없으면 데이터가 안 나옴 (교집합)

`SELECT m FROM Member m [INNER] JOIN m.team t` inner 생략 가능

- 외부 조인: Team이 없어도 Team을 null로 가지고 Member가 조회됨 (합집합)

`SELECT m FROM Member m LEFT [OUTER] JOIN m.team t` outer 생략 가능

- 세타 조인:

`select count(m) from Member m, Team t where m.username = t.name`

### **조인 - ON 절**

- ON절을 활용한 조인(JPA 2.1부터 지원)
    - 1. 조인 대상 필터링
    - 2. 연관 관계 없는 엔티티 외부 조인(하이버네이트 5.1부터)

### **조인 대상 필터링**

- 예) 회원과 팀을 조인하면서, 팀 이름이 A인 팀만 조인

JPQL:

`SELECT m, t FROM Member m LEFT JOIN m.team t on t.name = 'A'`

SQL:

`SELECT m.*, t.* FROM Member m LEFT JOIN Team t ON m.TEAM_ID=t.id and t.name='A'`

### **연관 관계 없는 엔티티 외부 조인**

- 예) 회원의 이름과 팀의 이름이 같은 대상 외부 조인

JPQL:

`SELECT m, t FROM Member m LEFT JOIN Team t on m.username = t.name`

SQL:

`SELECT m.*, t.* FROM Member m LEFT JOIN Team t ON m.username = t.name`

## **서브 쿼리**

- 나이가 평균보다 많은 회원

`select m from Member m where m.age > **(select avg(m2.age) from Member m2)**`

- 한 건이라도 주문한 고객

`select m from Member m where **(select count(o) from Order o where m = o.member)** > 0`

메인 쿼리랑 서브 쿼리랑 서로 관계가 없어야 성능이 잘 나옴

### **서브 쿼리 지원 함수**

- [NOT] EXISTS (subquery): 서브쿼리에 결과가 존재하면 참
- {ALL | ANY | SOME} (subquery)
- ALL 모두 만족하면 참
- ANY, SOME: 같은 의미, 조건을 하나라도 만족하면 참
- [NOT] IN (subquery): 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참

### **서브 쿼리 - 예제**

- 팀A 소속인 회원

`select m from Member m where **exists** (select t from m.team t where t.name = ‘팀A')`

- 전체 상품 각각의 재고보다 주문량이 많은 주문들

`select o from Order o where o.orderAmount > **ALL** (select p.stockAmount from Product p)`

- 어떤 팀이든 팀에 소속된 회원

`select m from Member m where m.team = **ANY** (select t from Team t)`

### **JPA 서브 쿼리 한계**

- JPA는 WHERE, HAVING 절에서만 서브 쿼리 사용 가능
- SELECT 절도 가능(하이버네이트에서 지원)
- SELECT (SELECT …) FROM …
- FROM절의 서브 쿼리는 현재 JPQL에서 불가능
    - 조인으로 풀 수 있으면 풀어서 해결

### **JPQL 타입 표현**

- 문자: ‘HELLO’, ‘She’’s’
- 숫자: 10L(Long), 10D(Double), 10F(Float)
- Boolean: TRUE, FALSE

`"select m.userName, 'HELLO', true, 10L from Member m"`

- ENUM: `jpabook.MemberType.Admin` (패키지명 포함)

`"select m from Member m where m.type = hellojpa.jpql.MemberType.ADMIN";`

- ENUM 파라미터 바인딩 방법

```java
String query = "select m from Member m where m.type = :type"; 
List<Member> result = em.createQuery(query, Member.class)   
		.setParameter("type", MemberType.ADMIN)   
		.getResultList();
```

- 엔티티 타입: TYPE(m) = Member (상속 관계에서 사용)

### **JPQL 기타**

- SQL과 문법이 같은 식
- EXISTS, IN
- AND, OR, NOT
- =, >, >=, <, <=, <>
- BETWEEN, LIKE, IS NULL

### **조건식 - CASE 식**

- 기본 CASE 식

```java
"select "
   + "case when m.age <= 10 then '학생요금' "
   + "when m.age >= 60 then '경로요금' "
   + "else '일반요금' "
	 + "end "
   + "from Member m";
```

- 단순 CASE 식

```java
"select "
   + "case t.name"
   + "when 'teamA' then '인센티브 120%' "
   + "when 'teamB' then '인센티브 110%' "
   + "else '인센티브 105%' "
   + "end "
   + "from Team t";
```

- COALESCE: 하나씩 조회해서 null이 아니면 반환

  사용자 이름이 없으면 이름 없는 회원을 반환

  `"select coalesce (m.userName, '이름 없는 회원') from Member m";`

- NULLIF: 두 값이 같으면 null 반환, 다르면 첫번째 값 반환

  사용자 이름이 ‘관리자’면 null을 반환하고 나머지는 본인의 이름을 반환

  `"select NULLIF (m.userName, '관리자') from Member m"`


### **JPQL 기본 함수**

- CONCAT 문자 더하기

  `"select concat('a', 'b') from Member m"`

- SUBSTRING 문자 자르기

  `"select substring(m.userName, 0, 2) from Member m"`

- TRIM 공백 제거
- LOWER, UPPER
- LENGTH
- LOCATE (a, b) - b에서 a가 몇 번째에 있는지 찾아라 (인덱스가 0부터가 아니라 1부터 시작, 없으면 0을 반환)

  `"select locate('ber', m.userName) from Member m"`

- ABS, SQRT, MOD 수학 관련 함수
- SIZE, INDEX(JPA 용도) size: 연관 관계에서 컬렉션 개수를 알 수 있음

  `"select size(t.members) from Team t"`


### **사용자 정의 함수 호출**

- 하이버네이트는 사용 전 방언에 추가해야 한다.
- 사용하는 DB 방언을 상속 받고, 사용자 정의 함수를 등록한다.

  `select function('group_concat', i.name) from Item i`
