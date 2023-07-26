# 성능 테스트 도구 (JMeter, nGrinder)
## Apache JMeter

- 웹 애플리케이션에 초점을 둔 다양한 서비스의 성능을 분석하고 측정하기 위한 부하 테스트 도구
- 다양한 애플리케이션/서버/프로토콜 유형의 성능을 테스트할 수 있다.
    - Web - HTTP, HTTPS (Java, NodeJS, PHP, ASP.NET, …)
    - SOAP / REST Webservices
    - FTP
    - Database via JDBC
    - LDAP
    - Message-oriented middleware (MOM) via JMS
    - Mail - SMTP(S), POP3(S) and IMAP(S)
    - Native commands or shell scripts
    - TCP
    - Java Objects
- 브라어저 또는 네이티브 애플리케이션에서 빠른 테스트 계획, 기록, 빌드 및 디버깅이 가능한 모든 기능을 갖춘 테스트 IDE
- 모든 JAVA 호환 OS에서 테스트를 로드할 수 있는 CLI 모드
- Jenkins와 연동 지원
- 바로 표시 가능한 동적 HTML 보고서 제공
- 완전한 멀티 스레딩 프레임워크를 통해 여러 스레드에서 동시에 샘플링하고 별도의 스레드 그룹에서 서로 다른 기능을 동시에 샘플링할 수 있다.
- 테스트 결과 캐싱 및 오프라인 분석
- GUI, 이메일, DB, SSL 등 지원하는 기능과 플러그인이 많다.

## nGrinder

- 네이버에서 성능 측정을 목적으로 개발 된 오픈소스 프로젝트
- 스크립트 작성, 테스트 실행, 모니터링, 결과 보고서 생성까지 한 번에 실행 가능한 오픈소스 스트레스 테스트 플랫폼
- nGrinder 두 구성 요소
    - nGrinder controller: 성능 테스터가 테스트 스크립트를 생성하고 테스크 실행을 구성할 수 있는 웹 애플리케이션
    - nGrinder agent: 부하를 발생시키는 가상 사용자 생성기
    - 컨트롤러와 에이전트는 각각 다른 컴퓨팅 환경에서 실행하는 것을 추천
- 기능
    - Jython 또는 Groovy 스크립트를 사용하여 테스트 시나리오를 생성하고 여러 에이전트를 사용하여 JVM에서 스트레스를 생성
    - 프로젝트 관리, 모니터링, 결과 관리 및 보고서 관리를 위한 웹 기반 인터페이스 제공
    - IDE에서 Groovy 스크립트를 개발 및 테스트하고 분산된 에이전트에서 실행
    - 스트레스를 발생시키는 에이전트와 스트레스 대상 머신 상태를 모니터링할 수 있다.
    - 1억명 이상 사용자를 보유한 대규모 시스템을 테스트하는 데 사용되는 검증된 솔루션

---

[https://ko.wikipedia.org/wiki/아파치_제이미터](https://ko.wikipedia.org/wiki/%EC%95%84%ED%8C%8C%EC%B9%98_%EC%A0%9C%EC%9D%B4%EB%AF%B8%ED%84%B0)

https://jmeter.apache.org/

https://naver.github.io/ngrinder/
