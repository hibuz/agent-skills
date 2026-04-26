---
name: documentation-and-adrs
description: 결정사항과 문서를 기록합니다. 아키텍처 결정을 내리거나, 공개 API를 변경하거나, 기능을 출시할 때, 또는 미래의 엔지니어와 에이전트가 코드베이스를 이해하는 데 필요한 컨텍스트를 기록해야 할 때 사용하세요.
---

# 문서화 및 ADR (Documentation and ADRs)

## 개요 (Overview)

코드뿐만 아니라 결정사항을 문서화하세요. 가장 가치 있는 문서는 '왜(why)' — 즉 결정으로 이어진 컨텍스트, 제약 조건, 트레이드오프(trade-offs)를 담고 있습니다. 코드는 '무엇(what)'이 만들어졌는지를 보여주지만, 문서는 '왜 이렇게 만들어졌는지'와 '어떤 대안들이 고려되었는지'를 설명합니다. 이 컨텍스트는 미래의 인간 엔지니어와 코드베이스에서 작업하는 에이전트들에게 필수적입니다.

## 사용 시기 (When to Use)

- 중요한 아키텍처 결정을 내릴 때
- 여러 접근 방식 중에서 하나를 선택할 때
- 공개 API를 추가하거나 변경할 때
- 사용자 경험을 바꾸는 기능을 출시할 때
- 프로젝트에 새로운 팀원(또는 에이전트)을 온보딩할 때
- 같은 내용을 반복해서 설명하고 있는 자신을 발견했을 때

**사용하지 말아야 할 때:** 뻔한 코드를 문서화하지 마세요. 코드가 이미 말하고 있는 내용을 반복하는 주석을 달지 마세요. 일회성 프로토타입을 위한 문서를 작성하지 마세요.

## 아키텍처 결정 기록 (Architecture Decision Records - ADR)

ADR은 중요한 기술적 결정 뒤에 숨겨진 추론을 캡처합니다. 당신이 작성할 수 있는 가장 가치 있는 문서입니다.

### ADR을 작성해야 하는 경우

- 프레임워크, 라이브러리 또는 주요 의존성을 선택할 때
- 데이터 모델이나 데이터베이스 스키마를 설계할 때
- 인증(authentication) 전략을 선택할 때
- API 아키텍처(REST vs. GraphQL vs. tRPC)를 결정할 때
- 빌드 도구, 호스팅 플랫폼 또는 인프라를 선택할 때
- 되돌리는 데 비용이 많이 드는 모든 결정을 내릴 때

### ADR 템플릿 (ADR Template)

ADR은 `docs/decisions/`에 일련번호와 함께 저장하세요:

```markdown
# ADR-001: 기본 데이터베이스로 PostgreSQL 사용

## 상태 (Status)
제안됨(Proposed) | 수용됨(Accepted) | ADR-XXX에 의해 대체됨(Superseded) | 폐기됨(Deprecated)

## 날짜 (Date)
2025-01-15

## 컨텍스트 (Context)
태스크 관리 애플리케이션을 위한 기본 데이터베이스가 필요합니다. 주요 요구 사항:
- 관계형 데이터 모델 (사용자, 태스크, 팀 간의 관계)
- 태스크 상태 변경을 위한 ACID 트랜잭션
- 태스크 콘텐츠에 대한 전체 텍스트 검색(Full-text search) 지원
- 관리형 호스팅 가능 (소규모 팀이며 운영 역량이 제한적임)

## 결정 (Decision)
PostgreSQL을 Prisma ORM과 함께 사용합니다.

## 고려된 대안들 (Alternatives Considered)

### MongoDB
- 장점: 유연한 스키마, 시작하기 쉬움
- 단점: 우리 데이터는 본질적으로 관계형임; 관계를 수동으로 관리해야 함
- 거절 이유: 문서 저장소(Document store)에서의 관계형 데이터는 복잡한 조인이나 데이터 중복을 초래함

### SQLite
- 장점: 설정 불필요, 임베디드, 빠른 읽기 속도
- 단점: 제한적인 동시 쓰기 지원, 프로덕션용 관리형 호스팅 부재
- 거절 이유: 프로덕션 환경의 다중 사용자 웹 애플리케이션에 적합하지 않음

### MySQL
- 장점: 성숙함, 널리 지원됨
- 단점: PostgreSQL이 JSON 지원, 전체 텍스트 검색, 에코시스템 도구 측면에서 더 뛰어남
- 거절 이유: PostgreSQL이 우리 기능 요구 사항에 더 적합함

## 결과 (Consequences)
- Prisma는 타입 안정성 있는 데이터베이스 접근 및 마이그레이션 관리를 제공함
- Elasticsearch를 추가하는 대신 PostgreSQL의 전체 텍스트 검색을 사용할 수 있음
- 팀에 PostgreSQL 지식이 필요함 (표준 기술이며 위험 낮음)
- 관리형 서비스(Supabase, Neon 또는 RDS)에서 호스팅함
```

### ADR 생명주기 (ADR Lifecycle)

```
제안됨(PROPOSED) → 수용됨(ACCEPTED) → (대체됨(SUPERSEDED) 또는 폐기됨(DEPRECATED))
```

- **이전 ADR을 삭제하지 마세요.** 과거의 컨텍스트를 담고 있습니다.
- 결정이 바뀌면, 이전 ADR을 참조하고 대체하는 새로운 ADR을 작성하세요.

## 인라인 문서화 (Inline Documentation)

### 주석을 달아야 할 때

'무엇(what)'이 아니라 '왜(why)'를 설명하세요:

```typescript
// 나쁨: 코드를 그대로 반복함
// 카운터를 1 증가시킴
counter += 1;

// 좋음: 명확하지 않은 의도를 설명함
// 속도 제한(Rate limit)에 슬라이딩 윈도우(sliding window)를 사용합니다.
// 윈도우 경계에서의 급격한 공격을 방지하기 위해 고정된 일정이 아닌
// 윈도우 경계에서 카운터를 리셋합니다.
if (now - windowStart > WINDOW_SIZE_MS) {
  counter = 0;
  windowStart = now;
}
```

### 주석을 달지 말아야 할 때

```typescript
// 자기설명적인 코드에 주석을 달지 마세요
function calculateTotal(items: CartItem[]): number {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}

// 나중에 해야 할 일에 TODO 주석을 남기지 마세요. 지금 바로 하세요.
// TODO: 에러 처리 추가  ← 그냥 추가하세요

// 주석 처리된 코드를 남겨두지 마세요.
// const oldImplementation = () => { ... }  ← 삭제하세요. Git 히스토리에 남아 있습니다.
```

### 알려진 함정 문서화 (Document Known Gotchas)

```typescript
/**
 * 중요: 이 함수는 첫 번째 렌더링 전에 호출되어야 합니다.
 * 하이드레이션(hydration) 후에 호출되면 SSR 중에 테마 컨텍스트를
 * 사용할 수 없으므로 스타일이 적용되지 않은 콘텐츠가 깜빡이는 현상이 발생합니다.
 *
 * 전체 설계 근거는 ADR-003을 참조하세요.
 */
export function initializeTheme(theme: Theme): void {
  // ...
}
```

## API 문서화 (API Documentation)

공개 API(REST, GraphQL, 라이브러리 인터페이스)의 경우:

### 타입과 함께 인라인으로 (TypeScript 권장 방식)

```typescript
/**
 * 새로운 태스크를 생성합니다.
 *
 * @param input - 태스크 생성 데이터 (제목 필수, 설명 선택)
 * @returns 서버에서 생성된 ID와 타임스탬프가 포함된 태스크 객체
 * @throws {ValidationError} 제목이 비어 있거나 200자를 초과하는 경우
 * @throws {AuthenticationError} 사용자가 인증되지 않은 경우
 *
 * @example
 * const task = await createTask({ title: '장보기' });
 * console.log(task.id); // "task_abc123"
 */
export async function createTask(input: CreateTaskInput): Promise<Task> {
  // ...
}
```

### REST API를 위한 OpenAPI / Swagger

```yaml
paths:
  /api/tasks:
    post:
      summary: 태스크 생성
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateTaskInput'
      responses:
        '201':
          description: 태스크 생성됨
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Task'
        '422':
          description: 유효성 검사 에러
```

## README 구조

모든 프로젝트는 다음을 포함하는 README를 가져야 합니다:

```markdown
# 프로젝트 이름

이 프로젝트가 무엇을 하는지에 대한 한 단락 설명.

## 빠른 시작 (Quick Start)
1. 저장소 클론
2. 의존성 설치: `npm install`
3. 환경 설정: `cp .env.example .env`
4. 개발 서버 실행: `npm run dev`

## 명령어 (Commands)
| 명령어 | 설명 |
|---------|-------------|
| `npm run dev` | 개발 서버 시작 |
| `npm test` | 테스트 실행 |
| `npm run build` | 프로덕션 빌드 |
| `npm run lint` | 린터 실행 |

## 아키텍처 (Architecture)
프로젝트 구조와 주요 설계 결정에 대한 간략한 개요.
상세 내용은 ADR을 링크하세요.

## 기여하기 (Contributing)
기여 방법, 코딩 표준, PR 프로세스.
```

## 변경 이력 관리 (Changelog Maintenance)

출시된 기능의 경우:

```markdown
# 변경 이력 (Changelog)

## [1.2.0] - 2025-01-20
### 추가 (Added)
- 태스크 공유: 사용자가 팀원과 태스크를 공유할 수 있음 (#123)
- 태스크 할당 시 이메일 알림 기능 (#124)

### 수정 (Fixed)
- 생성 버튼을 빠르게 클릭할 때 중복 태스크가 생성되는 문제 수정 (#125)

### 변경 (Changed)
- UX 개선을 위해 태스크 목록을 페이지당 50개씩 로드함 (기존 20개) (#126)
```

## 에이전트를 위한 문서화 (Documentation for Agents)

AI 에이전트 컨텍스트를 위한 특별한 고려 사항:

- **CLAUDE.md / 규칙 파일** — 프로젝트 관례를 문서화하여 에이전트가 이를 따르도록 함
- **Spec 파일** — 에이전트가 올바른 것을 만들 수 있도록 스펙을 최신으로 유지함
- **ADR** — 에이전트가 과거의 결정 이유를 이해하도록 도움 (중복된 결정 방지)
- **인라인 함정 알림** — 에이전트가 알려진 트랩에 빠지지 않도록 함

## 흔한 자기합리화 (Common Rationalizations)

| 자기합리화 | 실제 상황 |
|---|---|
| "코드가 스스로 설명하고 있어요" | 코드는 '무엇'을 보여줄 뿐입니다. '왜', 어떤 대안이 거절되었는지, 어떤 제약이 있는지 보여주지 않습니다. |
| "API가 안정되면 그때 문서를 쓸게요" | 문서를 작성할 때 API가 더 빨리 안정됩니다. 문서는 설계의 첫 번째 테스트입니다. |
| "아무도 문서를 읽지 않아요" | 에이전트는 읽습니다. 미래의 엔지니어도 읽습니다. 3개월 후의 당신 자신도 읽습니다. |
| "ADR은 오버헤드예요" | 10분의 ADR 작성이 6개월 후 똑같은 결정을 두고 벌어지는 2시간의 논쟁을 막아줍니다. |
| "주석은 낡기 마련이에요" | '왜'에 대한 주석은 안정적입니다. '무엇'에 대한 주석이 낡는 것이며, 그래서 전자의 경우만 주석을 써야 합니다. |

## 위험 신호 (Red Flags)

- 서면 근거가 없는 아키텍처 결정
- 문서나 타입이 없는 공개 API
- 프로젝트 실행 방법을 설명하지 않는 README
- 삭제 대신 주석 처리된 코드
- 몇 주째 남아 있는 TODO 주석
- 중요한 아키텍처 선택이 있는 프로젝트에 ADR이 없음
- 의도를 설명하는 대신 코드를 반복하는 문서

## 검증 (Verification)

문서화 후:

- [ ] 모든 중요한 아키텍처 결정에 대해 ADR이 존재하는가
- [ ] README가 빠른 시작, 명령어, 아키텍처 개요를 포함하는가
- [ ] API 함수에 파라미터와 반환 타입 문서화가 되어 있는가
- [ ] 알려진 함정들이 관련 위치에 인라인으로 문서화되었는가
- [ ] 주석 처리된 코드가 남아 있지 않은가
- [ ] 규칙 파일(CLAUDE.md 등)이 최신이며 정확한가
