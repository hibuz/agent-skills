---
name: security-and-hardening
description: 취약점에 대해 코드를 hardening. 사용자 입력, 인증, 데이터 저장, 또는 외부 연동을 다룰 때 사용. 신뢰할 수 없는 데이터를 받거나, 사용자 세션을 관리하거나, 서드파티 서비스와 상호작용하는 기능을 빌드할 때 사용.
---

# Security and Hardening

## Overview

웹 애플리케이션을 위한 보안 우선 개발 관행. 모든 외부 입력을 적대적으로, 모든 secret을 신성하게, 모든 인가 검사를 필수로 다루세요. 보안은 단계가 아닙니다 — 사용자 데이터, 인증, 또는 외부 시스템을 건드리는 모든 코드 줄에 대한 제약입니다.

## When to Use

- 사용자 입력을 받는 무언가 빌드
- 인증이나 인가 구현
- 민감한 데이터 저장이나 전송
- 외부 API나 서비스와 통합
- 파일 업로드, webhook, 또는 callback 추가
- 결제나 PII 데이터 처리

## 3계층 경계 시스템

### 항상 할 것 (예외 없음)

- 시스템 경계(API route, form 핸들러)에서 **모든 외부 입력 검증**
- **모든 데이터베이스 쿼리 parameterize** — 사용자 입력을 SQL에 절대 연결하지 말 것
- XSS 방지를 위해 **출력 encode** (프레임워크 auto-escaping 사용, 우회하지 말 것)
- 모든 외부 통신에 **HTTPS 사용**
- bcrypt/scrypt/argon2로 **비밀번호 해시** (평문 절대 저장 안 함)
- **security header 설정** (CSP, HSTS, X-Frame-Options, X-Content-Type-Options)
- 세션에 **httpOnly, secure, sameSite 쿠키 사용**
- 모든 릴리스 전 **`npm audit`** (또는 동등물) 실행

### 먼저 물어볼 것 (사람 승인 필요)

- 새 인증 흐름 추가나 auth 로직 변경
- 새 카테고리의 민감 데이터(PII, 결제 정보) 저장
- 새 외부 서비스 연동 추가
- CORS 구성 변경
- 파일 업로드 핸들러 추가
- rate limiting이나 throttling 수정
- 상승된 권한이나 role 부여

### 절대 하지 말 것

- version control에 **secret 절대 commit 안 함** (API 키, 비밀번호, 토큰)
- **민감 데이터 절대 log 안 함** (비밀번호, 토큰, 전체 신용카드 번호)
- **클라이언트 측 검증을 보안 경계로 절대 신뢰 안 함**
- 편의를 위해 **security header 절대 비활성화 안 함**
- 사용자 제공 데이터로 **`eval()`이나 `innerHTML` 절대 사용 안 함**
- **클라이언트 접근 가능 스토리지에 세션 절대 저장 안 함** (auth 토큰을 localStorage에)
- 사용자에게 **stack trace나 내부 에러 세부사항 절대 노출 안 함**

## OWASP Top 10 예방

### 1. Injection (SQL, NoSQL, OS Command)

```typescript
// 나쁨: 문자열 연결을 통한 SQL injection
const query = `SELECT * FROM users WHERE id = '${userId}'`;

// 좋음: parameterized 쿼리
const user = await db.query('SELECT * FROM users WHERE id = $1', [userId]);

// 좋음: parameterized 입력을 가진 ORM
const user = await prisma.user.findUnique({ where: { id: userId } });
```

### 2. Broken Authentication

```typescript
// 비밀번호 해싱
import { hash, compare } from 'bcrypt';

const SALT_ROUNDS = 12;
const hashedPassword = await hash(plaintext, SALT_ROUNDS);
const isValid = await compare(plaintext, hashedPassword);

// 세션 관리
app.use(session({
  secret: process.env.SESSION_SECRET,  // 코드가 아니라 환경에서
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,     // JavaScript로 접근 불가
    secure: true,       // HTTPS만
    sameSite: 'lax',    // CSRF 보호
    maxAge: 24 * 60 * 60 * 1000,  // 24시간
  },
}));
```

### 3. Cross-Site Scripting (XSS)

```typescript
// 나쁨: 사용자 입력을 HTML로 렌더링
element.innerHTML = userInput;

// 좋음: 프레임워크 auto-escaping 사용 (React는 기본으로 함)
return <div>{userInput}</div>;

// HTML을 반드시 렌더링해야 한다면, 먼저 sanitize
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(userInput);
```

### 4. Broken Access Control

```typescript
// 항상 인증만이 아니라 인가를 검사
app.patch('/api/tasks/:id', authenticate, async (req, res) => {
  const task = await taskService.findById(req.params.id);

  // 인증된 사용자가 이 리소스를 소유하는지 확인
  if (task.ownerId !== req.user.id) {
    return res.status(403).json({
      error: { code: 'FORBIDDEN', message: 'Not authorized to modify this task' }
    });
  }

  // 업데이트 진행
  const updated = await taskService.update(req.params.id, req.body);
  return res.json(updated);
});
```

### 5. Security Misconfiguration

```typescript
// security header (Express에 helmet 사용)
import helmet from 'helmet';
app.use(helmet());

// Content Security Policy
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'"],
    styleSrc: ["'self'", "'unsafe-inline'"],  // 가능하면 더 조이기
    imgSrc: ["'self'", 'data:', 'https:'],
    connectSrc: ["'self'"],
  },
}));

// CORS — 알려진 origin으로 제한
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || 'http://localhost:3000',
  credentials: true,
}));
```

### 6. Sensitive Data Exposure

```typescript
// API 응답에 민감 필드를 절대 반환 안 함
function sanitizeUser(user: UserRecord): PublicUser {
  const { passwordHash, resetToken, ...publicFields } = user;
  return publicFields;
}

// secret에 환경 변수 사용
const API_KEY = process.env.STRIPE_API_KEY;
if (!API_KEY) throw new Error('STRIPE_API_KEY not configured');
```

## 입력 검증 패턴

### 경계에서 Schema 검증

```typescript
import { z } from 'zod';

const CreateTaskSchema = z.object({
  title: z.string().min(1).max(200).trim(),
  description: z.string().max(2000).optional(),
  priority: z.enum(['low', 'medium', 'high']).default('medium'),
  dueDate: z.string().datetime().optional(),
});

// route 핸들러에서 검증
app.post('/api/tasks', async (req, res) => {
  const result = CreateTaskSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(422).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: 'Invalid input',
        details: result.error.flatten(),
      },
    });
  }
  // result.data는 이제 타입이 지정되고 검증됨
  const task = await taskService.create(result.data);
  return res.status(201).json(task);
});
```

### 파일 업로드 안전

```typescript
// 파일 타입과 크기 제한
const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'image/webp'];
const MAX_SIZE = 5 * 1024 * 1024; // 5MB

function validateUpload(file: UploadedFile) {
  if (!ALLOWED_TYPES.includes(file.mimetype)) {
    throw new ValidationError('File type not allowed');
  }
  if (file.size > MAX_SIZE) {
    throw new ValidationError('File too large (max 5MB)');
  }
  // 파일 확장자를 신뢰하지 말 것 — 중요하면 magic byte 확인
}
```

## npm audit 결과 Triage

모든 audit 발견이 즉각 조치를 요구하는 것은 아닙니다. 이 결정 트리를 사용:

```
npm audit가 취약점 보고
├── 심각도: critical 또는 high
│   ├── 취약 코드가 앱에서 도달 가능한가?
│   │   ├── 예 --> 즉시 수정 (의존성 업데이트, patch, 또는 교체)
│   │   └── 아니오 (dev 전용 dep, 사용 안 된 코드 경로) --> 곧 수정, 하지만 blocker 아님
│   └── 수정이 가능한가?
│       ├── 예 --> patch된 버전으로 업데이트
│       └── 아니오 --> workaround 확인, 의존성 교체 고려, 또는 리뷰 날짜와 함께 allowlist에 추가
├── 심각도: moderate
│   ├── production에서 도달 가능? --> 다음 릴리스 사이클에 수정
│   └── dev 전용? --> 편할 때 수정, backlog에 추적
└── 심각도: low
    └── 정기 의존성 업데이트 중 추적하고 수정
```

**핵심 질문:**
- 취약 함수가 실제로 당신의 코드 경로에서 호출되나?
- 의존성이 런타임 의존성인가 dev 전용인가?
- 배포 context를 고려할 때 취약점이 익스플로잇 가능한가 (예: 클라이언트 전용 앱의 서버 측 취약점)?

수정을 미룰 때, 이유를 문서화하고 리뷰 날짜를 설정하세요.

## Rate Limiting

```typescript
import rateLimit from 'express-rate-limit';

// 일반 API rate limit
app.use('/api/', rateLimit({
  windowMs: 15 * 60 * 1000, // 15분
  max: 100,                   // 윈도우당 100 요청
  standardHeaders: true,
  legacyHeaders: false,
}));

// auth endpoint에 더 엄격한 한도
app.use('/api/auth/', rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 10,  // 15분당 10회 시도
}));
```

## Secrets 관리

```
.env 파일:
  ├── .env.example  → commit됨 (placeholder 값을 가진 템플릿)
  ├── .env          → commit 안 됨 (실제 secret 포함)
  └── .env.local    → commit 안 됨 (로컬 override)

.gitignore는 포함해야 함:
  .env
  .env.local
  .env.*.local
  *.pem
  *.key
```

**commit 전 항상 확인:**
```bash
# 우발적으로 staged된 secret 확인
git diff --cached | grep -i "password\|secret\|api_key\|token"
```

## 보안 리뷰 체크리스트

```markdown
### Authentication
- [ ] Passwords hashed with bcrypt/scrypt/argon2 (salt rounds ≥ 12)
- [ ] Session tokens are httpOnly, secure, sameSite
- [ ] Login has rate limiting
- [ ] Password reset tokens expire

### Authorization
- [ ] Every endpoint checks user permissions
- [ ] Users can only access their own resources
- [ ] Admin actions require admin role verification

### Input
- [ ] All user input validated at the boundary
- [ ] SQL queries are parameterized
- [ ] HTML output is encoded/escaped

### Data
- [ ] No secrets in code or version control
- [ ] Sensitive fields excluded from API responses
- [ ] PII encrypted at rest (if applicable)

### Infrastructure
- [ ] Security headers configured (CSP, HSTS, etc.)
- [ ] CORS restricted to known origins
- [ ] Dependencies audited for vulnerabilities
- [ ] Error messages don't expose internals
```
## See Also

상세한 보안 체크리스트와 pre-commit 검증 단계는 `references/security-checklist.md`를 참고하세요.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "이건 내부 도구라 보안은 중요하지 않아" | 내부 도구도 침해됩니다. 공격자는 가장 약한 고리를 노립니다. |
| "보안은 나중에 추가할게" | 보안 retrofit은 내장보다 10배 어렵습니다. 지금 추가하세요. |
| "아무도 이걸 익스플로잇 안 할 거야" | 자동 스캐너가 찾아냅니다. obscurity에 의한 보안은 보안이 아닙니다. |
| "프레임워크가 보안을 처리해" | 프레임워크는 보장이 아니라 도구를 제공합니다. 여전히 올바르게 써야 합니다. |
| "그냥 프로토타입이야" | 프로토타입은 production이 됩니다. 첫날부터 보안 습관. |

## Red Flags

- 사용자 입력이 데이터베이스 쿼리, shell 커맨드, 또는 HTML 렌더링에 직접 전달됨
- 소스 코드나 commit 히스토리의 secret
- 인증이나 인가 검사 없는 API endpoint
- CORS 구성 누락 또는 wildcard(`*`) origin
- 인증 endpoint에 rate limiting 없음
- 사용자에게 노출된 stack trace나 내부 에러
- 알려진 critical 취약점이 있는 의존성

## Verification

보안 관련 코드 구현 후:

- [ ] `npm audit`가 critical이나 high 취약점을 안 보임
- [ ] 소스 코드나 git 히스토리에 secret 없음
- [ ] 모든 사용자 입력이 시스템 경계에서 검증됨
- [ ] 보호된 모든 endpoint에서 인증과 인가가 검사됨
- [ ] 응답에 security header 존재 (browser DevTools로 확인)
- [ ] 에러 응답이 내부 세부사항을 노출하지 않음
- [ ] auth endpoint에 rate limiting 활성
