# JPA와 외래키로 인한 데드락 해결기

프로젝트를 진행하던 중 배포를 한 뒤 에러가 발생해서 로그를 봤더니 데이터베이스에서 데드락이 발생하고 있었다.

따로 명시적인 락을 잡지 않았는데도 불구하고 데드락이 발생하니 처음엔 무슨 상황인지 이해할 수 없었다.

## 엔티티 구조
<img src="https://github-production-user-asset-6210df.s3.amazonaws.com/78652144/242762812-c297c4d9-5553-4c22-9e19-ad61fceb8bb9.png">

프로젝트의 핵심 도메인은 ‘지원서’이다. 동아리나 기업에 지원할 때 자소서나 지원서를 작성하게 되는데 지원서 작성과 보관을 도와주는 서비스기 때문이다.

하나의 ‘지원서’에는 여러 ‘문항’이 존재하기에 ‘지원 문항’과 그 ‘답변’을 하나로 묶은 ‘지원 항목’을 ‘지원서’와 일대다로 맺어주었다.

```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class ApplicationForm {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "application_form_id")
    private Long id;

    // ...

    @Embedded
    private ApplicationItems applicationItems = new ApplicationItems();
```

```java
@Embeddable
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class ApplicationItems {

    @OneToMany(mappedBy = "applicationForm", cascade = {CascadeType.PERSIST, CascadeType.REMOVE})
    private List<ApplicationItem> values = new ArrayList<>();
```

JPA를 사용하면서 일대다 양방향 매핑을 맺어주었다. 그 이유는 영속성 전이를 사용해서 `ApplicationForm` 내부의 일급 컬렉션인 `applicationItems`에 `ApplicationItem`을 추가하기만 하더라도 `insert` 쿼리를 내보내기 위해서고 `ApplicationForm`이 삭제되었을 때 `ApplicationItem`도 모두 삭제해 주기 위해서다. ‘지원서’와 ‘지원 항목’이 비즈니스 로직 상 밀접한 관련이 있고 같은 생명 주기를 공유하고 있기 때문에 이와 같이 설계했다.

## 데드락 발생 코드

```java
public Long addItem(Long applicationFormId,
                    LoginMember loginMember,
                    ApplicationItemCreateRequest request) {
        Writer writer = new Writer(loginMember.id());
        ApplicationForm form = writerCheckedFormService.checkWriterAndGet(applicationFormId, writer);
        ApplicationItem item = form.addItem(
                new ApplicationQuestion(request.applicationQuestion()),
                new ApplicationAnswer(request.applicationAnswer())
        );
        applicationFormRepository.flush();
        return item.getId();
}
```

위 코드는 ‘지원서’에 ‘지원 항목’을 추가하는 로직을 담당하는 코드다. 이 기능을 실행하면 ‘지원서’ 하나에 ‘지원 항목’이 3개였다면 4개가 되도록 추가할 수 있는 것이다. 영속성 전이를 사용해서 구현했기에 `ApplicationItem`에 대한 별도의 `save()` 호출 없이 자바 컬렉션에 저장하는 것 만으로 데이터베이스에 영속시킬 수 있었다.

> `flush()`를 호출하는 이유는 영속성 전이로 저장되는 엔티티의 경우 그냥 `save()`를 호출했을 때와 달리 PK를 바로 가져와주지 않기 때문이다. 트랜잭션 커밋 시점에 flush를 통해 영속성 전이에 의한 insert 쿼리가 발생하기에 `flush()`를 명시적으로 호출해서 ‘지원 항목’의 PK를 반환하고 있다.
>

```sql
Hibernate: 
    insert 
    into
        application_item
        (application_item_id, application_answer, application_form_id, application_question, category_id) 
    values
        (default, ?, ?, ?, ?)
Hibernate: 
    update
        application_form 
    set
        last_modified_time=?,
        recruiter=?,
        semester=?,
        member_id=?,
        writing_state=?,
        years=? 
    where
        application_form_id=?
```

쿼리는 위와 같이 발생한다.

1. 영속성 전이에 의한 insert 쿼리 발생
2. `ApplicatinoForm` 내부의 `ApplicatinoItem` 컬렉션에 변경이 발생했으니 변경 감지가 발생해서 update 쿼리 발생

필요없는 update 쿼리가 발생하는 것이 조금 마음에 걸렸지만 크게 신경쓰지는 않았다. 영속성 전이 기능을 통해 데이터베이스 종속적이지 않은 코드를 작성하는 것을 더 중요시 했었기 때문이다.

## 데드락 로그 확인

‘지원 항목 추가’ 기능에서 데드락이 발생한 것인데 동시에 2개 이상의 ‘지원 항목’을 추가하게 되면 데드락이 발생하는 것을 확인했다.

사용자가 만약 한 번에 2개 이상의 지원 항목을 추가한다면 프론트에서 2개의 API 요청을 한 번에 날리게 된다. 이 동시 요청에서 데드락이 발생하는 것인데 데이터베이스에 접속해서 로그를 확인해보니 아래와 같았다.

```bash
LATEST DETECTED DEADLOCK
------------------------
2023-05-19 14:10:34 22487951918848
*** (1) TRANSACTION:
TRANSACTION 11701, ACTIVE 0 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 5 lock struct(s), heap size 1128, 2 row lock(s), undo log entries 1
MySQL thread id 888, OS thread handle 22487844890368, query id 75644 10.0.0.157 root updating
update application_form set last_modified_time='2023-05-19 23:10:34.142917', recruiter='우아한테크코스 4기', semester='SECOND_HALF', member_id=1, writing_state='COMPLETE', years=2021 where application_form_id=3

*** (1) HOLDS THE LOCK(S):
RECORD LOCKS space id 19 page no 4 n bits 80 index PRIMARY of table `wannafly`.`application_form` trx id 11701 lock mode S locks rec but not gap
Record lock, heap no 6 PHYSICAL RECORD: n_fields 9; compact format; info bits 0
 0: len 8; hex 8000000000000003; asc         ;;
 1: len 6; hex 000000002da5; asc     - ;;
 2: len 7; hex 01000001960795; asc        ;;
 3: len 8; hex 8000000000000001; asc         ;;
 4: len 8; hex 434f4d504c455445; asc COMPLETE;;
 5: len 26; hex ec9ab0ec9584ed959ced858ced81acecbd94ec8aa42034eab8b0; asc                       4   ;;
 6: len 5; hex 99b0277217; asc   'r ;;
 7: len 4; hex 800007e5; asc     ;;
 8: len 11; hex 5345434f4e445f48414c46; asc SECOND_HALF;;

*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 19 page no 4 n bits 80 index PRIMARY of table `wannafly`.`application_form` trx id 11701 lock_mode X locks rec but not gap waiting
Record lock, heap no 6 PHYSICAL RECORD: n_fields 9; compact format; info bits 0
 0: len 8; hex 8000000000000003; asc         ;;
 1: len 6; hex 000000002da5; asc     - ;;
 2: len 7; hex 01000001960795; asc        ;;
 3: len 8; hex 8000000000000001; asc         ;;
 4: len 8; hex 434f4d504c455445; asc COMPLETE;;
 5: len 26; hex ec9ab0ec9584ed959ced858ced81acecbd94ec8aa42034eab8b0; asc                       4   ;;
 6: len 5; hex 99b0277217; asc   'r ;;
 7: len 4; hex 800007e5; asc     ;;
 8: len 11; hex 5345434f4e445f48414c46; asc SECOND_HALF;;

*** (2) TRANSACTION:
TRANSACTION 11699, ACTIVE 0 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 5 lock struct(s), heap size 1128, 2 row lock(s), undo log entries 1
MySQL thread id 886, OS thread handle 22488434579200, query id 75646 10.0.0.157 root updating
update application_form set last_modified_time='2023-05-19 23:10:34.135738', recruiter='우아한테크코스 4기', semester='SECOND_HALF', member_id=1, writing_state='COMPLETE', years=2021 where application_form_id=3

*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 19 page no 4 n bits 80 index PRIMARY of table `wannafly`.`application_form` trx id 11699 lock mode S locks rec but not gap
Record lock, heap no 6 PHYSICAL RECORD: n_fields 9; compact format; info bits 0
 0: len 8; hex 8000000000000003; asc         ;;
 1: len 6; hex 000000002da5; asc     - ;;
 2: len 7; hex 01000001960795; asc        ;;
 3: len 8; hex 8000000000000001; asc         ;;
 4: len 8; hex 434f4d504c455445; asc COMPLETE;;
 5: len 26; hex ec9ab0ec9584ed959ced858ced81acecbd94ec8aa42034eab8b0; asc                       4   ;;
 6: len 5; hex 99b0277217; asc   'r ;;
 7: len 4; hex 800007e5; asc     ;;
 8: len 11; hex 5345434f4e445f48414c46; asc SECOND_HALF;;

*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 19 page no 4 n bits 80 index PRIMARY of table `wannafly`.`application_form` trx id 11699 lock_mode X locks rec but not gap waiting
Record lock, heap no 6 PHYSICAL RECORD: n_fields 9; compact format; info bits 0
 0: len 8; hex 8000000000000003; asc         ;;
 1: len 6; hex 000000002da5; asc     - ;;
 2: len 7; hex 01000001960795; asc        ;;
 3: len 8; hex 8000000000000001; asc         ;;
 4: len 8; hex 434f4d504c455445; asc COMPLETE;;
 5: len 26; hex ec9ab0ec9584ed959ced858ced81acecbd94ec8aa42034eab8b0; asc                       4   ;;
 6: len 5; hex 99b0277217; asc   'r ;;
 7: len 4; hex 800007e5; asc     ;;
 8: len 11; hex 5345434f4e445f48414c46; asc SECOND_HALF;;

*** WE ROLL BACK TRANSACTION (2)
```

로그가 복잡하지만 간단하게 정리하면 다음과 같다.

1. 트랜잭션 11699번과 11701번이 동시에 ‘지원 항목 추가’ 쿼리 실행
2. 두 트랜잭션 모두 `application_form` 테이블에 S Lock을 획득
3. 두 트랜잭션 모두 `application_form`에 대해 X Lock 획득 시도
4. 다른 트랜잭션에서 S Lock을 걸고 있기 때문에 X Lock을 획득하기 위해 대기
5. 서로 기다리게 되면서 데드락 발생
6. 트랜잭션 11699번을 롤백

## 데드락 원인 파악

우선 S Lock과 X Lock이 무엇인지 이해하는 것이 필요하다.

- **shared lock**은 트랜잭션이 row를 읽기 위해 거는 락
    - 여러 트랜잭션이 동시에 데이터를 읽어도 문제 없고 shared lock끼리는 동시에 접근 가능
- **exclusive lock**은 트랜잭션이 row를 update하거나 delete하기 위해 거는 락
    - exclusive lock을 동시에 걸 순 없고 다른 트랜잭션이 읽는 것조차 막는다.

더 쉽게 풀자면 S Lock은 한 트랜잭션이 어떤 테이블 row를 읽을 때 다른 트랜잭션에서 이 row을 변경하지 못하도록 막는 잠금이다. 변경만 막고 단순 select나 S Lock을 함께 거는 것도 막지 않는다. 반면 X Lock은 다른 트랜잭션의 단순 select까지는 허용하지만 그 어떤 잠금(S Lock도 포함)도 거는 것을 허용하지 않는다.

위 데드락 로그에서 살펴 봤듯이 두 트랜잭션에서 같은 테이블 row에 대해 S Lock → X Lock 순으로 잠금을 획득하려고 할 때 동시 접근이 발생하면 데드락이 발생할 수밖에 없는 것이다. 두 트랜잭션이 서로가 S Lock을 해제하기를 기다리며  X Lock을 걸려고 기다리고 있기 때문이다.

| 트랜잭션 1 | 트랜잭션 2 |
| --- | --- |
| S Lock 획득 |  |
|  | S Lock 획득 |
| X Lock 획득 시도, 대기 |  |
|  | X Lock 획득 시도, 대기 |
|  | 데드락 탐지, 롤백 |
| X Lock 획득 |  |
| 커밋 |  |

## 외래키 제약과 잠금 전파

‘지원 항목 추가’ 기능의 쿼리가 S Lock → X Lock 순으로 잠금을 획득하는 것은 외래키 제약과 큰 관련이 있다. `application_form`과 `application_item`은 외래키 제약을 맺고 있다.

```sql
Hibernate: 
    insert 
    into
        application_item
        (application_item_id, application_answer, application_form_id, application_question, category_id) 
    values
        (default, ?, ?, ?, ?)
Hibernate: 
    update
        application_form 
    set
        last_modified_time=?,
        recruiter=?,
        semester=?,
        member_id=?,
        writing_state=?,
        years=? 
    where
        application_form_id=?
```

위  SQL을 다시 살펴보면 `applciation_item`에 insert 후 `application_form`에 update를 하고 있다. `application_form`에 대한 update 쿼리 수행 시 X Lock을 획득하는 것은 당연한 흐름이다. 그런데 S Lock은 대체 어디서 획득하는 것일일까? 답은 외래키 제약과 참조 무결성에 있다.

**참조 무결성**(referential integrity)은 관계 데이터베이스 관계 모델에서 외래키 제약이 걸린 테이블 간의 일관성(데이터 무결성)을 말한다. 쉽게 말하면 `appcliation_item`을 insert하거나 update하는 과정에서 다른 트랜잭션에서 `application_form`을 변경하거나 삭제하게 되면 무결성을 침해할 수 있다. 때문에 부모 테이블을 변경하지 못하도록 부모 테이블에 S Lock을 걸게 되는 것이다.

## 데드락 해결

원인은 파악했으니 이제 해결을 해야 한다. 해결 방법은 몇 가지가 있다.

1. 프론트에서 ‘지원 항목 추가’ API를 동시에 날리지 못하게 한다.

   → 사용자 경험 상 좋지 않고 애초에 동시에 날리면 안 되는 API가 말이 되지 않는다.

2. 외래키 제약을 해제한다.

   → 실제로 참조 관계를 갖고 있지만 외래키 제약을 맺지 않는 다른 테이블이 존재하기는 한다. 하지만 ‘지원서’와 ‘지원 항목’은 밀접한 관련을 갖고 있고 참조 무결성을 지켜야 하는 중요한 관계라고 판단해서 해제하지 않는 것으로 했다.

3. 필요 없는 update 쿼리를 삭제한다.

   → 위에서도 언급했듯이 update 쿼리는 영속성 전이, 변경 감지를 사용하면서 딸려 나가는 쿼리고 비즈니스 로직상 필요가 없다.


필요 없는 update 쿼리, 필요 없는 X Lock을 제거하는 것이 가장 좋다고 판단해서 3번 해결 방법을 채택하고 코드를 변경했다.

```java
public Long addItem(Long applicationFormId,
                        LoginMember loginMember,
                        ApplicationItemCreateRequest request) {
        Writer writer = new Writer(loginMember.id());
        ApplicationForm form = writerCheckedFormService.checkWriterAndGet(applicationFormId, writer);
        ApplicationItem item = new ApplicationItem(
                form,
                new ApplicationQuestion(request.applicationQuestion()),
                new ApplicationAnswer(request.applicationAnswer())
        );
        applicationItemRepository.save(item);
        return item.getId();
}
```

`ApplicationItem`을 영속성 전이에 의한 저장이 아닌 명시적인 `save()` 호출로 저장함으로써 insert 쿼리만 발생하게 하였다.

```sql
Hibernate: 
    insert 
    into
        application_item
        (application_item_id, application_answer, application_form_id, application_question, category_id) 
    values
        (default, ?, ?, ?, ?)
```

‘지원 항목 추가’에는 영속성 전이가 필요 없어졌지만 ‘지원서’를 저장할 때 ‘지원 항목’도 함께 저장하도록 되어 있기 때문에 `CascadeType.PERSIST`는 유지하고 있다.

## 결론

- 발생하는 쿼리를 변경함으로써 발생하는 데드락을 해결할 수 있었다.
- 외래키에 의한 데드락 문제는 은근 많이 발생하는 문제인 것 같다. 특히 이번 문제는 JPA 동작 방식과 외래키에 의한 잠금 전파 등에 대한 지식이 없으면 해결하기 어려웠을 것이다.
- JPA의 기능을 적극 활용하는 것은 좋지만 이번에 데드락 같은 문제나 필요 없는 쿼리, 잠금을 획득하는 부분이 있었다. 좀 더 조심해서 사용해야 겠다는 생각이 들었다.
- 도메인 주도 설계나 JPA를 사용하면 ‘데이터베이스 종속적이지 않은 코드(또는 설계)’를 한다고 하지만 결국 데이터베이스 종속적일 수밖에 없다는 생각이 들었다. DB의 동작 방식 때문에 코드를 변경하게 됐으니 말이다.
- 그래도 최대한 도메인 중심으로 설계 후 성능이나 버그에 맞춰 데이터베이스를 신경쓰는 절차 자체는 필요하다는 생각이 든다.
