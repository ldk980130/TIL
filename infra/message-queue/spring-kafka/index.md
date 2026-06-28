# Spring for Apache Kafka

- Spring 애플리케이션에서 카프카를 다루는 법 (공식 레퍼런스 기준 정리)
- 카프카 자체의 개념·내부 동작은 [실전 카프카 정리](../practical-kafka/index.md) 참고

## 단계별 정리

- [1. 개요와 환경 설정](01.spring-kafka-intro.md)
- [2. KafkaTemplate으로 메시지 발행](02.kafka-template-producer.md)
- [3. @KafkaListener로 메시지 소비](03.kafka-listener-consumer.md)
- [4. 직렬화 / 역직렬화](04.serialization.md)
- [5. 에러 핸들링과 재시도·DLT](05.error-handling-retry.md)
- [6. 오프셋 커밋과 AckMode](06.offset-commit-ack.md)
- [7. 컨테이너 동시성과 리밸런싱](07.concurrency-rebalance.md)
- [8. 카프카 트랜잭션과 exactly-once](08.transaction-exactly-once.md)
