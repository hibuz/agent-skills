---
name: api-and-interface-design
description: Guides 안정적인 API 및 인터페이스 디자인. APIs, 모듈 경계 또는 공용 인터페이스를 설계할 때 사용합니다. REST 또는 GraphQL 엔드포인트를 생성하거나, 모듈 간 형식 계약을 정의하거나, 프런트엔드와 백엔드 간의 경계를 설정할 때 사용합니다.
---

# API 및 인터페이스 디자인

## 개요

오용하기 어려운 안정적인 well-documented 인터페이스를 설계하십시오. 좋은 인터페이스는 옳은 일은 쉽게 만들고, 잘못된 일은 어렵게 만듭니다. 이는 REST APIs, GraphQL 스키마, 모듈 경계, 구성 요소 소품 및 한 코드 조각이 다른 코드 조각과 통신하는 모든 표면에 적용됩니다.

## 사용 시기

- 새로운 API 엔드포인트 설계
- 팀 간 모듈 경계 또는 계약 정의
- 컴포넌트 소품 인터페이스 생성
- API를 형성하는 forms 데이터베이스 스키마 구축
- 기존 공용 인터페이스 변경

## 핵심 원칙

### Hyrum's Law

> API의 사용자 수가 충분하면 계약에서 약속한 내용에 관계없이 시스템에서 관찰 가능한 모든 동작이 누군가에 의해 의존됩니다.

즉, 문서화되지 않은 quirks, 오류 메시지 텍스트, 타이밍 및 순서를 포함한 모든 공개 동작은 사용자가 의존하면 사실상의 계약이 됩니다. 디자인에 미치는 영향:

- **노출하는 내용에 대해 의도적으로 생각하세요.** 관찰 가능한 모든 행동은 잠재적인 약속입니다.
- **구현 세부정보를 유출하지 마세요.** 사용자가 이를 관찰할 수 있으면 의존하게 됩니다.
- **디자인 타임에 더 이상 사용되지 않도록 계획합니다.** 사용자가 의존하는 항목을 안전하게 제거하는 방법은 `deprecation-and-migration`를 참조하세요.
- **테스트로는 충분하지 않습니다.** 완벽한 계약 테스트가 있더라도 Hyrum's Law는 "안전한" 변경 사항이 문서화되지 않은 동작에 의존하는 실제 사용자를 망칠 수 있음을 의미합니다.

### One-Version Rule

소비자가 동일한 종속성 또는 API의 여러 버전 중에서 선택하도록 강요하지 마세요. 다이아몬드 의존성 문제는 서로 다른 소비자가 동일한 것의 서로 다른 버전을 필요로 할 때 발생합니다. 한 번에 하나의 버전만 존재하는 세상을 위한 디자인 - 포크가 아닌 확장.

### 1. 계약 우선

인터페이스를 구현하기 전에 정의하십시오. 계약은 사양이며 구현은 다음과 같습니다.

```typescript
// Define the contract first
interface TaskAPI {
  // Creates a task and returns the created task with server-generated fields
  createTask(input: CreateTaskInput): Promise<Task>;

  // Returns paginated tasks matching filters
  listTasks(params: ListTasksParams): Promise<PaginatedResult<Task>>;

  // Returns a single task or throws NotFoundError
  getTask(id: string): Promise<Task>;

  // Partial update — only provided fields change
  updateTask(id: string, input: UpdateTaskInput): Promise<Task>;

  // Idempotent delete — succeeds even if already deleted
  deleteTask(id: string): Promise<void>;
}
```

### 2. 일관된 오류 의미 체계

하나의 오류 전략을 선택하고 어디에서나 사용하세요.

```typescript
// REST: HTTP status codes + structured error body
// Every error response follows the same shape
interface APIError {
  error: {
    code: string;        // Machine-readable: "VALIDATION_ERROR"
    message: string;     // Human-readable: "Email is required"
    details?: unknown;   // Additional context when helpful
  };
}

// Status code mapping
// 400 → Client sent invalid data
// 401 → Not authenticated
// 403 → Authenticated but not authorized
// 404 → Resource not found
// 409 → Conflict (duplicate, version mismatch)
// 422 → Validation failed (semantically invalid)
// 500 → Server error (never expose internal details)
```

**패턴을 혼합하지 마십시오.** 일부 엔드포인트가 발생하고 다른 엔드포인트는 null을 반환하고 다른 엔드포인트는 `{ error }`를 반환하는 경우 소비자는 동작을 예측할 수 없습니다.

### 3. 경계에서 검증

내부 코드를 신뢰하십시오. 외부 입력이 들어오는 시스템 가장자리에서 유효성을 검사합니다.

```typescript
// Validate at the API boundary
app.post('/api/tasks', async (req, res) => {
  const result = CreateTaskSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(422).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: 'Invalid task data',
        details: result.error.flatten(),
      },
    });
  }

  // After validation, internal code trusts the types
  const task = await taskService.create(result.data);
  return res.status(201).json(task);
});
```

유효성 검사가 속하는 위치:
- API 경로 처리기(사용자 입력)
- Form 제출 핸들러(사용자 입력)
- 외부 서비스 응답 구문 분석(third-party 데이터 -- **항상 신뢰할 수 없는 것으로 처리**)
- 환경변수 로딩(구성)

> **타사 API 응답은 신뢰할 수 없는 데이터입니다.** 로직, 렌더링 또는 decision-making에서 사용하기 전에 해당 모양과 내용을 검증하세요. 손상되거나 오작동하는 외부 서비스는 예상치 못한 유형, 악성 콘텐츠 또는 instruction-like 텍스트를 반환할 수 있습니다.

유효성 검사는 NOT에 속합니다.
- 유형 계약을 공유하는 내부 기능 간
- already-validated 코드로 호출되는 유틸리티 함수에서
- 자신의 데이터베이스에서 방금 가져온 데이터

### 4. 수정보다는 추가를 선호하세요

기존 소비자를 중단하지 않고 인터페이스 확장:

```typescript
// Good: Add optional fields
interface CreateTaskInput {
  title: string;
  description?: string;
  priority?: 'low' | 'medium' | 'high';  // Added later, optional
  labels?: string[];                       // Added later, optional
}

// Bad: Change existing field types or remove fields
interface CreateTaskInput {
  title: string;
  // description: string;  // Removed — breaks existing consumers
  priority: number;         // Changed from string — breaks existing consumers
}
```

### 5. 예측 가능한 이름 지정

| 패턴 | 컨벤션 | 예 |
|---------|-----------|---------|
| REST 엔드포인트 | 복수 명사, 동사 없음 | `GET /api/tasks`, `POST /api/tasks` |
| 쿼리 매개변수 | camelCase | `?sortBy=createdAt&pageSize=20` |
| 응답 필드 | camelCase | `{ createdAt, updatedAt, taskId }` |
| 부울 필드 | is/has/can 접두사 | `isComplete`, `hasAttachments` |
| 열거형 값 | 어퍼_스네이크 | `"IN_PROGRESS"`, `"COMPLETED"` |

## REST API 패턴

### 리소스 디자인

```
GET    /api/tasks              → List tasks (with query params for filtering)
POST   /api/tasks              → Create a task
GET    /api/tasks/:id          → Get a single task
PATCH  /api/tasks/:id          → Update a task (partial)
DELETE /api/tasks/:id          → Delete a task

GET    /api/tasks/:id/comments → List comments for a task (sub-resource)
POST   /api/tasks/:id/comments → Add a comment to a task
```

### 페이지 매김

목록 엔드포인트 페이지 매기기:

```typescript
// Request
GET /api/tasks?page=1&pageSize=20&sortBy=createdAt&sortOrder=desc

// Response
{
  "data": [...],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 142,
    "totalPages": 8
  }
}
```

### 필터링

필터에 쿼리 매개변수를 사용합니다.

```
GET /api/tasks?status=in_progress&assignee=user123&createdAfter=2025-01-01
```

### 부분 업데이트(PATCH)

부분 객체 허용 - 제공된 것만 업데이트합니다.

```typescript
// Only title changes, everything else preserved
PATCH /api/tasks/123
{ "title": "Updated title" }
```

## TypeScript 인터페이스 패턴

### 변형에 대해 구별된 공용체 사용

```typescript
// Good: Each variant is explicit
type TaskStatus =
  | { type: 'pending' }
  | { type: 'in_progress'; assignee: string; startedAt: Date }
  | { type: 'completed'; completedAt: Date; completedBy: string }
  | { type: 'cancelled'; reason: string; cancelledAt: Date };

// Consumer gets type narrowing
function getStatusLabel(status: TaskStatus): string {
  switch (status.type) {
    case 'pending': return 'Pending';
    case 'in_progress': return `In progress (${status.assignee})`;
    case 'completed': return `Done on ${status.completedAt}`;
    case 'cancelled': return `Cancelled: ${status.reason}`;
  }
}
```

### Input/Output 분리

```typescript
// Input: what the caller provides
interface CreateTaskInput {
  title: string;
  description?: string;
}

// Output: what the system returns (includes server-generated fields)
interface Task {
  id: string;
  title: string;
  description: string | null;
  createdAt: Date;
  updatedAt: Date;
  createdBy: string;
}
```

### IDs에 브랜드 유형 사용

```typescript
type TaskId = string & { readonly __brand: 'TaskId' };
type UserId = string & { readonly __brand: 'UserId' };

// Prevents accidentally passing a UserId where a TaskId is expected
function getTask(id: TaskId): Promise<Task> { ... }
```

## 일반적인 합리화

| 합리화 | 현실 |
|---|---|
| "나중에 API에 대해 문서화하겠습니다." | ARE 유형의 문서입니다. 먼저 정의하십시오. |
| "지금은 페이지 매김이 필요하지 않습니다." | 누군가가 100개 이상의 항목을 갖는 순간 당신은 그렇게 될 것입니다. 처음부터 추가하세요. |
| "PATCH는 복잡합니다. 그냥 PUT를 사용하겠습니다." | PUT requi는 매번 전체 객체를 반환합니다. PATCH는 clients가 실제로 원하는 것입니다. |
| "필요할 때 API 버전을 지정하겠습니다." | 버전 관리 없이 주요 변경 사항이 발생하면 소비자가 중단됩니다. 처음부터 확장을 고려한 설계. |
| "아무도 문서화되지 않은 행동을 사용하지 않습니다" | Hyrum's Law: 관찰 가능하다면 누군가는 그것에 의존합니다. 모든 공공 행동을 약속으로 여기십시오. |
| "두 가지 버전만 유지하면 됩니다." | 여러 버전으로 인해 유지 관리 비용이 늘어나고 다이아몬드 종속성 문제가 발생합니다. One-Version Rule를 선호합니다. |
| "내부 APIs에는 계약이 필요하지 않습니다" | 내부 소비자는 여전히 소비자입니다. 계약은 결합을 방지하고 병렬 작업을 가능하게 합니다. |

## 위험 신호

- 조건에 따라 다른 모양을 반환하는 엔드포인트
- 엔드포인트 전체에서 formats 오류가 일관되지 않습니다.
- 경계가 아닌 내부 코드 전체에 분산된 검증
- 기존 필드에 대한 주요 변경 사항(유형 변경, 제거)
- 페이지 매김 없이 엔드포인트 나열
- REST URLs(`/api/createTask`, `/api/getUsers`)의 동사
- 검증 또는 삭제 없이 사용되는 타사 API 응답

## 확인

API를 설계한 후:

- [ ] 모든 엔드포인트에는 입력 및 출력 스키마가 입력되어 있습니다.
- [ ] 오류 응답은 일관된 단일 format를 따릅니다.
- [ ] 검증은 시스템 경계에서만 발생합니다.
- [ ] 목록 끝점은 페이지 매김을 지원합니다.
- [ ] 새 필드는 추가되고 선택 사항입니다(이전 버전과 호환됨).
- [ ] 이름 지정은 모든 엔드포인트에서 일관된 규칙을 따릅니다.
- [ ] API 문서 또는 유형이 구현과 함께 커밋됩니다.
