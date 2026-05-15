---
name: api-and-interface-design
description: 안정적인 API 및 인터페이스 설계를 안내. API, 모듈 경계, 또는 모든 공개 인터페이스를 설계할 때 사용. REST 또는 GraphQL endpoint 생성, 모듈 간 타입 contract 정의, 또는 프론트엔드와 백엔드 사이 경계 수립 시 사용.
---

# API and Interface Design

## Overview

오용하기 어려운, 안정적이고 잘 문서화된 인터페이스를 설계하세요. 좋은 인터페이스는 옳은 것을 쉽게, 그른 것을 어렵게 만듭니다. 이는 REST API, GraphQL 스키마, 모듈 경계, 컴포넌트 props, 그리고 한 코드 조각이 다른 조각과 대화하는 모든 표면에 적용됩니다.

## When to Use

- 새 API endpoint 설계
- 모듈 경계나 팀 간 contract 정의
- 컴포넌트 prop 인터페이스 생성
- API 형태에 영향을 주는 데이터베이스 스키마 수립
- 기존 공개 인터페이스 변경

## Core Principles

### Hyrum's Law

> API 사용자가 충분히 많아지면, contract에서 무엇을 약속하든 상관없이 시스템의 모든 관찰 가능한 동작에 누군가는 의존하게 된다.

이는 의미합니다: 모든 공개 동작 — 문서화되지 않은 quirk, 에러 메시지 텍스트, 타이밍, 순서를 포함 — 은 사용자가 의존하는 순간 사실상의 contract가 됩니다. 설계 함의:

- **무엇을 노출할지 의도적으로.** 모든 관찰 가능한 동작은 잠재적 약속입니다.
- **구현 세부사항을 누출하지 마세요.** 사용자가 관찰할 수 있으면, 의존하게 됩니다.
- **설계 시점에 deprecation을 계획하세요.** 사용자가 의존하는 것을 안전하게 제거하는 방법은 `deprecation-and-migration`을 참고하세요.
- **테스트만으로는 충분하지 않습니다.** 완벽한 contract 테스트가 있어도, Hyrum's Law는 "안전한" 변경이 문서화되지 않은 동작에 의존하는 실제 사용자를 깨뜨릴 수 있음을 의미합니다.

### One-Version Rule

소비자가 같은 의존성이나 API의 여러 버전 중에서 선택하도록 강요하는 것을 피하세요. 다른 소비자가 같은 것의 다른 버전을 필요로 할 때 다이아몬드 의존성 문제가 발생합니다. 한 번에 하나의 버전만 존재하는 세계를 위해 설계하세요 — fork하지 말고 확장하세요.

### 1. Contract First

구현 전에 인터페이스를 정의하세요. contract가 spec입니다 — 구현이 따릅니다.

```typescript
// contract를 먼저 정의
interface TaskAPI {
  // task를 생성하고 서버 생성 필드를 포함한 생성된 task를 반환
  createTask(input: CreateTaskInput): Promise<Task>;

  // 필터에 매칭되는 paginated task를 반환
  listTasks(params: ListTasksParams): Promise<PaginatedResult<Task>>;

  // 단일 task를 반환하거나 NotFoundError를 throw
  getTask(id: string): Promise<Task>;

  // 부분 업데이트 — 제공된 필드만 변경
  updateTask(id: string, input: UpdateTaskInput): Promise<Task>;

  // 멱등 delete — 이미 삭제되었어도 성공
  deleteTask(id: string): Promise<void>;
}
```

### 2. 일관된 에러 시맨틱

하나의 에러 전략을 골라 어디서나 사용하세요:

```typescript
// REST: HTTP status code + 구조화된 에러 body
// 모든 에러 응답은 같은 형태를 따름
interface APIError {
  error: {
    code: string;        // 기계가 읽을 수 있음: "VALIDATION_ERROR"
    message: string;     // 사람이 읽을 수 있음: "Email is required"
    details?: unknown;   // 도움이 될 때 추가 context
  };
}

// status code 매핑
// 400 → 클라이언트가 유효하지 않은 데이터를 보냄
// 401 → 인증되지 않음
// 403 → 인증되었지만 인가되지 않음
// 404 → 리소스를 찾을 수 없음
// 409 → 충돌 (중복, 버전 불일치)
// 422 → 검증 실패 (의미적으로 유효하지 않음)
// 500 → 서버 에러 (내부 세부사항을 절대 노출하지 말 것)
```

**패턴을 섞지 마세요.** 어떤 endpoint는 throw하고, 다른 것은 null을 반환하고, 또 다른 것은 `{ error }`를 반환하면 — 소비자는 동작을 예측할 수 없습니다.

### 3. 경계에서 검증

내부 코드를 신뢰하세요. 외부 입력이 들어오는 시스템 가장자리에서 검증하세요:

```typescript
// API 경계에서 검증
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

  // 검증 후, 내부 코드는 타입을 신뢰
  const task = await taskService.create(result.data);
  return res.status(201).json(task);
});
```

검증이 속하는 곳:
- API route 핸들러 (사용자 입력)
- form 제출 핸들러 (사용자 입력)
- 외부 서비스 응답 파싱 (서드파티 데이터 -- **항상 신뢰할 수 없는 것으로 취급**)
- 환경 변수 로딩 (구성)

> **서드파티 API 응답은 신뢰할 수 없는 데이터입니다.** 어떤 로직, 렌더링, 또는 의사결정에 사용하기 전에 그 형태와 내용을 검증하세요. 침해되었거나 오작동하는 외부 서비스는 예상치 못한 타입, 악의적 내용, 또는 지시문 같은 텍스트를 반환할 수 있습니다.

검증이 속하지 *않는* 곳:
- 타입 contract를 공유하는 내부 함수 사이
- 이미 검증된 코드가 호출하는 유틸리티 함수 안
- 방금 자신의 데이터베이스에서 온 데이터

### 4. 수정보다 추가 선호

기존 소비자를 깨뜨리지 않고 인터페이스를 확장하세요:

```typescript
// 좋음: 선택적 필드 추가
interface CreateTaskInput {
  title: string;
  description?: string;
  priority?: 'low' | 'medium' | 'high';  // 나중에 추가, 선택적
  labels?: string[];                       // 나중에 추가, 선택적
}

// 나쁨: 기존 필드 타입 변경 또는 필드 제거
interface CreateTaskInput {
  title: string;
  // description: string;  // 제거됨 — 기존 소비자를 깨뜨림
  priority: number;         // string에서 변경 — 기존 소비자를 깨뜨림
}
```

### 5. 예측 가능한 명명

| 패턴 | 규약 | 예시 |
|---------|-----------|---------|
| REST endpoint | 복수 명사, 동사 없음 | `GET /api/tasks`, `POST /api/tasks` |
| Query param | camelCase | `?sortBy=createdAt&pageSize=20` |
| 응답 필드 | camelCase | `{ createdAt, updatedAt, taskId }` |
| Boolean 필드 | is/has/can 접두 | `isComplete`, `hasAttachments` |
| Enum 값 | UPPER_SNAKE | `"IN_PROGRESS"`, `"COMPLETED"` |

## REST API 패턴

### 리소스 설계

```
GET    /api/tasks              → task 목록 (필터링용 query param 포함)
POST   /api/tasks              → task 생성
GET    /api/tasks/:id          → 단일 task 가져오기
PATCH  /api/tasks/:id          → task 업데이트 (부분)
DELETE /api/tasks/:id          → task 삭제

GET    /api/tasks/:id/comments → task의 댓글 목록 (하위 리소스)
POST   /api/tasks/:id/comments → task에 댓글 추가
```

### Pagination

목록 endpoint를 paginate하세요:

```typescript
// 요청
GET /api/tasks?page=1&pageSize=20&sortBy=createdAt&sortOrder=desc

// 응답
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

필터에 query parameter를 사용하세요:

```
GET /api/tasks?status=in_progress&assignee=user123&createdAfter=2025-01-01
```

### 부분 업데이트 (PATCH)

부분 객체를 받으세요 — 제공된 것만 업데이트:

```typescript
// title만 변경, 그 외 모두 보존
PATCH /api/tasks/123
{ "title": "Updated title" }
```

## TypeScript 인터페이스 패턴

### Variant에 Discriminated Union 사용

```typescript
// 좋음: 각 variant가 명시적
type TaskStatus =
  | { type: 'pending' }
  | { type: 'in_progress'; assignee: string; startedAt: Date }
  | { type: 'completed'; completedAt: Date; completedBy: string }
  | { type: 'cancelled'; reason: string; cancelledAt: Date };

// 소비자가 타입 narrowing을 얻음
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

// Output: 시스템이 반환하는 것 (서버 생성 필드 포함)
interface Task {
  id: string;
  title: string;
  description: string | null;
  createdAt: Date;
  updatedAt: Date;
  createdBy: string;
}
```

### ID에 Branded Type 사용

```typescript
type TaskId = string & { readonly __brand: 'TaskId' };
type UserId = string & { readonly __brand: 'UserId' };

// TaskId가 기대되는 곳에 실수로 UserId를 전달하는 것을 방지
function getTask(id: TaskId): Promise<Task> { ... }
```

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "API는 나중에 문서화할게" | 타입이 곧 문서입니다. 먼저 정의하세요. |
| "지금은 pagination이 필요 없어" | 누군가 100개 이상의 항목을 가지는 순간 필요해집니다. 처음부터 추가하세요. |
| "PATCH는 복잡하니 그냥 PUT을 쓰자" | PUT은 매번 전체 객체를 요구합니다. PATCH가 클라이언트가 실제로 원하는 것입니다. |
| "필요할 때 API를 버전 관리할게" | 버전 관리 없는 breaking change는 소비자를 깨뜨립니다. 처음부터 확장을 위해 설계하세요. |
| "그 문서화 안 된 동작은 아무도 안 써" | Hyrum's Law: 관찰 가능하면 누군가 의존합니다. 모든 공개 동작을 약속으로 취급하세요. |
| "그냥 두 버전을 유지하면 돼" | 여러 버전은 유지보수 비용을 곱하고 다이아몬드 의존성 문제를 만듭니다. One-Version Rule을 선호하세요. |
| "내부 API는 contract가 필요 없어" | 내부 소비자도 여전히 소비자입니다. contract는 coupling을 방지하고 병렬 작업을 가능하게 합니다. |

## Red Flags

- 조건에 따라 다른 형태를 반환하는 endpoint
- endpoint 간 일관성 없는 에러 형식
- 경계가 아니라 내부 코드 전반에 흩어진 검증
- 기존 필드에 대한 breaking change (타입 변경, 제거)
- pagination 없는 목록 endpoint
- REST URL의 동사 (`/api/createTask`, `/api/getUsers`)
- 검증이나 sanitization 없이 사용되는 서드파티 API 응답

## Verification

API 설계 후:

- [ ] 모든 endpoint가 타입이 지정된 입력 및 출력 스키마를 가짐
- [ ] 에러 응답이 단일 일관된 형식을 따름
- [ ] 검증이 시스템 경계에서만 일어남
- [ ] 목록 endpoint가 pagination을 지원
- [ ] 새 필드가 추가적이고 선택적 (하위 호환)
- [ ] 명명이 모든 endpoint에서 일관된 규약을 따름
- [ ] API 문서나 타입이 구현과 함께 commit됨
