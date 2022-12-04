# 6장 AOP
## 6.1 트랜잭션 코드의 분리

서비스 추상화를 통해 트랜잭션 기술에 독립적으로 트랜잭션을 적용할 수 있었다. 하지만 트랜잭션 경계 설정을 위한 코드가 여전히 거슬리는 것도 사실이다. 아래는 `UserService`라는 서비스의 한 비즈니스 로직 메서드이다.

```java
public class UserService {
	private final PlatformTransactionManager transactionmanager;
	// 비즈니스 로직 수행을 위한 의존성도 주입 받음

	public UserService(PlatformTransactionManager transactionManager) {
		this.transactionManager = transactionManager;
	}

	public void upgradeLevels() throws Exception {
		TransactionStatus = status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());

		try {
			// 비즈니스 로직
			this.transactionManager.commit(status);
		} catch (Exception e) {
			this.transactionManager.rollback(status);
			throw e;
		}
	}
}
```

비즈니스 로직 부분을 private 메서드로 분리하는 것도 방법이겠지만 여전히 트랜잭션을 담당하는 코드가 버젓이 비즈니스 코드와 함께 존재한다. 이를 DI를 통해 뽑아낼 수도 있다.

### DI 적용을 이용한 트랜잭션 분리

`UserService`는 어떤 클라이언트에게 DI되어 사용되고 있을 것이다.

```java
public class Client {
	private final UserService userSservice;

	public Client(UserService userService) {
		this.userService = userService;
	}
}
```

`UserService`가 인터페이스라면 이 인터페이스이 구현체를 여러개 만들 수도 있다. **비즈니스 로직만을 담당하는 구현체와, 트랜잭션 적용 코드를 적용하는 구현체를 따로 만드는 것이다.**

### 비즈니스 로직만 알고 있는 UserService 구현체

```java
public class UserServiceImpl implements UserService {
	// 비즈니스 로직 수행을 위한 의존성만 주입	

	public void upgradeLevels() throws Exception {
		// 비즈니스 로직만 수행
	}
}
```

트랜잭션 관련 코드를 전혀 고려하지 않는 모습으로 짜도 무방하다.

### 트랜잭션 부가 기능을 추가한 UserService 구현체

```java
public class UserServiceTx implements UserService {
	private final PlatformTransactionManager transactionmanager;
	private final UserService userService; // 비즈니스 로직을 알고 있는 구현체

	public UserService(PlatformTransactionManager transactionManager, UserService userService) {
		this.transactionManager = transactionManager;
		this.userService = userService;
	}

	public void upgradeLevels() throws Exception {
		TransactionStatus = status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());

		try {
			userService.upgradeLevels();

			this.transactionManager.commit(status);
		} catch (Exception e) {
			this.transactionManager.rollback(status);
			throw e;
		}
	}
}
```

비즈니스 로직만 담당하는 `UserService` 구현체와 `TransactionManager`를 함께 DI 받는다. 그리고 `Client` 코드에서는 `UserServiceTx` 구현체를 의존하게 하면 코드 상으로 트랜잭션 기술을 분리하면서 트랜잭션을 적용할 수 있다. 인터페이스를 의존하기에 `Client` 코드는 무엇 하나 고치지 않아도 된다.

### 트랜잭션 경계 설정 코드 분리의 장점

1. 비즈니스 로직을 담당하는 `UserServiceImpl` 코드를 작성할 때 트랜잭션에 대해 고민하지 않아도 된다.
    1. 트랜잭션의 적용 여부부터 로우레벨 트랜잭션 API는 물론이고 스프링의 트랜잭션 추상화 조차 필요 없다.
    2. DI를 통해 Tx 구현체가 먼저 실행되도록 만들기만 하면 된다.
2. 비즈니스 로직에 대한 테스트를 손쉽게 만들어낼 수 있다.

## 6.2 고립된 단위 테스트

가장 편하고 좋은 테스트 방법은 가능한 작은 단위로 쪼개서 테스트 하는 것이다.

작은 단위의 테스트가 좋은 이유는

- 테스트 실패 시 원인을 찾기 쉽다.
- 테스트 단위가 작아야 테스트의 의도나 내용이 분명해진다.
- 만들기도 쉽다.

하지만 작은 단위로 테스트하고 싶어도 테스트 대상이 다른 오브젝트와 환경에 의존하고 있다면 작은 단위로 테스트 하기 어렵다.

### 복잡한 의존 관계 속의 테스트

`UserService`를 생각해 보자. 해당 클래스가 비즈니스 로직을 수행하기 위해서는 `MailSender`라는 외부 API를 사용하는 오브젝트와 `UserDao`를 의존해야 한다고 가정하겠다. 여기다가 트랜잭션 경계 설정을 위한 `TransactionManager`까지 포함하면 총 3개의 오브젝트들과 커뮤니케이션 해야 한다.

`UserService`를 테스트하려면 이 세 가지 오브젝트들이 같이 실행될 뿐만 아니라 `UserDao`를 통해 `DataSource`를 비롯한 DB 관련 네트워크 통신까지 전부 함께 실행 된다. `MailSender`에 의해 외부 API까지 날린다고 하면 더 무거운 작업이다.

이런 경우의 문제점은 다음과 같다.

- 테스트를 준비하기 힘들다.
- 환경이 달라지면 동일한 결과를 내지 못할 수도 있다.
- 수행 속도가 느리다.
    - 때문에 테스트를 작성하고 실행하는 빈도가 떨어질 것이다.
- `UserService`의 문제가 아닌 `UserDao`의 문제 때문에 불필요한 시간을 낭비할 수도 있다.

### 테스트 대상 오브젝트 고립시키기

따라서 테스트 대상이 다른 의존 관계에 종속되고 영향 받지 않도록 고립시킬 필요가 있다. 이는 바로 **테스트 대역을 사용하는 것이다.**

고립된 테스트를 하면 테스트 대상이 다른 의존 대상에 영향 받지 않아 준비도 간단해진다. 뿐만 아니라 테스트 속도도 크게 향상된다.

고립된 테스트를 만들기 위한 목 오브젝틑 작성은 약간의 수고가 들어가지만 보상을 충분히 기대할만 하다.

### 단위 테스트와 통합 테스트

단위 테스트의 단위는 사용자가 정하기 나름이다. 기능 전체를 단위라고 볼 수도 있고 클래스나 메서드를 단위라고 볼 수도 있다. 여기서는 이전의 대상 클래스를 대역을 사용해서 고립시켜 하는 테스트도 단위 테스트라고 정의하겠다. 또 다른 계층의 오브젝트나 DB 등의 리소스가 참여하는 테스트를 통합 테스트라고 하겠다. 스프링의 테스트 컨텍스트 프레임워크를 이용한 테스트도 통합 테스트다.

**단위 테스트와 통합 테스트 선정 기준 가이드라인**

- 항상 단위 테스트를 먼저 고려한다.
- 한 클래스나 성격과 목적이 같은 긴밀한 클래스 몇 개를 모아서 외부와 고립시켜 (대역 사용) 테스트를 만든다. (테스트 작성도 간단, 실행 속도 향상, 외부 영향 없음)
- 외부 리소스를 사용해야만 가능한 테스트는 통합 테스트로 만든다.
- DAO처럼 단위 테스트로 만들기 어려운 코드도 있다. DAO는 그 자체로 로직을 담고 있기보다는 DB를 통해 로직을 수행하는 인터페이스와 같은 역할을 하기에 억지로 고립시켜도 가치가 없는 경우가 대부분이다. 따라서 DAO는 DB까지 연동하는 테스트로 만드는 편이 효과적이다.
- DAO 테스트는 DB 외부 리소스를 사용하기에 통합 테스트이지만, 코드에서 보자면 하나의 기능 단위를 테스트하는 것이기도 하다.
    - DAO를 테스트를 통해 충분히 검증해두면 DAO를 이용하는 클라이언트 코드는 DAO를 대역으로 사용해 스스로를 고립시킬 수 있다.
    - 물론 단위 테스트는 성공해도 통합에서 실패할 수도 있지만 충분한 단위 테스트를 거친다면 통합에서 오류가 발생할 확률은 줄어들고 발생한다 해도 쉽게 처리할 수 있다.
- 여러 단위를 통합한 테스트는 분명 필요하지만 단위 테스트를 충분히 거쳤다면 부담은 상대적으로 줄어든다.
- 단위 테스트로 만들기 어렵다고 판단되는 코드는 일단 통합 테스트를 고려해 본다.
- 가능하면 스프링의 지원 없이 직접 코드 레벨의 DI를 사용하면서 단위 테스트를 하는 것이 좋겠지만, 스프링의 설정 자체도 테스트 대상이고, 추상적인 레벨에서 테스트해야 할 경우도 있기에 스프링 테스트 컨텍스트 프레임워크를 이용한 통합 테스트를 작성한다.
