# 로드 밸런싱
- L4/L7 스위치, HAProxy, AWS ALB

## 로드 밸런서란

- **로드 밸런싱**
    - 네트워크 트래픽을 하나 이상의 서버나 장비로 분산하기 위해 사용되는 기술.
- **로드 밸런서**
    - 로드 밸런싱을 수행하는 소프트웨어나 하드웨어

로드 밸런싱을 통해 많은 인터넷 트래픽을 여러 웹 서버로 분산시킬 수 있다.

## 로드 밸런서를 적용하는 이유

1. 하나의 서버에서 감당할 수 없는 트래픽이 발생하는 것을 n대의 서버에게 분산시켜 서버의 자원이 고갈되는 것을 방지할 수 있다.
2. WAS가 SPOF가 되는 것을 방지할 수 있다.
    1. SPOF - a part of a system that, **if it fails, will stop the entire system from working**

사실 토이 프로젝트 등에선 로드 밸런싱을 적용할 만큼의 트래픽이 나오지 않는 것이 사실이다. 하지만 WAS가 한 대 뿐이라면 그 WAS가 어떤 원인이든 죽어버리면 전체 서비스 장애로 이어진다. 이를 해결하기 위해 스케일 아웃을 해서 적어도 2대의 WAS가 있는 편이 안전하다고 판단했다.

## 로드 밸런서 종류

### L4 Load Balancer

- IP, Port 기준으로 스케줄링 알고리즘을 통해 부하를 분산한다.
- TCP/IP 프로토콜을 사용하며 주로 Round Robin 방식을 이용한다.
- 데이터 안을 보지 않고 패킷 레벨에서만 정보를 얻기 때문에 빠르고 효율적이다 (저렴하다)
- 하지만 세세한 라우팅이 불가능하고 사용자의 IP가 수시로 바뀌는 경우 연속적인 서비스가 어렵다.

### L7 Load Balancer

- IP, Port 외에도 URI, Payload, Http Header, Cookie 등의 내용(애플리케이션 영역)을 기준으로 부하를 분산한다. (콘텐츠 기반 스위칭)
- 애플리케이션 계층에서 로드를 분산하기 때문에 상세한 라우팅이 가능하다
- 캐싱 기능 제공
- 비정상적인 트래픽을 사전에 필터링할 수 있어서 서비스 안전성이 높다.
- 분석해야 하는 정보가 많아 L4보다 작업이 어렵고 무거워진다.

### [HAProxy](https://ko.wikipedia.org/wiki/HAProxy)

- TCP, HTTP 기반 애플리케이션을 위한 고가용성 로드 밸런서와 리버스 프록시를 제공하는 자유-오픈 소프트웨어.
- 빠르고 효율적인 것으로 유명하다

### [ALB(Application Load Balancer)](https://docs.aws.amazon.com/ko_kr/elasticloadbalancing/latest/application/introduction.html#application-load-balancer-overview)

- AWS에 정의되어 있는 로드 밸런서
- 애플리케이션 영역에서 동작
- 기본 알고리즘은 라운드 로빈
- HTTP 헤더와 메서드 등으로 부하를 분산할 수 있다.

> **NGINX로 로드 밸런싱?**
Nginx를 로드 밸런서로 사용하는 이유는 간단하고 저렴하기 때문이다. L4, L7 스위치 등은 하드웨어로 구성해야하며 가격이 비싸다. 반면 Nginx를 사용하면, 서버에 소프트웨어만 설치하면 되므로 시간과 비용이 절약된다.
[https://hudi.blog/load-balancing-with-nginx/](https://hudi.blog/load-balancing-with-nginx/)
>

## NGINX 로드 밸런싱 설정

```bash
http {
    upstream backends {
        server backend1.example.com;
        server backend2.example.com;
        server 192.0.0.1 backup;
    }
    
    server {
        location / {
            proxy_pass http://backends;
        }
    }
}
```

- **upstream** - 서버 설정에서 NGINX가 받은 요청을 어떤 서버로 흘려 보낼 것인지 결정
    - 로드 밸런싱할 서버 인스턴스 그룹들의 ip 또는 도메인 이름을 block으로 묶음
- 위 설정대로면 `/`로 들어온 요청이 `proxy_pass`로 연결되는 데 이 때 `backends`라는 `upstream block` 안의 주소들에게 분배될 것이다.

## 로드 밸런싱 알고리즘

### 라운드 로빈

```bash
upstream backend {
   # no load balancing method is specified for Round Robin
   server backend1.example.com;
   server backend2.example.com;
}
```

- 서버에 들어온 요청을 순서대로 돌아가며 배정하는 방식
- 순서대로 분배하기 때문에 여러 서버가 동일한 스펙이면서 서버와의 연결이 오래 지속되지 않는 경우 적절

### Least Connections (가중 라운드 로빈)

```bash
upstream backend {
    least_conn;
    server backend1.example.com;
    server backend2.example.com;
}
```

- 서버 가중치를 고려하여 활성 연결 수가 가장 적은 서버로 요청을 분배

### IP Hash

```bash
upstream backend {
    ip_hash;
    server backend1.example.com;
    server backend2.example.com;
}
```

- 요청의 IP 주소를 특정 서버로 매핑하는 방식
- 사용자가 항상 동일한 서버로 연결되는 것을 보장함
- 만약 한 서버를 로드 밸런시엥서 제외하고 싶다면 down 추가

```bash
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com down;
}
```

### Generic Hash

요청이 들어오는 서버가 사용자 정의 키에 의해 결정된다. 사용자 정의 키에는 문자열, 변수 등의 조합이 될 수도 있다.

```bash
upstream backend {
    hash $request_uri consistent;
    server backend1.example.com;
    server backend2.example.com;
}
```

### Least Time (NGINX Plus only)

각 요청에 대해 NGINX Plus가 최소 평균 지연 시간과 최소 활성 커넥션을 가진 서버를 선택한다.

```bash
upstream backend {
    least_time header;
    server backend1.example.com;
    server backend2.example.com;
}
```

최소 지연 시간은 다음 세 변수에 의해 계산이 달라진다.

- `header` - 서버로부터 첫 번째 바이트를 수신하는 시간
- `last_byte` -  서버로부터 전체 응답을 수신하는 시간
- `last_byte inflight` - 불완전한 요청을 고려하여 서버로부터 전체 응답을 수신하는 시간

### Random

각 요청이 무작위로 서버로 전송된다.

`two` 매개변수가 정의된 경우, NGINX는 서버 가중치를 고려하여 무작위로 2개의 서버를 선택한 뒤 아래 방법을 사용하여 서버 하나를 선택한다.

- `least_conn` - 최소 활성 커넥션 수
- `least_time=header`(NGINX Plus) - 서버로부터 응답 헤더를 수신하는 최소 평균 시간
- `least_time=last_byte`(NGINX Plus) - 서버로부터 전체 응답을 수신하기 위한 최소 평균 시간

```bash
upstream backend {
    random two least_time=last_byte;
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com;
    server backend4.example.com;
}
```

## ec2-smody-nginx default.config

```bash
upstream backends { # 트래픽을 분산시킬 IP와 Port 정보
        server 192.168.2.xxx:8080;
        server 192.168.2.yyy:8080;
}

server {
  listen 443 ssl;
  server_name admin.smody.co.kr;

  ssl_certificate /etc/letsencrypt/live/admin.smody.co.kr/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/admin.smody.co.kr/privkey.pem;
  include /etc/letsencrypt/options-ssl-nginx.conf;
  ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

  location / {
    proxy_pass http://backends; # upstream 블럭과 매핑
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```

- 설정하고 `sudo nginx -s reload` 실행

### 참고

아마존 웹 서비스(AWS Discovery Book)

[https://jiwondev.tistory.com/189](https://jiwondev.tistory.com/189)

[https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=innolifes&logNo=222078920240](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=innolifes&logNo=222078920240)

[https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)

[https://hyeon9mak.github.io/nginx-upstream-multi-server/](https://hyeon9mak.github.io/nginx-upstream-multi-server/)
