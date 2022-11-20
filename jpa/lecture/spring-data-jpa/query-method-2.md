# 쿼리 메소드 기능 2

## 스프링 데이터 JPA 페이징과 정렬

### 페이징과 정렬 파라미터

- `org.springframework.data.domain.Sort`: 정렬 기능
- `org.springframework.data.domain.Pageable` : 페이징 기능 (내부에 Sort 포함)

> 패키지를 보면 두 인터페이스 모두 JPA가 아니다.
관계형 DB든 몽고 DB든 페이징을 공통화 시킨 것이다.
>

### 특별한 반환 타입

- `org.springframework.data.domain.Page` : 추가 count 쿼리 결과를 포함하는 페이징
    - 총 데이터 개수가(totalCount)필요한 경우 사용
    - 전체 페이지 수를 알고 싶을 때
    - <1> <2>… <99>
- `org.springframework.data.domain.Slice` : 추가 count 쿼리 없이 다음 페이지만 확인 가능
  (내부적으로 limit + 1조회)
    - 총 데이터 개수가 필요 없는 경우
    - 예) 모바일로 스크롤 내리다가 limit까지 다 내리면 <더보기> 버튼이 있는 것
    - limit + 1개 조회 된 정보로 다음 페이지가 있는 것까지 파악 가능함
- List (자바 컬렉션): 추가 count 쿼리 없이 결과만 반환

### 페이징과 정렬 사용 예제

```java
Page<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용
Slice<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용 안함
List<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용 안함
List<Member> findByUsername(String name, Sort sort);
```

### Page 사용 예제 실행 코드

```java
@Test
public void page() throws Exception {
		//given
		memberRepository.save(new Member("member1", 10));
		memberRepository.save(new Member("member2", 10));
		memberRepository.save(new Member("member3", 10));
		memberRepository.save(new Member("member4", 10));
		memberRepository.save(new Member("member5", 10));

		//when
		PageRequest pageRequest = PageRequest.of(
			0, 3, // 0페이지에서 3개 가져와
			Sort.by(Sort.Direction.DESC, "username")
		);
		Page<Member> page = memberRepository.findByAge(10, pageRequest);

		//then
		List<Member> content = page.getContent(); //조회된 데이터
		assertThat(content.size()).isEqualTo(3); //조회된 데이터 수
		assertThat(page.getTotalElements()).isEqualTo(5); //전체 데이터 수
		assertThat(page.getNumber()).isEqualTo(0); //페이지 번호
		assertThat(page.getTotalPages()).isEqualTo(2); //전체 페이지 번호
		assertThat(page.isFirst()).isTrue(); //첫번째 항목인가?
		assertThat(page.hasNext()).isTrue(); //다음 페이지가 있는가?
}
```

- 두 번째 파라미터로 받은 `Pagable`은 인터페이스다. 따라서 실제 사용할 때는 해당 인터페이스를 구현한 `org.springframework.data.domain.PageRequest` 객체를 사용한다.
- `PageRequest` 생성자의 첫 번째 파라미터에는 현재 페이지를, 두 번째 파라미터에는 조회할 데이터 수를 입력한다. 여기에 추가로 정렬 정보도 파라미터로 사용할 수 있다. 참고로 페이지는 0부터 시작한다.
- 전체 페이지 번호(전체 데이터 개수)를 알 수 있는 이유는 추가로 count 쿼리를 날리기 때문이다.

### Slice 인터페이스

```java
public interface Slice<T> extends Streamable<T> {
		int getNumber(); //현재 페이지
		int getSize(); //페이지 크기
		int getNumberOfElements(); //현재 페이지에 나올 데이터 수
		List<T> getContent(); //조회된 데이터
		boolean hasContent(); //조회된 데이터 존재 여부
		Sort getSort(); //정렬 정보
		boolean isFirst(); //현재 페이지가 첫 페이지 인지 여부
		boolean isLast(); //현재 페이지가 마지막 페이지 인지 여부
		boolean hasNext(); //다음 페이지 여부
		boolean hasPrevious(); //이전 페이지 여부
		Pageable getPageable(); //페이지 요청 정보
		Pageable nextPageable(); //다음 페이지 객체
		Pageable previousPageable();//이전 페이지 객체
		<U> Slice<U> map(Function<? super T, ? extends U> converter); //변환기
}
```

- 전체 데이터 개수에 대한 건 가져오지 않는다. (count 쿼리 X)
- 이전 페이지와 다음 페이지에 대한 정보만 들어 있다.

### Page 반환 사용 시 주의 사항

- 전체 데이터 개수를 알기 위해 Page 반환 타입을 사용하는 경우 select 쿼리가 join 등으로 복잡한 경우가 있다.
- 이 경우 count 쿼리도 복잡한 그대로 날아가서 성능에 문제가 생길 수도 있다.
    - 원본 쿼리에 count만 붙여서 나감
    - 덧붙여서 fetch join을 Page 반환 타입과 함께 사용할 경우 오류가 발생한다.
        - fetch join은 객체 그래프를 조회하는 기능이기 때문에 연관된 부모가 꼭 있어야 하는데 수를 뽑은 count()로 조회 결과가 변경되기 때문
- 때문에 count 쿼리를 따로 분리하는 기능을 제공한다.

```java
@Query(value = “select m from Member m”,
			 countQuery = “select count(m.username) from Member m”)
Page<Member> findMemberAllCountBy(Pageable pageable);
```

### 페이지를 유지하면서 엔티티를 DTO로 변환하기

```java
Page<Member> page = memberRepository.findByAge(10, pageRequest);
Page<MemberDto> dtoPage = page.map(m -> new MemberDto());
```

## 벌크성 수정 쿼리

### JPA를 사용한 벌크성 수정 쿼리

```java
public int bulkAgePlus(int age) {
		return em.createQuery("update Member m set m.age = m.age + 1 where m.age >= :age")
                    .setParameter("age", age)
                    .executeUpdate();
}
```

### 스프링 데이터 JPA를 사용한 벌크성 수정 쿼리

```java
@Modifying
@Query("update Member m set m.age = m.age + 1 where m.age >= :age")
int bulkAgePlus(@Param("age") int age);
```

- 벌크성 수정, 삭제 쿼리는 `@Modifying` 어노테이션을 사용
    - 사용하지 않으면 다음 예외 발생
    - `org.hibernate.hql.internal.QueryExecutionRequestException: Not supported for
      DML operations`
- 벌크성 쿼리를 실행하고 나서 영속성 컨텍스트 초기화: `@Modifying(clearAutomatically = true)`
  (이 옵션의 기본값은 false)
    - 벌크 연산은 영속성 컨텍스트를 무시하고 실행한다.
    - 때문에 이 옵션 없이 회원을 findById 로 다시 조회하면 영속성 컨텍스트에 과거 값이 남아서 문제가 될 수 있다. 만약 다시 조회해야 하면 꼭 영속성 컨텍스트를 초기화 하자.

## @EntityGraph

스프링 데이터 JPA는 JPA가 제공하는 엔티티 그래프 기능을 편리하게 사용하게 도와준다. 이 기능을
사용하면 JPQL 없이 페치 조인을 사용할 수 있다. (JPQL + 엔티티 그래프도 가능)

### 페치 조인 사용 코드

```java
@Query("select m from Member m left join fetch m.team")
List<Member> findMemberFetchJoin();
```

### @EntityGraph 사용 코드

```java
//공통 메서드 오버라이드
@Override
@EntityGraph(attributePaths = {"team"})
List<Member> findAll();

//JPQL + 엔티티 그래프
@EntityGraph(attributePaths = {"team"})
@Query("select m from Member m")
List<Member> findMemberEntityGraph();

//메서드 이름으로 쿼리에서 특히 편리하다.
@EntityGraph(attributePaths = {"team"})
List<Member> findByUsername(String username)
```

- 사실상 페치 조인(FETCH JOIN)의 간편 버전
- LEFT OUTER JOIN 사용
    - fetch join은 inner join, outer join을 선택할 수 있다. (기본은 inner join)
    - `@EntityGraph`는 inner, outer를 선택할 수 없고 outer join이 사용됨

## JPA Hint & Lock

### JPA Hint

JPA 쿼리 힌트(SQL 힌트가 아니라 JPA 구현체(하이버네이트)에게 제공하는 힌트)

> 아무런 정보(힌트)가 없으면 JPA는 영속성 컨텍스트 안에서 스냅샷을 이용하여 엔티티의 변경을 항상 감지하고 변경 감지를 실행해야 한다. (비용이 든다.)

조회만 하고 싶다면 변경 감지를 동작시킬 필요가 없다.
>

```java
@QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true"))
@Query("select m from Member m where m.username = :username")
Member findReadOnly(@Param("username") String username);
```

- `org.springframework.data.jpa.repository.QueryHints` 어노테이션을 사용
- 엔티티의 정보가 수정되어도 변경 감지가 동작하지 않아 update 쿼리가 나가지 않는다.
- 스냅샷 자체가 만들어지지 않아 메모리 이점도 있다.

> 실제로 전체 기능 중 조회 용 메서드에 전부 readOnly 설정을 해줘봐야야 성능 개선이 얼마 되지 않는다. 성능 문제가 되는 건 진짜 복잡한 조회 쿼리 몇 개가 문제다.
복잡하고 트래픽이 많은 API 몇 개에 넣는 거지 싹 다 넣는 건 번거로울 수 있다. (성능 테스트를 먼저 해보자)
>
> 성능 툴: apache ab, jmeter, ngrinder([http://naver.github.io/ngrinder/](http://naver.github.io/ngrinder/)
> 
> 스프링5.1 버전 이후부터는 `@Transactinal(readOnly=true)` 어노테이션을 설정하면 `@QueryHint의 readOnly`도 자동으로 동작된다.
>

### LOCK

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
List<Member> findByUsername(String name);
```

- `org.springframework.data.jpa.repository.Lock` 어노테이션을 사용
- 이 부분은 내용이 깊다. 책을 참고하자
