# 컨텍스트 스위칭
## 컨텍스트 스위칭

- 프로세스/스레드가 실행되다가 OS가 개입하여 CPU에 할당된 프로세스/스레드를 바꾸는 것
- 여러 프로세스/스레드를 동시에 실행시키기 위해 필요하다.
- 오버헤드가 발생하기 때문에 자주 일어나면 성능이 저하
- 기존 프로세스/스레드의 상태를 저장하고 새 프로세스/스레드의 저장 상태를 로드
    - ‘상태’란 곧 ‘컨텍스트’

### 컨텍스트

- 프로세스/스레드의 상태
- CPU, 메모리 등

### 컨텍스트 스위칭 발생 시점

1. 주어진 time slice(quantum)를 다 사용
2. I/O 작업을 해야할 때
3. 다른 리소스를 기다려야 할 때
4. 등

### OS 커널(kernel)

- 컨텍스트 스위칭을 실행 시키는 주체
- 각종 리소스를 관리, 감독하는 역할

> 컴퓨터 과학에서 커널은 컴퓨터 운영 체제의 핵심이 되는 컴퓨터 프로그램으로, 시스템의 모든 것을 완전히 통제한다.
참고 - [위키](https://ko.wikipedia.org/wiki/%EC%BB%A4%EB%84%90_(%EC%BB%B4%ED%93%A8%ED%8C%85))
>

### 프로세스 간, 스레드 간 컨텍스트 스위칭 공통점

1. 커널 모드에서 실행
    1. 통제권이 커널에게 넘어감
2. CPU의 레지스터 상태를 교체
    1. 레지스터는 각종 명령어들을 실행하기 위해 필요한 데이터를 저장하고 있음

### 스레드 컨텍스트 스위칭

- T1이 작업 수행 → 커널 모드 시작 → T1의 상태 저장 → T2의 상태 로딩 → 커널 모드 종료 → T2 작업 시작

### 프로세스 컨텍스트 스위칭

- P1(T1)이 작업 수행 → 커널 모드 시작 → P1(T1)의 상태 저장 → P2(T2)의 상태 로딩 → {**메모리 관련 처리 추가**} → 커널 모드 종료 → P2(T2)작업 시작
- 가상 메모리 주소 관련 처리를 추가로 수행
    - MMU( Memory Management Unit)가 P2 메모리 주소 영역 보도록 변경
    - TLB를 완전히 비움

> **MMU(Memory Management Unit)**
런타임 중에 가상 주소를 물리적 주소를 매핑하는 하드웨어 디바이스이다. 사용자 프로그램은 논리적 주소를 다루고 물리적 주소를 신경 쓰지 않는다.
>

> **TLB(Translation Lookaside Buffer)**
가상 메모리 주소를 물리적인 주소로 변환하는 속도를 높이기 위해 사용되는 캐시
>

### 스레드 컨텍스트 스위칭이 빠른 이유

- 메모리 주소 관련 처리는 하지 않기 때문이다.

### 캐시 오염

- 컨텍스트 스위칭이 미치는 간접적인 영향
- CPU는 자주 쓰는 데이터는 메모리가 아닌 캐시에 적재한 뒤 사용한다.
- 컨텍스트 스위칭 직후엔 캐시에 가봤자 이전 프로세의 정보만 있다.
    - 스레드 간 스위칭이라면 같은 프로세스 내의 메모리를 공유해서 의미가 있을 수도 있다.
- 캐시를 활용하지 못하기에 성능이 떨어진다.

---

Operating System Concepts - Tenth Edition

[https://www.youtube.com/watch?v=Xh9Nt7y07FE](https://www.youtube.com/watch?v=Xh9Nt7y07FE)

[https://blog.naver.com/PostView.naver?blogId=kgr2626&logNo=222147205118&redirect=Dlog&widgetTypeCall=true&directAccess=false](https://blog.naver.com/PostView.naver?blogId=kgr2626&logNo=222147205118&redirect=Dlog&widgetTypeCall=true&directAccess=false)
