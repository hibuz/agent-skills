---
name: documentation-and-adrs
description: 결정과 문서를 기록. 아키텍처 결정, 공개 API 변경, 기능 ship, 또는 미래 엔지니어와 agent가 코드베이스를 이해하는 데 필요한 context를 기록할 필요가 있을 때 사용.
---

# Documentation and ADRs

## Overview

코드만이 아니라 결정을 문서화하세요. 가장 가치 있는 문서는 *왜*를 포착합니다 — 결정으로 이어진 context, 제약, trade-off. 코드는 *무엇*이 만들어졌는지 보여줍니다; 문서는 *왜 이렇게 만들어졌는지*와 *어떤 대안이 고려되었는지*를 설명합니다. 이 context는 코드베이스에서 작업할 미래의 사람과 agent에게 필수적입니다.

## When to Use

- 중대한 아키텍처 결정을 내릴 때
- 경합하는 접근 중에서 선택할 때
- 공개 API를 추가하거나 변경할 때
- 사용자 대면 동작을 바꾸는 기능을 ship할 때
- 새 팀원(또는 agent)을 프로젝트에 온보딩할 때
- 같은 것을 반복해서 설명하고 있는 자신을 발견할 때

**사용하지 말아야 할 때:** 명백한 코드를 문서화하지 마세요. 코드가 이미 말하는 것을 다시 말하는 주석을 추가하지 마세요. 버릴 프로토타입에 문서를 쓰지 마세요.

## Architecture Decision Records (ADR)

ADR은 중대한 기술 결정 뒤의 추론을 포착합니다. 작성할 수 있는 가장 가치 높은 문서입니다.

### ADR을 언제 작성하는가

- 프레임워크, 라이브러리, 또는 주요 의존성 선택
- 데이터 모델이나 데이터베이스 스키마 설계
- 인증 전략 선택
- API 아키텍처 결정 (REST vs. GraphQL vs. tRPC)
- 빌드 도구, 호스팅 플랫폼, 또는 인프라 중 선택
- 되돌리기에 비싼 모든 결정

### ADR 템플릿

ADR을 순차 번호와 함께 `docs/decisions/`에 저장:

```markdown
# ADR-001: Use PostgreSQL for primary database

## Status
Accepted | Superseded by ADR-XXX | Deprecated

## Date
2025-01-15

## Context
We need a primary database for the task management application. Key requirements:
- Relational data model (users, tasks, teams with relationships)
- ACID transactions for task state changes
- Support for full-text search on task content
- Managed hosting available (for small team, limited ops capacity)

## Decision
Use PostgreSQL with Prisma ORM.

## Alternatives Considered

### MongoDB
- Pros: Flexible schema, easy to start with
- Cons: Our data is inherently relational; would need to manage relationships manually
- Rejected: Relational data in a document store leads to complex joins or data duplication

### SQLite
- Pros: Zero configuration, embedded, fast for reads
- Cons: Limited concurrent write support, no managed hosting for production
- Rejected: Not suitable for multi-user web application in production

### MySQL
- Pros: Mature, widely supported
- Cons: PostgreSQL has better JSON support, full-text search, and ecosystem tooling
- Rejected: PostgreSQL is the better fit for our feature requirements

## Consequences
- Prisma provides type-safe database access and migration management
- We can use PostgreSQL's full-text search instead of adding Elasticsearch
- Team needs PostgreSQL knowledge (standard skill, low risk)
- Hosting on managed service (Supabase, Neon, or RDS)
```

### ADR 라이프사이클

```
PROPOSED → ACCEPTED → (SUPERSEDED 또는 DEPRECATED)
```

- **옛 ADR을 삭제하지 마세요.** 역사적 context를 포착합니다.
- 결정이 바뀌면, 옛것을 참조하고 대체하는 새 ADR을 작성하세요.

## 인라인 문서

### 언제 주석을 다는가

*무엇*이 아니라 *왜*를 주석으로:

```typescript
// 나쁨: 코드를 다시 말함
// Increment counter by 1
counter += 1;

// 좋음: 명백하지 않은 의도를 설명
// Rate limit uses a sliding window — reset counter at window boundary,
// not on a fixed schedule, to prevent burst attacks at window edges
if (now - windowStart > WINDOW_SIZE_MS) {
  counter = 0;
  windowStart = now;
}
```

### 언제 주석을 달지 않는가

```typescript
// 자명한 코드에 주석 달지 말 것
function calculateTotal(items: CartItem[]): number {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}

// 지금 해야 할 것에 TODO 주석 남기지 말 것
// TODO: add error handling  ← 그냥 추가하세요

// 주석 처리된 코드 남기지 말 것
// const oldImplementation = () => { ... }  ← 삭제하세요, git에 히스토리 있음
```

### 알려진 Gotcha 문서화

```typescript
/**
 * IMPORTANT: This function must be called before the first render.
 * If called after hydration, it causes a flash of unstyled content
 * because the theme context isn't available during SSR.
 *
 * See ADR-003 for the full design rationale.
 */
export function initializeTheme(theme: Theme): void {
  // ...
}
```

## API 문서

공개 API(REST, GraphQL, 라이브러리 인터페이스)의 경우:

### 타입과 함께 인라인 (TypeScript에 권장)

```typescript
/**
 * Creates a new task.
 *
 * @param input - Task creation data (title required, description optional)
 * @returns The created task with server-generated ID and timestamps
 * @throws {ValidationError} If title is empty or exceeds 200 characters
 * @throws {AuthenticationError} If the user is not authenticated
 *
 * @example
 * const task = await createTask({ title: 'Buy groceries' });
 * console.log(task.id); // "task_abc123"
 */
export async function createTask(input: CreateTaskInput): Promise<Task> {
  // ...
}
```

### REST API용 OpenAPI / Swagger

```yaml
paths:
  /api/tasks:
    post:
      summary: Create a task
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateTaskInput'
      responses:
        '201':
          description: Task created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Task'
        '422':
          description: Validation error
```

## README 구조

모든 프로젝트에는 다음을 다루는 README가 있어야 합니다:

```markdown
# Project Name

One-paragraph description of what this project does.

## Quick Start
1. Clone the repo
2. Install dependencies: `npm install`
3. Set up environment: `cp .env.example .env`
4. Run the dev server: `npm run dev`

## Commands
| Command | Description |
|---------|-------------|
| `npm run dev` | Start development server |
| `npm test` | Run tests |
| `npm run build` | Production build |
| `npm run lint` | Run linter |

## Architecture
Brief overview of the project structure and key design decisions.
Link to ADRs for details.

## Contributing
How to contribute, coding standards, PR process.
```

## Changelog 유지보수

ship된 기능의 경우:

```markdown
# Changelog

## [1.2.0] - 2025-01-20
### Added
- Task sharing: users can share tasks with team members (#123)
- Email notifications for task assignments (#124)

### Fixed
- Duplicate tasks appearing when rapidly clicking create button (#125)

### Changed
- Task list now loads 50 items per page (was 20) for better UX (#126)
```

## Agent를 위한 문서

AI agent context에 대한 특별 고려:

- **CLAUDE.md / rules 파일** — agent가 따르도록 프로젝트 규약 문서화
- **Spec 파일** — agent가 올바른 것을 빌드하도록 spec 최신 유지
- **ADR** — agent가 과거 결정이 왜 내려졌는지 이해하도록 도움 (재결정 방지)
- **인라인 gotcha** — agent가 알려진 함정에 빠지는 것을 방지

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "코드가 자체 문서화돼" | 코드는 무엇을 보여줍니다. 왜, 어떤 대안이 거부됐는지, 어떤 제약이 적용되는지는 보여주지 않습니다. |
| "API가 안정되면 문서를 쓸게" | API는 문서화할 때 더 빨리 안정됩니다. 문서가 설계의 첫 테스트입니다. |
| "아무도 문서를 안 읽어" | agent는 읽습니다. 미래 엔지니어는 읽습니다. 3개월 후의 당신도 읽습니다. |
| "ADR은 오버헤드야" | 10분짜리 ADR이 6개월 후 같은 결정에 대한 2시간 논쟁을 방지합니다. |
| "주석은 구식이 돼" | *왜*에 대한 주석은 안정적입니다. *무엇*에 대한 주석이 구식이 됩니다 — 그래서 전자만 쓰는 것입니다. |

## Red Flags

- 작성된 근거 없는 아키텍처 결정
- 문서나 타입 없는 공개 API
- 프로젝트 실행 방법을 설명하지 않는 README
- 삭제 대신 주석 처리된 코드
- 몇 주째 있는 TODO 주석
- 중대한 아키텍처 선택이 있는 프로젝트에 ADR 없음
- 의도를 설명하는 대신 코드를 다시 말하는 문서

## Verification

문서화 후:

- [ ] 모든 중대한 아키텍처 결정에 ADR이 존재
- [ ] README가 quick start, command, 아키텍처 개요를 다룸
- [ ] API 함수에 파라미터와 반환 타입 문서가 있음
- [ ] 알려진 gotcha가 중요한 곳에 인라인으로 문서화됨
- [ ] 주석 처리된 코드가 남지 않음
- [ ] rules 파일(CLAUDE.md 등)이 최신이고 정확함
