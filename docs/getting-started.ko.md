# agent-skills 시작하기

agent-skills는 Markdown 지침을 받는 모든 AI 코딩 agent에서 동작합니다. 이 가이드는 보편적인 접근 방식을 다룹니다. 도구별 설정은 전용 가이드를 참고하세요.

## Skill은 어떻게 동작하는가

각 skill은 특정 엔지니어링 워크플로를 기술하는 Markdown 파일(`SKILL.md`)입니다. agent의 context에 로드되면 agent는 그 워크플로를 따릅니다 — 검증 단계, 피해야 할 안티패턴, 종료 기준을 포함합니다.

**skill은 참조 문서가 아닙니다.** agent가 따르는 단계별 프로세스입니다.

## 빠른 시작 (모든 Agent)

### 1. repository clone

```bash
git clone https://github.com/addyosmani/agent-skills.git
```

### 2. skill 선택

`skills/` 디렉터리를 둘러보세요. 각 하위 디렉터리에는 다음을 담은 `SKILL.md`가 있습니다:
- **When to use** — 이 skill이 적용됨을 나타내는 트리거
- **Process** — 단계별 워크플로
- **Verification** — 작업 완료를 확인하는 방법
- **Common rationalizations** — agent가 단계를 건너뛰는 데 쓸 수 있는 핑계
- **Red flags** — skill이 위반되고 있다는 신호

### 3. skill을 agent에 로드

관련 `SKILL.md` 내용을 agent의 시스템 프롬프트, rules 파일, 또는 대화에 복사하세요. 가장 흔한 방식:

**시스템 프롬프트:** 세션 시작 시 skill 내용을 붙여넣습니다.

**rules 파일:** 프로젝트의 rules 파일(CLAUDE.md, .cursorrules 등)에 skill 내용을 추가합니다.

**대화:** 지침을 줄 때 skill을 참조합니다: "이 변경에는 test-driven-development 프로세스를 따르세요."

### 4. 검색에 메타 skill 사용

`using-agent-skills` skill을 로드한 상태로 시작하세요. 작업 유형을 적절한 skill에 매핑하는 플로차트가 들어 있습니다.

## 권장 설정

### 최소 (여기서 시작)

rules 파일에 세 가지 필수 skill을 로드하세요:

1. **spec-driven-development** — 무엇을 만들지 정의하기 위해
2. **test-driven-development** — 동작함을 증명하기 위해
3. **code-review-and-quality** — merge 전 품질을 검증하기 위해

이 세 가지는 AI 보조 개발에서 가장 중요한 품질 공백을 다룹니다.

### 전체 라이프사이클

포괄적인 커버리지를 위해 단계별로 skill을 로드하세요:

```
프로젝트 시작:  spec-driven-development → planning-and-task-breakdown
개발 중:        incremental-implementation + test-driven-development
merge 전:       code-review-and-quality + security-and-hardening
배포 전:        shipping-and-launch
```

### Context 인식 로딩

모든 skill을 한 번에 로드하지 마세요 — context를 낭비합니다. 현재 작업에 관련된 skill을 로드하세요:

- UI 작업 중? `frontend-ui-engineering` 로드
- 디버깅 중? `debugging-and-error-recovery` 로드
- CI 설정 중? `ci-cd-and-automation` 로드

## Skill Anatomy

모든 skill은 동일한 구조를 따릅니다:

```
YAML frontmatter (name, description)
├── Overview — 이 skill이 무엇을 하는가
├── When to Use — 트리거와 조건
├── Core Process — 단계별 워크플로
├── Examples — 코드 샘플과 패턴
├── Common Rationalizations — 핑계와 반박
├── Red Flags — skill이 위반되고 있다는 신호
└── Verification — 종료 기준 체크리스트
```

전체 명세는 [skill-anatomy.ko.md](skill-anatomy.ko.md)를 참고하세요.

## Agent 사용하기

`agents/` 디렉터리에는 사전 구성된 agent persona가 들어 있습니다:

| Agent | 목적 |
|-------|---------|
| `code-reviewer.md` | 5축 코드 리뷰 |
| `test-engineer.md` | 테스트 전략 및 작성 |
| `security-auditor.md` | 취약점 탐지 |

전문 리뷰가 필요할 때 agent 정의를 로드하세요. 예를 들어, 코딩 agent에게 "code-reviewer agent persona로 이 변경을 리뷰하라"고 요청하고 agent 정의를 제공하세요.

## Command 사용하기

`.claude/commands/` 디렉터리에는 Claude Code용 슬래시 커맨드가 들어 있습니다:

| 커맨드 | invoke되는 Skill |
|---------|---------------|
| `/spec` | spec-driven-development |
| `/plan` | planning-and-task-breakdown |
| `/build` | incremental-implementation + test-driven-development |
| `/test` | test-driven-development |
| `/review` | code-review-and-quality |
| `/ship` | shipping-and-launch |

## Reference 사용하기

`references/` 디렉터리에는 보조 체크리스트가 들어 있습니다:

| 참조 | 함께 사용 |
|-----------|----------|
| `testing-patterns.md` | test-driven-development |
| `performance-checklist.md` | performance-optimization |
| `security-checklist.md` | security-and-hardening |
| `accessibility-checklist.md` | frontend-ui-engineering |

skill이 다루는 것 이상의 상세 패턴이 필요할 때 참조를 로드하세요.

## Spec 및 task 산출물

`/spec`과 `/plan` 커맨드는 작업용 산출물(`SPEC.md`, `tasks/plan.md`, `tasks/todo.md`)을 생성합니다. 작업이 진행되는 동안 이것들을 **살아 있는 문서(living document)**로 다루세요:

- 개발 중에는 version control에 두어 사람과 agent가 공유된 진실의 원천을 갖도록 하세요.
- 범위나 결정이 바뀌면 업데이트하세요.
- repo가 이 파일들을 장기적으로 원하지 않는다면, merge 전에 삭제하거나 해당 폴더를 `.gitignore`에 추가하세요 — 이 워크플로는 이 파일들이 영구적일 것을 요구하지 않습니다.

## 팁

1. **비자명한 작업에는 spec-driven-development로 시작**하세요
2. 코드를 작성할 때는 **항상 test-driven-development를 로드**하세요
3. **검증 단계를 건너뛰지 마세요** — 그것이 핵심입니다
4. **skill을 선별적으로 로드**하세요 — context가 많다고 항상 좋은 것은 아닙니다
5. **리뷰에 agent를 사용**하세요 — 다른 관점은 다른 문제를 잡아냅니다
