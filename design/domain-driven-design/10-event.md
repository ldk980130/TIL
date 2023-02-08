# Chapter10 이벤트

## 10.1 시스템 간 강결합 문제

- 두 바운디드 컨텍스트 간의 강결합이 존재하면 여러 문제가 발생하게 된다.

### 주문을 취소하면 환불 처리를 해야하는 예시

```java
public class CancelOrderService {
	private final RefundService refundService;

	@Transactional
	public void cancel(OrderNo orderNo) {
		// ...
		refundService.refund(order.getPaymentId());
		// ...
	}
}
```

- 이 경우 외부 서비스가 정상이 아닐 경우 트랜잭션 처리를 어떻게 해야 할지 애매하다.
- 환불을 처리하는 외부 시스템 응답 시간이 길어지면 대기 시간도 길어지기에 성능에 직접적인 영향을 받게 된다.
- 추가 요구 사항이 있을 경우 더 복잡해진다.
    - erx) 주문 취소 시 환불 뿐만 아니라 취소했다는 내용을 통지해야 한다는 추가 요구 사항
    - 로직이 더 비대해지고 트랜잭션 처리가 복잡해진다.

## 10.2 이벤트 개요

- 바운디드 컨텍스트 간 강결합은 이벤트를 사용해 해결할 수 있다.
- ‘~할 때’, ‘~가 발생하면’, ‘만약 ~하면’과 같은 요구사항은 도메인 상태 변경과 관련된 경우가 많고 이는 이벤트를 이용해 구현할 수 있다.

### 10.2.1 이벤트 관련 구성요소

- **이벤트**
- **이벤트 생성 주체**
    - 엔티티, 벨류, 도메인 서비스와 같은 도메인 객체
    - 로직을 실행해 상태가 바뀌면 이벤트를 발생 시킨다.
- **이벤트 디스패처(퍼블리셔)**
    - 이벤트 생성 주체와 이벤트 핸들러를 연결시킨다.
    - 구현 방식에 따라 이벤트 생성과 처리를 동기나 비동기로 실행한다.
- **이벤트 핸들러(구독자)**
    - 이벤트 생성 주체가 발생시킨 이벤트에 반응해 원하는 기능을 실행한다.

### 10.2.2 이벤트의 구성

- 이벤트 종류
    - 클래스 이름으로 이벤트 종류를 표현
    - ex) `ShippingInfoChangedEvent`
- 이벤트 발생 시간
- 추가 데이터
    - 이벤트와 관련된 정보
    - ex) 주문 번호, 신규 배송지 정보 등
    - 이벤트와 관련 없는 정보를 담을 필요는 없다.

### 10.2.3 이벤트 용도

- 첫 번째 용도는 **트리거**다.
    - 도메인 상태가 바뀔 때 다른 후처리가 필요할 때
    - ex) ‘주문 취소’를 트리거로 ‘환불 처리’
    - ex) ‘예매 완료’를 트리거로 ‘SMS 통지’
- 두 번째 용도는 **서로 다른 시스템 간의 동기화**이다.
    - ex) ‘배송지 변경’ 시 외부 배송 서비스에 바뀐 배송지 정보를 전송해야 한다.

### 10.2.4 이벤트 장점

- 이벤트를 사용하면 **서로 다른 도메인 로직이 섞이는 것을 방지**할 수 있다.

    ```java
    public class Order {
    	public void cancel() {
    		verifyNotYetShipped();
    		this.state = OrderState.CANCELED;
    		Events.raise(new OrderCanceldEvent(number.getNumber()));
    	}
    }
    ```

    - ‘주문’ 도메인에서 ‘결제’ 도메인으로의 의존도 제거했다.
- 이벤트를 사용하면 **기능 확장도 용이**하다.
    - 이벤트 생성 주체 코드는 건드릴 필요 없이 이벤트 핸들러만 추가하면 된다.

## 10.3 이벤트, 핸들러, 디스패처 구현

- 이벤트 관련 코드는 다음과 같다.
    - **이벤트 클래스**: 이벤트를 표현
    - **디스패처**: 스프링이 제공하는 `ApplicationEventPublisher`를 이용
    - **Events**: 이벤트를 발행하고`ApplicationEventPublisher`를 사용
    - **이벤트 핸들러**: 스프링이 제공하는 기능을 사용하여 이벤트를 수신 처리한다.

### 10.3.1 이벤트 클래스

```java
public class OrderCancedEvent {
	private String orderNumber;

	// 생성자, getter ...
}
```

- 이벤트 자체를 위한 상위 타입은 존재하지 않는다.
    - 모든 이벤트가 공통으로 갖는 프로퍼티가 있다면 상위 클래스를 만들어 상속하게 할 수도 있다.
- 이벤트 클래스 이름을 결정할 때 과거 시제를 사용하면 된다.
    - ex) `OrderCancedEvent`
- 이벤트 처리에 필요한 최소한의 데이터를 포함해야 한다.

### 10.3.2 Events 클래스와 ApplicationEventPublisher

- 이벤트를 발생시킬 때 스프링이 제공하는 `ApplicationEventPublisher`를 사용하면 된다.
    - `applicationEventPublisher.publishEvent(…)`
- 도메인 엔티티 객체 내부에서 이벤트를 발생시키도록 구현할 수도 있다.

    ```java
    public class Order {
    	public void cancel() {
    		verifyNotYetShipped();
    		this.state = OrderState.CANCELED;
    		Events.raise(new OrderCanceldEvent(number.getNumber()));
    	}
    }
    ```

    - `Events` 클래스는 `ApplicationEventPublisher`를 사용해 이벤트를 발생시키도록 구현한 것이다.

    ```java
    import org.springframework.context.ApplicationEventPublisher;
    
    public class Events {
    	private static ApplicationEventPublisher publisher;
    	
    	static void setPublisher(ApplicationEventPublisher publisher) {
    		Events.publisher = publisher;
    	}
    
    	public static void raise(Object event) {
    		if (publisher != null) {
    			publisher.publisherEvent(event);
    		}
    	}
    }
    ```

    - `Events` 클래스의 초기화는 아래와 같이 구현한다.

    ```java
    @Configuration
    public class EventsConfig {
    	// ApplicationContext는 ApplicationEventPublisher를 상속하고 있다.
    	@AutoWired
    	private ApplicationContext applicationContext;
    
    	@Bean
    	public InitializingBean eventsInitializer() {
    		return () -> Events.setPublisher(applicationContext);
    	}
    }
    ```

    - `InitializingBean` 인터페이스는 스프링 빈 객체를 초기화할 때 사용하는 인터페이스다.

### 10.3.3 이벤트 발생과 이벤트 핸들러

- 이벤트 처리는 스프링이 제공하는 `@EventListener` 애너테이션을 사용해 구현한다.

    ```java
    @EventListener(OrderCancedEvent.class)
    public void handle(OrderCanceldEvent event) {
    	refundService.refund(event.getOrderNumber());
    }
    ```


### 10.3.4 흐름 정리

1. 도메인 기능을 실행
2. 도메인 기능은 `Events.raise()`를 이용해 이벤트 발생시킴
3. `Events.raise()`는 `ApplicationEventPublisher`를 이용해 이벤트 출판
4. `ApplicationEventPublisher`는 `@EventListener` 메서드를 찾아 실행
- 이렇게만 처리하면 이벤트 발생과 이벤트 처리가 같은 트랜잭션 범위 내에서 실행된다.

## 10.4 동기 이벤트 처리 문제

- 이벤트를 이용해 강결합은 해결했지만 **외부 서비스에 영향 받는 문제는 남아있다.**
    - 동기적으로 이벤트가 처리되기에 외부 서비스의 응답 시간이 길어지면 성능에 문제가 발생하는 것은 아직 해결하지 못했다.
    - 같은 트랜잭션으로 묶이기에 외부 서비스에서 예외가 발생했을 때 이벤트 발생 로직도 롤백할지, 커밋할지 애매한 문제도 남아있다.
- 위 문제를 해소하는 방법은 두 가지가 있다.
    - 이벤트를 비동기로 처리
    - 이벤트와 트랜잭션을 연계
