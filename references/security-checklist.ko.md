# Security Checklist

웹 애플리케이션 보안을 위한 빠른 참조. `security-and-hardening` skill과 함께 사용하세요.

## 목차

- [Pre-Commit 점검](#pre-commit-checks)
- [인증(Authentication)](#authentication)
- [인가(Authorization)](#authorization)
- [입력 검증](#input-validation)
- [Security Header](#security-headers)
- [CORS 구성](#cors-configuration)
- [데이터 보호](#data-protection)
- [의존성 보안](#dependency-security)
- [에러 처리](#error-handling)
- [OWASP Top 10 빠른 참조](#owasp-top-10-quick-reference)

## Pre-Commit 점검

- [ ] 코드에 secrets 없음 (`git diff --cached | grep -i "password\|secret\|api_key\|token"`)
- [ ] `.gitignore`가 커버: `.env`, `.env.local`, `*.pem`, `*.key`
- [ ] `.env.example`이 placeholder 값 사용 (실제 secrets 아님)

## 인증(Authentication)

- [ ] 비밀번호가 bcrypt(≥12 rounds), scrypt, 또는 argon2로 해시됨
- [ ] 세션 쿠키: `httpOnly`, `secure`, `sameSite: 'lax'`
- [ ] 세션 만료 구성됨 (합리적인 max-age)
- [ ] login endpoint에 rate limiting (15분당 ≤10회 시도)
- [ ] 비밀번호 재설정 토큰: 시간 제한적(≤1시간), 일회용
- [ ] 반복 실패 후 계정 잠금 (선택, 알림과 함께)
- [ ] 민감한 작업에 MFA 지원 (선택이나 권장)

## 인가(Authorization)

- [ ] 보호된 모든 endpoint가 인증 검사
- [ ] 모든 리소스 접근이 소유권/role 검사 (IDOR 방지)
- [ ] admin endpoint가 admin role 검증 요구
- [ ] API 키가 필요 최소 권한으로 scope됨
- [ ] JWT 토큰 검증됨 (signature, expiration, issuer)

## 입력 검증

- [ ] 모든 사용자 입력이 시스템 경계에서 검증됨 (API route, form 핸들러)
- [ ] 검증이 allowlist 사용 (denylist 아님)
- [ ] 문자열 길이 제약 (min/max)
- [ ] 숫자 범위 검증됨
- [ ] email, URL, 날짜 형식이 적절한 라이브러리로 검증됨
- [ ] 파일 업로드: 타입 제한, 크기 제한, 내용 검증
- [ ] SQL 쿼리 parameterize됨 (문자열 연결 없음)
- [ ] HTML 출력 encode됨 (프레임워크 auto-escaping 사용)
- [ ] redirect 전 URL 검증됨 (open redirect 방지)

## Security Header

```
Content-Security-Policy: default-src 'self'; script-src 'self'
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 0  (비활성화, CSP에 의존)
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

## CORS 구성

```typescript
// 제한적 (권장)
cors({
  origin: ['https://yourdomain.com', 'https://app.yourdomain.com'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
})

// production에서 절대 사용 금지:
cors({ origin: '*' })  // 모든 origin 허용
```

## 데이터 보호

- [ ] 민감한 필드가 API 응답에서 제외됨 (`passwordHash`, `resetToken` 등)
- [ ] 민감한 데이터를 logging 안 함 (비밀번호, 토큰, 전체 CC 번호)
- [ ] PII가 저장 시 암호화됨 (규정상 필요한 경우)
- [ ] 모든 외부 통신에 HTTPS
- [ ] 데이터베이스 백업 암호화됨

## 의존성 보안

```bash
# 의존성 감사
npm audit

# 가능한 곳 자동 수정
npm audit fix

# critical 취약점 확인
npm audit --audit-level=critical

# 의존성 최신 유지
npx npm-check-updates
```

## 에러 처리

```typescript
// production: 일반 에러, 내부 정보 없음
res.status(500).json({
  error: { code: 'INTERNAL_ERROR', message: 'Something went wrong' }
});

// production에서 절대 금지:
res.status(500).json({
  error: err.message,
  stack: err.stack,         // 내부 정보 노출
  query: err.sql,           // 데이터베이스 세부사항 노출
});
```

## OWASP Top 10 빠른 참조

| # | 취약점 | 예방 |
|---|---|---|
| 1 | Broken Access Control | 모든 endpoint에 인증 검사, 소유권 검증 |
| 2 | Cryptographic Failures | HTTPS, 강력한 해싱, 코드에 secrets 없음 |
| 3 | Injection | parameterized 쿼리, 입력 검증 |
| 4 | Insecure Design | 위협 모델링, spec 주도 개발 |
| 5 | Security Misconfiguration | security header, 최소 권한, 의존성 감사 |
| 6 | Vulnerable Components | `npm audit`, 의존성 최신 유지, 최소 의존성 |
| 7 | Auth Failures | 강력한 비밀번호, rate limiting, 세션 관리 |
| 8 | Data Integrity Failures | 업데이트/의존성 검증, 서명된 artifact |
| 9 | Logging Failures | 보안 이벤트 log, secrets는 log 안 함 |
| 10 | SSRF | URL 검증/allowlist, outbound 요청 제한 |
