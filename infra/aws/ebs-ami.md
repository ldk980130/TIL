# EBS, AMI
## Elastic Block Store

- EC2 인스턴스에 사용할 영구 블록 스토리지 볼륨을 제공한다.
    - EC2에 장착하여 사용 가능한 가상 저장 장치
- 각 EBS 볼륨은 가용 영역 내에 자동으로 복제되어 구성 요소 장애로부터 보호해주고, 고가용성 및 내구성을 제공한다.
- 워크로드 실행에 필요한 지연 시간이 짧고 일관된 성능을 제공한다.
- EBS를 사용하면 몇 분 내에 사용량을 많게 또는 적게 확장할 수 있다.
- **EBS Based** : 반 영구적인 파일의 저장 가능
    - Snapshot 기능
    - 인스턴스 업그레이드 가능
    - STOP이 가능
    - 컴퓨팅 파워와 저장소(파일)의 분리

- **Instance Storage** : 휘발성이나 빠른 방식
    - 빠르지만 저장이 필요 없는 경우
    - STOP이 불가능함

## Amazon Machine Image

- 인스턴스를 시작하는 데 필요한 정보를 제공
- 동일한 구성의 인스턴스가 여러 개 필요할 때 한 AMI에서 여러 대를 시작할 수 있다.
- AMI는 다음을 포함한다.
    - 1개 이상의 EBS 스냅샷 또는 인스턴스 저장 지원 AMI의 경우 인스턴스 루트 볼륨에 대한 탬플릿 (ex OS, 애플리케이션 서버, 애플리케이션)
    - AMI를 사용하여 인스턴스를 시작할 수 있는 AWS 계정을 제어하는 시작 권한
    - 시작될 때 인스턴스에 연결할 볼륨을 지정하는 블록 디바이스 매핑

---

[https://www.youtube.com/watch?v=ieG7hup-R8U](https://www.youtube.com/watch?v=ieG7hup-R8U)