# HTTP 헤더 - 일반 헤더
## HTTP 헤더

- HTTP 전송에 필요한 모든 부가 정보
- 예) 바디의 내용, 바디의 크기, 압축, 인증, 요청 클라이언트, 서버 정보, 캐시 관리 정보
- 필요 시 임의의 헤더 필드 추가 가능

## HTTP 표준

- 1999년 RFC2616 폐기
- 2014년 RFC7230~7235 등장
- 엔티티 바디라는 용어 사라짐

### RFC723X 변화

- 엔티티(entity) → 표현(Representation)
- Representation = representation Metadata + Representation Data
- 표현 = 표현 메타 데이터 + 표현 데이터

## 헤더 분류

- General 헤더: 메시지 전체에 적용되는 정보, 예) Connection: close
- Request 헤더: 요청 정보, 예) User-Agent: Mozilla/5.0 (Macintosh; ..)
- Response 헤더: 응답 정보, 예) Server: Apache
- Entity 헤더: 엔티티 바디 정보, 예) Content-Type: text/html, Content-Length: 3423
- 표현은 요청이나 응답에서 전달할 실제 데이터
- 표현 헤더는 표현 데이터를 해석할 수 있는 젇보 제공
    - 유형, 길이, 압축 정보 등

## 표현

- Content-Type: 표현 데이터의 형식
    - text/html; charset=utf-8
    - application/json
    - image/png
- Content-Encoding: 표현 데이터의 압축 방식
    - gzip
    - deflate
    - identity
- Content-Language: 표현 데이터의 자연 언어
    - 표현 데이터의 자연 언어를 표현
    - ko, en, en-US
- Content-Length: 포현 데이터의 길이
    - 바이트 단위
    - Transfer-Encoding(전송 코딩)을 사용하면 사용하면 안 됨

## 협상(콘텐츠 네고시에이셴)

**클라이언트가 선호하는 표현 요청**

- Accept: 클라이언트가 선호하는 미디어 타입 전달
- Accept-Charset: 클라이언트가 선호하는 문자 인코딩
- Accept-Encoding: 클라이언트가 선호하는 압축 인코딩
- Accept-Language: 클라이언트가 선호하는 자연 언어
- 협상 헤더는 요청 시에만 사용

### 협상과 우선순위1

**Quality Values(q)**
- Quality Values(q) 값 사용
- 0~1, 클수록 높은 우선순위
- 생략하면 1
- `Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7`
1. ko-KR;q=1 (q생략)
2. ko;q=0.9
3. en-US;q=0.8
4. en:q=0.7

### 협상과 우선순위2
**Quality Values(q)**
- 구체적인 것이 우선한다.
- `Accept: text/*, text/plain, text/plain;format=flowed, */*`
    1. text/plain;format=flowed
    2. text/plain
    3. text/*
    4. **/**

### 협상과 우선순위3

- 구체적인 것을 기준으로 미디어 타입을 맞춘다.
- `Accept: text/*;q=0.3, text/html;q=0.7, text/html;level=1,
  text/html;level=2;q=0.4, */*;q=0.5`

## 전송 방식

- 단순 전송
    - Content-Length
- 압축 전송
    - Content-Encoding
- 분할 전송
    - Transfer-Encoding
    - Content-Length를 넣으면 안 된다. 길이가 처음에 예상이 안 됨
- 범위 전송
    - Content-Range

## 일반 정보

- **From: 유저 에이전트의 이메일 정보**
    - 일반적으로 잘 사용되지 않음
    - 검색 엔진 같은 곳에서 주로 사용
    - 요청에서 사용
- **Referer: 이전 웹 페이지 주소**
    - 현재 요청된 페이지의 이전 웹 페이지 주소 (진짜 많이 씀)
    - Google 페이지에서 Naver로 이동할 때 Referer: www.google.com을 포함
    - Referer를 사용해서 유입 경로 분석 가능
    - 요청에서 사용
    - 참고: referer는 referrer의 오타
- **User-Agent: 유저 에이전트 애플리케이션 정보**
    - 클라이언트 애플리케이션 정보 (웹 브라우저 정보 등등)
    - 통계 정보로 활용되어 어떤 브라우저에서 장애가 발생하는지 파악 가능
    - 요청에서 사용
- **Server: 요청을 처리하는 오리진 서버의 소프트웨어 정보**
    - 응답에서 사용
- **Date: 메시지가 생성된 날짜**
    - 응답에서 사용

## 특별한 정보

- **Host: 요청한 호스트 정보 (도메인)**
    - 요청에서 사용, 필수
    - 하나의 서버가 여러 도메인을 처리해야 할 때
    - 하나의 IP 주소에 여러 도메인이 적용되어 있을 때

- **Location: 페이지 리다이렉션**
    - 웹 브라우저는 3XX 응답 결과에 Location 헤더가 있으면, Location 위치로 자동 이동(리다이렉트)
    - 201 (Created): Location 값은 요청에 의해 생성된 리소스 URI
    - 3XX (Redirection): Location 값은 요청을 자동으로 리다이렉션하기 위한 대상 리소스를 가리킴
- **Allow: 허용 가능한 HTTP 메서드**
    - 405 (Method Not Allowed)에서 응답에 포함해야 함
    - 서버에 URI 경로는 존재하지만 매핑되는 HTTP 메서드가 없는 경우 Allow 헤더에 허용 가능한 메서드를 담아 보낸다.
    - Allow: GET, HEAD, PUT
    - 많이 구현되어 있지는 않다.
- **Retry-After: 유저 에이전트가 다음 요청을 하기까지 기다려야 하는 시간**
    - 503 (Service Unavailable): 서비스가 언제까지 불능인지 알려줄 수 있음
    - 날짜 혹은 초 단위로까지 표기 가능

## 인증

- **Authorization: 클라이언트 인증 정보를 서버에 전달**
    - Authorization: Basic xxxxxxxxxxxxxx
    - 인증 메커니즘에는 여러가지가 있다. (Auth 등) 그래서 value에 들어가는 값이 다 달라서 공부하면서 어떤 값을 넣어야 하는지 알아야 함
- **WWW-Authenticate: 리소스 접근시 필요한 인증 방법 정의**
    - 401 Unauthorized 응답과 함께 사용
    - WWW-Authenticate: Newauth realm="apps", type=1, title="Login to \"apps\"", Basic realm="simple"

## 쿠키

- Set-Cookie: 서버에서 클라이언트로 쿠키 전달 (응답)
- Cookie: 클라이언트가 서버에서 받은 쿠키를 저장하고, HTTP 요청 시 서버로 전달
- 모든 요청에 쿠키 정보 자동 포함

- 예) 예) set-cookie: **sessionId=abcde1234**; **expires**=Sat, 26-Dec-2020 00:00:00 GMT; **path**=/; **domain**=.google.com; **Secure**
    - sessionId - 세션의 키값
    - expires - 쿠키 만료 시간
    - path - 이러한 경로에 대해서 쿠키 허용
    - domain - 이러한 도메인에 대해서 허용
    - Secure - 쿠키의 보안 정보
- 사용처
    - 사용자 로그인
    - 광고 정보 트래킹 (이 사용자가 이런 광고를 주로 보는구나)
- 쿠키 정보는 항상 서버에 전송 됨
    - 네트워크 추가 트래픽 유발
    - 최소한의 정보만 사용 (세선 Id, 인증 토큰)
    - 서버에 전송하지 않고, 웹 브라우저 내부에 데이터를 저장하고 싶으면 웹 스토리지 참고
- 주의!
    - 보안에 민감한 데이터는 저장하면 안 됨 (주민 번호, 신용카드 번호 등등)

### 쿠키 - 생명 주기

- Set-Cookie: **expires**=Sat, 26-Dec-2020 04:39:21 GMT
    - 만료되면 쿠키 삭제
- Set-Cookie: max-age=3600 (3600초)
    - 0이나 음수를 지정하면 쿠키 삭제
- 세션 쿠키: 만료 날짜를 생략하면 브라우저 종료 시까지만 유지
- 영속 쿠키: 만료 날짜를 입력하면 해당 날짜까지 유지

### 쿠키 - 도메인

- 예) domain=example.org
- **명시: 명시한 문서 기준 도메인 + 서브 도메인 포함**
    - domain=naver.com을 지정해서 쿠키 생성
    - naver.com 뿐만 아니라 blog.naver.com도 쿠키 접근
- **생략: 현재 문서 기준 도메인만 적용**
    - domain 지정을 생략하면 다른 도메인에서는 쿠키에 접근하지 못한다.

### 쿠키 - 경로

- 예) path=/home
- **이 경로를 포함한 하위 경로 페이지만 쿠키 접근**
- **일반적으로 path=/ 루트로 지정**

### 쿠키 - 보안

- Secure
    - 쿠키는 http, https를 구분하지 않고 전송
    - Secure를 적용하면 https인 경우에만 전송
- HttpOnly
    - XSS 공격 방지
    - 자바 스크립트에서 접근 불가
    - HTTP 전송에만 사용
- SameSite
    - XSRF 공격 방지
    - 요청 도메인과 쿠키에 설정된 도메인이 같은 경우에만 쿠키 전송
