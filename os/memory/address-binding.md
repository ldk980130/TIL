# Address Binding
## 논리적 주소와 물리적 주소

- 논리적주소(Logical address)
    - 프로그램 실행 중에 CPU에 의해 생성된  가상 주소
    - 프로세스에서 독립적으로 가지는 상대적인 주소 (프로세스마다 0부터 시작)
    - 프로세스는 OS에서 물리적 주소로 변한되는 논리적 주소를 사용해 메모리에 접근한다.
- 물리적 주소(Physical address)
    - 데이터가 저장되는 메모리의 실제 주소
    - 가상 주소와 달리 실제 메모리의 위치
    - 물리적 주소는 MMU(Memory Management Unit)에 의해 논리적 주소에서 변환된다.
- 논리적 주소와 물리적 주소의 공통점
    - 메모리에서 특정 위치를 식별하는 데 사용된다.
    - 이진, 16진수, 10진수 등 다양한 형식으로 표현될 수 있다.
    - 유한한 범위를 가지며 이는 사용되는 비트 수에 따라 결정된다.

## Address Binding

- Address Binding은 어떤 프로그램이 어떤 물리적 주소에 적재되는지를 결정하는 과정이다.
- 프로그램은 CPU에서 실해오디기 위해 메모리에 로드되는데 CPU는 프로그램을 프로세스로 변환하고 논리적 주소를 생성하여 메인 메모리와 통신한다.
- 논리적 주소를 사용하면 프로세스가 물리적 메모리 위치를 몰라도 메모리에 접근할 수 있는 추상화 계층을 제공하게 된다.
- 논리적 주소는 페이지 테이블을 통해 물리적 주소에 매핑된다.
    - MMU가 페이지 테이블을 사용

### MMU (Memory Management Unit)

- MMU는 CPU가 메모리에 접근하는 것을 관리하는 컴퓨터 하드웨어 부품
    - 가상 메모리를 실제 메모리 주소로 변환
- 실제 메모리 주소로의 변환을 위해 MMU는 TLB라는 고속 보조기억장치를 참조한다.
    - TLB에 원하는 데이터가 없으면 페이지 테이블에서 정보를 얻어온다.

### TLB (Translation Lookaside Buffer)

- 가상 메모리 주소를 물리 주소로 변환하는 속도를 높이기 위해 사용되는 캐시
- 최근에 발생한 가상 메모리 주소와 물리 주소 변환 테이블을 저장
- CPU와 CPU 캐시, CPU 캐시와 메인 메모리 사이 등 여러 레벨 캐시들 사이에서 주소 변환에 사용할 수 있다.
- CPU는 1차적으로 TLB에 접근하여 원하는 페이지가 있는지 탐색하고 MMU의 페이지 테이블을 참조한다.

### Address Binding 과정

![image](https://github.com/ldk980130/TIL/assets/78652144/c94d3727-8268-4183-b4cf-6360a5d746ba)

1. 소스 코드에는 심볼릭 주소가 존재
    1. 변수, 함수, 클래스 등의 식별자
2. 어셈블러(또는 컴파일러)는 심볼릭 주소를 relative numerical address에 매핑하고 object 파일을 출력하여 링커에 전달
3. 링커는 객체 파일과 기타 코드, 라이브러리 등을 가져와 실행 파일을 생성
4. 로더에 출력을 전달되고 로더는 메모리를 로드

### Address Binding 종류

주소를 binding하는 시점에 따라 분류된다.

- Compile Time
    - 프로세스의 물리적 주소가 컴파일할 때 결정된다.
    - 소스 코드의 심볼릭 주소가 바로 물리적 주소로 변환된다.
    - 컴파일러가 절대 주소(고정된 주소)를 생성하기에 만약 위치가 변경되면 재컴파일 해야 한다.
    - 프로세스 내부에서 사용하는 논리적 주소와 물리적 주소가 동일
    - 컴파일 후에는 주소가 다 정해져 버리기 때문에 다른 프로세스가 차지한 메모리 때문에 실행되지 않을 수 있다. → PC에서 돌아가는 프로세스가 하나인 경우에만 유용(임베디드 시스템)
- Load Time
    - 컴파일러가 바인딩을 수행하지 않고 심볼릭 주소를 논리적 주소로 변환한다.
    - Loader가 프로세스를 메모리에 load하는 시점에 물리적 주소를 결정
    - 프로세스 내부의 논리적 주소와 물리적 주소가 다르다.
    - 프로세스 내의 메모리를 참조하는 명령어들을 전부 바꿔야 하기에 로딩 시간이 커질 수 있다.
- Execution Time (run time)
    - 프로세스가 실행될 때 메모리 주소를 변환하는 방법
    - CPU가 주소를 참조할 때마다 address mapping table을 이용하여 binding
    - 이 때 MMU를 사용하여 변환

---

https://rebro.kr/178

https://www.geeksforgeeks.org/logical-and-physical-address-in-operating-system/

[https://ko.wikipedia.org/wiki/메모리_관리_장치](https://ko.wikipedia.org/wiki/%EB%A9%94%EB%AA%A8%EB%A6%AC_%EA%B4%80%EB%A6%AC_%EC%9E%A5%EC%B9%98)

[https://ko.wikipedia.org/wiki/변환_색인_버퍼](https://ko.wikipedia.org/wiki/%EB%B3%80%ED%99%98_%EC%83%89%EC%9D%B8_%EB%B2%84%ED%8D%BC)

https://www.baeldung.com/cs/address-binding-in-operating-systems
