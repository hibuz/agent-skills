# Security Checklist

Web Application 보안을 위한 빠른 참조서입니다. `security-and-hardening` 기술과 함께 사용하세요.

## Table of Contents

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

## Pre-Commit Checks

- [ ] 코드에 Secret(비밀 정보) 없음 (`git diff --cached | grep -i "password\|secret\|api_key\|token"`)
- [ ] `.gitignore`에 다음이 포함됨: `.env`, `.env.local`, `*.pem`, `*.key`
- [ ] `.env.example`에 Placeholder 값을 사용함 (실제 Secret 아님)

## Authentication

- [ ] 비밀번호를 bcrypt (≥12 rounds), scrypt 또는 argon2로 Hashing함
- [ ] Session Cookie 설정: `httpOnly`, `secure`, `sameSite: 'lax'`
- [ ] Session 만료 설정됨 (합리적인 max-age)
- [ ] Login Endpoint에 Rate Limiting 적용 (15분당 ≤10회 시도)
- [ ] 비밀번호 재설정 Token: 시간 제한(≤1시간), 일회용
- [ ] 반복된 실패 시 계정 잠금 (선택 사항, 알림 포함)
- [ ] 민감한 작업에 MFA 지원 (선택 사항이나 권장됨)

## Authorization

- [ ] 모든 보호된 Endpoint에서 Authentication 확인
- [ ] 모든 리소스 접근 시 소유권/Role 확인 (IDOR 방지)
- [ ] Admin Endpoint에 Admin Role 확인 필요
- [ ] API Key 권한을 최소한으로 제한(Scope)
- [ ] JWT Token 검증 (Signature, Expiration, Issuer)

## Input Validation

- [ ] 모든 사용자 입력을 시스템 Boundary(API Route, Form Handler)에서 검증
- [ ] 검증 시 Allowlist 사용 (Denylist가 아님)
- [ ] 문자열 길이 제한 (Min/Max)
- [ ] 숫자 범위 검증
- [ ] 이메일, URL, 날짜 형식을 적절한 라이브러리로 검증
- [ ] 파일 업로드: Type 제한, 크기 제한, 콘텐츠 검증
- [ ] SQL Query Parameterized 처리 (문자열 연결 금지)
- [ ] HTML 출력 Encoding (Framework의 Auto-escaping 사용)
- [ ] Redirect 전 URL 검증 (Open Redirect 방지)

## Security Headers

```
Content-Security-Policy: default-src 'self'; script-src 'self'
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 0  (비활성화, CSP에 의존)
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

## CORS Configuration

```typescript
// 제한적 설정 (권장)
cors({
  origin: ['https://yourdomain.com', 'https://app.yourdomain.com'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
})

// Production에서 절대 사용 금지:
cors({ origin: '*' })  // 모든 Origin 허용
```

## Data Protection

- [ ] API Response에서 민감한 필드 제외 (`passwordHash`, `resetToken` 등)
- [ ] 민감한 데이터를 로깅하지 않음 (비밀번호, Token, 전체 신용카드 번호 등)
- [ ] PII(개인정보)를 At Rest 상태에서 암호화 (규정상 필요한 경우)
- [ ] 모든 외부 통신에 HTTPS 사용
- [ ] Database 백업 암호화

## Dependency Security

```bash
# Dependency Audit 실행
npm audit

# 가능한 경우 자동으로 수정
npm audit fix

# Critical 취약점 확인
npm audit --audit-level=critical

# Dependency 업데이트 유지
npx npm-check-updates
```

## Error Handling

```typescript
// Production: 일반적인 Error 메시지, 내부 정보 없음
res.status(500).json({
  error: { code: 'INTERNAL_ERROR', message: 'Something went wrong' }
});

// Production에서 절대 사용 금지:
res.status(500).json({
  error: err.message,
  stack: err.stack,         // 내부 정보 노출
  query: err.sql,           // Database 세부 정보 노출
});
```

## OWASP Top 10 Quick Reference

| # | Vulnerability | Prevention |
|---|---|---|
| 1 | Broken Access Control | 모든 Endpoint에서 Auth 체크, 소유권 확인 |
| 2 | Cryptographic Failures | HTTPS, 강력한 Hashing, 코드 내 Secret 금지 |
| 3 | Injection | Parameterized Query, Input Validation |
| 4 | Insecure Design | Threat Modeling, Spec-driven Development |
| 5 | Security Misconfiguration | Security Header, 최소 권한, Dependency Audit |
| 6 | Vulnerable Components | `npm audit`, Dependency 업데이트 유지, 최소한의 Dependency |
| 7 | Auth Failures | 강력한 비밀번호, Rate Limiting, Session 관리 |
| 8 | Data Integrity Failures | 업데이트/Dependency 검증, Signed Artifact |
| 9 | Logging Failures | 보안 이벤트 로깅, Secret 로깅 금지 |
| 10 | SSRF | URL 검증/Allowlist, 아웃바운드 요청 제한 |
