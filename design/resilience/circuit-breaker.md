# 서킷 브레이커 (Circuit Breaker)

- 실패·지연이 **지속되는** 다운스트림 호출을 차단해 **빠르게 실패(fail fast)**시키는 패턴
- [재시도](retry.md)와 짝 — 재시도는 *일시적* 실패, 서킷 브레이커는 *지속적* 실패를 담당
- 목적: 죽어가는 서비스에 부하를 더하는 cascade를 막고, 호출자의 자원(스레드·커넥션)을 보호

## 왜 필요한가

- 다운스트림이 느려지거나 죽으면 호출자가 계속 호출 → 스레드·커넥션이 응답 대기로 묶임 → **호출자까지 자원 고갈 → cascading failure**
- 재시도는 이를 악화(부하 가중)
- 서킷 브레이커: 실패가 임계를 넘으면 회로를 "열어" 즉시 실패시킴 → ① 다운스트림에 회복 시간 ② 호출자 자원 보호

## 3상태 (state machine)

```
 CLOSED ──(실패율 임계 초과)──▶ OPEN ──(wait 경과)──▶ HALF_OPEN
    ▲                                                    │ │
    └────────────(시험 호출 성공)─────────────────────────┘ │
                                  (시험 호출 실패) ◀──────────┘ → OPEN
```

- **CLOSED**: 정상. 호출을 통과시키며 실패를 집계. 임계 초과 시 OPEN으로
- **OPEN**: 호출을 다운스트림에 보내지 않고 **즉시 실패**. `wait duration` 경과 후 HALF_OPEN으로
- **HALF_OPEN**: 제한된 수의 **시험 호출**만 허용 → 성공하면 CLOSED(회복), 실패하면 다시 OPEN

## 트립(개방) 조건

- **실패율 임계**: 최근 N건 중 일정 비율(예: 50%) 실패 → sliding window(count-based / time-based)로 측정
- **느린 호출 비율**: 임계 시간보다 느린 호출을 실패로 카운트(slow call) — 지연도 장애로 취급
- **최소 호출 수**: 일정 건수(`minimumNumberOfCalls`) 이상 모인 뒤에만 비율 평가 → 1~2건에 성급히 트립하는 것 방지
- **실패로 셀 예외 선택**: 4xx 같은 비즈니스 에러는 실패에서 제외(`ignoreExceptions`), 타임아웃·5xx만 카운트

## Resilience4j 주요 설정

- `failureRateThreshold` / `slowCallRateThreshold` / `slowCallDurationThreshold`
- `slidingWindowType`(COUNT_BASED / TIME_BASED), `slidingWindowSize`
- `minimumNumberOfCalls`
- `waitDurationInOpenState`
- `permittedNumberOfCallsInHalfOpenState`
- `recordExceptions` / `ignoreExceptions`

## Fallback (graceful degradation)

- OPEN이거나 호출 실패 시 대체 응답을 반환 — 캐시값·기본값·축소된 응답·나중 처리용 큐
- "전체 장애" 대신 "부분 저하"로 흡수

## 재시도와의 조합

- 역할 분담: **retry = 일시적 실패(몇 번 더 시도)**, **circuit breaker = 지속적 실패(그만 시도)**
- 합성 순서(Resilience4j 기본): **Retry가 바깥, CircuitBreaker가 안쪽** → 재시도마다 CB를 거치고, CB가 OPEN이면 남은 재시도는 즉시 실패(불필요한 재시도 차단)
- **timeout과도 필수 조합**: 멈춘 호출을 timeout으로 끊어야 CB가 그 실패를 인지

## 형제 패턴 (resilience family)

- **timeout**: 호출 시간 상한
- **bulkhead**: 자원 풀(스레드/세마포)을 격리해 한 다운스트림 장애가 전체 풀을 삼키지 않게
- **rate limiter**: 호출량 제한
- 이들을 함께 엮어 회복탄력성을 구성

## 요약

- 3상태(CLOSED/OPEN/HALF_OPEN)로 "지속 실패면 차단, 주기적으로 회복 탐색"
- 트립은 실패율·느린 호출 비율 + 최소 호출 수로 판단
- retry·timeout·bulkhead·fallback과 조합해 cascading failure를 방지

## 참고

- [Martin Fowler — CircuitBreaker](https://martinfowler.com/bliki/CircuitBreaker.html)
- [Resilience4j — CircuitBreaker](https://resilience4j.readme.io/docs/circuitbreaker)
- [Microsoft Azure — Circuit Breaker pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/circuit-breaker)
