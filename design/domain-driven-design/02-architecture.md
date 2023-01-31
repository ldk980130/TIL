# Chapter2 아키텍처 개요

## 2.1 네 개의 영역

‘표현’, ‘응용’, ‘도메인’, ‘인프라스트럭처’는 아키텍처를 설계할 때 출현하는 전형적인 네 가지 영역이다.

### 표현 영역

- UI 영역이라고도 불린다.
- 표현 영역은 사용자 요청을 받아 응용 영역에 전달하고 결과를 받아 다시 사용자에게 보여주는 역할을 한다.
    - HTTP 요청을 응용 영역이 필요로 하는 형식으로 변환하고 응답을 다시 HTTP 응답으로 변환하여 전송한다.
- 스프링 MVC 프레임워크가 표현 영역을 위한 기술에 해당한다.
- 표현 영역의 사용자는 사람일 수도, REST API를 호출하는 외부 시스템일 수도 있다.

### 응용 영역

- 응용 영역은 시스템이 사용자에게 제공해야 할 기능을 구현한다.
    - ‘주문 등록’, ‘주문 최소’ 등
- 응용 영역은 기능 구현을 위해 도메인 영역의 도메인 모델을 사용한다.
    - 로직을 직접 수행하기보다 도메인 모델에 로직 수행을 위임

```java
public class OrderService {
	
	@Transactional
	public void cancelOrder(String orderId) {
		Order order = getOrderById(orderId);
		order.cancel();
	}

	// ...
}
```

### 도메인 영역

- 도메인 영역은 도메인 모델을 구현한다.
- 도메인 모델은 도메인의 핵심 로직을 구현한다.

### 인프라스트럭처 영역

- 인프라스트럭처 영역은 논리적 개념을 표현하기보다 실제 구현 기술에 대한 것을 다룬다.
    - RDBMS, 메시징 큐, 레디스 등
- 도메인 영역, 응용영역, 표현 영역은 구현 기술을 사용한 코드를 직접 만들지 않는다.

## 2.2 계층 구조 아키텍처

- 네 영역을 구성할 때 일반적으로 표현 영역과 응용 영역은 도메인 영역을 사용하고, 도메인 영역은 인프라스트럭처 영역을 사용한다.
    - 도메인 복잡도에 따라 응용과 도메인을 분리하기도 하고 합치기도 한다.
- 계층 구조를 엄격하게 하면 의존 방향이 상위 계층에서 하위 계층으로만 흐른다.
    - 표현 → 응용 → 도메인 → 인프라스트럭처

### 인프라스트럭처 계층에 종속되는 문제

- 구현의 편리함을 위해 응용 계층이 인프라스트럭처 계층에 의존하기도 한다.
- 이 때 발생하는 두 가지 문제점이 있다.
    - 응용 계층을 테스트하기 어렵다.
    - 구현 방식을 변경하기 어렵다.
- 아래 코드는 가격 할인을 담당하는 응용 계층이 인프라 계층인 `RuleEngine` 클래스를 의존하는 예시이다.

    ```java
    public class CalculateDiscountServie {
    
    	private final DroolsRuleEngine ruleEngine;
    
    	// ...
    
    	public Money calculateDiscount(OrderLine orderLines, String customerId) {
    		Customer customer = findCustomer(customerId);
    
    		MutableMoney money = new MutableMoney(0);
    		List<?> facts = Arrays.asList(customer, money);
    		facts.addAll(orderLines);
    		ruleEngine.evalute("discountCalculation", facts);
    		return money.toImmutableMoney();
    	}
    ```

    - `CalculateDiscountService`를 테스트하려면 `RuleEngine`이 완벽하게 동작해야 하기에 테스트가 번거롭다.
    - `caculateDiscount()` 메서드 안의 코드는 `RuleEngine` 클래스를 사용하기위해 필요한 코드들이기 때문에 다른 구현 기술을 사용하려면 이 코드들을 다 고쳐야 한다.
