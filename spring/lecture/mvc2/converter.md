# Converter
## **스프링 타입 컨버터**

### **스프링 타입 컨버터 소개**

문자를 숫자로 변환하거나, 반대로 숫자를 문자로 변환해야 하는 것처럼 애플리케이션을 개발하다 보면 타입을 변환해야 하는 경우가 상당히 많다.

```java
@GetMapping("/hello-v1") 
public String helloV1(HttpServletRequest request) {    
		String data = request.getParameter("data"); // 문자 타입 조회    
		Integer intValue = Integer.*valueOf*(data); // 숫자 타입으로 변경    
		System.*out*.println("intValue = " + intValue);    
		return "ok"; 
}
```

- `String data = request.getParameter("data")`
- HTTP 요청 파라미터는 모두 문자로 처리된다. 따라서 요청 파라미터를 자바에서 다른 타입으로 변환해서 사용하고 싶으면 다음과 같이 숫자 타입으로 변환하는 과정을 거쳐야 한다.

```java
@GetMapping("/hello-v2")
public String helloV2(@RequestParam Integer data) {
		System.*out*.println("data = " + data);
		return "ok";
}
```

- 스프링이 제공하는 `@RequestParam`을 사용하면 이 문자 10을 Integer 타입의 숫자 10으로 편리하게 받을 수 있다.
- 이것은 스프링이 중간에서 타입을 변환해주었기 때문이다
- 이러한 예는 `@ModelAttribute`, `@PathVariable`에서도 확인할 수 있다.
- 문자를 숫자로 변경하는 예시를 들었지만, 반대로 숫자를 문자로 변경하는 것도 가능하고, Boolean 타입을 숫자로 변경하는 것도 가능하다. 만약 개발자가 새로운 타입을 만들어서 변환하고 싶으면 어떻게 하면 될까?

### **컨버터 인터페이스**
```java
package org.springframework.core.convert.converter;

public interface Converter<S, T> {
    T convert(S sourcE);
}
```

## **타입 컨버터 - Converter**

**예) `StringToIntegerConverter` - 문자를 숫자로 변환하는 타입 컨버터**

```java
public class StringToIntegerConverter implements Converter<String, Integer> {

		@Override
		public Integer convert(String source) {
				return Integer.valueOf(source);
		}
}
```

### **사용자 정의 타입 컨버터**

**IpPort.class**

```java
@Getter
@EqualsAndHashCode 
public class IpPort {

		private String ip;
		private int port;

		public IpPort(String ip, int port) {
				this.ip = ip;
				this.port = port;
		}
}
```

**StringToIpPortConverter**

```java
public class StringToIpPortConverter implements Converter<String, IpPort> {

		@Override
		public IpPort convert(String source) {
				//"127.0.0.1:8080"
				String[] split = source.split(":");
				String ip = split[0];
				int port = Integer.parseInt(split[1]);
				return new IpPort(ip, port);
		}
}
```

**IpPortToStringConverter**

```java
public class IpPortToStringConvert implements Converter<IpPort, String> {

		@Override
		public String convert(IpPort source) {
				return source.getIp() + ":" + source.getPort();
		}
}
```

**참고**

- 스프링은 용도에 따라 다양한 방식의 타입 컨버터를 제공한다.
- `Converter` -> 기본 타입 컨버터
- `ConverterFactory`-> 전체 클래스 계층 구조가 필요할 때
- `GenericConverter`-> 정교한 구현, 대상 필드의 애노테이션 정보 사용 가능
- `ConditionalGenericConverter`-> 특정 조건이 참인 경우에만 실행
- 스프링은 문자, 숫자, 불린, Enum 등 일반적인 타입에 대한 대부분의 컨버터를 기본으로 제공한다.
- IDE에서 `Converter`, `ConverterFactory`, `GenericConverter`의 구현체를 찾아보면 수 많은 컨버터를 확인할 수 있다.

## **컨버전 서비스 - ConversionService**

이렇게 타입 컨버터를 하나하나 직접 찾아서 타입 변환에 사용하는 것은 매우 불편하다. 그래서 스프링은 개별 컨버터를 모아두고 그것들을 묶어서 편리하게 사용할 수 있는 기능을 제공하는데, 이것이 바로 컨버전 서비스(`ConversionService`)이다.

### **ConversionService 인터페이스**

```java
public interface ConversionService {

		boolean canConvert(@Nullable Class<?> sourceType, Class<?> targetType);

		boolean canConvert(@Nullable TypeDescriptor sourceType, TypeDescriptor targetType);

		@Nullable
		<T> T convert(@Nullable Object source, Class<T> targetType);

		@Nullable
		Object convert(@Nullable Object source, @Nullable TypeDescriptor sourceType, TypeDescriptor targetType);
}
```

컨버전 서비스 인터페이스는 단순히 컨버팅이 가능한가? 확인하는 기능과, 컨버팅 기능을 제공한다.

### **DefaultConversionService**

`DefaultConversionService`는 `ConversionService`인터페이스를 구현했는데, 추가로 컨버터를 등록하는 기능도 제공한다.

```java
@Test

public void conversionService() throws Exception {
		//given
		DefaultConversionService conversionService = new DefaultConversionService();

		//when
		conversionService.addConverter(new StringToIntegerConverter());
		conversionService.addConverter(new IntegerToStringConverter());
		conversionService.addConverter(new StringToIpPortConverter());
		conversionService.addConverter(new IpPortToStringConverter());

		//then
		assertThat(conversionService.convert("10", Integer.class)).isEqualTo(10);
		assertThat(conversionService.convert(10, String.class)).isEqualTo("10");
		assertThat(conversionService.convert("127.0.0.1:8080", IpPort.class))
				.isEqualTo(new IpPort("127.0.0.1", 8080));
		assertThat(conversionService.convert(new IpPort("127.0.0.1", 8080), String.class))
				.isEqualTo("127.0.0.1:8080");
}
```

**등록과 사용 분리**

컨버터를 등록할 때는 타입 컨버터를 명확하게 알아야 한다. 반면에 컨버터를 사용하는 입장에서는 타입 컨버터를 전혀 몰라도 된다. 따라서 타입을 변환을 원하는 사용자는 컨버전 서비스 인터페이스에만 의존하면 된다. 물론 컨버전 서비스를 등록하는 부분과 사용하는 부분을 분리하고 의존 관계 주입을 사용해야 한다

**인터페이스 분리 원칙 - ISP(Interface Segregation Principal)**

인터페이스 분리 원칙은 클라이언트가 자신이 이용하지 않는 메서드에 의존하지 않아야 한다.

`DefaultConversionService`는 다음 두 인터페이스를 구현했다.

- `ConversionService`: 컨버터 사용에 초점
- `ConverterRegistry`: 컨버터 등록에 초점
- 컨버터를 사용하는 클라이언트는 `ConversionService`만 의존하면 되므로, 컨버터를 어떻게 등록하고 관리하는지는 전혀 몰라도 된다. 결과적으로 컨버터를 사용하는 클라이언트는 꼭 필요한 메서드만 알게된다.
- 스프링은 내부에서 `ConversionService`를 사용해서 타입을 변환한다.

## **스프링에 Converter 적용하기**

### **WebConfig - 컨버터 등록**

```java
@Configuration 
public class WebConfig implements WebMvcConfigurer {
		@Override    
		public void addFormatters(FormatterRegistry registry) {       
				registry.addConverter(new StringToIntegerConverter());       
				registry.addConverter(new IntegerToStringConverter());       
				registry.addConverter(new StringToIpPortConverter());       
				registry.addConverter(new IpPortToStringConverter());    
		} 
}
```

스프링은 내부에서 `ConversionService`를 제공한다. 우리는 `WebMvcConfigurer`가 제공하는 `addFormatters()`를 사용해서 추가하고 싶은 컨버터를 등록하면 된다. 이렇게 하면 스프링은 내부에서 사용하는 `ConversionService`에 컨버터를 추가해준다

**IpPort 변환 예**

```java
// http://localhost:8080/ip-port?ipPort=127.0.0.1:8080
@GetMapping("/ip-port") 
public String ipPort(@RequestParam IpPort ipPort) {    
		System.*out*.println("ipPort Ip = " + ipPort.getIp());    
		System.*out*.println("ipPort port = " + ipPort.getPort());    
		return "ok"; 
}
```

- ?ipPort=127.0.0.1:8080 쿼리 스트링이 `@RequestParam IpPort ipPort` 에서 객체 타입으로 잘 변환 된 것을 확인할 수 있다.
- `@RequestParam`은 `@RequestParam`을 처리하는 `ArgumentResolver`인`RequestParamMethodArgumentResolver`에서 `ConversionService`를 사용해서 타입을 변환한다.
