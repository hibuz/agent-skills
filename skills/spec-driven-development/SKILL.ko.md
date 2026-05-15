---
name: spec-driven-development
description: 코딩 전에 spec을 생성. 새 프로젝트, 기능, 또는 중대한 변경을 시작하고 아직 명세가 없을 때 사용. 요구사항이 불명확하거나, 모호하거나, 막연한 아이디어로만 존재할 때 사용.
---

# Spec-Driven Development

## Overview

코드를 작성하기 전에 구조화된 명세를 작성하세요. spec은 당신과 사람 엔지니어 사이의 공유된 진실의 원천입니다 — 무엇을 빌드하고, 왜, 그리고 언제 끝났는지 어떻게 알지를 정의합니다. spec 없는 코드는 추측입니다.

## When to Use

- 새 프로젝트나 기능 시작
- 요구사항이 모호하거나 불완전
- 변경이 여러 파일이나 모듈을 건드림
- 아키텍처 결정을 내리려 함
- 작업이 구현에 30분 이상 걸림

**사용하지 말아야 할 때:** 한 줄 수정, 오타 교정, 또는 요구사항이 모호하지 않고 자기 완결적인 변경.

## Gated 워크플로

spec-driven development는 네 페이즈입니다. 현재 페이즈가 검증될 때까지 다음 페이즈로 진행하지 마세요.

```
SPECIFY ──→ PLAN ──→ TASKS ──→ IMPLEMENT
   │          │        │          │
   ▼          ▼        ▼          ▼
 사람        사람     사람        사람
 리뷰        리뷰     리뷰        리뷰
```

### Phase 1: Specify

높은 수준의 비전으로 시작하세요. 요구사항이 구체적이 될 때까지 사람에게 명확화 질문을 하세요.

**가정을 즉시 표면화하세요.** 어떤 spec 내용도 쓰기 전에, 가정하는 것을 나열:

```
ASSUMPTIONS I'M MAKING:
1. This is a web application (not native mobile)
2. Authentication uses session-based cookies (not JWT)
3. The database is PostgreSQL (based on existing Prisma schema)
4. We're targeting modern browsers only (no IE11)
→ Correct me now or I'll proceed with these.
```

모호한 요구사항을 조용히 채우지 마세요. spec의 전체 목적은 코드가 작성되기 *전에* 오해를 표면화하는 것입니다 — 가정은 가장 위험한 형태의 오해입니다.

**이 여섯 가지 핵심 영역을 다루는 spec 문서를 작성:**

1. **Objective** — 무엇을 왜 빌드하나? 사용자는 누구? 성공은 어떻게 보이나?

2. **Commands** — 도구 이름만이 아니라 flag를 포함한 전체 실행 가능한 커맨드.
   ```
   Build: npm run build
   Test: npm test -- --coverage
   Lint: npm run lint --fix
   Dev: npm run dev
   ```

3. **Project Structure** — 소스 코드가 어디에, 테스트가 어디에, 문서가 어디에.
   ```
   src/           → 애플리케이션 소스 코드
   src/components → React 컴포넌트
   src/lib        → 공유 유틸리티
   tests/         → unit 및 integration 테스트
   e2e/           → end-to-end 테스트
   docs/          → 문서
   ```

4. **Code Style** — 스타일을 보여주는 실제 코드 스니펫 하나가 그것을 묘사하는 세 단락보다 낫습니다. 명명 규약, 포맷팅 규칙, 좋은 출력 예시 포함.

5. **Testing Strategy** — 어떤 프레임워크, 테스트가 어디에, 커버리지 기대, 어떤 관심사에 어떤 테스트 수준.

6. **Boundaries** — 3계층 시스템:
   - **항상 할 것:** commit 전 테스트 실행, 명명 규약 따름, 입력 검증
   - **먼저 물어볼 것:** 데이터베이스 스키마 변경, 의존성 추가, CI config 변경
   - **절대 하지 말 것:** secret commit, vendor 디렉터리 편집, 승인 없이 실패하는 테스트 제거

**Spec 템플릿:**

```markdown
# Spec: [Project/Feature Name]

## Objective
[What we're building and why. User stories or acceptance criteria.]

## Tech Stack
[Framework, language, key dependencies with versions]

## Commands
[Build, test, lint, dev — full commands]

## Project Structure
[Directory layout with descriptions]

## Code Style
[Example snippet + key conventions]

## Testing Strategy
[Framework, test locations, coverage requirements, test levels]

## Boundaries
- Always: [...]
- Ask first: [...]
- Never: [...]

## Success Criteria
[How we'll know this is done — specific, testable conditions]

## Open Questions
[Anything unresolved that needs human input]
```

**지시를 성공 기준으로 reframe하세요.** 막연한 요구사항을 받으면, 구체적 조건으로 번역:

```
REQUIREMENT: "Make the dashboard faster"

REFRAMED SUCCESS CRITERIA:
- Dashboard LCP < 2.5s on 4G connection
- Initial data load completes in < 500ms
- No layout shift during load (CLS < 0.1)
→ Are these the right targets?
```

이는 "더 빠르게"가 무슨 뜻인지 추측하기보다 명확한 목표를 향해 loop하고, 재시도하고, 문제를 해결하게 합니다.

### Phase 2: Plan

검증된 spec으로, 기술 구현 plan 생성:

1. 주요 컴포넌트와 그 의존성 식별
2. 구현 순서 결정 (무엇이 먼저 빌드되어야 하나)
3. 위험과 완화 전략 메모
4. 병렬로 빌드 가능한 것 vs. 순차여야 하는 것 식별
5. 페이즈 사이 검증 체크포인트 정의

plan은 리뷰 가능해야 합니다: 사람이 읽고 "그래, 그게 올바른 접근" 또는 "아니, X를 바꿔"라고 말할 수 있어야 합니다.

### Phase 3: Tasks

plan을 별개의 구현 가능한 task로 분해:

- 각 task는 단일 집중 세션에 완료 가능해야 함
- 각 task에 명시적 수용 기준 있음
- 각 task에 검증 단계 포함 (test, build, 수동 확인)
- task가 인지된 중요도가 아니라 의존성으로 순서됨
- 어떤 task도 ~5개 파일 이상 변경을 요구하지 않아야 함

**Task 템플릿:**
```markdown
- [ ] Task: [Description]
  - Acceptance: [What must be true when done]
  - Verify: [How to confirm — test command, build, manual check]
  - Files: [Which files will be touched]
```

### Phase 4: Implement

`skills/incremental-implementation/SKILL.md`(`incremental-implementation`)와 `skills/test-driven-development/SKILL.md`(`test-driven-development`)를 따라 task를 한 번에 하나씩 실행하세요. agent에게 전체 spec을 범람시키기보다 각 단계에서 올바른 spec 섹션과 소스 파일을 로드하기 위해 `skills/context-engineering/SKILL.md`(`context-engineering`)를 사용하세요.

## Spec을 살아 있게 유지

spec은 일회성 산출물이 아니라 살아 있는 문서입니다:

- **결정이 바뀌면 업데이트** — 데이터 모델이 바뀌어야 함을 발견하면, spec을 먼저 업데이트한 뒤 구현.
- **범위가 바뀌면 업데이트** — 추가되거나 잘린 기능은 spec에 반영되어야 함.
- **spec을 commit** — spec은 코드와 함께 version control에 속함.
- **PR에서 spec 참조** — 각 PR이 구현하는 spec 섹션으로 다시 링크.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "이건 단순하니 spec이 필요 없어" | 단순한 작업은 *긴* spec이 필요 없지만, 여전히 수용 기준이 필요합니다. 두 줄 spec이면 괜찮습니다. |
| "코드 작성 후 spec을 쓸게" | 그건 명세가 아니라 문서입니다. spec의 가치는 코드 *전에* 명료함을 강제하는 것입니다. |
| "spec이 우리를 늦출 거야" | 15분짜리 spec이 몇 시간의 재작업을 방지합니다. 15분의 waterfall이 15시간의 디버깅을 이깁니다. |
| "어차피 요구사항은 바뀔 거야" | 그래서 spec이 살아 있는 문서입니다. 구식 spec도 spec 없는 것보다 낫습니다. |
| "사용자가 원하는 것을 알아" | 명확한 요청에도 암묵적 가정이 있습니다. spec이 그 가정을 표면화합니다. |

## Red Flags

- 작성된 요구사항 없이 코드 작성 시작
- "끝"이 무슨 뜻인지 명확히 하기 전에 "그냥 빌드 시작할까요?"라고 물음
- 어떤 spec이나 task 목록에도 언급되지 않은 기능 구현
- 문서화하지 않고 아키텍처 결정
- "무엇을 빌드할지 명백하다"고 spec을 건너뜀

## Verification

구현으로 진행하기 전, 확인:

- [ ] spec이 여섯 가지 핵심 영역을 모두 다룸
- [ ] 사람이 spec을 리뷰하고 승인함
- [ ] 성공 기준이 구체적이고 테스트 가능
- [ ] 경계(Always/Ask First/Never)가 정의됨
- [ ] spec이 repository의 파일에 저장됨
