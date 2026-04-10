---
name: documentation-and-adrs
description: Records 결정 및 문서화. 아키텍처 결정을 내릴 때, 공개 APIs 변경, 기능 제공 또는 미래 엔지니어와 agents가 코드베이스를 이해하는 데 필요한 컨텍스트를 기록해야 할 때 사용합니다.
---

# 문서 및 ADRs

## 개요

코드뿐만 아니라 의사결정도 문서화하세요. 가장 가치 있는 문서는 *이유*, 즉 결정을 내리게 된 맥락, 제약 조건, trade-offs를 포착합니다. 코드는 *무엇*이 built인지 보여줍니다. 문서에는 *왜 built가 이런 식으로 이루어졌는지*와 *어떤 대안이 고려되었는지* 설명되어 있습니다. 이 컨텍스트는 코드베이스에서 작업하는 미래의 인간과 agents에게 필수적입니다.

## 사용 시기

- 중요한 아키텍처 결정을 내립니다.
- 경쟁적인 접근 방식 중에서 선택
- 공개 API 추가 또는 변경
- user-facing 동작을 변경하는 기능 제공
- 프로젝트에 새로운 팀 구성원(또는 agents) 온보딩
- 같은 내용을 반복해서 설명하고 있는 자신을 발견할 때

**사용하지 말아야 할 때:** 명백한 코드를 문서화하지 마세요. 코드에 이미 표시된 내용을 restate하는 주석을 추가하지 마세요. 일회용 프로토타입에 대한 문서를 작성하지 마십시오.

## Architecture Decision Records (ADRs)

ADRs는 중요한 기술적 결정의 이면에 있는 추론을 포착합니다. 이는 귀하가 작성할 수 있는 highest-value 문서입니다.

### ADR를 작성해야 하는 경우

- 프레임워크, 라이브러리 또는 주요 종속성 선택
- 데이터 모델 또는 데이터베이스 스키마 설계
- 인증 전략 선택
- API 아키텍처 결정(REST 대 GraphQL 대 tRPC)
- build 도구, 호스팅 플랫폼 orm 또는 인프라 중에서 선택
- 취소하는 데 비용이 많이 드는 결정

### ADR 템플릿

일련번호를 사용하여 ADRs를 `docs/decisions/`에 저장합니다.

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

### ADR 수명주기

```
PROPOSED → ACCEPTED → (SUPERSEDED or DEPRECATED)
```

- **이전 ADRs를 삭제하지 마세요.** 역사적 맥락을 포착합니다.
- 결정이 변경되면 이전 결정을 참조하고 대체하는 새로운 ADR를 작성합니다.

## 인라인 문서

### 댓글을 작성해야 하는 경우

*무엇*이 아닌 *이유*를 댓글로 달아주세요.

```typescript
// BAD: Restates the code
// Increment counter by 1
counter += 1;

// GOOD: Explains non-obvious intent
// Rate limit uses a sliding window — reset counter at window boundary,
// not on a fixed schedule, to prevent burst attacks at window edges
if (now - windowStart > WINDOW_SIZE_MS) {
  counter = 0;
  windowStart = now;
}
```

### NOT가 댓글을 달 때

```typescript
// Don't comment self-explanatory code
function calculateTotal(items: CartItem[]): number {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}

// Don't leave TODO comments for things you should just do now
// TODO: add error handling  ← Just add it

// Don't leave commented-out code
// const oldImplementation = () => { ... }  ← Delete it, git has history
```

### 문서에 알려진 문제

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

공용 APIs(REST, GraphQL, 라이브러리 인터페이스)의 경우:

### 유형이 포함된 인라인(TypeScript에 권장됨)

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

### OpenAPI / Swagger for REST APIs

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

모든 프로젝트에는 다음을 포함하는 README가 있어야 합니다.

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

## 변경 로그 유지 관리

제공되는 기능의 경우:

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

## Agents에 대한 문서

AI agent 컨텍스트에 대한 특별 고려사항:

- **CLAUDE.md / 규칙 파일** — agents가 이를 따르도록 프로젝트 규칙을 문서화합니다.
- **사양 파일** — agents build가 올바르게 작동하도록 사양을 계속 업데이트하세요.
- **ADRs** — agents가 과거 결정이 내려진 이유를 이해하도록 도와줍니다(re-deciding 방지).
- **인라인 문제** — agents가 알려진 함정에 빠지는 것을 방지합니다.

## 일반적인 합리화

| 합리화 | 현실 |
|---|---|
| "코드는 self-documenting입니다" | 코드는 무엇을 보여줍니다. 이유, 거부된 대안 또는 적용되는 제약 조건은 표시되지 않습니다. |
| "API가 안정되면 문서를 작성하겠습니다" | APIs는 문서화 시 더욱 빠르게 안정화됩니다. 문서는 디자인의 첫 번째 테스트입니다. |
| "아무도 문서를 읽지 않습니다" | Agents 않습니다. 미래의 엔지니어들은 그렇습니다. 귀하의 3-months-later 자체가 그렇습니다. |
| "ADRs가 오버헤드입니다." | 10분 ADR는 6개월 후 동일한 결정에 대한 2시간 토론을 방지합니다. |
| "댓글이 오래되었습니다." | *이유*에 대한 댓글은 안정적입니다. *무엇*에 대한 댓글은 오래된 것입니다. 그래서 former만 작성하는 것입니다. |

## 위험 신호

- 서면 근거가 없는 아키텍처 결정
- 문서나 유형이 없는 공개 APIs
- 프로젝트 실행 방법을 설명하지 않는 README
- 삭제 대신 주석 처리된 코드
- 몇 주 동안 있었던 TODO 댓글
- 중요한 아키텍처 선택이 있는 프로젝트에는 ADRs가 없습니다.
- 의도를 설명하는 대신 restates 코드를 설명하는 문서

## 확인

문서화한 후:

- [ ] ADRs는 모든 중요한 아키텍처 결정에 존재합니다.
- [ ] README는 quick 시작, 명령 및 아키텍처 개요를 다룹니다.
- [ ] API 함수에는 매개변수 및 반환 유형 문서가 있습니다.
- [ ] 알려진 문제는 중요한 위치에 인라인으로 문서화됩니다.
- [ ] commented-out 코드가 남아 있지 않습니다.
- [ ] 규칙 파일(CLAUDE.md 등)이 최신이고 정확합니다.
