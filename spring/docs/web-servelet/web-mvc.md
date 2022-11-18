# [1. Spring Web MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc)

Spring Web MVC는 Servlet API를 기반으로 구축된 오리지널 웹 프레임워크이며 스프링 프레임워크에 처음부터 포함되었다.

정식 명칭인 “Spring Wev MVC”는 소스 모듈의 이름에서 유래했지만, 더 일반적으로 “Spring MVC”로 알려져 있다.

## [1.1. DispatcherServlet](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet)

Spring MVC는 다른 많은 웹 프레임워크와 마찬가지로 중앙 서블릿인 프론트 컨트롤러 패턴을 중심으로 설계되었다.

중앙 서블릿인 `DispatcherServlet`은 요청을 처리하는 공유된 알고리즘을 제공하지만 실제 작업은 구성 가능한 위임된 컴포넌트(configurable delegate components)가 수행한다.

이 모델은 유연하고 다양한 워크 플로우를 지원한다.

`DispatcherServlet`은 다른 `servlet`과 마찬가지로 자바 configuration이나 web.xml을 사용하여 Servlet 사양에 따라 매핑되어야 한다.

또한 `DispatcherServlet`은 Spring Configuration을 사용하여 request mapping, view resolution, exception handling 등을 찾는다.

## [1.7. CORS](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-cors)

Spring MVC는 CORS를 핸들링할 수 있는 방법을 제공한다.

### 1.7.1 Introduction

보안의 이유로 브라우저는 다른 origin에 있는 리소스에 대한 AJAX 호출을 금지한다.

CORS(Cross-Origin Resource Sharing)는 IFRAME 또는 JSONP를 기반으로 하는 덜 안전한 방법을 사용하는 대신 어떤 종류의 교차 도메인 요청을 승인할지 지정할 수 있는 대부분의 브라우저에서 구현하는 W#C specification이다.

### 1.7.2 Processing

CORS 사양은 preflight, simple, actual request로 나뉜다. 더 자세한 내용은 [이 글](https://developer.mozilla.org/ko/docs/Web/HTTP/CORS) 참고

Spring MVC `HandlerMapping` 구현은 CORS에 대한 지원을 제공한다. 성공적으로 핸들러에 대한 요청을 매핑한 후, `HandlerMapping` 구현체는 CORS configuration을 체크하고 추가 작업을 수행한다. preflight 요청은 직접 처리되는 반면 simple과 actual CORS 요청은 인터셉트 되어 검증되고 필요한 CORS 응답을 헤더에 설정한다.

cross-origin 요청을 허용해주기 위해선 명시적인 CORS configuration이 있어야 한다. 일치하는 CORS configuration이 없으면 preflight 요청이 거부된다. 그리고 simple과 actual CORS 요청에 대해 알맞은 CORS 응답 헤더가 추가되지 않으므로 브라우저는 이를 거부한다.

각 `HandlerMapping`은 URL 패턴 기반 `CorsConfiguration` 매핑을 이용하여 개별적으로 구성할 수 있다. 많은 경우에 애플리케이션들이 MVC java configuration이나 XML namespace를 사용하여 매핑을 선언하며, 하나의 글로벌 매핑이 모든 `HandlerMapping` 인스턴스에 전달된다.

`HandlerMapping` 수준의 글로벌 CORS configuration을 보다 세분화된 handler-level CORS configuration과 결합할 수 있다. 예를 들어 controller 주석이 달린 클래스 또는 메서드 수준에 `@CrossOrigin` 주석을 사용할 수 있다.

### [1.7.3 @CrossOrigin](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-cors-controller)

`@CrossOrigin` 주석은 다음 예제와 같이 주석이 달린 컨트롤러 메서드에서 cross-origin 요청을 활성화한다. (클래스 레벨에도 붙일 수 있으며 그 경우 클래스 하위 모든 메서드에 적용된다.)

```java
@RestController
@RequestMapping("/account")
public class AccountController {

    @CrossOrigin
    @GetMapping("/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @DeleteMapping("/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```

기본적으로 `@CrossOrigin`은 다음을 모두 허용한다.

- All origins
- All headers
- All HTTP methods

그런데 이렇게 하면 중요한 사용자별 정보(쿠키 및 CSRF 토큰)를 노출하는 신뢰 수준이 설정되므로 `allowCredentials`는 활성화되지 않으며 적절한 경우에만 사용해야 한다. `allowOrigins`를 사용하도록 설정한 경우 하나 이상의 특정 도메인(”*”이 아닌)으로 설정하거나 `allowOriginPatterns` 속성을 사용하여 동적 origin 집합과 일치시킬 수 있다.

### [1.7.4 Global ****Configuration****](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-cors-global)

`HandlerMappging`에서 URL 기반 `CorsConfiguration` 매핑을 개별적으로 설정할 수 있다.  하지만 대부분 MVC java configuration이나 MVC XML namespace를 사용하여 이 작업을 수행한다.

기본적으로 global configuration은 다음을 모두 허용한다.

- All origins
- All headers
- GET, HEAD, POST methods

그런데 이렇게 하면 중요한 사용자별 정보(쿠키 및 CSRF 토큰)를 노출하는 신뢰 수준이 설정되므로 `allowCredentials`는 활성화되지 않으며 적절한 경우에만 사용해야 한다. `allowOrigins`를 사용하도록 설정한 경우 하나 이상의 특정 도메인(”*”이 아닌)으로 설정하거나 `allowOriginPatterns` 속성을 사용하여 동적 origin 집합과 일치시킬 수 있다.

### **Java Configuration**

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {

        registry.addMapping("/api/**")
            .allowedOrigins("https://domain2.com")
            .allowedMethods("PUT", "DELETE")
            .allowedHeaders("header1", "header2", "header3")
            .exposedHeaders("header1", "header2")
            .allowCredentials(true).maxAge(3600);

        // Add more mappings...
    }
}
```
