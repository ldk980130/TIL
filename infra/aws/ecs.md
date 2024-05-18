# Amazon Elastic Container Service
- Amazon Elastic Container Service(Amazon ECS)
    - 컨테이너 애플리케이션을 쉽게 배포, 관리, 확대할 수 있는 완전 관리형 컨테이너 오케스트레이션 서비스
    - AWS, ECR(Elastic Conatainer Registry), Docker 등 서드 파티 도구와 통합
    - 애플리케이션 구축에 더 집중 가능

## Amazon ECS 용어 및 구성 요소

- Capacity - 컨테이너가 실행되는 인프라
- Controller - 컨테이너에서 실행되는 애플리케이션을 배포, 관리
- Provisioning - 애플리케이션, 컨테이너를 배포 및 관리하는 데 사용할 수 있는 도구

### Amazon ECS Capacity

- 컨테이너가 실행되는 인프라
- Capacity 옵션
    - EC2 인스턴스
    - Fargate (서버리스)
    - 온프레미스 가상 머신(VM) 또는 서버
- Capacity는 다음 AWS 리소스 중 하나에 있을 수 있다.
    - 가용 영역
    - 로컬 영역
    - Wavelength Zone
    - AWS 리전
    - AWS Outposts

### Amazone ECS Controller

- Amazon ECS 스케줄러
    - 애플리케이션을 관리하는 소프트웨어

### Amazon ECS 프로비저닝

- 프로비저닝을 위한 여러 옵션이 존재한다.
- AWS Management Console
    - ECS 리소스에 접근 가능한 웹 인터페이스 제공
- AWS CLI
    - ECS 포함한 다양한 자원에 접근 가능한 명령을 제공
- AWS SDK
    - 언어별 API를 제공하고 많은 연결 세부 정보를 처리
- Copilot
    - ECS에서 프로덕션 지원 컨테이너 애플리케이션을 구축, 배포, 운영할 수 있는 오픈 소스 제공
- AWS CDK
    - 익숙한 언어를 사용하여 프로비저니에 사용할 수 있는 오픈 소스 소프트웨어 개발 프레임워크 제공
    - AWS CloudFormation을 통해 안전하고 반복 가능한 방식으로 리소스를 프로비저닝 한다.

## 애플리케이션 수명 주기

<img width=500 height=300 src="https://github.com/ldk980130/TIL/assets/78652144/e23e9371-f5ce-4850-8614-cd3589da75ce">

- 컨테이너에서 실행할 애플리케이션 설계가 필요하다.
    - 컨테이너는 도커 이미지를 통해 생성 (Dockerfile을 통해 구축)
    - 이러한 이미지는 빌드 후 Amazon ECR에 저장된다.
- 이미지 저장 후 Amazon ECS 작업 정의(task definition)를 생성한다.
    - 작업 정의는 애플리케이션의 파라미터 및 하나 이상의 컨테이너를 설명하는 JSON 형식의 텍스트 파일이다.
    - 이미지 및 파라미터, 사용할 컨테이너, 데이터 볼륨 등을 지정 가능
- 작업 정의 후 클러스터에 Service 또는 Task로 배포한다.
    - 클러스터 - 등록된 capacity 인프라에서 실행되는 작업 또는 서비스의 논리적 그룹

- 태스크 (Task)
    - 클러스터 내 작업 정의를 인스턴스화한 것
    - 독립 실행형 태스크를 실행하거나 Service의 일부로 태스크를 실행할 수 있다.
    - ECS의 Service로 ECS 클러스터에서 원하는 수의 태스크를 동시 실행하고 유지 가능하다.
    - 태스크가 실패하거나 중지하면 ECS 서비스 스케줄러가 태스크 정의에 따라 다른 인스턴스를 시작하는 방식으로 동작한다.
- 컨테이너 에이전트 (Container Agent)
    - ECS 클러스터 내 각 컨테이너 인스턴스에서 실행 됨
    - 실행 중인 태스크와 컨테이너의 리소스 사용률 정보를 ECS로 전송
    - ECS로부터 요청을 수신할 때마다 태스크를 시작 및 중지한다.

### 참고

- https://docs.aws.amazon.com/ko_kr/AmazonECS/latest/developerguide/Welcome.html
