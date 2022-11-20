# 확장 기능
### 사용자 정의 레포지토리 구현

- 스프링 데이터 JPA 리포지토리는 인터페이스만 정의하고 구현체는 스프링이 자동 생성
- 스프링 데이터 JPA가 제공하는 인터페이스를 직접 구현하면 구현해야 하는 기능이 너무 많음
- 다양한 이유로 인터페이스의 메서드를 직접 구현하고 싶다면?
    - JPA 직접 사용( EntityManager )
    - 스프링 JDBC Template 사용
    - MyBatis 사용
    - 데이터베이스 커넥션 직접 사용 등등...
    - Querydsl 사용

### 사용자 정의 구현 클래스 만들기

```java
public interface MemberRepositoryCustom {
		List<Member> findMemberCustom();
}
```

```java
@RequiredArgsConstructor
public class MemberRepositoryImpl implements MemberRepositoryCustom {

		private final EntityManager em;

		@Override
		public List<Member> findMemberCustom() {
		    return em.createQuery("select m from Member m")
                        .getResultList();
		}
}
```

```java
public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {
}
```

- MemberRepository를 클라이언트 코드에서 쓰면 된다.
    - `List<Member> result = memberRepository.findMemberCustom();`
- 직접 정의할 메서드는 MemberRepositoryImpl에서 구현하면 된다.
    - 규칙: 리포지토리 인터페이스 이름 + Impl
    - 스프링 데이터 JPA가 인식해서 스프링 빈으로 등록
    - 스프링 데이터 2.x 부터는 사용자 정의 구현 클래스에 리포지토리 인터페이스 이름 + Impl 을 적용하는 대신에 **사용자 정의 인터페이스 명 + Impl 방식도 지원한다.**
    - 예를 들어서 위 예제의 `MemberRepositoryImpl` 대신에 `MemberRepositoryCustomImpl` 같이 구현해도 된다
    - 기존 방식보다 이 방식이 사용자 정의 인터페이스 이름과 구현 클래스 이름이 비슷하므로 더 직관적이다.
    - 추가로 여러 인터페이스를 분리해서 구현하는 것도 가능하기 때문에 새롭게 변경된 이 방식을 사용하는 것을 더 권장한다.
- 스프링 데이터 JPA가 제공하는 기능은 MemberRepository에서 하던대로 선언하면 된다.

> XXXRepository 하나에 핵심 비즈니스 로직이나 화면에 맞춘 통계성 쿼리를 모두 선언하고 (사용자 정의 레포지토리로) 구현하면 한 레포지토리가 너무 비대해진다.
영한님은 통계성 복잡한 쿼리(dto 등을 사용하는)를 사용하는 레포지토리와 핵심 비즈니스 로직이 있는 레포지토리를 분리하신다고 한다.
>

> 참고: 항상 사용자 정의 리포지토리가 필요한 것은 아니다. 그냥 임의의 리포지토리를 만들어도 된다.
예를 들어 MemberQueryRepository를 인터페이스가 아닌 클래스로 만들고 스프링 빈으로 등록해서 그냥 직접 사용해도 된다. 물론 이 경우 스프링 데이터 JPA와는 아무런 관계 없이 별도로 동작한다
>

## Auditing

- 엔티티를 생성, 변경할 때 변경한 사람과 시간을 추적하고 싶으면?
    - 등록일
    - 수정일
    - 등록자
    - 수정자

### 스프링 데이터 JPA 사용

**설정**

- `@EnableJpaAuditing` 스프링 부트 설정 클래스에 적용해야 함
- `@EntityListeners(AuditingEntityListener.class)` 엔티티에 적용
- 사용 어노테이션
    - `@CreatedDate`
    - `@LastModifiedDate`
    - `@CreatedBy`
    - `@LastModifiedBy`

```java
@EnableJpaAuditing
@SpringBootApplication
public class DataJpaApplication {
		public static void main(String[] args) {
		    SpringApplication.run(DataJpaApplication.class, args);
		}
}
```

```java
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
@Getter
public class BaseEntity { // 얘를 상속하면 됨

		@CreatedDate
		@Column(updatable = false)
		private LocalDateTime createdDate;

		@LastModifiedDate
		private LocalDateTime lastModifiedDate;
		
		@CreatedBy
		@Column(updatable = false)
		private String createdBy;
		
		@LastModifiedBy
		private String lastModifiedBy;
}
```

등록자, 수정자를 처리해주는 `AuditorAware` 스프링 빈 등록

```java
@Bean
public AuditorAware<String> auditorProvider() {
    return () -> Optional.of(UUID.randomUUID().toString());
}
```

실무에서는 세션 정보나, 스프링 시큐리티 로그인 정보에서 ID를 받음

## Web 확장 - 도메인 클래스 컨버터

```java
@RestController
@RequiredArgsConstructor
public class MemberController {

		private final MemberRepository memberRepository;

		@GetMapping("/members/{id}")
		public String findMember(@PathVariable("id") Member member) {
				return member.getUsername();
		}
}
```

- HTTP 요청은 회원 id 를 받지만 도메인 클래스 컨버터가 중간에 동작해서 회원 엔티티 객체를 반환
- 도메인 클래스 컨버터도 리파지토리를 사용해서 엔티티를 찾음

> 주의: 도메인 클래스 컨버터로 엔티티를 파라미터로 받으면, 이 엔티티는 단순 조회용으로만 사용해야 한다.
(트랜잭션이 없는 범위에서 엔티티를 조회했으므로, 엔티티를 변경해도 DB에 반영되지 않는다.)
>

## Web 확장 - 페이징과 정렬

### 페이징과 정렬 예제

```java
@GetMapping("/members")
public Page<Member> list(Pageable pageable) {
		Page<Member> page = memberRepository.findAll(pageable);
		return page;
}
```

- 파라미터로 `Pageable`을 받을 수 있다.
- `Pageable`은 인터페이스, 실제는 `org.springframework.data.domain.PageRequest` 객체 생성

**요청 파라미터**

- 예) `/members?page=0&size=3&sort=id,desc&sort=username,desc`
- page: 현재 페이지, 0부터 시작한다.
- size: 한 페이지에 노출할 데이터 건수
- sort: 정렬 조건을 정의한다. 예) 정렬 속성,정렬 속성...(ASC | DESC), 정렬 방향을 변경하고 싶으면 sort 파라미터 추가 ( asc 생략 가능)

> page 정보만 주면 size 기본값은 20개로 설정된다.
>

**기본값**

- 글로벌 설정

```java
spring.data.web.pageable.default-page-size=20 /# 기본 페이지 사이즈/
spring.data.web.pageable.max-page-size=2000 /# 최대 페이지 사이즈/
```

- 개별 설정

`@PageableDefault`

```java
@RequestMapping(value = "/members_page", method = RequestMethod.GET)
public String list(@PageableDefault(size = 12, sort = “username”, direction = Sort.Direction.DESC) Pageable pageable) {
 ...
}
```

**접두사**

- 페이징 정보가 둘 이상이면 접두사로 구분
- `@Qualifier`에 접두사명 추가 "{접두사명}_xxx”
- 예제: `/members?member_page=0&order_page=1`

```java
public String list(
		@Qualifier("member") Pageable memberPageable,
		@Qualifier("order") Pageable orderPageable, ...
```

### Page 내용을 DTO로 반환하기

- `Page.map()` 사용

```java
@GetMapping("/members")
public Page<MemberDto> list(Pageable pageable) {
		Page<Member> page = memberRepository.findAll(pageable);
		Page<MemberDto> pageDto = page.map(MemberDto::new);
		return pageDto;
}
```

### Page를 1부터 시작하기

1. Pageable, Page를 파리미터와 응답 값으로 사용히지 않고, 직접 클래스를 만들어서 처리한다. 그리고 직접 `PageRequest`(`Pageable` 구현체)를 생성해서 리포지토리에 넘긴다. 물론 응답 값도 Page 대신에 직접 만들어서 제공해야 한다.
2. `spring.data.web.pageable.one-indexed-parameters`를 true 로 설정한다. 그런데 이 방법은 web에서 page 파라미터를 -1 처리 할 뿐이다. 따라서 응답 값인 Page 에 모두 0 페이지 인덱스를 사용하는 한계가 있다.

> `Page<Member>` 객체를 그대로 리턴하는 경우 페이징 내용 정보 뿐만 아니라 아래와 같은 메타 데이터도 함께 보낸다. 이 메타 데이터는 1부터 시작한다는 걸 반영하지 않고 0으로 시작한다는 가정 하에 보내진다. 즉 page=1로 보내 받은 응답 메타 데이터에 pageNumber는 아래와 같이 0으로 되어 있다.
>

```json
{ 
  "content": [
 ...
 ],
	 "pageable": {
	 "offset": 0,
	 "pageSize": 10,
	 "pageNumber": 0 //0 인덱스
 }, 
  "number": 0, //0 인덱스
  "empty": false
}
```
