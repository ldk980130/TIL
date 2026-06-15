# 재시도 (Retry)

- 일시적(transient) 실패를 재시도로 흡수하는 패턴
- 핵심은 "**언제**(재시도 대상) · **어떻게**(백오프) · **얼마나**(한도)" 그리고 잘못 쓰면 장애를 증폭한다는 점
- 전제: **멱등성** — 재시도는 같은 연산을 반복하므로 멱등하지 않으면 중복이 발생

## 무엇을 재시도하나

- **재시도 가능(transient)**: 네트워크 타임아웃, 503(Service Unavailable), 429(Too Many Requests), 낙관적 락 충돌, 데드락, 일시적 커넥션 실패
- **재시도 불가(permanent)**: 400/422 검증 실패, 401/403 인증·인가, 비즈니스 규칙 위반 — 재시도해도 똑같이 실패
- 판단 기준: HTTP 상태코드 / 예외 타입 / `Retry-After` 헤더
- 잘못 분류하면: 영구 실패를 재시도 → 자원 낭비, 일시 실패를 포기 → 가용성 손해

## 멱등성 전제

- 재시도 = 같은 연산 반복 → 비멱등이면 중복 결제·중복 차감
- 멱등 확보: 자연 멱등 연산, 또는 idempotency key·원장 (→ `database/nosql/redis/06.stock-management.md`)
- 특히 **타임아웃 후 재시도**가 위험: 요청은 성공했는데 응답만 유실됐을 수 있어 → 멱등 키가 없으면 중복 실행

## 백오프 전략

- **고정 지연(fixed)**: 매번 같은 간격 — 단순하나 동시 실패 시 재시도가 같은 순간에 몰림
- **지수 백오프(exponential)**: `delay = base × 2^attempt` — 간격을 늘려 다운스트림 회복 시간 확보
- **지수 백오프 + 지터(jitter)**: 무작위를 섞어 여러 클라이언트의 재시도가 동시에 몰리는 것(retry storm)을 방지

```text
full jitter:         sleep = random(0, min(cap, base × 2^n))
equal jitter:        temp  = min(cap, base × 2^n);  sleep = temp/2 + random(0, temp/2)
decorrelated jitter: sleep = min(cap, random(base, prev_sleep × 3))
```

- 지터 없는 지수 백오프는 "동시에 실패 → 동일 지연 후 동시에 재시도"라 부하 스파이크가 반복됨

## 한도와 포기

- **최대 시도 횟수(max attempts)** + **최대 경과 시간(deadline)** 을 함께 둠
- 포기 시 처리: DLQ(이후 수동/배치 처리), fallback(대체 응답·캐시), 에러 표면화
- 무한 재시도 금지 — poison message가 영원히 도는 것을 막아야 함

## 재시도의 위험 — 증폭과 cascade

- 재시도는 **이미 과부하인 다운스트림에 부하를 더해** 상황을 악화시킴(retry storm)
- **계층별 재시도 곱셈**: 3개 계층이 각 3회 재시도하면 최대 27배 증폭 → **한 계층에서만** 재시도해야 함
- 완화책
    - backoff + jitter로 시점 분산
    - **circuit breaker**: 다운스트림이 지속 실패하면 빠르게 실패시켜 재시도 자체를 차단
    - **retry budget**: 재시도를 전체 요청의 일정 비율(예: 10%)로 제한 (Google SRE) — 토큰 버킷으로 구현
    - load shedding: 과부하 시 일부 요청을 거절
    - 서버가 "과부하니 재시도 말라"는 신호 제공(예: 429 + `Retry-After`)

## 타임아웃·서킷브레이커와의 조합

- 재시도에는 **시도별 timeout이 필수** — 멈춘 호출을 재시도하면 무의미. 추가로 전체 deadline으로 총 시간을 제한
- 역할 분담: **재시도 = 일시적 실패** 대응, **circuit breaker = 지속적 실패** 대응(열리면 재시도를 차단). 둘은 보완 관계

## 적용 맥락별

- **HTTP**: 429/503 + `Retry-After` 존중. GET/PUT/DELETE는 멱등이라 안전, POST는 idempotency key 필요
- **DB 동시성**: 낙관적 락 충돌·데드락은 짧은 backoff로 즉시 재시도가 효과적. 단 고경합이면 재시도 폭주 → 큐잉/비관적 락 고려
- **메시지**: at-least-once 재전송 → **멱등 컨슈머**가 전제 (outbox 재발행)
- **분산 락 획득**: 락을 못 얻으면 무작위 지연 후 재시도 (→ `database/nosql/redis/04.redlock.md`)

## 요약

| 요소 | 핵심 |
|---|---|
| 대상 | transient만 (5xx·429·락 충돌·타임아웃), permanent(4xx)는 제외 |
| 백오프 | 지수 + 지터 (retry storm 방지) |
| 한도 | max attempts + deadline, 초과 시 DLQ/fallback |
| 안전장치 | 멱등성 전제 · 한 계층에서만 · circuit breaker·retry budget로 증폭 차단 |

## 참고

- [AWS Builders' Library — Timeouts, retries, and backoff with jitter](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/)
- Google SRE Book — Addressing Cascading Failures (retry budget)
