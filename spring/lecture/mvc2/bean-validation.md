# Bean Validation
## **Bean Validation**

- 먼저 Bean Validation은 특정한 구현체가 아니라 Bean Validation 2.0(JSR-380)이라는 기술 표준이다.
- 쉽게 이야기해서 검증 애노테이션과 여러 인터페이스의 모음이다. 마치 JPA가 표준 기술이고 그 구현체로 하이버네이트가 있는 것과 같다
- Bean Validation을 구현한 기술 중에 일반적으로 사용하는 구현체는 하이버네이트 Validator이다. 이름이 하이버네이트가 붙어서 그렇지 ORM과는 관련이 없다.
- 검증 애노테이션 모음:  [https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/#validator-defineconstraints-spec](https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/#validator-defineconstraints-spec)

## **Bean Validation 애노테이션 적용**

```java
@Data 
public class Item {   

		private Long id;  
 
		@NotBlank  
		private String itemName;  
 
		@NotNull  
		@Range(min = 1000, max = 1000000)  
		private Integer price;   

		@NotNull 
		@Max(9999)  
		private Integer quantity;
```

- **@NotBlank** : 빈값 + 공백만 있는 경우를 허용하지 않는다.
- **@NotNull** : null을 허용하지 않는다.
- **@Range**(min = 1000, max = 1000000) : 범위 안의 값이어야 한다.
- **@Max**(9999) : 최대 9999까지만 허용한다.

## **Bean Validation - 에러 코드**

NotBlank 라는 오류 코드를 기반으로 `MessageCodesResolver`를 통해 다양한 메시지 코드가 순서대로생성된다.

**@NotBlank**

- NotBlank.item.itemName
- NotBlank.itemName
- NotBlank.java.lang.String
- NotBlank

**@Range**

- Range.item.price
- Range.price
- Range.java.lang.Integer
- Range

## **메시지 등록**

**errors.properties**

```yaml
# Bean Validation 추가 
NotBlank={0} 공백X 
Range={0}, {2} ~ {1} 허용
Max={0}, 최대{1}
```

- {0} 은 필드명이고, {1} , {2} ...은 각 애노테이션 마다 다르다.

### **BeanValidation 메시지 찾는 순서**

1. 생성된 메시지 코드 순서대로 messageSource에서 메시지 찾기
2. 애노테이션의 message 속성 사용 `@NotBlank(message = "공백! {0}")`
3. 라이브러리가 제공하는 기본 값 사용 -> 공백일 수 없습니다.

## **Bean Validation - 오브젝트 오류**

**글로벌 오류 추가**

```yaml
if (item.getPrice() != null && item.getQuantity() != null) {
		int resultPrice = item.getPrice() * item.getQuantity();  
		if (resultPrice < 10000) {   
				bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null); 
		}
}
```

- ScriptAssert를 이용하는 방법이 있지만 기능이 제한적이라 자바 코드로 글로벌 오류를 추가하는 것을 추천.

## **Bean Validation - groups**

동일한 모델 객체를 등록할 때와 수정할 때 각각 다르게 검증하는 방법을 알아보자

### **방법 2가지**

- BeanValidation의 `groups`기능을 사용한다.
- Item을 직접 사용하지 않고, ItemSaveForm, ItemUpdateForm 같은 폼 전송을 위한 별도의 모델 객체를 만들어서 사용한다

### **BeanValidation groups 기능 사용**

1. groups를 체크하기 위한 인터페이스를 만든다 
2. 적용하고 싶은 필드에 groups를 적용한다.

```java
public class Item {

		@NotNull(groups = UpdateCheck.class)
		private Long id;

		@NotBlank(groups = {SaveCheck.class, UpdateCheck.class})
		private String itemName;

		@NotNull(groups = {SaveCheck.class, UpdateCheck.class})
		@Range(min = 1000, max = 1000000, groups = {SaveCheck.class, UpdateCheck.class})
		private Integer price;

		@NotNull(groups = {SaveCheck.class, UpdateCheck.class})
		@Max(value = 9999, groups = {SaveCheck.class})
		private Integer quantity;
```

1. 컨트롤러 Validated 어노테이션에 추가해준다.

`@Validated(SaveCheck.class)`

`@Validated(UpdateCheck.class)`

참고: `@Valid`에는 `groups`를 적용할 수 있는 기능이 없다. 따라서 `groups`를 사용하려면 `@Validated` 를 사용해야 한다.

사실 `groups`기능은 실제 잘 사용되지는 않는데, 그 이유는 실무에서는 주로 다음에 등장하는 등록용 폼 객체와 수정용 폼 객체를 분리해서 사용하기 때문이다

### **Form 전송 객체 분리**

수정의 경우 등록과 수정은 완전히 다른 데이터가 넘어온다. 생각해보면 회원 가입시 다루는 데이터와 수정시 다루는 데이터는 범위에 차이가 있다. 예를 들면 등록시에는 로그인id, 주민번호 등등을 받을 수 있지만, 수정시에는 이런 부분이 빠진다. 그리고 검증 로직도 많이 달라진다. 그래서

## **Bean Validation - HTTP 메시지 컨버터**

`@Valid` , `@Validated`는 `HttpMessageConverter` (`@RequestBody` )에도 적용할 수 있다

```java
@PostMapping("/add") 
public Object addItem(
		@RequestBody @Validated ItemSaveForm form, 
		BindingResult bindingResult
)
```

- `@ModelAttribute`는 HTTP 요청 파라미터(URL 쿼리 스트링, POST Form)를 다룰 때 사용한다.
- `@RequestBody`는 HTTP Body의 데이터를 객체로 변환할 때 사용한다. 주로 API JSON 요청을 다룰 때 사용한다.

### **@ModelAttribute vs @RequestBody**

`@ModelAttribute`는 각각의 필드 단위로 세밀하게 적용된다.

`HttpMessageConverter`는 `@ModelAttribute`와 다르게 각각의 필드 단위로 적용되는 것이 아니라, 전체 객체 단위로 적용된다.

- `@ModelAttribute`는 필드 단위로 정교하게 바인딩이 적용된다. 특정 필드가 바인딩 되지 않아도 나머지 필드는 정상 바인딩 되고, Validator를 사용한 검증도 적용할 수 있다.
- `@RequestBody`는 `HttpMessageConverter`단계에서 JSON 데이터를 객체로 변경하지 못하면 이후 단계 자체가 진행되지 않고 예외가 발생한다. 컨트롤러도 호출되지 않고, `Validator`도 적용할 수 없다.
