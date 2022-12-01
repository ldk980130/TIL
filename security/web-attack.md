# 웹 공격과 방어
## SQL injection

- 코드 인젝션의 기법으로 클라이언트의 입력 값을 조작하여 서버의 DB를 공격할 수 있는 방식
- 주로 입력 데이터를 제대로 필터링 하지 못했을 경우 발생한다.

### SQL injection 종류

- **Error based SQL Injection**
    - DB에 고의적으로 오류를 발생 시켜 에러 출력을 통해 DB 구조를 파악하고 필요한 정보 습
    - 아래와 같은 로그인 쿼리가 있다고 하자.
        - `SELECT * FROM users where user_id = ? and password = ?`
    - id를 입력하는 곳에 `' OR 1 = 1 —`을 입력하면 아래와 같이 된다.
        - `SELECT * FROM users where user_id = '' OR 1 = 1 --' and password = ?`
    - 싱글 쿼터(’)를 닫기 위한 싱글 쿼터와 `OR 1 = 1`로 `WHERE` 절을 모두 참으로 만들고 `--`를 통해 뒤 구문을 모두 주석 처리 한다.
    - 결과 적으로 User 테이블 전체를 조회하게 된다.
- **Union based SQL Injection**
    - 2개 이상의 쿼리를 요청하여 결과를 얻는 UNION 연산자를 사용하는 공격으로 원래의 요청에 한 개의 추가 쿼리를 삽입하여 정보를 얻어 내는 기법이다.
    - 아래와 같은 글 제목이나 내용으로 게시글을 조회하는 쿼리가 있다.
        - `SELECT * FROM board where title like '%input' OR contents like '%input%`
    - 검색 입력 값으로 `' UNION SELECT null, id, password FROM users —`을 입력하면
        - `SELECT * FROM board where title like '%' UNION SELECT null, id, password FROM users --' OR contents like '%input%`
    - `UNION` 구문을 통해 board와 user를 합치게 되고 select를 통해 회원 정보를 빼낼 수 있다. 필요 없는 구문은 주석 처리 한다.
- **Blind SQL Injection**
    - 쿼리의 결과를 참과 거짓으로만 출력하는 페이지에서 사용하는 공격이다.

### SQL injection 방어

- **입력 값에 대한 검증**
- **Prepared Statement 구문 사용**
    - Prepared Statement 구문을 사용하면 입력 값이 DB 파라미터로 들어가기 전에 DBMS가 미리 컴파일 하여 실행하지 않고 대기한다. 그 후 입력 값을 문자열로 인식하게 하여 공격 쿼리가 들어가도 의미 없는 단순 문자열이기 때문에 공격자의 의도대로 실행되지 않는다.
    - JdbcTemplate을 사용한다면 `PreparedStatement` 객체로 파라미터를 바인딩 하기 때문에 공격에 안전하다.
    - JPA를 사용해도 Prepared Statement 구문을 사용한다.
- **Error Message 노출 금지**
    - 데이터베이스 에러 발생 시 예외 처리를 한 후 위험한 정보가 노출 되는 것을 막아야 한다.
- **웹 방화벽 사용**
    - 웹 공격 방어에 특화되어 있는 웹 방화벽을 사용하는 것도 방법이다. 소프트웨어 형, 하드웨어 형, 프록시 형의 종류가 있다.

## XSS (Cross Site Scripting)

- 공격자가 웹 페이지에 악성 스크립트를 삽입하여 공격하는 방식이다.
- 공격자가 사용자의 정보(쿠키, 세션 등)를 탈취하거나, 비정상적인 기능을 수행하게 할 수 있다.

### XSS 종류

- **Stored XSS**
    - 공격자가 제공한 데이터가 **서버에 저장된 후** 지속적으로 서비스를 제공하는 정상 페이지에서 다른 사용자에게 스크립트가 노출되는 기법
    - 예) 게시판 애플리케이션에서 공격자가 게시글로 악성 스크립트를 작성하여 저장한다. 그 후 다른 사용자가 그 게시글을 조회했을 때 스크립트가 동작하는 방식이다.
- **Reflected XSS**
    - 웹 애플리케이션의 저장된 파라미터를 사용할 때 발생하는 취약점을 이용한 공격법
        - “검색어” 같은 쿼리 스트링을 URL에 담아 전송 했을 때 서버가 필터링을 처지지 않고 쿼리에 포함된 스크립트를 응답 페이지에 담아 전송함으로써 발생
    - 스크립트가 대상 웹사이트에 있지 않고 다른 매체(타 사이트, 이메일)에 포함될 수 있다.
    - Stored XSS와 다르게 DB에 스크립트가 저장되지 않고 응답 페이지로 바로 클라이언트에 전달된다는 차이점이 있다.
- **DOM Based XSS**

### 리액트에서의 xss 방어

- 기본적으로 React Dom은 JSX에 삽입된 모든 값을 렌더링하기 전에 이스케이프 하므로 악성 스크립트가 동작하지 않는다.
- 즉 스크립트가 단순 문자열로 변환된다는 것

## CSRF(Cross Site Request Forgery)

- 사용자가 자신의 의지와 무관하게 공격자가 의도한 행동을 하여 웹 페이지를 보안에 취약하게 하거나, 수정, 삭제 등의 작업을 하게 만드는 방식이다.
- XSS는 사용자가 웹사이트를 신용하는 점을 노린 것이라면, CSRF는 웹사이트가 사용자의 브라우저를 신용하는 상태를 노린 것이다.
- CSRF 공격이 이루어지려면 다음의 조건이 만족 되어야 한다.
    - 위조 요청을 전송하는 서비스에 희생자가 로그인 상태
    - 로그인 정보(세션 id, 토큰 등)가 쿠키에 저장
    - 희생자가 해커가 만든 피싱 사이트에 접속

### CSRF 공격 과정

1. 사용자가 이용하려는 사이트에 로그인 상태
2. 로그인 이후 쿠키에 로그인 정보가 저장된다.
3. 공격자는 사용자가 악성 스크립트 페이지를 누르도록 유도한다.
    1. 악성 스크립트를 게시글로 작성하여 클릭을 유도
    2. 메일 등으로 링크를 직접 전달
4. 사용자가 악성 스크립트로 작성된 페이지에 접근 시 쿠키에 저장된 로그인 정보가 자동적으로 원 서버로 요청됨
5. 서버는 쿠키를 통해 사용자의 정상적인 요청이라 판단하고 요청을 처리
- 위와 같은 과정으로 사용자가 의도하지 않은 요청이 처리될 위험이 있다.

### CSRF 방어

- **Referer 검증**
    - HTTP Referer 헤더에는 이전 웹 페이지 주소가 들어 있다. 이를 통해 유입 경로 분석 가능
    - 사용자 요청에 Referer 헤더를 검증하면 보통의 경우 host와 Referer 값이 일치한다.
- Spring Security엔 기본적으로 CSRF 공격에 대한 방지를 수행한다.

---

[https://www.youtube.com/watch?v=laQAQeuuJF4](https://www.youtube.com/watch?v=laQAQeuuJF4)

[https://ko.wikipedia.org/wiki/사이트_간_스크립팅](https://ko.wikipedia.org/wiki/%EC%82%AC%EC%9D%B4%ED%8A%B8_%EA%B0%84_%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8C%85)

[https://ko.wikipedia.org/wiki/사이트_간_요청_위조](https://ko.wikipedia.org/wiki/%EC%82%AC%EC%9D%B4%ED%8A%B8_%EA%B0%84_%EC%9A%94%EC%B2%AD_%EC%9C%84%EC%A1%B0)

[https://www.youtube.com/watch?v=bSGqBoZd8WM](https://www.youtube.com/watch?v=bSGqBoZd8WM)

[https://junhyunny.github.io/information/security/spring-boot/spring-security/cross-site-reqeust-forgery/](https://junhyunny.github.io/information/security/spring-boot/spring-security/cross-site-reqeust-forgery/)
