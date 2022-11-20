# 기본 문법
- `EntityManager`로 `JPAQueryFactory` 생성Querydsl은 JPQL 빌더
- JPQL: 문자(실행 시점 오류), Querydsl: 코드(컴파일 시점 오류)
- JPQL: 파라미터 바인딩 직접, Querydsl: 파라미터 바인딩 자동 처리

```java
@PersistenceContext
private EntityManager em;
private final JPAQueryFactory queryFactory = new JPAQueryFactory(em);
```

> `JPAQueryFactory`를 필드로 제공하면 동시성 문제는 어떻게 될까?

동시성 문제는 `JPAQueryFactory`를 생성할 때 제공하는 `EntityManager(em)`에 달려있다. 스프링 프레임워크는 여러 쓰레드에서 동시에 같은 `EntityManager`에 접근해도, 트랜잭션 마다 별도의 영속성 컨텍스트를 제공하기 때문에, 동시성 문제는 걱정하지 않아도 된다.
>

## 기본 Q-Type 활용

```java
@Test
public void startQuerydsl2() {
		//member1을 찾아라.
		QMember m = new QMember("m");
		Member findMember = queryFactory
			.select(m)
			.from(m)
			.where(m.username.eq("member1"))
			.fetchOne();
		assertThat(findMember.getUsername()).isEqualTo("member1");
}
}
```

### Q클래스 인스턴스를 사용하는 2가지 방법

```java
QMember qMember = new QMember("m"); //별칭 직접 지정
QMember qMember = QMember.member; //기본 인스턴스 사용
```

### 기본 인스턴스를 static import와 함께 사용

- 깔끔하게 쓸 수 있어서 권장

```java
import static study.querydsl.entity.QMember.*;

		@Test 
		void startQuerydsl3() {
			//member1을 찾아라.
			Member findMember = queryFactory
				.select(member)
				.from(member)
				.where(member.username.eq("member1"))
				.fetchOne();
			assertThat(findMember.getUsername()).isEqualTo("member1");
		}
```

> 기본 인스턴스를 사용하면 앨리어스가 `member1`이 된다. 같은 테이블끼리 조인할 일이 있을 때 별칭을 따로 지정해서 사용하면 된다.
>

## 검색 조건 쿼리

### and 조건

```java
Member member = queryFactory
			.selectFrom(QMember.member)
			.where(QMember.member.username.eq("member1")
				.and(QMember.member.age.eq(10)))
			.fetchOne();
```

- where()에 파라미터로 검색조건을 추가하면 AND 조건이 추가됨
    - `.where(*member*.username.eq("member1"), *member*.age.eq(10))`
- 이 경우 null 값은 무시 메서드 추출을 활용해서 동적 쿼리를 깔끔하게 만들 수 있음 뒤에서 설명

### JPQL이 제공하는 모든 검색 조건 제공

```java
member.username.eq("member1") // username = 'member1'
member.username.ne("member1") //username != 'member1'
member.username.eq("member1").not() // username != 'member1'

member.username.isNotNull() //이름이 is not null

member.age.in(10, 20) // age in (10,20)
member.age.notIn(10, 20) // age not in (10, 20)
member.age.between(10,30) //between 10, 30

member.age.goe(30) // age >= 30
member.age.gt(30) // age > 30
member.age.loe(30) // age <= 30
member.age.lt(30) // age < 30

member.username.like("member%") //like 검색
member.username.contains("member") // like ‘%member%’ 검색
member.username.startsWith("member") //like ‘member%’ 검색
```

## 결과 조회

- `fetch()` : 리스트 조회, 데이터 없으면 빈 리스트 반환
- `fetchOne()` : 단 건 조회
    - 결과가 없으면 : null
    - 결과가 둘 이상이면 : `com.querydsl.core.NonUniqueResultException`
- `fetchFirst()` : limit(1).fetchOne()
- `fetchResults()` : 페이징 정보 포함, total count 쿼리 추가 실행
- `fetchCount()` : count 쿼리로 변경해서 count 수 조회

## 정렬

```java
List<Member> result = queryFactory
			.selectFrom(member)
			.where(member.age.eq(100))
			.orderBy(member.age.desc(), member.username.asc().nullsLast())
			.fetch();
```

- `desc()` , `asc()` : 일반 정렬
- `nullsLast()` , `nullsFirst()` : null 데이터 순서 부여

## 페이징

```java
List<Member> result = queryFactory
			.selectFrom(member)
			.orderBy(member.username.desc())
			.offset(1)
			.limit(2)
			.fetch();
```

> `fetchResults`를 사용해서 페이징할 수도 있지만 이렇게 하면 count 쿼리가 추가적으로 나간다. 그런데 count 쿼리가 최적화 되어 나가는 것이 아니라 데이터 조회하는 코드 그대로 나가기 때문에 조회할 때 조인 등을 썼다면 count할 때도 조인을 쓰기 때문에 성능이 안나올 수도 있다. 이럴 때는 count 전용 쿼리를 별도로 작성해야 한다.
>

## 집합

### 집합 함수

```java
@Test
void aggregation() {
		List<Tuple> result = queryFactory
			.select(
				member.count(),
				member.age.sum(),
				member.age.avg(),
				member.age.max(),
				member.age.min()
			)
			.from(member)
			.fetch();

		Tuple tuple = result.get(0);
		assertThat(tuple.get(member.count())).isEqualTo(4);
		assertThat(tuple.get(member.age.sum())).isEqualTo(100);
		assertThat(tuple.get(member.age.avg())).isEqualTo(25);
		assertThat(tuple.get(member.age.max())).isEqualTo(40);
		assertThat(tuple.get(member.age.min())).isEqualTo(10);
}
```

- 여러 타입을 한 번에 가져올 때 `Tuple`을 쓸 수 있다.
- DTO로 바로 뽑는 방식을 사용할 수도 있다.

### GroupBy 사용

```java
/**
 * 팀의 이름과 평균 연령을 구해라
 */
@Test
void group() {
		List<Tuple> results = queryFactory
			.select(team.name, member.age.avg())
			.from(member)
			.join(member.team, team)
			.groupBy(team.name)
			.fetch();
		Tuple teamA = results.get(0);
		Tuple teamB = results.get(1);

		assertThat(teamA.get(team.name)).isEqualTo("teamA");
		assertThat(teamA.get(member.age.avg())).isEqualTo(15);
		assertThat(teamB.get(team.name)).isEqualTo("teamB");
		assertThat(teamB.get(member.age.avg())).isEqualTo(35);
}
```

- `groupBy` 결과를 제한하려면 `having`

```java
…
.groupBy(item.price)
.having(item.price.gt(1000))
 …
```

## 조인

### 기본 조인

조인의 기본 문법은 첫 번째 파라미터에 조인 대상을 지정하고, 두 번째 파라미터에 별칭(alias)으로 사용할 Q 타입을 지정하면 된다.
`join(조인 대상, 별칭으로 사용할 Q타입)`

```java
List<Member> result = queryFactory
			.selectFrom(member)
			.join(member.team, team)
			.where(team.name.eq("teamA"))
			.fetch();
```

- `join()` , `innerJoin()` : 내부 조인(inner join)
- `leftJoin()` : left 외부 조인(left outer join)
- `rightJoin()` : rigth 외부 조인(rigth outer join)
- JPQL의 on 과 성능 최적화를 위한 fetch 조인 제공 다음 on 절에서 설명

### 세타 조인

연관 관계가 없는 필드로 조인

```java
/**
 * 회원의 이름이 팀 이름과 같은 회원 조회
 */
@Test
void theta_join() {
		em.persist(new Member("teamA"));
		em.persist(new Member("teamB"));

		List<Member> results = queryFactory
			.selectFrom(member)
			.from(member, team)
			.where(member.username.eq(team.name))
			.fetch();

		assertThat(results)
			.extracting("username")
			.containsExactly("teamA", "teamB");
}
```

- from 절에 여러 엔티티를 선택해서 세타 조인
- 외부 조인 불가능 다음에 설명할 조인 on을 사용하면 외부 조인 가능

### 조인 on 절

ON절을 활용한 조인(JPA 2.1부터 지원)

1. 조인 대상 필터링
2. 연관 관계 없는 엔티티 외부 조인

**조인 대상 필터링**

```java
/**
 * 회원과 팀을 조인하면서, 팀 이름이 teamA인 팀만 조인, **회원은 모두 조회**
 * JPQL - select m, t from member m left join m.team t on t.name = 'teamA'
 */
@Test
void join_on_filtering() {
		List<Tuple> result = queryFactory
			.select(member, team)
			.from(member)
			.leftJoin(member.team, team).on(team.name.eq("teamA"))
			.fetch();

		for (Tuple tuple : result) {
			System.out.println("tuple = " + tuple);
		}
}
```

> 참고: on 절을 활용해 조인 대상을 필터링 할 때, 외부 조인이 아니라 내부 조인(inner join)을 사용하면, where 절에서 필터링 하는 것과 기능이 동일하다. `.where(team.name.eq(”teamA”)`
따라서 on 절을 활용한 조인 대상 필터링을 사용할 때, 내부 조인 이면 익숙한 where 절로 해결하고, 정말 외부 조인(left join)이 필요한 경우에만 이 기능을 사용하자.
>

**연관 관계 없는 엔티티 외부 조인**

```java
/**
 * 연관관계 없는 엔티티 외부 조인
 * 회원의 이름이 팀 이름과 같은 대상 외부 조인
 */
@Test
void join_on_no_relation() {
		em.persist(new Member("teamA"));
		em.persist(new Member("teamB"));
		em.persist(new Member("teamC"));

		List<Tuple> results = queryFactory
			.select(member, team)
			.from(member)
			.leftJoin(team).on(member.username.eq(team.name))
			.fetch();

		for (Tuple tuple : results) {
			System.out.println("tuple = " + tuple);
		}
}
```

- 하이버네이트 5.1부터 on 을 사용해서 서로 관계가 없는 필드로 외부 조인하는 기능이 추가되었다.
- 물론 내부 조인도 가능하다.
- 주의! 문법을 잘 봐야 한다. `leftJoin()` 부분에 일반 조인과 다르게 엔티티 하나만 들어간다.
    - 일반 조인: `leftJoin(member.team, team)`
        - 조인 대상 id로 매칭
    - on조인: `from(member).leftJoin(team).on(xxx)`

## 조인 - 페치 조인

페치 조인은 SQL에서 제공하는 기능은 아니다. SQL조인을 활용해서 연관된 엔티티를 SQL 한번에
조회하는 기능이다. 주로 성능 최적화에 사용하는 방법이다.

```java
@Test
void fetchJoinUser() {
		em.flush();
		em.clear();

		Member findMember = queryFactory
			.selectFrom(member)
			.join(member.team, team).fetchJoin()
			.where(member.username.eq("member1"))
			.fetchOne();

		boolean loaded = entityManagerFactory.getPersistenceUnitUtil().isLoaded(findMember.getTeam());
		assertThat(loaded).as("페치 조인 적용").isTrue();
}
```

- `join(),` `leftJoin()` 등 조인 기능 뒤에 `fetchJoin()` 이라고 추가하면 된다.

## 서브 쿼리

`com.querydsl.jpa.JPAExpressions` 사용

### **서브 쿼리 eq 사용**

```java
/**
 * 나이가 가장 많은 회원 조회
 */
@Test
void subQuery() {
		QMember memberSub = new QMember("memberSub");

		List<Member> result = queryFactory
			.selectFrom(member)
			.where(member.age.eq(
				JPAExpressions
					.select(memberSub.age.max())
					.from(memberSub)
			))
			.fetch();

		assertThat(result)
			.extracting("age")
			.containsExactly(40);
}
```

### **서브 쿼리 goe 사용**

```java
/**
 * 나이가 평균 이상인 회원
 */
@Test
void subQuery_goe() {
		QMember memberSub = new QMember("memberSub");

		List<Member> result = queryFactory
			.selectFrom(member)
			.where(member.age.goe(
				JPAExpressions
					.select(memberSub.age.avg())
					.from(memberSub)
			))
			.fetch();

		assertThat(result)
			.extracting("age")
			.containsExactly(30, 40);

```

### **서브 쿼리 in 사용**

```java
/**
 * 10살 초과인 회원 조회
 */
	@Test
	void subQuery_in() {
		QMember memberSub = new QMember("memberSub");

		List<Member> result = queryFactory
			.selectFrom(member)
			.where(member.age.in(
				JPAExpressions
					.select(memberSub.age)
					.from(memberSub)
					.where(member.age.gt(10))
			))
			.fetch();

		assertThat(result)
			.extracting("age")
			.containsExactly(20, 30, 40);
}
```

### **select 절에 subquery**

```java
@Test
void selectSubQuery() {
		QMember memberSub = new QMember("memberSub");

		List<Tuple> results = queryFactory
			.select(
				member.username,
				JPAExpressions
					.select(memberSub.age.avg())
					.from(memberSub)
			)
			.from(member)
			.fetch();

		for (Tuple tuple : results) {
			System.out.println("tuple = " + tuple);
		}
}
```

### **static import 활용**

```java
import static com.querydsl.jpa.JPAExpressions.select;

List<Member> result = queryFactory
			.selectFrom(member)
			.where(member.age.eq(
				select(memberSub.age.max())
					.from(memberSub)
			))
			.fetch();
```

### from 절의 서브 쿼리 한계

JPA JPQL 서브 쿼리의 한계점으로 from 절의 서브 쿼리(인라인 뷰)는 지원하지 않는다. 당연히 Querydsl도 지원하지 않는다. 하이버네이트 구현체를 사용하면 select 절의 서브 쿼리는 지원한다. Querydsl도 하이버네이트 구현체를 사용하면 select 절의 서브 쿼리를 지원한다.

**from 절의 서브쿼리 해결방안**

1. 서브쿼리를 join으로 변경한다. (가능한 상황도 있고, 불가능한 상황도 있다.)
2. 애플리케이션에서 쿼리를 2번 분리해서 실행한다.
3. nativeSQL을 사용한다.

## Case 문

select, 조건절(where), order by에서 사용 가능

### 단순한 조건

```java
@Test
void basicCase() {
		List<String> result = queryFactory
			.select(member.age
				.when(10).then("열살")
				.when(20).then("스무살")
				.otherwise("기타"))
			.from(member)
			.fetch();

		for (String s : result) {
			System.out.println("s = " + s);
		}
}
```

### 복잡한 조건

```java
@Test
void complexCase() {
		List<String> result = queryFactory
			.select(new CaseBuilder()
				.when(member.age.between(0, 20)).then("0~20살")
				.when(member.age.between(21, 30)).then("21~30살")
				.otherwise("기타"))
			.from(member)
			.fetch();
		
		for (String s : result) {
			System.out.println("s = " + s);
		}
}
```

### orderBy에서 Case 문 함께 사용하기

예를 들어서 다음과 같은 임의의 순서로 회원을 출력하고 싶다면?

1. 0 ~ 30살이 아닌 회원을 가장 먼저 출력
2. 0 ~ 20살 회원 출력
3. 21 ~ 30살 회원 출력

```java
@Test
void orderByCase() {
		NumberExpression<Integer> rankPath = new CaseBuilder()
			.when(member.age.between(0, 20)).then(2)
			.when(member.age.between(21, 30)).then(1)
			.otherwise(3);

		List<Tuple> results = queryFactory
			.select(member.username, member.age, rankPath)
			.from(member)
			.orderBy(rankPath.desc())
			.fetch();
		
		for (Tuple result : results) {
			String username = result.get(member.username);
			Integer age = result.get(member.age);
			Integer rank = result.get(rankPath);
			System.out.println("username = " + username + " age=" + age + " rank=" + rank);
	}
```

## 상수 문자 더하기

**상수가 필요하면 `Expressions.constant(xxx)` 사용**

```java
Tuple result = queryFactory
 .select(member.username, Expressions.constant("A"))
 .from(member)
 .fetchFirst();
```

**문자 더하기 concat**

```java
String result = queryFactory
 .select(member.username.concat("_").concat(member.age.stringValue()))
 .from(member)
 .where(member.username.eq("member1"))
 .fetchOne();
```

결과: member1_10

> 참고: `member.age.stringValue()` 부분이 중요한데, 문자가 아닌 다른 타입들은 `stringValue()` 로 문자로 변환할 수 있다. 이 방법은 ENUM을 처리할 때도 자주 사용한다
>
