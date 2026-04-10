---
name: security-and-hardening
description: 취약점에 대비하여 코드를 강화합니다. 사용자 입력, 인증, 데이터 저장 또는 외부 통합을 처리할 때 사용합니다. 신뢰할 수 없는 데이터를 허용하거나, 사용자 세션을 관리하거나, third-party 서비스와 상호 작용하는 기능을 building할 때 사용합니다.
---

# 보안 및 강화

## 개요

웹 애플리케이션을 위한 보안 우선 개발 관행. 모든 외부 입력을 적대적인 것으로, 모든 비밀을 신성한 것으로, 모든 인증 확인을 필수로 취급하십시오. 보안은 단계가 아닙니다. 사용자 데이터, 인증 또는 외부 시스템과 관련된 모든 코드 줄에 대한 제약입니다.

## 사용 시기

- Building 사용자 입력을 허용하는 모든 것
- 인증 또는 승인 구현
- 민감한 데이터를 저장하거나 전송하는 경우
- 외부 APIs 또는 서비스와 통합
- 파일 업로드, 웹훅 또는 콜백 추가
- 결제 또는 PII 데이터 처리

## Three-Tier 경계 시스템

### 항상 수행(예외 없음)

- 시스템 경계에서 **모든 외부 입력 유효성 검사**(API 경로, form 핸들러)
- **모든 데이터베이스 쿼리 매개변수화** — 사용자 입력을 SQL에 연결하지 마십시오.
- **XSS를 방지하기 위해 출력을 인코딩**(프레임워크 auto-escaping를 사용하고 이를 우회하지 않음)
- **모든 외부 통신에는 HTTPS**를 사용하세요.
- bcrypt/scrypt/argon2를 사용하여 **passwords** 해시(일반 텍스트를 저장하지 않음)
- **보안 헤더 설정** (CSP, HSTS, X-Frame-Options, X-Content-Type-Options)
- **세션에 httpOnly, 보안, sameSite 쿠키를 사용**
- **매 릴리스 전에 `npm audit`**(또는 equivalent)를 실행하세요.

### 먼저 물어보세요(Requires 사람 승인 요청)

- 새로운 인증 흐름 추가 또는 인증 로직 변경
- 민감한 데이터의 새로운 카테고리 저장(PII, 결제 정보)
- 새로운 외부 서비스 통합 추가
- CORS 구성 변경
- 파일 업로드 핸들러 추가
- 속도 제한 또는 조절 수정
- 승격된 권한이나 역할 부여

### 절대 하지 마세요

- 버전 관리에 **비밀을 커밋하지 마세요**(API 키, passwords, 토큰)
- **민감한 데이터는 절대 기록하지 마세요**(passwords, 토큰, 전체 신용카드 번호)
- **client-side 검증**을 보안 경계로 절대 신뢰하지 마세요.
- **편의를 위해 보안 헤더를 비활성화하지 마세요**
- **user-provided 데이터와 함께 `eval()` 또는 `innerHTML`**를 사용하지 마세요.
- **client-accessible 저장소에 세션을 저장하지 마세요**(인증 토큰의 경우 localStorage)
- **스택 추적** 또는 내부 오류 세부정보를 사용자에게 노출하지 마세요.

## OWASP 상위 10가지 예방 조치

### 1. 인젝션(SQL, NoSQL, OS 명령어)

```typescript
// BAD: SQL injection via string concatenation
const query = `SELECT * FROM users WHERE id = '${userId}'`;

// GOOD: Parameterized query
const user = await db.query('SELECT * FROM users WHERE id = $1', [userId]);

// GOOD: ORM with parameterized input
const user = await prisma.user.findUnique({ where: { id: userId } });
```

### 2. 인증 실패

```typescript
// Password hashing
import { hash, compare } from 'bcrypt';

const SALT_ROUNDS = 12;
const hashedPassword = await hash(plaintext, SALT_ROUNDS);
const isValid = await compare(plaintext, hashedPassword);

// Session management
app.use(session({
  secret: process.env.SESSION_SECRET,  // From environment, not code
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,     // Not accessible via JavaScript
    secure: true,       // HTTPS only
    sameSite: 'lax',    // CSRF protection
    maxAge: 24 * 60 * 60 * 1000,  // 24 hours
  },
}));
```

### 3. 크로스 사이트 스크립팅(XSS)

```typescript
// BAD: Rendering user input as HTML
element.innerHTML = userInput;

// GOOD: Use framework auto-escaping (React does this by default)
return <div>{userInput}</div>;

// If you MUST render HTML, sanitize first
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(userInput);
```

### 4. 손상된 액세스 제어

```typescript
// Always check authorization, not just authentication
app.patch('/api/tasks/:id', authenticate, async (req, res) => {
  const task = await taskService.findById(req.params.id);

  // Check that the authenticated user owns this resource
  if (task.ownerId !== req.user.id) {
    return res.status(403).json({
      error: { code: 'FORBIDDEN', message: 'Not authorized to modify this task' }
    });
  }

  // Proceed with update
  const updated = await taskService.update(req.params.id, req.body);
  return res.json(updated);
});
```

### 5. 잘못된 보안 구성

```typescript
// Security headers (use helmet for Express)
import helmet from 'helmet';
app.use(helmet());

// Content Security Policy
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'"],
    styleSrc: ["'self'", "'unsafe-inline'"],  // Tighten if possible
    imgSrc: ["'self'", 'data:', 'https:'],
    connectSrc: ["'self'"],
  },
}));

// CORS — restrict to known origins
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || 'http://localhost:3000',
  credentials: true,
}));
```

### 6. 민감한 데이터 노출

```typescript
// Never return sensitive fields in API responses
function sanitizeUser(user: UserRecord): PublicUser {
  const { passwordHash, resetToken, ...publicFields } = user;
  return publicFields;
}

// Use environment variables for secrets
const API_KEY = process.env.STRIPE_API_KEY;
if (!API_KEY) throw new Error('STRIPE_API_KEY not configured');
```

## 입력 유효성 검사 패턴

### 경계에서 스키마 유효성 검사

```typescript
import { z } from 'zod';

const CreateTaskSchema = z.object({
  title: z.string().min(1).max(200).trim(),
  description: z.string().max(2000).optional(),
  priority: z.enum(['low', 'medium', 'high']).default('medium'),
  dueDate: z.string().datetime().optional(),
});

// Validate at the route handler
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
  // result.data is now typed and validated
  const task = await taskService.create(result.data);
  return res.status(201).json(task);
});
```

### 파일 업로드 안전

```typescript
// Restrict file types and sizes
const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'image/webp'];
const MAX_SIZE = 5 * 1024 * 1024; // 5MB

function validateUpload(file: UploadedFile) {
  if (!ALLOWED_TYPES.includes(file.mimetype)) {
    throw new ValidationError('File type not allowed');
  }
  if (file.size > MAX_SIZE) {
    throw new ValidationError('File too large (max 5MB)');
  }
  // Don't trust the file extension — check magic bytes if critical
}
```

## npm 감사 결과 분류

모든 감사 결과에 즉각적인 조치가 필요한 것은 아닙니다. 다음 의사결정 트리를 사용하십시오.

```
npm audit reports a vulnerability
├── Severity: critical or high
│   ├── Is the vulnerable code reachable in your app?
│   │   ├── YES --> Fix immediately (update, patch, or replace the dependency)
│   │   └── NO (dev-only dep, unused code path) --> Fix soon, but not a blocker
│   └── Is a fix available?
│       ├── YES --> Update to the patched version
│       └── NO --> Check for workarounds, consider replacing the dependency, or add to allowlist with a review date
├── Severity: moderate
│   ├── Reachable in production? --> Fix in the next release cycle
│   └── Dev-only? --> Fix when convenient, track in backlog
└── Severity: low
    └── Track and fix during regular dependency updates
```

**주요 질문:**
- 취약한 함수가 실제로 코드 경로에서 호출됩니까?
- 종속성은 런타임 종속성입니까, 아니면 dev-only입니까?
- 배포 상황에 따라 취약점을 악용할 수 있습니까(예: client-only 앱의 server-side 취약점)?

수정을 연기하는 경우 이유를 문서화하고 검토 날짜를 설정하세요.

## 속도 제한

```typescript
import rateLimit from 'express-rate-limit';

// General API rate limit
app.use('/api/', rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,                   // 100 requests per window
  standardHeaders: true,
  legacyHeaders: false,
}));

// Stricter limit for auth endpoints
app.use('/api/auth/', rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 10,  // 10 attempts per 15 minutes
}));
```

## 비밀 관리

```
.env files:
  ├── .env.example  → Committed (template with placeholder values)
  ├── .env          → NOT committed (contains real secrets)
  └── .env.local    → NOT committed (local overrides)

.gitignore must include:
  .env
  .env.local
  .env.*.local
  *.pem
  *.key
```

**커밋하기 전에 항상 확인하세요.**
```bash
# Check for accidentally staged secrets
git diff --cached | grep -i "password\|secret\|api_key\|token"
```

## 보안 검토 체크리스트

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
## 참고 항목

자세한 보안 체크리스트 및 pre-commit 확인 단계는 `references/security-checklist.md`를 참조하세요.

## 일반적인 합리화

| 합리화 | 현실 |
|---|---|
| "이것은 내부 도구이므로 보안은 중요하지 않습니다." | 내부 도구가 손상되었습니다. 공격자는 가장 약한 링크를 표적으로 삼습니다. |
| "나중에 보안을 추가하겠습니다" | 보안 개조는 bui를 설치하는 것보다 10배 더 어렵습니다. 지금 추가하세요. |
| "아무도 이것을 악용하려고 시도하지 않을 것입니다" | 자동 스캐너가 이를 찾아냅니다. 모호한 보안은 보안이 아닙니다. |
| "프레임워크가 보안을 처리합니다" | 프레임워크는 보장이 아닌 도구를 제공합니다. 여전히 올바르게 사용해야 합니다. |
| "그것은 단지 프로토타입일 뿐이다" | 프로토타입이 생산이 됩니다. 처음부터 보안 습관. |

## 위험 신호

- 데이터베이스 쿼리, 셸 명령 또는 HTML 렌더링에 직접 전달된 사용자 입력
- 소스 코드 또는 커밋 기록의 비밀
- 인증 또는 승인 확인이 없는 API 엔드포인트
- CORS 구성 또는 와일드카드(`*`) 원본이 누락되었습니다.
- 인증 엔드포인트에 속도 제한이 없습니다.
- 스택 추적이나 내부 오류가 사용자에게 노출됨
- 알려진 심각한 취약점이 있는 종속성

## 확인

security-relevant 코드를 구현한 후:

- [ ] `npm audit`에는 심각하거나 높은 취약점이 표시되지 않습니다.
- [ ] 소스 코드나 Git 기록에 비밀이 없습니다.
- [ ] 시스템 경계에서 검증된 모든 사용자 입력
- [ ] 보호되는 모든 엔드포인트에서 인증 및 권한 부여 확인
- [ ] 응답에 보안 헤더가 있음(브라우저 DevTools로 확인)
- [ ] 오류 응답은 내부 세부 정보를 노출하지 않습니다.
- [ ] 인증 엔드포인트에서 속도 제한이 활성화되었습니다.
