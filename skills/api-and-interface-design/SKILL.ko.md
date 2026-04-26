---
name: api-and-interface-design
description: 안정적인 API 및 Interface 설계를 가이드합니다. API, Module 경계 또는 모든 Public Interface를 설계할 때 사용하세요. REST 또는 GraphQL Endpoint를 생성하거나, Module 간의 Type Contract를 정의하거나, Frontend와 Backend 간의 경계를 설정할 때 사용합니다.
---

# API and Interface Design

## Overview

오용하기 어려운 안정적이고 잘 문서화된 Interface를 설계하세요. 좋은 Interface는 올바른 일을 하기 쉽게 만들고, 잘못된 일을 하기 어렵게 만듭니다. 이는 REST API, GraphQL Schema, Module 경계, Component Prop 및 한 조각의 코드가 다른 조각과 통신하는 모든 접점에 적용됩니다.

## When to Use

- 새로운 API Endpoint 설계 시
- 팀 간의 Module 경계 또는 Contract 정의 시
- Component Prop Interface 생성 시
- API 형태에 영향을 주는 Database Schema 구축 시
- 기존 Public Interface 변경 시

## Core Principles

### Hyrum's Law (하이럼의 법칙)

> API 사용자가 충분히 많아지면, 계약(Contract)에서 무엇을 약속했든 관계없이 시스템의 관찰 가능한 모든 동작은 누군가에게 의존하게 됩니다.

이것은 다음을 의미합니다: 사용자가 그것에 의존하게 되는 순간, 문서화되지 않은 특징, Error 메시지 텍스트, 타이밍, 순서를 포함한 모든 공개된 동작은 사실상의 Contract가 됩니다. 설계 시 시사점:

- **노출하는 것에 신중하세요.** 모든 관찰 가능한 동작은 잠재적인 약속(Commitment)이 됩니다.
- **Implementation Detail(구현 세부 사항)을 유출하지 마세요.** 사용자가 관찰할 수 있다면, 그들은 그것에 의존할 것입니다.
- **설계 시점부터 Deprecation을 계획하세요.** 사용자가 의존하는 것을 안전하게 제거하는 방법은 `deprecation-and-migration`을 참조하세요.
- **Test만으로는 충분하지 않습니다.** 완벽한 Contract Test가 있더라도, Hyrum's Law에 따르면 "안전한" 변경이 문서화되지 않은 동작에 의존하는 실제 사용자를 망가뜨릴 수 있습니다.

### The One-Version Rule

사용자가 동일한 Dependency나 API의 여러 버전 중에서 선택하도록 강요하지 마세요. 서로 다른 사용자가 동일한 것의 다른 버전을 필요로 할 때 Diamond Dependency 문제가 발생합니다. 한 번에 하나의 버전만 존재하는 세상을 위해 설계하세요 — Fork 대신 Extend(확장)하세요.

### 1. Contract First

구현하기 전에 Interface를 먼저 정의하세요. Contract가 Spec이며, 구현은 그 뒤를 따릅니다.

```typescript
// Contract를 먼저 정의
interface TaskAPI {
  // Task를 생성하고 서버에서 생성된 필드가 포함된 생성된 Task를 반환
  createTask(input: CreateTaskInput): Promise<Task>;

  // Filter와 일치하는 Paginated Task 반환
  listTasks(params: ListTasksParams): Promise<PaginatedResult<Task>>;

  // 단일 Task 반환 또는 NotFoundError 발생
  getTask(id: string): Promise<Task>;

  // Partial Update — 제공된 필드만 변경
  updateTask(id: string, input: UpdateTaskInput): Promise<Task>;

  // Idempotent(멱등성) Delete — 이미 삭제되었더라도 성공
  deleteTask(id: string): Promise<void>;
}
```

### 2. Consistent Error Semantics

하나의 Error 전략을 선택하고 모든 곳에 사용하세요:

```typescript
// REST: HTTP Status Code + 구조화된 Error Body
// 모든 Error Response는 동일한 형태를 따름
interface APIError {
  error: {
    code: string;        // 기계 판독 가능: "VALIDATION_ERROR"
    message: string;     // 사람이 읽을 수 있음: "Email is required"
    details?: unknown;   // 도움이 되는 추가 Context
  };
}

// Status Code 매핑
// 400 → Client가 잘못된 데이터를 보냄
// 401 → 인증되지 않음
// 403 → 인증되었으나 권한이 없음
// 404 → 리소스를 찾을 수 없음
// 409 → 충돌 (중복, 버전 불일치)
// 422 → 유효성 검사 실패 (의미적으로 유효하지 않음)
// 500 → Server Error (내부 세부 정보를 절대 노출하지 말 것)
```

**패턴을 섞지 마세요.** 어떤 Endpoint는 Throw하고, 어떤 곳은 Null을 반환하고, 또 다른 곳은 `{ error }`를 반환한다면 — 사용자는 동작을 예측할 수 없습니다.

### 3. Validate at Boundaries

내부 코드를 믿으세요. 외부 입력이 들어오는 시스템의 가장자리(Boundary)에서 유효성을 검사하세요:

```typescript
// API Boundary에서 유효성 검사
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

  // 유효성 검사 후, 내부 코드는 Type을 신뢰함
  const task = await taskService.create(result.data);
  return res.status(201).json(task);
});
```

유효성 검사가 필요한 곳:
- API Route Handler (사용자 입력)
- Form Submission Handler (사용자 입력)
- 외부 서비스 Response 파싱 (제3자 데이터 -- **항상 신뢰할 수 없는 데이터로 취급**)
- Environment Variable 로딩 (Configuration)

> **제3자 API Response는 Untrusted Data입니다.** 로직, 렌더링 또는 의사 결정에 사용하기 전에 형태와 콘텐츠의 유효성을 검사하세요. 침해되었거나 오작동하는 외부 서비스는 예상치 못한 Type, 악의적인 콘텐츠 또는 명령 같은 텍스트를 반환할 수 있습니다.

유효성 검사가 필요하지 않은 곳:
- Type Contract를 공유하는 내부 함수 사이
- 이미 검증된 코드에 의해 호출되는 Utility 함수 내
- 자신의 Database에서 방금 가져온 데이터

### 4. Prefer Addition Over Modification

기존 사용자를 깨뜨리지 않고 Interface를 확장하세요:

```typescript
// Good: Optional 필드 추가
interface CreateTaskInput {
  title: string;
  description?: string;
  priority?: 'low' | 'medium' | 'high';  // 나중에 추가됨, Optional
  labels?: string[];                       // 나중에 추가됨, Optional
}

// Bad: 기존 필드의 Type을 변경하거나 필드 삭제
interface CreateTaskInput {
  title: string;
  // description: string;  // 삭제됨 — 기존 사용자를 깨뜨림
  priority: number;         // string에서 변경됨 — 기존 사용자를 깨뜨림
}
```

### 5. Predictable Naming

| Pattern | Convention | Example |
|---------|-----------|---------|
| REST endpoints | 복수 명사, 동사 없음 | `GET /api/tasks`, `POST /api/tasks` |
| Query params | camelCase | `?sortBy=createdAt&pageSize=20` |
| Response fields | camelCase | `{ createdAt, updatedAt, taskId }` |
| Boolean fields | is/has/can 접두사 | `isComplete`, `hasAttachments` |
| Enum values | UPPER_SNAKE | `"IN_PROGRESS"`, `"COMPLETED"` |

## REST API Patterns

### Resource Design

```
GET    /api/tasks              → Task 목록 (Filtering을 위한 Query Param 포함)
POST   /api/tasks              → Task 생성
GET    /api/tasks/:id          → 단일 Task 가져오기
PATCH  /api/tasks/:id          → Task 수정 (Partial)
DELETE /api/tasks/:id          → Task 삭제

GET    /api/tasks/:id/comments → Task의 Comment 목록 (Sub-resource)
POST   /api/tasks/:id/comments → Task에 Comment 추가
```

### Pagination

목록 Endpoint를 Paginate하세요:

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

### Filtering

Filter를 위해 Query Parameter를 사용하세요:

```
GET /api/tasks?status=in_progress&assignee=user123&createdAfter=2025-01-01
```

### Partial Updates (PATCH)

Partial Object를 받으세요 — 제공된 것만 업데이트합니다:

```typescript
// title만 변경되고 나머지는 유지됨
PATCH /api/tasks/123
{ "title": "Updated title" }
```

## TypeScript Interface Patterns

### 변형(Variant)을 위해 Discriminated Union 사용

```typescript
// Good: 각 변형이 명시적임
type TaskStatus =
  | { type: 'pending' }
  | { type: 'in_progress'; assignee: string; startedAt: Date }
  | { type: 'completed'; completedAt: Date; completedBy: string }
  | { type: 'cancelled'; reason: string; cancelledAt: Date };

// 사용자는 Type Narrowing을 얻음
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
// Input: 호출자가 제공하는 것
interface CreateTaskInput {
  title: string;
  description?: string;
}

// Output: 시스템이 반환하는 것 (Server에서 생성된 필드 포함)
interface Task {
  id: string;
  title: string;
  description: string | null;
  createdAt: Date;
  updatedAt: Date;
  createdBy: string;
}
```

### ID를 위해 Branded Type 사용

```typescript
type TaskId = string & { readonly __brand: 'TaskId' };
type UserId = string & { readonly __brand: 'UserId' };

// TaskId가 필요한 곳에 실수로 UserId를 넘기는 것을 방지
function getTask(id: TaskId): Promise<Task> { ... }
```

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "API 문서는 나중에 쓸게요" | Type이 곧 문서입니다. 먼저 정의하세요. |
| "지금은 Pagination이 필요 없어요" | 누군가 100개 이상의 아이템을 가지는 순간 필요하게 됩니다. 처음부터 추가하세요. |
| "PATCH는 복잡하니 그냥 PUT을 쓰죠" | PUT은 매번 전체 Object를 요구합니다. 사용자가 실제로 원하는 것은 PATCH입니다. |
| "필요할 때 API 버전을 매길게요" | 버전 관리 없는 Breaking Change는 사용자를 망가뜨립니다. 처음부터 확장을 고려해 설계하세요. |
| "그 문서화되지 않은 동작은 아무도 안 써요" | Hyrum's Law: 관찰 가능하다면 누군가는 그것에 의존합니다. 모든 공개된 동작을 약속으로 취급하세요. |
| "그냥 두 버전을 유지하면 돼요" | 여러 버전은 유지보수 비용을 배가시키고 Diamond Dependency 문제를 만듭니다. One-Version Rule을 선호하세요. |
| "내부 API는 Contract가 필요 없어요" | 내부 사용자도 여전히 사용자입니다. Contract는 Coupling(결합)을 방지하고 병렬 작업을 가능하게 합니다. |

## Red Flags

- 조건에 따라 다른 형태를 반환하는 Endpoint
- Endpoint마다 일관성 없는 Error 형식
- Boundary가 아닌 내부 코드 곳곳에 흩어져 있는 유효성 검사
- 기존 필드에 대한 Breaking Change (Type 변경, 삭제)
- Pagination이 없는 목록 Endpoint
- REST URL에 동사 사용 (`/api/createTask`, `/api/getUsers`)
- 유효성 검사나 Sanitization 없이 사용되는 제3자 API Response

## Verification

API 설계 후:

- [ ] 모든 Endpoint가 Typed Input 및 Output Schema를 가짐
- [ ] Error Response가 하나의 일관된 형식을 따름
- [ ] 유효성 검사가 시스템 Boundary에서만 발생함
- [ ] 목록 Endpoint가 Pagination을 지원함
- [ ] 새로운 필드는 추가적이며 Optional임 (하위 호환성)
- [ ] 모든 Endpoint에서 명명 규칙이 일관됨
- [ ] API 문서나 Type이 구현과 함께 Commit됨
