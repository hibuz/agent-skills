# 보안 체크리스트

웹 애플리케이션 보안에 대한 Quick 참조입니다. `security-and-hardening` skill와 함께 사용하세요.

## 목차

- [Pre-Commit Checks](#pre-commit-checks)
- [Authentication](#authentication)
- [Authorization](#authorization)
- [Input Validation](#input-validation)
- [Security Headers](#security-headers)
- [CORS Configuration](#cors-configuration)
- [Data Protection](#data-protection)
- [Dependency Security](#dependency-security)
- [Error Handling](#error-handling)
- [OWASP Top 10 Quick Reference](#owasp-top-10-quick-reference)

## 커밋 전 확인

- [ ] 코드에 비밀이 없습니다(`git diff --cached | grep -i "password\|secret\|api_key\|token"`)
- [ ] `.gitignore` 커버: `.env`, `.env.local`, `*.pem`, `*.key`
- [ ] `.env.example`는 자리 표시자 값을 사용합니다(실제 비밀 아님).

## 인증

- [ ] bcrypt(≥12 라운드), scrypt 또는 argon2로 해시된 Passwords
- [ ] 세션 쿠키: `httpOnly`, `secure`, `sameSite: 'lax'`
- [ ] 세션 만료가 구성됨(합리적인 max-age)
- [ ] Rate limiting on login endpoint (≤10 attempts per 15 minutes)
- [ ] 비밀번호 재설정 토큰: time-limited(1시간 이하), single-use
- [ ] 반복적인 실패 후 계정 잠금(선택 사항, 알림 포함)
- [ ] MFA 민감한 작업에 지원됨(선택 사항이지만 권장됨)

## 승인

- [ ] 보호된 모든 엔드포인트는 인증을 확인합니다.
- [ ] 모든 리소스 액세스는 소유권을 확인합니다./role(IDOR 방지)
- [ ] 관리 엔드포인트 require 관리 역할 확인 필요
- [ ] API 키는 필요한 최소 권한으로 범위가 지정되었습니다.
- [ ] JWT 토큰이 검증되었습니다(서명, 만료, 발급자).

## 입력 유효성 검사

- [ ] 시스템 경계에서 검증된 모든 사용자 입력(API 경로, form 핸들러)
- [ ] 검증에서는 허용 목록(거부 목록 아님)을 사용합니다.
- [ ] 문자열 길이가 제한됨(min/max)
- [ ] 검증된 숫자 범위
- [ ] 이메일, URL 및 날짜 formats가 적절한 라이브러리로 검증됨
- [ ] 파일 업로드: rest형, 크기 제한, 콘텐츠 확인됨
- [ ] SQL 쿼리가 매개변수화됨(문자열 연결 없음)
- [ ] HTML 출력 인코딩됨(프레임워크 auto-escaping 사용)
- [ ] 리디렉션 전에 URLs가 검증되었습니다(열린 리디렉션 방지).

## 보안 헤더

```
Content-Security-Policy: default-src 'self'; script-src 'self'
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 0  (disabled, rely on CSP)
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

## CORS 구성

```typescript
// Restrictive (recommended)
cors({
  origin: ['https://yourdomain.com', 'https://app.yourdomain.com'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
})

// NEVER use in production:
cors({ origin: '*' })  // Allows any origin
```

## 데이터 보호

- [ ] API 응답에서 제외된 민감한 필드(`passwordHash`, `resetToken` 등)
- [ ] 민감한 데이터가 기록되지 않음(passwords, 토큰, 전체 CC 숫자)
- [ ] rest에서 암호화된 PII(규정에 따라 quired가 필요한 경우)
- [ ] HTTPS 모든 외부 통신용
- [ ] 데이터베이스 백업이 암호화됨

## 종속성 보안

```bash
# Audit dependencies
npm audit

# Fix automatically where possible
npm audit fix

# Check for critical vulnerabilities
npm audit --audit-level=critical

# Keep dependencies updated
npx npm-check-updates
```

## 오류 처리

```typescript
// Production: generic error, no internals
res.status(500).json({
  error: { code: 'INTERNAL_ERROR', message: 'Something went wrong' }
});

// NEVER in production:
res.status(500).json({
  error: err.message,
  stack: err.stack,         // Exposes internals
  query: err.sql,           // Exposes database details
});
```

## OWASP 상위 10개 Quick 참조

| # | 취약점 | 예방 |
|---|---|---|
| 1 | 손상된 액세스 제어 | 모든 엔드포인트에 대한 인증 확인, 소유권 확인 |
| 2 | 암호화 실패 | HTTPS, 강력한 해싱, 코드에 비밀 없음 |
| 3 | 주입 | 매개변수화된 쿼리, 입력 유효성 검사 |
| 4 | 안전하지 않은 디자인 | 위협 모델링, spec-driven 개발 |
| 5 | 보안 구성 오류 | 보안 헤더, 최소 권한, 감사 부서 |
| 6 | 취약한 구성요소 | `npm audit`, Deps 업데이트 유지, 최소 Deps |
| 7 | 인증 실패 | 강력한 비밀번호rds, 속도 제한, 세션 관리 |
| 8 | 데이터 무결성 실패 | 업데이트/dependencies, 서명된 아티팩트 확인 |
| 9 | 로깅 실패 | 보안 이벤트를 기록하고 비밀은 기록하지 않음 |
| 10 | SSRF | /allowlist URLs, restrict 아웃바운드 요청 확인 |
