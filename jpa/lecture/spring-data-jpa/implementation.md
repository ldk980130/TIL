# 스프링 데이터 JPA 분석
## 스프링 데이터 JPA 구현체 분석

- 스프링 데이터 JPA가 제공하는 공통 인터페이스의 구현체
- `org.springframework.data.jpa.repository.support.SimpleJpaRepository`

```java
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {
    @Transactional
    @Override
    public <S extends T> S save(S entity) {
        // ...
    }
}
```

- `@Repository` 적용
    - 스프링의 컴포넌트 스캔 대상이 됨
    - JPA 예외를 스프링이 추상화한 예외로 변환
        - JPA나 JDBC 예외가 터지는 게 아니라 스프링이 제공하는 예외가 터짐
        - 데이터베이스 처리 기술이 바뀌어도 상위 계층에서 예외를 다룰 때 변동이 없다.
- `@Transactional` 트랜잭션 적용
    - JPA의 모든 변경은 트랜잭션 안에서 동작
    - 스프링 데이터 JPA는 변경(등록, 수정, 삭제) 메서드를 트랜잭션 처리
    - 서비스 계층에서 트랜잭션을 시작하지 않으면 리파지토리에서 트랜잭션 시작
    - 서비스 계층에서 트랜잭션을 시작하면 리파지토리는 해당 트랜잭션을 전파 받아서 사용
    - 그래서 **스프링 데이터 JPA**를 사용할 때 트랜잭션이 없어도 데이터 등록, 변경이 가능했음(사실은 트랜잭션이 리포지토리 계층에 걸려있는 것임)
- `@Transactional(readOnly = true)`
    - 데이터를 단순히 조회만 하고 변경하지 않는 트랜잭션에서 `readOnly = true` 옵션을 사용하면 플러시를 생략해서 약간의 성능 향상을 얻을 수 있음
    - h2에서는 괜찮았는데 MySql에서는 readOnly에서 insert 등 사용하면 예외 발생하더라..

### 읽기 전용 쿼리의 성능 최적화

엔티티가 영속성 컨텍스트에 관리되면 1차 캐시부터 변경 감지까지 얻을 수 있다. 하지만 스냅샷을 보관하는 등 더 많은 메모리를 사용하는 단점이 있다. 이를 해결하는 방법이 있다.

- 스칼라 타입으로 조회
    - `select o.id, o.name from Order`
    - 스칼라 타입으로 모든 필드를 조회하면 영속성 컨텍스트가 관리하지 않는다.
- [읽기 전용 쿼리 힌트 사용](https://www.notion.so/2-7a0c78705aee4b3b8f2896cae5b524b6)
    - 스냅샷을 보관하지 않아 메모리상 이점을 얻을 수 있다. 스냅샷이 없으므로 변경이 반영되지 않음
- 읽기 전용 트랜잭션 사용 (`@Transactional(readOnly = true`)
    - 플러시를 호출하지 않음

### **매우 중요!!! save()**

`save()` 메서드*

```java
if (entityInformation.isNew(entity)) {
    em.persist(entity);
    return entity;
} else {
    return em.merge(entity);
}
```

- 새로운 엔티티면 저장( persist )
- 새로운 엔티티가 아니면 병합( merge )
    - DB에 있는 원래 데이터(엔티티)를 가져온 뒤 바꿔치기
    - select 쿼리가 한 번 일어남 (단점)
    - 데이터 수정은 변경 감지를 써야 한다. merge를 쓰면 안 된다.
    - merge는 엔티티가 어쩌다 영속 상태를 벗어났을 때 다시 영속 상태로 만들 때 써야 한다.

## 새로운 엔티티를 구별하는 방법

`entityInformation.isNew(entity)`

- 새로운 엔티티를 판단하는 기본 전략
    - 식별자가 객체일 때 null 로 판단
    - 식별자가 자바 기본 타입일 때 0으로 판단
    - `Persistable` 인터페이스를 구현해서 판단 로직 변경 가능

    ```java
    package org.springframework.data.domain;
    
    public interface Persistable<ID> {
    		ID getId();
    		boolean isNew();
    }
    ```

  > 참고: JPA 식별자 생성 전략이 `@GenerateValue`면 `save()` 호출 시점에 식별자가 없으므로 새로운 엔티티로 인식해서 정상 동작한다. 그런데 JPA 식별자 생성 전략이 `@Id`만 사용해서 직접 할당이면 이미 식별자 값이 있는 상태로 `save()`를 호출한다. 따라서 이 경우 `merge()`가 호출된다. `merge()`는 우선 DB를 호출해서 값을 확인하고, DB에 값이 없으면 새로운 엔티티로 인지하므로 매우 비효율 적이다. 따라서 `Persistable`를 사용해서 새로운 엔티티 확인 여부를 직접 구현하게는 효과적이다.
  참고로 등록시간(`@CreatedDate`)을 조합해서 사용하면 이 필드로 새로운 엔티티 여부를 편리하게 확인할 수 있다. (`@CreatedDate`에 값이 없으면 새로운 엔티티로 판단)
  >

  ### Persistable 구현

```java
    @EntityListeners(AuditingEntityListener.class)
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public class Item implements Persistable<String> {
    		@Id
    		private String id;
  
    		@CreatedDate
    		private LocalDateTime createdDate;
    		
    		public Item(String id) {
    		    this.id = id;
    		}
    
    		@Override
    		public String getId() {
    		    return id;
    		}
    
    		@Override
    		public boolean isNew() {
    		    return createdDate == null;
    		}
    }
```
