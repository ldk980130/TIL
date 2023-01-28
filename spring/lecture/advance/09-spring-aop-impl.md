## 기본적인 구현

```java
@Slf4j
@Aspect
public class AspectV1 {

    //hello.aop.order 패키지와 하위 패키지
    @Around("execution(* hello.aop.order..*(..))")
    public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("[log] {}", joinPoint.getSignature()); //join point 시그니처
        return joinPoint.proceed();
    }
}
```

- `@Around` 애노테이션의 값이 포인트컷이 된다.
- `doLog()` 메서드 자체가 어드바이스가 된다.

## 포인트컷 분리

```java
@Slf4j
@Aspect
public class AspectV2 {

    //hello.aop.order 패키지와 하위 패키지
    @Pointcut("execution(* hello.aop.order..*(..))")
    private void allOrder(){} //pointcut signature

    @Around("allOrder()")
    public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("[log] {}", joinPoint.getSignature()); //join point 시그니처
        return joinPoint.proceed();
    }
}
```

- `@Pointcut`에 포인트컷 표현식을 사용한다.
- 메서드 이름과 파라미터를 합쳐서 포인트컷 시그니처라 한다.
    - `allOrder()`
- 반환 타입은 `void`여야 한다.
- 코드 내용은 비워둔다.
- 이렇게 분리하면 하나의 포인트컷 표현식을 여러 어드바이스에서 함께 사용할 수 있단. 접근 제어자가 public이면 다른 클래스에서도 접근할 수 있다.

## 어드바이스 추가

```java
@Slf4j
@Aspect
public class AspectV3 {

    //hello.aop.order 패키지와 하위 패키지
    @Pointcut("execution(* hello.aop.order..*(..))")
    private void allOrder(){} //pointcut signature

    //클래스 이름 패턴이 *Service
    @Pointcut("execution(* *..*Service.*(..))")
    private void allService(){}

    @Around("allOrder()")
    public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("[log] {}", joinPoint.getSignature()); //join point 시그니처
        return joinPoint.proceed();
    }

    //hello.aop.order 패키지와 하위 패키지 이면서 클래스 이름 패턴이 *Service
    @Around("allOrder() && allService()")
    public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {
        try {
            log.info("[트랜잭션 시작] {}", joinPoint.getSignature());
            Object result = joinPoint.proceed();
            log.info("[트랜잭션 커밋] {}", joinPoint.getSignature());
            return result;
        } catch (Exception e) {
            log.info("[트랜잭션 롤백] {}", joinPoint.getSignature());
            throw e;
        } finally {
            log.info("[리소스 릴리즈] {}", joinPoint.getSignature());
        }
    }
}
```

- `@Around("allOrder() && allService()")`
    - 포인트컷은 이렇게 조합할 수 있다. (`&&`, `||`, `!`)

## 포인트컷 참조

다음과 같이 포인트컷을 공용으로 사용하기 위해 별도의 외부 클래스에 모아두어도 된다. 참고로 외부에서 호출할 때는 포인트컷의 접근 제어자를 public으로 열어두어야 한다.

```java
public class Pointcuts {

    //hello.aop.order 패키지와 하위 패키지
    @Pointcut("execution(* hello.aop.order..*(..))")
    public void allOrder(){} //pointcut signature

    //클래스 이름 패턴이 *Service
    @Pointcut("execution(* *..*Service.*(..))")
    public void allService(){}

    //allOrder && allService
    @Pointcut("allOrder() && allService()")
    public void orderAndService() {}
}
```

사용하는 방법은 패키지명을 포함한 클래스 이름과 포인트컷 시그니처를 모두 지정하면 된다.

```java
@Around("hello.aop.order.aop.Pointcuts.allOrder()")
    public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("[log] {}", joinPoint.getSignature()); //join point 시그니처
        return joinPoint.proceed();
    }

    @Around("hello.aop.order.aop.Pointcuts.orderAndService()")
    public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {

        try {
            log.info("[트랜잭션 시작] {}", joinPoint.getSignature());
            Object result = joinPoint.proceed();
            log.info("[트랜잭션 커밋] {}", joinPoint.getSignature());
            return result;
        } catch (Exception e) {
            log.info("[트랜잭션 롤백] {}", joinPoint.getSignature());
            throw e;
        } finally {
            log.info("[리소스 릴리즈] {}", joinPoint.getSignature());
        }
    }
```

## 어드바이스 순서

- `org.springframework.core.annotation.@Order`을 적용해야 한다.
- 그런데 어드바이스 단위가 아니라 클래스 단위로 적용할 수 있다.
- **따라서 애스펙트를 별도의 클래스로 분리해야 한다.**
- 숫자가 작을 수록 먼저 실행된다.

```java
@Aspect
@Order(2)
public static class LogAspect {
    @Around("hello.aop.order.aop.Pointcuts.allOrder()")
    public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("[log] {}", joinPoint.getSignature()); //join point 시그니처
        return joinPoint.proceed();
    }
}

@Aspect
@Order(1)
public static class TxAspect {
    @Around("hello.aop.order.aop.Pointcuts.orderAndService()")
    public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {

        try {
            log.info("[트랜잭션 시작] {}", joinPoint.getSignature());
            Object result = joinPoint.proceed();
            log.info("[트랜잭션 커밋] {}", joinPoint.getSignature());
            return result;
        } catch (Exception e) {
            log.info("[트랜잭션 롤백] {}", joinPoint.getSignature());
            throw e;
        } finally {
            log.info("[리소스 릴리즈] {}", joinPoint.getSignature());
        }
    }
}
```

## 어드바이스 종류

- `@Around` : 메서드 호출 전후에 수행, 가장 강력한 어드바이스, 조인 포인트 실행 여부 선택, 반환 값 변환, 예외 변환 등이 가능
- `@Before` : 조인 포인트 실행 이전에 실행
- `@AfterReturning` : 조인 포인트가 정상 완료후 실행
- `@AfterThrowing` : 메서드가 예외를 던지는 경우 실행
- `@After` : 조인 포인트가 정상 또는 예외에 관계없이 실행(finally)

```java
@Around("hello.aop.order.aop.Pointcuts.orderAndService()")
public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {

    try {
        //@Before
        log.info("[트랜잭션 시작] {}", joinPoint.getSignature());
        Object result = joinPoint.proceed();
        //@AfterReturning
        log.info("[트랜잭션 커밋] {}", joinPoint.getSignature());
        return result;
    } catch (Exception e) {
        //@AfterThrowing
        log.info("[트랜잭션 롤백] {}", joinPoint.getSignature());
        throw e;
    } finally {
        //@After
        log.info("[리소스 릴리즈] {}", joinPoint.getSignature());
    }
 }

@Before("hello.aop.order.aop.Pointcuts.orderAndService()")
public void doBefore(JoinPoint joinPoint) {
    log.info("[before] {}", joinPoint.getSignature());
}

@AfterReturning(value = "hello.aop.order.aop.Pointcuts.orderAndService()", returning = "result")
public void doReturn(JoinPoint joinPoint, Object result) {
    log.info("[return] {} return={}", joinPoint.getSignature(), result);
}

@AfterThrowing(value = "hello.aop.order.aop.Pointcuts.orderAndService()", throwing = "ex")
public void doThrowing(JoinPoint joinPoint, Exception ex) {
    log.info("[ex] {} message={}", ex);
}

@After(value = "hello.aop.order.aop.Pointcuts.orderAndService()")
public void doAfter(JoinPoint joinPoint) {
    log.info("[after] {}", joinPoint.getSignature());
}
```

- 모든 어드바이스는 `org.aspectj.lang.JoinPoint`를 첫번째 파라미터에 사용할 수 있다. (생략해도 된다.)
- 단 `@Around`는 `ProceedingJoinPoint`을 사용해야 한다.
- 참고로 `ProceedingJoinPoint`는 `org.aspectj.lang.JoinPoint`의 하위 타입이다.
- JoinPoint 주요 기능
    - `getArgs()`: 메서드 인수 반환
    - `getThis()`: 프록시 반환
    - `getTarget()`: 타겟 반환
    - `getSignature()`: 조인되는 메서드에 대한 설명 반환
    - `toString()`: 조인되는 방법에 대한 유용한 설명을 반환