---
name: spec-driven-development
description: 코딩하기 전에 사양을 만듭니다. 새 프로젝트, 기능 또는 중요한 변경을 시작할 때 사용하며 아직 사양이 없습니다. requirements가 불분명하거나 모호하거나 모호한 아이디어로만 존재할 때 사용합니다.
---

# 사양 중심 개발

## 개요

코드를 작성하기 전에 구조화된 사양을 작성하세요. 사양은 귀하와 인간 엔지니어 사이의 공유된 정보 소스입니다. 이는 우리가 building하는 내용, 이유 및 완료 여부를 어떻게 알 수 있는지 정의합니다. 사양이 없는 코드는 추측입니다.

## 사용 시기

- 새로운 프로젝트나 기능 시작하기
- Requirements가 모호하거나 불완전합니다.
- 변경 사항이 여러 파일이나 모듈에 영향을 미칩니다.
- 아키텍처 결정을 내리려고 합니다.
- 작업을 구현하는 데 30분 이상 소요

**사용하지 말아야 할 때:** 한 줄 수정, 오타 수정 또는 requirements가 명확하고 self-contained인 변경 사항.

## 더 게이트 Workflow

사양 중심 개발에는 4단계가 있습니다. 현재 단계가 검증될 때까지 다음 단계로 진행하지 마세요.

```
SPECIFY ──→ PLAN ──→ TASKS ──→ IMPLEMENT
   │          │        │          │
   ▼          ▼        ▼          ▼
 Human      Human    Human      Human
 reviews    reviews  reviews    reviews
```

### 1단계: 지정

high-level 비전으로 시작하세요. requirements가 구체적일 때까지 사람에게 명확한 질문을 하세요.

**즉시 가정을 드러냅니다.** 사양 콘텐츠를 작성하기 전에 가정하는 내용을 나열하세요.

```
ASSUMPTIONS I'M MAKING:
1. This is a web application (not native mobile)
2. Authentication uses session-based cookies (not JWT)
3. The database is PostgreSQL (based on existing Prisma schema)
4. We're targeting modern browsers only (no IE11)
→ Correct me now or I'll proceed with these.
```

모호한 requirements를 자동으로 채우지 마세요. 사양의 전체 목적은 코드가 작성되기 *전에* 오해를 표면화하는 것입니다. 가정은 오해의 가장 위험한 form입니다.

**다음 6가지 핵심 영역을 다루는 사양 문서를 작성하세요.**

1. **목적** — 우리는 building이며 그 이유는 무엇입니까? 사용자는 누구입니까? 성공은 어떤 모습인가요?

2. **명령** — 도구 이름뿐만 아니라 플래그가 포함된 전체 실행 가능 명령입니다.
   ```
   Build: npm run build
   Test: npm test -- --coverage
   Lint: npm run lint --fix
   Dev: npm run dev
   ```

3. **프로젝트 구조** — 소스 코드가 있는 곳, 테스트가 진행되는 곳, 문서가 속한 곳.
   ```
   src/           → Application source code
   src/components → React components
   src/lib        → Shared utilities
   tests/         → Unit and integration tests
   e2e/           → End-to-end tests
   docs/          → Documentation
   ```

4. **코드 스타일** — 귀하의 스타일을 보여주는 하나의 실제 코드 조각이 스타일을 설명하는 세 개의 문단보다 좋습니다. 명명 규칙, formatting 규칙 및 좋은 출력의 예를 포함합니다.

5. **테스팅 전략** — 테스트가 실행되는 프레임워크, 적용 범위 기대치, 관심 있는 테스트 수준.

6. **경계** — Three-tier 시스템:
   - **항상 수행:** 커밋 전에 테스트를 실행하고 명명 규칙을 따르고 입력을 검증합니다.
   - **먼저 물어보세요:** 데이터베이스 스키마 변경, 종속성 추가, CI 구성 변경
   - **절대로 하지 마세요:** 비밀 커밋, 공급업체 디렉터리 편집, 승인 없이 실패한 테스트 제거

**사양 템플릿:**

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

**지침을 성공 기준으로 재구성합니다.** 모호한 요청을 받으면 이를 구체적인 조건으로 변환합니다.

```
REQUIREMENT: "Make the dashboard faster"

REFRAMED SUCCESS CRITERIA:
- Dashboard LCP < 2.5s on 4G connection
- Initial data load completes in < 500ms
- No layout shift during load (CLS < 0.1)
→ Are these the right targets?
```

이를 통해 "더 빠르다"는 것이 무엇을 의미하는지 추측하는 대신 명확한 목표를 향해 루프, 재시도 및 problem-solve를 수행할 수 있습니다.

### 2단계: 계획

검증된 사양을 사용하여 기술 구현 계획을 생성합니다.

1. 주요 구성 요소와 해당 종속성을 식별합니다.
2. 구현 순서를 결정합니다(먼저 built가 무엇이어야 함)
3. 위험 및 완화 전략 참고
4. built가 병렬일 수 있는 것과 순차적이어야 하는 것을 식별합니다.
5. 단계 간 검증 체크포인트 정의

계획은 검토 가능해야 합니다. 사람은 이를 읽고 "예, 그게 올바른 접근 방식입니다" 또는 "아니요, X를 변경하세요"라고 말할 수 있어야 합니다.

### 3단계: 작업

계획을 실행 가능한 개별 작업으로 나눕니다.

- 각 작업은 집중된 단일 세션에서 완료할 수 있어야 합니다.
- 각 작업에는 명시적인 승인 기준이 있습니다.
- 각 작업에는 검증 단계(테스트, build, 수동 확인)가 포함됩니다.
- 작업은 인지된 중요도가 아닌 의존성에 따라 정렬됩니다.
- ~5개 이상의 파일을 변경하는 작업이 필요하지 않습니다.

**작업 템플릿:**
```markdown
- [ ] Task: [Description]
  - Acceptance: [What must be true when done]
  - Verify: [How to confirm — test command, build, manual check]
  - Files: [Which files will be touched]
```

### 4단계: 구현

`incremental-implementation` 및 `test-driven-development` skills에 따라 한 번에 하나씩 작업을 실행합니다. 전체 사양으로 agent를 넘치게 만드는 대신 `context-engineering`를 사용하여 각 단계에서 올바른 사양 섹션과 소스 파일을 로드하세요.

## 사양 유지

사양은 one-time 아티팩트가 아닌 살아있는 문서입니다.

- **의사결정이 변경되면 업데이트** — 데이터 모델을 변경해야 하는 경우 먼저 사양을 업데이트한 다음 구현하세요.
- **범위 변경 시 업데이트** — 추가되거나 삭제된 기능은 사양에 반영되어야 합니다.
- **사양 커밋** — 사양은 코드와 함께 버전 제어에 속합니다.
- **PRs의 사양 참조** — 각 PR가 구현하는 사양 섹션으로 다시 연결됩니다.

## 일반적인 합리화

| 합리화 | 현실 |
|---|---|
| "이것은 간단합니다. 사양은 필요하지 않습니다." | 간단한 작업에는 *긴* 사양이 필요하지 않지만 여전히 승인 기준이 필요합니다. two-line 사양은 괜찮습니다. |
| "코딩한 후 스펙을 작성하겠습니다" | 그것은 사양이 아니라 문서입니다. 사양의 가치는 코드 *전에* 명확성을 강제하는 데 있습니다. |
| "스펙으로 인해 속도가 느려질 것입니다" | 15분 사양으로 몇 시간의 재작업이 방지됩니다. 15분 만에 폭포수를 만드는 것이 15시간 만에 디버깅하는 것보다 빠릅니다. |
| "Requirements는 어쨌든 변경됩니다." | 이것이 바로 스펙이 살아있는 문서인 이유입니다. 오래된 사양이라도 사양이 없는 것보다는 낫습니다. |
| "사용자는 자신이 원하는 것이 무엇인지 알고 있습니다." | 명확한 요청에도 암묵적인 가정이 있습니다. 사양은 이러한 가정을 표면화합니다. |

## 위험 신호

- 작성된 requirements 없이 코드 작성 시작
- "그냥 building을 시작해야 할까요?"라고 묻습니다. "완료"가 무엇을 의미하는지 명확히 하기 전에
- 사양이나 작업 목록에 언급되지 않은 기능 구현
- 문서화하지 않고 아키텍처 결정을 내립니다.
- "build가 무엇인지 분명하기 때문에" 사양을 건너뜁니다.

## 확인

구현을 진행하기 전에 다음을 확인하세요.

- [ ] 사양은 6가지 핵심 영역을 모두 포괄합니다.
- [ ] 사람이 사양을 검토하고 승인했습니다.
- [ ] 성공 기준이 구체적이고 테스트 가능함
- [ ] 경계(Always/Ask First/Never)가 정의됩니다.
- [ ] 사양은 repository의 파일에 저장됩니다.
