# 빈 스코프
지금까지 우리는 스프링 빈이 스프링 컨테이너의 시작과 함께 생성되어서 스프링 컨테이너가 종료될 때까지 유지된다고 학습했다. 이것은 스프링 빈이 기본적으로 싱글톤 스코프로 생성되기 때문이다. 스코프는 번역 그대로 빈이 존재할 수 있는 범위를 뜻한다.

**스프링은 다음과 같은 다양한 스코프를 지원한다.**

- **싱글톤**: 기본 스코프, 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프
- **프로토 타입**: 스프링 컨테이너는 프로토타입 빈의 생성과 의존 관계 주입까지(초기화 작업까지)만 관여하고 더는 관리하지 않는 매우 짧은 범위의 스코프
- **웹 관련 스코프**
    - **request**: 웹 요청이 들어오고 나갈 때까지 유지되는 스코프
    - **session**: 웹 세션이 생성되고 종료될 때까지 유지되는 스코프
    - **application**: 웹의 서블릿 컨텍스와 같은 범위로 유지되는 스코프

## 프로토타입 스코프

프로토타입 스코프를 스프링 컨테이너에 조회하면 스프링 컨테이너는 항상 새로운 인스턴스를 생성해서 반환한다.

```java
// 컴포넌트 스캔 자동 등록
@Scope("prototype")
@Component
public class HelloBean {}

// 수동 등록
@Scope("prototype")
@Bean
PrototypeBean HelloBean() {
	 return new HelloBean();
}
```

1. 프로토타입 스코프의 빈을 스프링 컨테이너에 요청한다.
2. 스프링 컨테이너는 이 시점에 프로토타입 빈을 생성하고, 필요한 의존 관계를 주입한다.
3. 스프링 컨테이너는 생성한 프로토타입 빈을 클라이언트에 반환한다.
4. 이후에 스프링 컨테이너에 같은 요청이 오면 항상 새로운 프로토타입 빈을 생성해서 반환한다
- **핵심은 스프링 컨테이너는 프로토타입 빈을 생성하고 의존 관계 주입, 초기화까지만 처리한다는 것**
- 프로토타입 빈을 관리할 책임은 프로토타입 빈을 받은 클라이언트에 있다 그래서 `@PreDestory`같은 종료 메서드가 호출되지 않는다.

**프로토타입 스코프 - 싱글톤 빈과 함께 사용시 문제점**

스프링은 일반적으로 싱글톤 빈을 사용하므로, 싱글톤 빈이 프로토타입 빈을 사용하게 된다. 그런데 싱글톤 빈은 생성 시점에만 의존 관계 주입을 받기 때문에, 프로토타입 빈이 새로 생성되기는 하지만, 싱글톤 빈과 함께 계속 유지되는 것이 문제다.

프로토타입 빈을 사용한 로직에서 항상 새로운 인스턴스의 빈을 사용하고 싶은 게 문제!

### `ObjectProvider`

- 의존관계를 외부에서 주입(DI) 받는게 아니라 직접 필요한 의존 관계를 찾는 것을 Dependency Lookup (DL) 의존관계 조회(탐색) 이라한다.
- 지정한 빈을 컨테이너에서 대신 찾아주는 DL 서비스를 제공하는 것이 바로 `ObjectProvider` 이다. 참고로 과거에는 `ObjectFactory`가 있었는데, 여기에 편의 기능을 추가해서 `ObjectProvider`가 만들어졌다.

```java
@Autowired
private ObjectProvider<PrototypeBean> prototypeBeanProvider;

public int logic() {
		PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
		prototypeBean.addCount();
		int count = prototypeBean.getCount();
		return count;
}
```

- `ObjectProvider`의 `getObject()`를 호출하면 내부에서는 스프링 컨테이너를 통해 해당 빈을 찾아서 반환한다. (DL)

**특징**

- `ObjectFactory`: 기능이 단순, 별도의 라이브러리 필요 없음, 스프링에 의존
- `ObjectProvider`: `ObjectFactory`상속, 옵션, 스트림 처리등 편의 기능이 많고, 별도의 라이브러리 필요 없음, 스프링에 의존

**정리**
그러면 프로토타입 빈을 언제 사용할까? 매번 사용할 때 마다 의존관계 주입이 완료된 새로운 객체가 필요하면 사용하면 된다. 그런데 실무에서 웹 애플리케이션을 개발해보면, 싱글톤 빈으로 대부분의 문제를 해결할 수 있기 때문에 프로토타입 빈을 직접적으로 사용하는 일은 매우 드물다.

`ObjectProvider`등은 프로토타입 뿐만 아니라 DL이 필요한 경우는 언제든지 사용할 수 있다

## 웹 스코프

- **request**: HTTP 요청 하나가 들어오고 나갈 때 까지 유지되는 스코프, 각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고, 관리된다.
- **session**: HTTP Session과 동일한 생명주기를 가지는 스코프
- **application**: 서블릿 컨텍스트( ServletContext )와 동일한 생명주기를 가지는 스코프
- **websocket**: 웹 소켓과 동일한 생명주기를 가지는 스코프

### 문제점

```java
@Service
@RequiredArgsConstructor
public class LogDemoService {
		private final MyLogger myLogger;

		public void logic(String id) {
				myLogger.log("service id = " + id);
		}
}
```

- `MyLogger`가 **request** 스코프 빈이라고 하자. `LogDemoService`는 싱글톤 빈이다.
- 싱글톤 빈인 `LogDemoService`를 생성하고 `MyLogger`에 대한 의존성이 필요해 주입하려고 하니 실제 HTTP 요청이 와야 생성되는 `MyLogger`는 컨텍스트에 존재하지 않아 오류가 발생한다.

### Provider로 해결

```java
@Service
@RequiredArgsConstructor
public class LogDemoService {

	private final ObjectProvider<MyLogger> myLoggerProvider;

	public void logic(String id) {
			MyLogger myLogger = myLoggerProvider.getObject();
			myLogger.log("service id = " + id);
	}
}
```

- `ObjectProvider`덕분에 `ObjectProvider.getObject()` 를 호출하는 시점까지 request scope 빈의
  생성을 지연할 수 있다.
- `ObjectProvider.getObject()`를 호출하시는 시점에는 HTTP 요청이 진행중이므로 request scope
  빈의 생성이 정상 처리된다

### 스코프와 프록시

```java
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class MyLogger {
}
```

- `proxyMode = ScopedProxyMode.TARGET_CLASS`를 추가해주자.
  - 적용 대상이 인터페이스가 아닌 클래스면 `TARGET_CLASS`를 선택
  - 적용 대상이 인터페이스면 `INTERFACES`를 선택
- 이렇게 하면 `MyLogger`의 가짜 프록시 클래스를 만들어두고 HTTP request와 상관 없이 가짜 프록시 클래스를 다른 빈에 미리 주입해 둘 수 있다.
- 이 방법을 사용하면 `ObjectProvider`를 사용하던 컨트롤러나 서비스에서 `ObjectProvider`를 사용 안하도록 코드를 원래대로 돌릴 수 있다.

**동작 정리**

- CGLIB라는 라이브러리로 내 클래스를 상속 받은 가짜 프록시 객체를 만들어서 주입한다.
- 이 가짜 프록시 객체는 실제 요청이 오면 그때 내부에서 실제 빈을 요청하는 위임 로직이 들어있다.
- 가짜 프록시 객체는 실제 request scope와는 관계가 없다. 그냥 가짜이고, 내부에 단순한 위임 로직만 있고, 싱글톤처럼 동작한다.
- 사실 Provider를 사용하든, 프록시를 사용하든 핵심 아이디어는 진짜 객체 조회를 꼭 필요한 시점까지 지연 처리 한다는 점이다.
