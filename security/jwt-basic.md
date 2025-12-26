# JWT (JSON Web Token) 개념 정리

---

## 1. JWT 기본 구조

JWT는 `.`으로 구분된 3개의 Base64 인코딩된 부분으로 구성됩니다.

```
eyJhbGciOiJSUzI1NiIsImtpZCI6ImtleV8xMjMifQ.eyJzdWIiOiJ1c2VyXzAxSCIsImV4cCI6MTcwMDAwMDAwMH0.signature
|___________________________________|___________________________________________________|_________|
           Header                                    Payload                              Signature
```

### Header (헤더)
```json
{
  "alg": "RS256",      // 서명 알고리즘
  "kid": "key_abc123"  // Key ID - 어떤 키로 서명했는지
}
```

### Payload (페이로드)
```json
{
  "sub": "user_01H...",              // Subject - 주체 (사용자 ID 등)
  "iss": "https://auth.example.com", // Issuer - 발급자
  "aud": "my-client-id",             // Audience - 대상자
  "exp": 1700000000,                 // Expiration - 만료 시간
  "iat": 1699990000                  // Issued At - 발급 시간
}
```

### Signature (서명)
- Header + Payload를 비밀키로 서명한 값
- 위변조 검증에 사용

---

## 2. 표준 Claims (클레임)

| Claim | 이름 | 설명 | 검증 |
|-------|------|------|------|
| `iss` | Issuer | 토큰 발급자 | 권장 |
| `sub` | Subject | 토큰의 주체 (사용자 ID 등) | 용도에 따라 |
| `aud` | Audience | 토큰의 대상자 (client_id 등) | 권장 |
| `exp` | Expiration | 만료 시간 (Unix timestamp) | 필수 |
| `nbf` | Not Before | 이 시간 이전에는 유효하지 않음 | 선택 |
| `iat` | Issued At | 발급 시간 | 선택 |

### iss (Issuer) 상세

**"누가 이 토큰을 발급했는가?"**

```json
{ "iss": "https://auth.example.com" }
```

- 토큰을 발급한 인증 서버의 식별자
- 검증 시 신뢰할 수 있는 발급자인지 확인

```
[정상]
토큰의 iss: "https://auth.example.com"
서버 설정:  "https://auth.example.com" ← 일치 ✓

[공격 시도]
해커가 자체 서버에서 토큰 발급
토큰의 iss: "https://hacker.com"
서버 설정:  "https://auth.example.com" ← 불일치 ✗ 거부
```

### aud (Audience) 상세

**"이 토큰은 누구를 위한 것인가?"**

```json
{ "aud": "my-api-server" }
// 또는 여러 대상
{ "aud": ["service-a", "service-b"] }
```

- 토큰을 사용할 수 있는 서비스/클라이언트 지정
- 다른 서비스가 토큰을 오용하는 것 방지

```
[시스템 구성]
- Auth Server: 토큰 발급
- Service A: client_id = "service-a"
- Service B: client_id = "service-b"

[정상 요청]
사용자가 Service A용 토큰 발급받음
토큰의 aud: "service-a"
Service A에서 검증: aud == "service-a" ✓

[오용 시도]
같은 토큰으로 Service B 접근 시도
토큰의 aud: "service-a"
Service B에서 검증: aud != "service-b" ✗ 거부
```

### iss vs aud 비교

| | iss (Issuer) | aud (Audience) |
|---|---|---|
| 질문 | 누가 발급했나? | 누구를 위한 건가? |
| 값 예시 | `https://auth.example.com` | `my-client-id` |
| 검증 목적 | 신뢰할 수 있는 발급자인가? | 나를 위한 토큰인가? |

### 실제 흐름

```
1. 클라이언트 → Auth Server: "Service A용 토큰 주세요"

2. Auth Server → 클라이언트: 토큰 발급
   {
     "iss": "https://auth.example.com",  ← 내(Auth Server)가 발급함
     "aud": "service-a",                  ← Service A용임
     "sub": "user123",
     "exp": 1700000000
   }

3. 클라이언트 → Service A: 토큰으로 API 호출

4. Service A 검증:
   ✓ iss == "https://auth.example.com" (신뢰하는 발급자)
   ✓ aud == "service-a" (나를 위한 토큰)
   ✓ exp > 현재시간 (만료 안됨)
   → 요청 승인
```

---

## 3. kid (Key ID)

### 개념
- JWT 헤더에 포함된 **서명 키 식별자**
- 여러 개의 키 중 어떤 키로 서명했는지 알려줌

```json
// JWT Header
{
  "alg": "RS256",
  "kid": "key_abc123"  // 이 키로 서명함
}
```

### 왜 필요한가?
- 서비스는 여러 개의 키를 가질 수 있음
- 키 로테이션 시 새 키와 기존 키가 공존
- 검증 시 kid를 보고 올바른 공개키를 선택

---

## 4. JWKS (JSON Web Key Set)

### 개념
- **공개키 목록**을 JSON 형식으로 제공하는 엔드포인트
- JWT 서명 검증에 사용

### 형식
```json
{
  "keys": [
    {
      "kid": "key_abc123",
      "kty": "RSA",
      "alg": "RS256",
      "n": "0vx7agoebG...",  // RSA modulus
      "e": "AQAB"             // RSA exponent
    },
    {
      "kid": "key_def456",
      "kty": "RSA",
      "alg": "RS256",
      "n": "1b3aGoebG...",
      "e": "AQAB"
    }
  ]
}
```

### 일반적인 JWKS 엔드포인트 패턴
```
https://auth.example.com/.well-known/jwks.json
```

---

## 5. 키 로테이션 (Key Rotation)

### 개념
보안을 위해 주기적으로 서명 키를 교체하는 것

### 로테이션 과정
```
[1단계] 기존 상태
JWKS: [key_abc123]
JWT:  kid=key_abc123 → 검증 성공 ✓

[2단계] 새 키 추가 (로테이션 시작)
JWKS: [key_abc123, key_def456]  ← 새 키 추가
새 JWT: kid=key_def456 → 검증 성공 ✓
기존 JWT: kid=key_abc123 → 검증 성공 ✓

[3단계] 기존 키 제거 (로테이션 완료)
JWKS: [key_def456]  ← 기존 키 제거
```

### 클라이언트 대응
```kotlin
// kid를 못 찾으면 JWKS를 재fetch하여 재시도
findKey(kid)?.let { return it }

// 2차 시도: JWKS 재fetch 후 검색
jwkSet = refreshJwkSet()
return findKey(kid) ?: throw Exception("Key not found")
```

---

## 6. JWT 검증 흐름

```
1. JWT 파싱
   └─ Header, Payload, Signature 분리

2. Header에서 kid 추출
   └─ 어떤 키로 서명했는지 확인

3. JWKS에서 해당 kid의 공개키 조회
   └─ 없으면 JWKS 재fetch (키 로테이션 대응)

4. 서명 검증
   └─ 공개키로 Signature 검증

5. 표준 Claims 검증
   ├─ exp: 만료 여부
   ├─ nbf: 유효 시작 시간
   ├─ iss: 발급자 일치 여부
   └─ aud: 대상자 일치 여부

6. 비즈니스 로직용 Claims 추출
   └─ sub, 커스텀 클레임 등
```

---

## 7. 서명 알고리즘

### 대칭키 (Symmetric)
- **HS256, HS384, HS512**
- 서명/검증에 동일한 비밀키 사용
- 키 공유 필요 → 보안 위험

### 비대칭키 (Asymmetric) - 권장
- **RS256, RS384, RS512** (RSA)
- **ES256, ES384, ES512** (ECDSA)
- 개인키로 서명, 공개키로 검증
- 공개키는 JWKS로 배포 가능

```
발급자                           검증자
  │                               │
  │ 개인키로 서명                   │
  │ ───────────────────────────► │
  │        JWT 전달                │ 공개키로 검증
  │                               │ (JWKS에서 조회)
```

---

## 8. Access Token vs ID Token

| 구분 | Access Token | ID Token |
|------|--------------|----------|
| 목적 | API 호출 권한 | 사용자 신원 확인 |
| 대상 | Resource Server | Client Application |
| 포함 정보 | 권한(scope), 만료시간 | 사용자 프로필 |
| 유효기간 | 짧음 (5-60분) | 짧음 |

---

## 9. jjwt 라이브러리 사용 예시 (Kotlin)

```kotlin
// JwtParser 생성
val jwtParser = Jwts.parser()
    .keyLocator(jwksKeyLocator)                    // JWKS 기반 키 조회
    .requireIssuer("https://auth.example.com")     // iss 검증
    .build()

// JWT 검증 및 파싱
val claims = jwtParser
    .parseSignedClaims(jwtString)  // 서명 검증 + 파싱
    .payload                        // Claims 추출

// Claims 사용
val userId = claims.subject              // sub
val customClaim = claims["custom"] as String
```

---

## 참고 자료

- [JWT.io - JWT Debugger](https://jwt.io/)
- [RFC 7519 - JSON Web Token](https://datatracker.ietf.org/doc/html/rfc7519)
- [RFC 7517 - JSON Web Key](https://datatracker.ietf.org/doc/html/rfc7517)
- [jjwt GitHub Repository](https://github.com/jwtk/jjwt)
