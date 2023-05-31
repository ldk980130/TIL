# CORS
- CORS (Cross-Origin Resurce Sharing)은 브라우저가 서버에 리소스를 요청할 때 허용해야 하는 Origins을 표시하도록 하는 HTTP 헤더 기반 메커니즘이다.
- 일반적으로 CORS는 브라우저가 교차 출처 리소스를 호스팅하는 서버에 preflight 요청을 통해 서버가 요청을 허용하는지 확인한다.
    - preflight에 브라우저는 실제 요청에 해당하는 HTTP 헤더와 메서드를 전송한다.
- 보안상의 이유로 브라우저는 기본적으로 cross-origin (교차 출처) 요청을 제한한다.
- CORS 메커니즘은 브라우저와 서버 간 안전한 교차 출처 요청 및 데이터 전송을 지원한다.

## SOP (Same-Origin Policy)

- SOP란 같은 출처에서만 리소스를 공유할 수 있다는 규칙
    - 프로토콜, 도메인, 포트가 같다면 같은 Origin
- SOP 제약이 없다면 CSRF나 XSS와 같은 보안 공격에 취약할 수밖에 없다.
- 다른 Origin에서의 요청으로 리소스를 요청하면 브라우저는 CORS 에러를 발생시킨다.
- 즉 CORS란 다른 출처의 리소스 공유에 대한 허용/비허용 정책이다.

## CORS 동작 과정

- CORS에는 3가지 요청 시나리오가 존재하지만 모든 시나리오가 아래의 과정을 따른다.
1. 클라이언트에서 HTTP요청의 헤더에 `Origin`을 담아 전달
    1. `Origin: https://does.com`
2. 서버는 응답헤더에 `Access-Control-Allow-Origin`을 담아 클라이언트로 전달
    1. 이 외에도 `Access-Control-Allow-Headers`, `Access-Control-Allow-Methods`가 있다.
3. 클라이언트에서 `Origin`과 서버가 보내준 `Access-Control-Allow-Origin`을 비교
    1. 만약 유효하지 않다면 그 응답을 사용하지 않고 버린다. (CORS 에러)
    2. 서버가 허용하지 않은 HTTP 헤더나 메서드로 요청을 하는 경우에도 CORS 에러가 발생한다.

### 시나리오 1 - Preflight

- 가장 일반적인 시나리오로 요청을 한 번에 보내지 않고 예비 요청과 본 요청으로 나누어 서버로 전송한다.
- 본 요청 전 예비 요청으로 HTTP OPTIONS 메서드를 사용해 CORS 허용 여부를 확인한다.
    - `Access-Control-Max-Age` 헤더를 통해 캐시될 시간을 명시하면 예비 요청을 캐싱 시켜 그 다음 요청부터 본 요청을 바로 보내도록 최적화를 시킬 수도 있다.
- 예비 요청에서 CORS 에러가 발생하면 본 요청을 보내지 않는다.

### 시나리오 2 - Simple Request

- Simple Request는 예비 요청을 보내지 않고 바로 본 요청을 보내면서 CORS 허용 여부를 판단하는 방식이다.
- 본 요청을 보낸 후 서버의 응답에 CORS 관련 헤더가 포함되고 이 헤더를 검증하여 위반 여부를 판단한다.
- Simple Request를 사용하는 조건
    - 요청의 메소드는 `GET`, `HEAD`, `POST` 중 하나여야 한다.
    - `Accept`, `Accept-Language`, `Content-Language`, `Content-Type`, `DPR`, `Downlink`, `Save-Data`, `Viewport-Width`, `Width` 헤더일 경우 에만 적용된다.
    - `Content-Type` 헤더가 `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain`중 하나여야한다.
- Simple Request를 사용할 수 없는 요청이라면 Preflight Request로 동작한다.

> 일반적으로 Rest API 방식으로 Json 데이터를 주고 받는 경우가 많기에 Preflight 방식이 대부분 사용된다고 보면 된다.
>

### 시나리오 3 - Credentialed Request

- 클라이언트에서 서버에게 자격 인증 정보(`Credential`)를 실어 요청할때 사용되는 요청 (쿠키, 헤더의 토큰 등)
- 브라우저에서 기본적으로 쿠키 등의 인증 관련 정보를 요청에 담지 않도록 하지만 Credentialed Request의 경우 요청에 인증 정보를 담게 된다.
- 클라이언트에서 인증 정보를 보내도록 설정하기
    - axios 라이브러리를 사용하는 경우: `withCredentials: true`
- 서버에서 인증된 요청에 대한 헤더 설정하기
    - 응답 헤더의 `Access-Control-Allow-Credentials` 항목을 `true`로 설정해야 한다.
    - 응답 헤더의 `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`,  `Access-Control-Allow-Headers`의 값에 와일드카드 문자(`"*"`)는 사용할 수 없다.
- Credentialed Request에도 예비 요청을 포함한다.

## Spring 서버에서 CORS 허용하기

### 스프링 MVC에서의 설정

```java

@Configuration
public class WebConfig implements WebMvcConfigurer {
   
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("http://localhost:8080", "http://localhost:8081")
                .allowedMethods("GET", "POST")
                .allowCredentials(true)
                .maxAge(3000); 
    }
}
```

### 스프링 시큐리티 사용 시 설정

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
                .cors()
                .configurationSource(corsConfigurationSource())    

        // ...
    }

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration corsConfig = new CorsConfiguration();
        corsConfig.setAllowedOriginPatterns(List.of(corsAllowedOrigins.split(",")));
        corsConfig.setAllowedMethods(List.of("HEAD","POST","GET","DELETE","PUT", "OPTIONS"));
        corsConfig.setAllowedHeaders(List.of("*"));
        corsConfig.setAllowCredentials(true);
        corsConfig.setExposedHeaders(List.of(HttpHeaders.LOCATION, HttpHeaders.SET_COOKIE));

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", corsConfig);
        return source;
    }
```

## CORS 테스트 해보기

- 아래 요청에 200이면 허용, 403이면 거부

```bash
curl \
--verbose \
--request OPTIONS \
'http://localhost:8080/api/test' \
--header 'Origin: http://localhost:3000' \
--header 'Access-Control-Request-Headers: Origin, Accept, Content-Type' \
--header 'Access-Control-Request-Method: DELETE'
```

---

https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS

https://inpa.tistory.com/entry/WEB-%F0%9F%93%9A-CORS-%F0%9F%92%AF-%EC%A0%95%EB%A6%AC-%ED%95%B4%EA%B2%B0-%EB%B0%A9%EB%B2%95-%F0%9F%91%8F#%F0%9F%92%AC_%EA%B2%B0%EA%B5%AD_cors_%ED%95%B4%EA%B2%B0%EC%B1%85%EC%9D%80_%EC%84%9C%EB%B2%84%EC%9D%98_%ED%97%88%EC%9A%A9%EC%9D%B4_%ED%95%84%EC%9A%94
