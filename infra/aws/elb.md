# ELB, Auto Scaling
## Elastic Load Balancer

- ELB는 애플리케이션 트래픽을 EC2, 컨테이너, IP 주소, Lambda 함수와 같은 여러 대상에 자동으로 **분산** 시킨다.
- 단일 가용 영역 또는 여러 가용 영역에서 다양한 애플리케이션 부하를 처리할 수 있다.
- **고가용성**, **자동 확장/축소, 강력한 보안**

### ELB 특징

- **IP가 지속적으로 바뀜**
    - 지속적으로 IP 주소가 바뀜
    - 따라서 도메인 기반으로 사용해야 함
- **Health Check**
    - 직접 트래픽을 발생시켜 Instance가 살아있는지 체크
    - InService, OutofService 두 상태로 나뉨
- 3종류가 존재함
    - Application Load Balancer
        - (네트워크 계층의) 애플리케이션 레벨 (똑똑함)
    - Network Load Balancer
        - TCP/IP 레벨 (빠름)
        - Elastic IP 할당 가능
    - Classic Load Balancer
        - 옛날에 사용하던

### Sticky Session

- 여러 서버가 있을 떼 로그인 세션의 서버 간 불일치를 해결하는 방법 중 하나
    - A 서버에 세션을 저장한 유저는 계속 A 서버에만 요청하도록 하는 방식

## Auto Scaling

- 애플리케이션을 모니터링하고 용량을 자동으로 조정
- 최대한 저렴한 비용으로 안정적으로 예측 가능한 성능을 유지
- 몇 분 만에 손쉽게 애플리케이션 규모 조정을 설정할 수 있음

### EC2 Auto Scaling의 목표

- 최소한의 인스턴스
- 원하는 인스턴스 개수 목표 유지
- 최대 인스턴스 개수 이하로 유지
- AZ에 골고루 분산될 수 있도록 분배
- 항상 서비스가 유지될 수 있는 인스턴스 확보

### EC2 Auto Scaling의 구성

- **Launch Configuration**: 무엇을 어떻게 실행
    - EC2 타입, 사이즈
    - AMI
    - 보안 그룹, Key, IAM
    - User Data
        - EC2가 실행될 때 초기에 실행시키는 액션의 집합
        - 애플리케이션을 띄우기 위한 조치를 설정할 수 있음 (깃 클론, 배포 진행..)
- **Monitoring**: 언제 실행시킬 것인가? + 상태 확인
    - 예) CPU 점유율이 일정 %을 넘어섰을 때 추가로 실행 or 2개 이상이 필요한 스틱에서 EC2 하나가 죽었을 때
    - Cloud Watch나 ELB와 연계
- **Desired Capacity**: 얼만큼 실행 시킬 것인가?
    - 예) 최소 1개 ~ 최대 3개
- **Lifecycle Hook**: 인스턴스 시작/종료 시 callback
    - 다른 서비스와 연계하여 전/후처리 가능 → Cloud Watch Event, Event/SNS/SQS, Lambda
    - 기본 3600초 동안 기다림
---

[https://youtu.be/rMx0MHlgpMY](https://youtu.be/rMx0MHlgpMY)
