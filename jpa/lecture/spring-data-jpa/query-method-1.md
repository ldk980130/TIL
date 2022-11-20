# 쿼리 메소드 기능 1

### 쿼리 메소드 기능 3가지

- 메소드 이름으로 쿼리 생성
- 메소드 이름으로 JPA NamedQuery 호출
- `@Query` 어노테이션을 사용해서 리파지토리 인터페이스에 쿼리 직접 정의

## 메소드 이름으로 쿼리 생성

스프링 데이터 JPA는 메소드 이름을 분석해서 JPQL 생성

> 이름과 나이를 기준으로 회원을 조회하려면?
>

### 순수 JPA

```java
public List<Member> findByUsernameAndAgeGreaterThan(String username, int age) {
 return em.createQuery("select m from Member m where m.username = :username and m.age > :age")
			 .setParameter("username", username)
			 .setParameter("age", age)
			 .getResultList();
}
```

### 스프링 데이터 JPA

```java
public interface MemberRepository extends JpaRepository<Member, Long> { 
    List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
}
```

### [쿼리 메소드 필터 조건](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation)

### [스프링 데이터 JPA가 제공하는 쿼리 메소드 기능](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.query-creation)

- 조회: `find…By`, `read…By`, `query…By` `get…By`,
    - 예:) findHelloBy 처럼 ...에 식별하기 위한 내용(설명)이 들어가도 된다.
- COUNT: `count…By` 반환타입 long
- EXISTS: `exists…By` 반환타입 boolean
- 삭제: `delete…By`, `remove…By` 반환타입 long
- DISTINCT: `findDistinct`, `findMemberDistinctBy`
- LIMIT: findFirst3, `findFirst`, `findTop`, `findTop3`

> 참고: 이 기능은 엔티티의 필드명이 변경되면 인터페이스에 정의한 메서드 이름도 꼭 함께 변경해야 한다.
그렇지 않으면 애플리케이션을 시작하는 시점에 오류가 발생한다.
이렇게 애플리케이션 로딩 시점에 오류를 인지할 수 있는 것이 스프링 데이터 JPA의 매우 큰 장점이다.
>

### 단점

> 필터링 조건 등이 많아지면 메소드 이름이 너무 길어진다. 그래서 파라미터가 2개까지는 괜찮은데 3개부터는 다른 방법을 이용할 수도?
>

## JPA NakmedQuery

JPA의 NamedQuery를 호출할 수 있음

- `@NamedQuery` 어노테이션으로 Named 쿼리 정의*
- `@NamedQuery`**는 애플리케이션 로딩 시점에 쿼리 문법을 파싱해서 체크할 수 있다.**
    - 잘못되면 터짐

```java
@Entity
@NamedQuery(name="Member.findByUsername",
						 query="select m from Member m where m.username = :username")
public class Member {
 ...
}
```

```java
List<Member> resultList = em.createNamedQuery("Member.findByUsername", Member.class)
			.setParameter("username", username)
			.getResultList();
```

### 스프링 데이터 JPA로 Named 쿼리 호출

```java
public interface MemberRepository extends JpaRepository<Member, Long> { 
//** 여기 선언한 Member 도메인 클래스

		List<Member> findByUsername(@Param("username") String username);
}
```

- 스프링 데이터 JPA는 선언한 "도메인 클래스 + .(점) + 메서드 이름"으로 Named 쿼리를 찾아서 실행
- 만약 실행할 Named 쿼리가 없으면 메서드 이름으로 쿼리 생성 전략을 사용한다.
- 필요하면 전략을 변경할 수 있지만 권장하지 않는다.

> **실무에서 NamedQuery 사용할 일이 없다고 함!!**
>

대신 `@Query`를 사용해서 리파지토리 메소드에 쿼리를 직접 정의한다.

## @Query, 리포지토리 메소드에 쿼리 정의하기

### 메서드에 JPQL 쿼리 작성

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

		@Query("select m from Member m where m.username= :username and m.age = :age")
		List<Member> findUser(@Param("username") String username, @Param("age") int age);
}
```

- `@org.springframework.data.jpa.repository.Query` 어노테이션을 사용
- 실행할 메서드에 정적 쿼리를 직접 작성하므로 이름 없는 Named 쿼리라 할 수 있음
- **JPA Named 쿼리처럼 애플리케이션 실행 시점에 문법 오류를 발견할 수 있음 (매우 큰 장점!)**
- 위처럼 JPQL을 직접 사용하는 경우 `@Param`을 붙여줘야 한다.

> 참고: 실무에서는 메소드 이름으로 쿼리 생성 기능은 파라미터가 증가하면 메서드 이름이 매우
지저분해진다. 따라서 `@Query` 기능을 자주 사용하게 된다.
>

## @Query, 값, DTO 조회하기

### 단순히 값 하나를 조회

```java
@Query("select m.username from Member m")
List<String> findUsernameList();
```

- JPA 값 타입(`@Embedded`)도 이 방식으로 조회할 수 있다.

### DTO로 직접 조회

```java
@Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) " +
			 "from Member m join m.team t")
List<MemberDto> findMemberDto();
```

- DTO의 풀 패키지 이름으로 new 생성자를 사용해야 한다.
- 주의! DTO로 직접 조회 하려면 JPA의 new 명령어를 사용해야 한다. 그리고 다음과 같이 생성자가 맞는 DTO가 필요하다. (JPA와 사용 방식이 동일하다.)

```java
@Data
public class MemberDto {

	private Long id;
	private String username;
	private String teamName;

	public MemberDto(Long id, String username, String teamName) {
		this.id = id;
		this.username = username;
		this.teamName = teamName;
	}
}
```

## 파라미터 바인딩

```java
select m from Member m where m.username = ?0 //위치 기반
select m from Member m where m.username = :name //이름 기반
```

- 위치 기반은 거의 사용하지 않는다.
    - 위치 바뀌면 버그가 날 수 있음
- 가급적 이름 기반을 쓰자

### 파라미터 바인딩

```java
import org.springframework.data.repository.query.Param

public interface MemberRepository extends JpaRepository<Member, Long> {

		@Query("select m from Member m where m.username = :name")
		Member findMembers(@Param("name") String username);
}
```

### 컬렉션 파라미터 바인딩

- Collection 타입으로 in절 지원

```java
@Query("select m from Member m where m.username in :names")
List<Member> findByNames(@Param("names") List<String> names);
```

## [반환 타입](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repository-query-return-types)

```java
List<Member> findByUsername(String name); //컬렉션
Member findByUsername(String name); //단건
Optional<Member> findByUsername(String name); //단건 Optional
```

**조회 결과가 많거나 없으면?**

- 컬렉션
    - 결과 없음: 빈 컬렉션 반환
- 단건 조회
    - 결과 없음: null 반환
    - 결과가 2건 이상: `javax.persistence.NonUniqueResultException` 예외 발생

  > 참고: 단건으로 지정한 메서드를 호출하면 스프링 데이터 JPA는 내부에서 JPQL의
  `Query.getSingleResult()` 메서드를 호출한다. 이 메서드를 호출했을 때 조회 결과가 없으면 `javax.persistence.NoResultException` 예외가 발생하는데 개발자 입장에서 다루기가 상당히 불편하다. 스프링 데이터 JPA는 단건을 조회할 때 이 예외가 발생하면 예외를 무시하고 대신에 null을 반환한다.
  >

  > 단건 조회에서 결과가 2건 이상일 때 JPA가`javax.persistence.NonUniqueResultException`을 발생 시키지만 스프링은 이 예외를 잡아서 `IncorrectResultSizeDataAccessException`으로 변환 시켜 던진다.
  (JPA를 쓰든 안 쓰든 클라이언트 코드(service계층 등)에서 일관되게 예외를 다루기 위해서)
>
