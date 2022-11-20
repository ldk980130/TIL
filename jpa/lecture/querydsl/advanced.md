# 중급 문법
## 프로젝션과 결과 반환 - 기본

### 프로젝션 대상이 하나

```java
List<String> result = queryFactory
 .select(member.username)
 .from(member)
 .fetch();
```

- 프로젝션 대상이 하나면 타입을 명확하게 지정할 수 있음
- 프로젝션 대상이 둘 이상이면 튜플이나 DTO로 조회

### 튜플 조회

- 프로젝션 대상이 둘 이상일 때 사용
- `com.querydsl.core.Tuple`

```java
@Test
void tupleProjection() {
		List<Tuple> tuples = queryFactory
			.select(member.username, member.age)
			.from(member)
			.fetch();

		for (Tuple tuple : tuples) {
			String username = tuple.get(member.username);
			Integer age = tuple.get(member.age);
			System.out.println("username = " + username + " age = " + age);
		}
}
```

> Tuple은 querydsl 패키지에 포함된 객체다. Repository 계층 안에서 쓰는 건 괜찮지만 Service나 다른 영역에서까지 사용하면 좋은 설계가 아니다. Repository 내부에서 쓰는 기술을 숨기자
>

## 프로젝션과 결과 반환 - DTO 조회

### 순수 JPA에서 DTO 조회

```java
List<MemberDto> result = em.createQuery(
		 "select new study.querydsl.dto.MemberDto(m.username, m.age) " +
		 "from Member m", MemberDto.class)
 .getResultList();
```

- 순수 JPA에서 DTO를 조회할 때는 new 명령어를 사용해야함
- DTO의 package이름을 다 적어줘야해서 지저분함
- 생성자 방식만 지원함

### Querydsl 빈 생성(Bean population)

**프로퍼티 접근 방법 (setter 사용)**

```java
@Test
void findDtoBySetter() {
		List<MemberDto> result = queryFactory
			.select(Projections.bean(MemberDto.class,
				member.username,
				member.age))
			.from(member)
			.fetch();
		for (MemberDto memberDto : result) {
			System.out.println("memberDto = " + memberDto);
		}
}
```

- `Projections.bean()` 사용
- `setter()`와 기본 생성자 필요

**필드 직접 접근**

```java
@Test
void findDtoByField() {
		List<MemberDto> result = queryFactory
			.select(Projections.fields(MemberDto.class,
				member.username,
				member.age))
			.from(member)
			.fetch();
		for (MemberDto memberDto : result) {
			System.out.println("memberDto = " + memberDto);
		}
}
```

- `Projections.fields()` 사용
- `setter()` 없어도 됨
- 별칭이 다를 경우
    - 프로퍼티나, 필드 접근 생성 방식에서 이름이 다를 때 해결 방안
    - `ExpressionUtils.as(source, alias)` : 필드나, 서브 쿼리에 별칭 적용
    - `username.as("memberName")` : 필드에 별칭 적용
    - 원래는 username이지만 조회하려는 UserDto에는 name이라고 가정

```java
List<UserDto> fetch = queryFactory
		 .select(Projections.fields(UserDto.class, 
                                    member.username.as("name"),
                                    ExpressionUtils.as(
                                        JPAExpressions
                                                .select(memberSub.age.max())
                                                .from(memberSub), "age")
						 )
		 ).from(member)
		 .fetch();
```

**생성자 사용**

```java
@Test
void findDtoByConstructor() {
		List<MemberDto> result = queryFactory
			.select(Projections.constructor(MemberDto.class,
				member.username,
				member.age))
			.from(member)
			.fetch();
		for (MemberDto memberDto : result) {
			System.out.println("memberDto = " + memberDto);
		}
}
```

- `Projections.constructor()` 사용
- 이름이 아니라 타입을 보고 들어가게 됨 (별칭이 안 맞아도 타입이 맞으면 들어 감)
- 하지만 생성자 파라미터 개수가 안 맞았을 경우 컴파일 타임에 에러를 잡아주지 못한다.

## 프로젝션과 결과 반환 - @QueryProjection

**생성자 + @QueryProjection**

- `./gradlew compileQuerydsl`
- `QMemberDto` 생성 확인

```java
@Data
@NoArgsConstructor
public class MemberDto {

	private String username;
	private int age;

	@QueryProjection
	public MemberDto(String username, int age) {
		this.username = username;
		this.age = age;
	}
}
```

```java
@Test
void findDtoByQueryProjection() {
		List<MemberDto> result = queryFactory
			.select(new QMemberDto(member.username, member.age))
			.from(member)
			.fetch();
		for (MemberDto memberDto : result) {
			System.out.println("memberDto = " + memberDto);
		}
}
```

- 방법은 컴파일러로 타입을 체크할 수 있으므로 가장 안전한 방법이다. 다만 DTO에 QueryDSL
  어노테이션을 유지해야 하는 점과 DTO까지 Q 파일을 생성해야 하는 단점이 있다.

## 동적 쿼리 - BooleanBuilder 사용

```java
@Test
void dynamicQuery_BooleanBuilder() {
		String usernameParam = "member1";
		Integer ageParam = 10;

		List<Member> result = searchMember1(usernameParam, ageParam);
		assertThat(result.size()).isEqualTo(1);
}

private List<Member> searchMember1(String usernameCond, Integer ageCond) {
		BooleanBuilder builder = new BooleanBuilder();
		if (usernameCond != null) {
			builder.and(member.username.eq(usernameCond));
		}
		if (ageCond != null) {
			builder.and(member.age.eq(ageCond));
		}
		return queryFactory
			.selectFrom(member)
			.where(builder)
			.fetch();
}
```

- usernameParam과 ageParam 값이 null일 때와 아닐 때 동적으로 where 문에 조건을 추가해서 쿼리를 만들 수 있다.

## 동적 쿼리 - Where 다중 파라미터 사용

```java
@Test
void dynamicQuery_WhereParam() {
		String usernameParam = "member1";
		Integer ageParam = 10;

		List<Member> result = queryFactory
			.selectFrom(member)
			.where(usernameEq(usernameParam), ageEq(ageParam))
			.fetch();
		assertThat(result.size()).isEqualTo(1);
}

private BooleanExpression usernameEq(String usernameCond) {
		if (usernameCond != null) {
			return member.username.eq(usernameCond);
		}
		return null;
}

private BooleanExpression ageEq(Integer ageCond) {
		if (ageCond != null) {
			return member.age.eq(ageCond);
		}
		return null;
}
```

- where 조건에 null 값은 무시된다.
- 메서드를 다른 쿼리에서도 재활용 할 수 있다.
- 쿼리 자체의 가독성이 높아진다.
- 영한님은 간단한 경우 삼항 연산자로 하신다고 하심

```java
private BooleanExpression usernameEq(String usernameCond) { 
		return usernameCond != null ? member.username.eq(usernameCond) : null;
}
private BooleanExpression ageEq(Integer ageCond) {
		return ageCond != null ? member.age.eq(ageCond) : null;
}
```

- 조합 가능

    ```java
    List<Member> result = queryFactory
    			.selectFrom(member)
    			.where(usernameEq(usernameParam).and(ageEq(ageParam)))
    			.fetch();
    ```

    - 조합할 경우 `BooleanExpression`이 null이 반환될 경우 메서드 체이닝을 하면 예외가 발생할 것이다. 따라서 다음과 같이 `BooleanBuilder`를 반환하는 방법이 있다.

    ```java
    private BooleanBuilder usernameEq(String username) {
    		return nullSafeBuilder(() -> member.username.eq(username));
    }
    
    private BooleanBuilder ageEq(Integer age) {
    		return nullSafeBuilder(() -> member.age.eq(age));
    }
    
    public static BooleanBuilder nullSafeBuilder(Supplier<BooleanExpression> expressionSupplier) {
    		try {
    			return new BooleanBuilder(expressionSupplier.get());
    		} catch (IllegalArgumentException e) {
    			return new BooleanBuilder();
    		}
    }
    ```


## BooleanBuilder 활용 클래스 만들어 보기

```java
public class DynamicQuery {

	private final BooleanBuilder builder;

	private DynamicQuery() {
		this.builder = new BooleanBuilder();
	}

	public static DynamicQuery builder() {
		return new DynamicQuery();
	}

	public DynamicQuery and(Supplier<BooleanExpression> expressionSupplier) {
		try {
			this.builder.and(expressionSupplier.get());
		} catch (IllegalArgumentException ignored) {}
		return this;
	}

	public BooleanBuilder build() {
		return this.builder;
	}
}
```

```java
@Test
void dynamicQuery_CustomBooleanBuilder() {
		String usernameParam = "member1";
		Integer ageParam = null;

		List<Member> result = queryFactory
			.selectFrom(member)
			.where(DynamicQuery.builder()
				.and(() -> member.username.eq(usernameParam))
				.and(() -> member.age.eq(ageParam))
				.build())
			.fetch();

		assertThat(result.size()).isEqualTo(1);
}
```
