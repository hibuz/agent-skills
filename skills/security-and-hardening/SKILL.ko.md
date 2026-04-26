---
name: security-and-hardening
description: 취약점으로부터 코드를 강화합니다. 사용자 입력, 인증, 데이터 저장 또는 외부 통합을 다룰 때 사용합니다. 신뢰할 수 없는 데이터를 수용하거나, 사용자 세션을 관리하거나, 서드파티 서비스와 상호작용하는 기능을 구축할 때 사용합니다.
---

# 보안 및 강화 (Security and Hardening)

## 개요 (Overview)

웹 애플리케이션을 위한 보안 우선 개발 관행입니다. 모든 외부 입력은 적대적인 것으로, 모든 비밀 정보(secret)는 신성한 것으로, 모든 인가(authorization) 체크는 필수적인 것으로 취급하세요. 보안은 개발의 한 단계가 아니라 사용자 데이터, 인증 또는 외부 시스템과 상호작용하는 모든 코드 줄에 적용되는 제약 조건입니다.

## 사용 시점

- 사용자 입력을 받는 모든 항목을 구축할 때
- 인증(authentication) 또는 인가(authorization)를 구현할 때
- 민감한 데이터를 저장하거나 전송할 때
- 외부 API 또는 서비스와 통합할 때
- 파일 업로드, 웹훅(webhooks) 또는 콜백을 추가할 때
- 결제 또는 개인정보(PII) 데이터를 다룰 때

## 3계층 경계 시스템 (The Three-Tier Boundary System)

### 항상 할 것 (예외 없음)

- **모든 외부 입력을 검증하세요.** 시스템 경계(API 라우트, 폼 핸들러)에서 수행합니다.
- **모든 데이터베이스 쿼리를 매개변수화(parameterize)하세요.** 사용자 입력을 SQL에 직접 연결하지 마세요.
- **출력을 인코딩하여 XSS를 방지하세요.** 프레임워크의 자동 이스케이프 기능을 사용하고 이를 우회하지 마세요.
- **모든 외부 통신에 HTTPS를 사용하세요.**
- **비밀번호를 해싱하세요.** bcrypt/scrypt/argon2를 사용하며 절대로 평문으로 저장하지 마세요.
- **보안 헤더를 설정하세요.** (CSP, HSTS, X-Frame-Options, X-Content-Type-Options)
- **세션에 httpOnly, secure, sameSite 쿠키를 사용하세요.**
- **모든 릴리스 전에 `npm audit`(또는 이와 동등한 도구)을 실행하세요.**

### 먼저 물어볼 것 (인간의 승인 필요)

- 새로운 인증 흐름을 추가하거나 인증 로직을 변경할 때
- 새로운 카테고리의 민감한 데이터(PII, 결제 정보 등)를 저장할 때
- 새로운 외부 서비스 통합을 추가할 때
- CORS 설정을 변경할 때
- 파일 업로드 핸들러를 추가할 때
- 속도 제한(rate limiting) 또는 스로틀링(throttling)을 수정할 때
- 승격된 권한이나 역할을 부여할 때

### 절대 하지 말 것

- **비밀 정보를 커밋하지 마세요.** 버전 관리 시스템에 API 키, 비밀번호, 토큰 등을 올리지 마세요.
- **민감한 데이터를 로그에 남기지 마세요.** 비밀번호, 토큰, 신용카드 번호 전체 등을 포함합니다.
- **클라이언트 측 검증을 보안 경계로 신뢰하지 마세요.**
- **편의를 위해 보안 헤더를 비활성화하지 마세요.**
- **사용자가 제공한 데이터에 `eval()`이나 `innerHTML`을 사용하지 마세요.**
- **클라이언트가 접근 가능한 저장소에 세션을 저장하지 마세요.** (auth 토큰을 위해 localStorage 사용 금지)
- **스택 트레이스(stack traces)나 내부 에러 세부 사항을 사용자에게 노출하지 마세요.**

## OWASP Top 10 방지

### 1. 인젝션 (SQL, NoSQL, OS Command Injection)

```typescript
// 나쁨: 문자열 연결을 통한 SQL 인젝션 위험
const query = `SELECT * FROM users WHERE id = '${userId}'`;

// 좋음: 매개변수화된 쿼리
const user = await db.query('SELECT * FROM users WHERE id = $1', [userId]);

// 좋음: 매개변수화된 입력을 사용하는 ORM
const user = await prisma.user.findUnique({ where: { id: userId } });
```

### 2. 취약한 인증 (Broken Authentication)

```typescript
// 비밀번호 해싱
import { hash, compare } from 'bcrypt';

const SALT_ROUNDS = 12;
const hashedPassword = await hash(plaintext, SALT_ROUNDS);
const isValid = await compare(plaintext, hashedPassword);

// 세션 관리
app.use(session({
  secret: process.env.SESSION_SECRET,  // 코드가 아닌 환경 변수에서 가져옴
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,     // 자바스크립트로 접근 불가
    secure: true,       // HTTPS 전용
    sameSite: 'lax',    // CSRF 방지
    maxAge: 24 * 60 * 60 * 1000,  // 24시간
  },
}));
```

### 3. 교차 사이트 스크립팅 (XSS)

```typescript
// 나쁨: 사용자 입력을 HTML로 렌더링
element.innerHTML = userInput;

// 좋음: 프레임워크의 자동 이스케이프 사용 (React는 기본적으로 이 작업을 수행함)
return <div>{userInput}</div>;

// HTML을 반드시 렌더링해야 하는 경우, 먼저 새니타이징(sanitize)하세요.
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(userInput);
```

### 4. 취약한 접근 제어 (Broken Access Control)

```typescript
// 인증뿐만 아니라 항상 인가(authorization)를 확인하세요.
app.patch('/api/tasks/:id', authenticate, async (req, res) => {
  const task = await taskService.findById(req.params.id);

  // 인증된 사용자가 해당 리소스의 소유자인지 확인
  if (task.ownerId !== req.user.id) {
    return res.status(403).json({
      error: { code: 'FORBIDDEN', message: '이 작업을 수정할 권한이 없습니다' }
    });
  }

  // 업데이트 진행
  const updated = await taskService.update(req.params.id, req.body);
  return res.json(updated);
});
```

### 5. 보안 설정 오류 (Security Misconfiguration)

```typescript
// 보안 헤더 (Express에서는 helmet 사용 권장)
import helmet from 'helmet';
app.use(helmet());

// 콘텐츠 보안 정책 (CSP)
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'"],
    styleSrc: ["'self'", "'unsafe-inline'"],  // 가능한 한 좁게 설정
    imgSrc: ["'self'", 'data:', 'https:'],
    connectSrc: ["'self'"],
  },
}));

// CORS — 알려진 오리진으로 제한
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || 'http://localhost:3000',
  credentials: true,
}));
```

### 6. 민감한 데이터 노출 (Sensitive Data Exposure)

```typescript
// API 응답에 민감한 필드를 절대 포함하지 마세요.
function sanitizeUser(user: UserRecord): PublicUser {
  const { passwordHash, resetToken, ...publicFields } = user;
  return publicFields;
}

// 비밀 정보에는 환경 변수를 사용하세요.
const API_KEY = process.env.STRIPE_API_KEY;
if (!API_KEY) throw new Error('STRIPE_API_KEY가 설정되지 않았습니다');
```

## 입력 검증 패턴 (Input Validation Patterns)

### 경계에서의 스키마 검증

```typescript
import { z } from 'zod';

const CreateTaskSchema = z.object({
  title: z.string().min(1).max(200).trim(),
  description: z.string().max(2000).optional(),
  priority: z.enum(['low', 'medium', 'high']).default('medium'),
  dueDate: z.string().datetime().optional(),
});

// 라우트 핸들러에서 검증
app.post('/api/tasks', async (req, res) => {
  const result = CreateTaskSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(422).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: '잘못된 입력입니다',
        details: result.error.flatten(),
      },
    });
  }
  // result.data는 이제 타입이 지정되고 검증된 상태입니다.
  const task = await taskService.create(result.data);
  return res.status(201).json(task);
});
```

### 파일 업로드 안전성

```typescript
// 파일 유형 및 크기 제한
const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'image/webp'];
const MAX_SIZE = 5 * 1024 * 1024; // 5MB

function validateUpload(file: UploadedFile) {
  if (!ALLOWED_TYPES.includes(file.mimetype)) {
    throw new ValidationError('허용되지 않는 파일 형식입니다');
  }
  if (file.size > MAX_SIZE) {
    throw new ValidationError('파일이 너무 큽니다 (최대 5MB)');
  }
  // 파일 확장자를 신뢰하지 마세요 — 중요한 경우 매직 바이트(magic bytes)를 확인하세요.
}
```

## npm audit 결과 분류

모든 감사 결과에 즉각적인 조치가 필요한 것은 아닙니다. 다음 의사결정 트리를 사용하세요:

```
npm audit 취약점 보고
├── 심각도: 치명적(critical) 또는 높음(high)
│   ├── 해당 취약 코드가 앱에서 도달 가능한가요?
│   │   ├── 예 --> 즉시 수정 (의존성 업데이트, 패치 또는 교체)
│   │   └── 아니요 (개발 전용 의존성, 사용되지 않는 코드 경로) --> 조만간 수정하되, 블로커는 아님
│   └── 픽스(fix)가 제공되었나요?
│       ├── 예 --> 패치된 버전으로 업데이트
│       └── 아니요 --> 우회 방법 확인, 의존성 교체 고려 또는 리뷰 날짜와 함께 허용 목록에 추가
├── 심각도: 중간(moderate)
│   ├── 프로덕션에서 도달 가능한가요? --> 다음 릴리스 사이클에서 수정
│   └── 개발 전용인가요? --> 편한 시점에 수정하고 백로그에서 관리
└── 심각도: 낮음(low)
    └── 정기적인 의존성 업데이트 시 추적 및 수정
```

**핵심 질문:**
- 취약한 함수가 실제로 당신의 코드 경로에서 호출되나요?
- 해당 의존성이 런타임 의존성인가요, 개발 전용인가요?
- 당신의 배포 컨텍스트(예: 클라이언트 전용 앱에서의 서버 측 취약점)에서 해당 취약점이 악용될 수 있나요?

수정을 미룰 때는 이유를 문서화하고 리뷰 날짜를 설정하세요.

## 속도 제한 (Rate Limiting)

```typescript
import rateLimit from 'express-rate-limit';

// 일반 API 속도 제한
app.use('/api/', rateLimit({
  windowMs: 15 * 60 * 1000, // 15분
  max: 100,                   // 창당 100개 요청
  standardHeaders: true,
  legacyHeaders: false,
}));

// 인증 엔드포인트에 대한 더 엄격한 제한
app.use('/api/auth/', rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 10,  // 15분당 10회 시도
}));
```

## 비밀 정보 관리 (Secrets Management)

```
.env 파일:
  ├── .env.example  → 커밋됨 (자리 표시자 값이 포함된 템플릿)
  ├── .env          → 커밋 안 함 (실제 비밀 정보 포함)
  └── .env.local    → 커밋 안 함 (로컬 오버라이드)

.gitignore에 반드시 포함할 것:
  .env
  .env.local
  .env.*.local
  *.pem
  *.key
```

**커밋 전 항상 확인하세요:**
```bash
# 실수로 스테이징된 비밀 정보가 있는지 확인
git diff --cached | grep -i "password\|secret\|api_key\|token"
```

## 보안 리뷰 체크리스트 (Security Review Checklist)

```markdown
### 인증 (Authentication)
- [ ] 비밀번호가 bcrypt/scrypt/argon2로 해싱됨 (salt rounds ≥ 12)
- [ ] 세션 토큰이 httpOnly, secure, sameSite로 설정됨
- [ ] 로그인에 속도 제한이 적용됨
- [ ] 비밀번호 재설정 토큰이 만료됨

### 인가 (Authorization)
- [ ] 모든 엔드포인트에서 사용자 권한을 확인함
- [ ] 사용자는 자신의 리소스에만 접근할 수 있음
- [ ] 관리자 작업에 관리자 역할 확인이 필요함

### 입력 (Input)
- [ ] 모든 사용자 입력이 경계에서 검증됨
- [ ] SQL 쿼리가 매개변수화됨
- [ ] HTML 출력이 인코딩/이스케이프됨

### 데이터 (Data)
- [ ] 코드나 버전 관리 시스템에 비밀 정보가 없음
- [ ] API 응답에서 민감한 필드가 제외됨
- [ ] PII가 미사용 시 암호화됨 (해당하는 경우)

### 인프라 (Infrastructure)
- [ ] 보안 헤더가 설정됨 (CSP, HSTS 등)
- [ ] CORS가 알려진 오리진으로 제한됨
- [ ] 의존성에 취약점이 있는지 감사함
- [ ] 에러 메시지가 내부 정보를 노출하지 않음
```
## 더 보기

상세한 보안 체크리스트 및 커밋 전 검증 단계는 `references/security-checklist.ko.md`를 참조하세요.

## 일반적인 합리화

| 합리화 | 실제 |
|---|---|
| "이것은 내부 도구라 보안이 중요하지 않아요" | 내부 도구도 침해될 수 있습니다. 공격자는 가장 약한 고리를 노립니다. |
| "보안은 나중에 추가할게요" | 보안을 사후에 보강하는 것은 구축 시 적용하는 것보다 10배 더 어렵습니다. 지금 추가하세요. |
| "아무도 이것을 악용하려 하지 않을 거예요" | 자동화된 스캐너가 찾아낼 것입니다. 모호함에 의한 보안은 보안이 아닙니다. |
| "프레임워크가 보안을 처리해 줘요" | 프레임워크는 도구를 제공할 뿐 보증을 제공하지 않습니다. 여전히 올바르게 사용해야 합니다. |
| "그냥 프로토타입일 뿐이에요" | 프로토타입은 프로덕션이 됩니다. 첫날부터 보안 습관을 들이세요. |

## 레드 플래그 (Red Flags)

- 사용자 입력이 데이터베이스 쿼리, 쉘 명령어 또는 HTML 렌더링에 직접 전달됨
- 소스 코드나 커밋 히스토리에 비밀 정보가 있음
- 인증이나 인가 체크가 없는 API 엔드포인트
- CORS 설정 누락 또는 와일드카드(`*`) 오리진
- 인증 엔드포인트에 속도 제한 없음
- 스택 트레이스나 내부 에러가 사용자에게 노출됨
- 알려진 치명적 취약점이 있는 의존성 사용

## 검증 (Verification)

보안 관련 코드를 구현한 후 다음을 확인하세요:

- [ ] `npm audit` 결과 치명적 또는 높음 취약점이 없음
- [ ] 소스 코드나 git 히스토리에 비밀 정보가 없음
- [ ] 모든 사용자 입력이 시스템 경계에서 검증됨
- [ ] 모든 보호된 엔드포인트에서 인증 및 인가가 확인됨
- [ ] 응답에 보안 헤더가 포함됨 (브라우저 DevTools로 확인)
- [ ] 에러 응답이 내부 세부 사항을 노출하지 않음
- [ ] 인증 엔드포인트에서 속도 제한이 활성화됨
