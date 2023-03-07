# Expression-Based Access Control

> 스프링 시큐리티는 SpEL 표현식을 사용한다. 표현식 평가는 root object를 사용하여 평가된다. 스프링 시큐리티는 특정 클래스를 루트 객체로 사용하여 기본 제공 표현식을 제공하고 현재와 같은 엑세스 규칙을 제공한다.
>

## [Common Built-In Expressions](https://docs.spring.io/spring-security/reference/servlet/authorization/expression-based.html#el-common-built-in)

- `SecurityExpressionRoot`가 바로 표현식 루트 객체이다.
    - 웹과 메서드 시큐리티에 모두 사용할 수 있는 일반적인 표현식이 제공된다.

| Expression | Description |
| --- | --- |
| hasRole(String role) | 현재 principal이 지정된 role을 가지고 있으면 true를 반환한다. |
| hasAnyRole(String…​ roles) | 현재 principal이 제공된 역할 중 하나라도 가지고 있으면 true를 반환한다. |
| hasAuthority(String authority) | 현재 principal이 지정된 authority를 가지고 있으면 true를 반환한다. |
| hasAnyAuthority(String…​ authorities) | 현재 principal이 제공된 authority들을 가지고 있으면 true를 반환한다. |
| principal | 현재 사용자를 나타내는 principal 객체에 접근할 수 있다. |
| authentication | SecurityContext에서 가져온 Authentication 객체에 직접 엑세스할 수 있다. |
| permitAll | 항상 true를 반환 |
| denyAll | 항상 false를 반환 |
| isAnonymous() | 현재 사용자가 익명 사용자면 true를 반환한다. |
| isRememberMe() | 현재 사용자가 remember-me 사용자면 true를 반환한다. |
| isAuthenticated() | 현재 사용자가 익명도이 아니면 true를 반환한다. |
| isFullyAuthenticated() | 현재 사용자가 익명도, rememer-me도 아니면 true를 반환한다. |
| hasPermission(Object target, Object permission) | 사용자가 주어진 권한에 대해 제공된 대상에 접근할 수 있으면 true를 반환한다. ex) hasPermission(domainObject, ‘read’) |
| hasPermission(Object targetId, String targetType, Object permission) | 사용자가 주어진 권한에 대해 제공된 대상에 접근할 수 있으면 true를 반환한다. ex) hasPermission(1, 'com.example.domain.Message', 'read'). |

## [Referring to Beans in Web Security Expressions](https://docs.spring.io/spring-security/reference/servlet/authorization/expression-based.html#el-access-web-beans)

- 사용 가능한 표현식을 확장하려는 경우 스프링 빈을 쉽게 참조할 수 있다.

```java
public class WebSecurity {
		public boolean check(Authentication authentication, HttpServletRequest request) {
				...
		}
}
```

```java
http
    .authorizeHttpRequests(authorize -> authorize
        .requestMatchers("/user/**").access(new WebExpressionAuthorizationManager("@webSecurity.check(authentication,request)"))
        ...
    )
```

## [Path Variables in Web Security Expressions](https://docs.spring.io/spring-security/reference/servlet/authorization/expression-based.html#el-access-web-path-variables)

- URL 내에서 경로 변수를 참조하려면 다음과 같이 할 수 있다.

```java
public class WebSecurity {
		public boolean checkUserId(Authentication authentication, int id) {
				...
		}
}
```

```java
http
	.authorizeHttpRequests(authorize -> authorize
		.requestMatchers("/user/{userId}/**").access("@webSecurity.checkUserId(authentication,#userId)")
		...
	);
```

## [Method Security Expressions](https://docs.spring.io/spring-security/reference/servlet/authorization/expression-based.html#_method_security_expressions)

- Method Security는 단순 허용, 거부 규칙보다 조금 더 복잡하다.
- Spring Security 3.0에서는 표현식 사용에 대한 포괄적인 지원을 위해 몇 가지 새로운 어노테이션을 도입했다.

### @Pre and @Post Annotaions

- 호출 전후의 권한 확인을 허용하고 제출된 컬렉션 인수 또는 리턴 값의 필터링을 지원하는 네 가지 어노테이션이 있다.
    - `@PreAuthorize`
    - `@PreFilter`
    - `@PostAuthorize`
    - `@PostFilter`
- 메서드 시큐리티 기능을 사용하려면 `global-method-security`를 설정해야 한다.

```xml
<global-method-security pre-post-annotations="enabled"/>
```

### Access Control using @PreAuthorize and @PostAuthorize

- 가장 많이 사용되는 어노테이션은 `@PreAuthorize`으로 메서드가 호출될 수 있는지 없는지를 결정한다.

    ```java
    @PreAuthorize("hasRole('USER')")
    public void create(Contact contact);
    ```

    - `ROLE_USER`에게만 위 메서드를 허용한다.
    - 이 예제의 경우 일반 설정을 통해서도 할 수 있긴하다.
- 아래 예제는 현재 사용자가 `contant`에 `admin` 권한을 가지고 있는지 검사한다.

    ```java
    @PreAuthorize("hasPermission(#contact, 'admin')")
    public void deletePermission(Contact contact, Sid recipient, Permission permission);
    ```

    - built-in `hasPermission()` 표현식은 Spring Security ACL 모듈에 연결된다.
    - 표현식 변수의 이름을 통해 메서드 argunemt에 접근할 수 있다.
- Spring Security는 여러 방법으로 메서드 argument를 확인할 수 있다.
    - `DefaultSecurityParameterNameDiscover`를 이용
    - `@P`나 `@Param` 어노테이션을 통해 아래와 같이 사용할 수도 있다.

    ```java
    import org.springframework.security.access.method.P;
    
    @PreAuthorize("#c.name == authentication.name")
    public void doSomething(@P("c") Contact contact);
    ```

    ```java
    import org.springframework.data.repository.query.Param;
    
    @PreAuthorize("#n == authentication.name")
    Contact findContactByName(@Param("n") String name);
    ```

    - 바로 위 코드 표현식에 `authentication`이 등장하는데 이는 Security Context 내부의 `Authentication`이다.
    - Context 내부의 `UserDetails`에도 `principal` 표현식으로 접근할 수 있다.

### Filtering using @PreFilter and @PostFilter

- Spring Security는 표현식을 사용하여 collections, arrays, maps, streams의 필터링을 지원한다.
    - 리턴 값에 대해 가장 일반적으로 수행된다.
- `@PostFilter` 어노테이션을 사용하면 반환된 컬렉션 또는 맵을 반복하여 표현식이 거짓인 요소를 모두 제거한다.

    ```java
    @PreAuthorize("hasRole('USER')")
    @PostFilter("hasPermission(filterObject, 'read') or hasPermission(filterObject, 'admin')")
    public List<Contact> getAll();
    ```

    - `filterObject`는 현재 컬렉션의 객체를 참조한다.
    - `Map`이 반환 값인 경우 `filterObject.key` 또는 `filterObject.value`를 사용할 수 있다.

## [Built-In Expressions](https://docs.spring.io/spring-security/reference/servlet/authorization/expression-based.html#el-method-built-in)

- method security에 많은 built-in 표현식이 있지만 `hasPermission()` 표현식은 자세히 살펴볼 필요가 있다.

### The PermissionEvaluator interface

- hasPermission() 표현식은 PermissionEvaluator 인스턴스로 위임한다.
- 이는 표현식과 Spring Security ACL 시스템을 연결하여 추상적 권한을 기반으로 도메인 객체에 대한 권한 제약 조건을 지정할 수 있게 하기 위한 것이다.
    - ACL 모듈은 명시적인 종속성이 필요 없기에 다른 구현으로 교체할 수 있다.
- 인터페이스에는 두 메서드가 있다.

    ```java
    boolean hasPermission(Authentication authentication, Object targetDomainObject,
    							Object permission);
    
    boolean hasPermission(Authentication authentication, Serializable targetId,
    							String targetType, Object permission);
    ```

    - `Authentication`이 제공되지 않을 때를 제외하고 사용 가능한 표현식에 직접 매핑된다.
    - 첫 번째 버전은 현재 사용자에게 해당 객체에 대한 권한이 있는 경우 표현식은 참을 반환한다.
    - 두 번째 버전은 객체가 로드되지 않았지만 해당 식별자를 알고 있는 경우 사용된다.
- `hasPermission()` 표현식을 사용하려면 애플리케이션 컨텍스트에서 `PermissionEvaluator`를 명시적으로 구성해야 한다.
