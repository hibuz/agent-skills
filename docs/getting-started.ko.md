# Getting Started with agent-skills

agent-skills는 Markdown 지침을 수용하는 모든 AI 코딩 Agent와 함께 작동합니다. 이 가이드는 범용적인 접근 방식을 다룹니다. 도구별 설정은 전용 가이드를 참조하세요.

## How Skills Work

각 기술은 특정 엔지니어링 Workflow를 설명하는 Markdown 파일(`SKILL.md`)입니다. Agent의 Context에 로드되면 Agent는 검증 단계, 피해야 할 Anti-pattern, 종료 기준(Exit Criteria)을 포함한 Workflow를 따릅니다.

**기술은 참조 문서가 아닙니다.** Agent가 따르는 단계별 프로세스입니다.

## Quick Start (모든 Agent 공통)

### 1. 리포지토리 클론

```bash
git clone https://github.com/addyosmani/agent-skills.git
```

### 2. 기술 선택

`skills/` 디렉토리를 살펴보세요. 각 하위 디렉토리에는 다음을 포함하는 `SKILL.md`가 있습니다:
- **When to use** — 이 기술이 적용됨을 나타내는 트리거
- **Process** — 단계별 Workflow
- **Verification** — 작업이 완료되었음을 확인하는 방법
- **Common rationalizations** — Agent가 단계를 건너뛰기 위해 사용할 수 있는 변명
- **Red flags** — 기술이 위반되고 있다는 징후

### 3. Agent에 기술 로드

관련 `SKILL.md` 내용을 Agent의 System Prompt, Rule 파일 또는 대화창에 복사하세요. 가장 일반적인 방법은 다음과 같습니다:

**System Prompt:** 세션 시작 시 기술 내용을 붙여넣습니다.

**Rules 파일:** 프로젝트의 Rule 파일(CLAUDE.md, .cursorrules 등)에 기술 내용을 추가합니다.

**Conversation:** 지침을 줄 때 기술을 참조합니다: "이 변경 사항에 대해 test-driven-development 프로세스를 따라줘."

### 4. 탐색을 위해 Meta-skill 사용

`using-agent-skills` 기술이 로드된 상태로 시작하세요. 여기에는 작업 유형을 적절한 기술에 매핑하는 순서도가 포함되어 있습니다.

## Recommended Setup

### Minimal (여기서 시작하세요)

Rule 파일에 다음 세 가지 필수 기술을 로드하세요:

1. **spec-driven-development** — 무엇을 만들지 정의하기 위해
2. **test-driven-development** — 작동함을 증명하기 위해
3. **code-review-and-quality** — Merge 전 품질을 검증하기 위해

이 세 가지는 AI 지원 개발에서 가장 중요한 품질 격차를 해결해 줍니다.

### Full Lifecycle

포괄적인 커버리지를 위해 단계별로 기술을 로드하세요:

```
프로젝트 시작 시:  spec-driven-development → planning-and-task-breakdown
개발 중:          incremental-implementation + test-driven-development
Merge 전:         code-review-and-quality + security-and-hardening
Deploy 전:        shipping-and-launch
```

### Context-Aware Loading

모든 기술을 한꺼번에 로드하지 마세요 — Context 낭비입니다. 현재 작업과 관련된 기술만 로드하세요:

- UI 작업 중? `frontend-ui-engineering` 로드
- 디버깅 중? `debugging-and-error-recovery` 로드
- CI 설정 중? `ci-cd-and-automation` 로드

## Skill Anatomy

모든 기술은 동일한 구조를 따릅니다:

```
YAML frontmatter (name, description)
├── Overview — 이 기술이 하는 일
├── When to Use — 트리거 및 조건
├── Core Process — 단계별 Workflow
├── Examples — 코드 샘플 및 패턴
├── Common Rationalizations — 변명 및 반박
├── Red Flags — 기술 위반 징후
└── Verification — 종료 기준 Checklist
```

전체 사양은 [skill-anatomy.ko.md](skill-anatomy.ko.md)를 참조하세요.

## Using Agents

`agents/` 디렉토리에는 미리 구성된 Agent 페르소나가 포함되어 있습니다:

| Agent | Purpose |
|-------|---------|
| `code-reviewer.md` | 5개 축 코드 리뷰 |
| `test-engineer.md` | 테스트 전략 및 작성 |
| `security-auditor.md` | 취약점 탐지 |

전문화된 리뷰가 필요할 때 Agent 정의를 로드하세요. 예를 들어, 코딩 Agent에게 "code-reviewer Agent 페르소나를 사용하여 이 변경 사항을 리뷰해줘"라고 요청하고 Agent 정의를 제공하세요.

## Using Commands

`.claude/commands/` 디렉토리에는 Claude Code를 위한 슬래시 명령어가 포함되어 있습니다:

| Command | Skill Invoked |
|---------|---------------|
| `/spec` | spec-driven-development |
| `/plan` | planning-and-task-breakdown |
| `/build` | incremental-implementation + test-driven-development |
| `/test` | test-driven-development |
| `/review` | code-review-and-quality |
| `/ship` | shipping-and-launch |

## Using References

`references/` 디렉토리에는 보조 Checklist가 포함되어 있습니다:

| Reference | Use With |
|-----------|----------|
| `testing-patterns.md` | test-driven-development |
| `performance-checklist.md` | performance-optimization |
| `security-checklist.md` | security-and-hardening |
| `accessibility-checklist.md` | frontend-ui-engineering |

기술이 다루는 범위를 넘어서는 상세한 패턴이 필요할 때 Reference를 로드하세요.

## Spec and task artifacts

`/spec` 및 `/plan` 명령어는 작업 Artifact(`SPEC.md`, `tasks/plan.md`, `tasks/todo.md`)를 생성합니다. 작업이 진행되는 동안 이를 **살아있는 문서**로 취급하세요:

- 인간과 Agent가 공유된 진실의 원천(Source of Truth)을 가질 수 있도록 개발 중에 이를 Version Control에 유지하세요.
- 범위나 결정 사항이 변경되면 이를 업데이트하세요.
- 리포지토리에 이 파일들이 장기적으로 남는 것을 원치 않는다면, Merge 전에 삭제하거나 해당 폴더를 `.gitignore`에 추가하세요 — Workflow상 영구적으로 남을 필요는 없습니다.

## Tips

1. 사소하지 않은 모든 작업은 **spec-driven-development로 시작**하세요.
2. 코드를 작성할 때는 **항상 test-driven-development를 로드**하세요.
3. **검증 단계를 건너뛰지 마세요** — 그것이 이 모든 과정의 핵심입니다.
4. **기술을 선택적으로 로드**하세요 — Context가 많다고 항상 좋은 것은 아닙니다.
5. **리뷰를 위해 Agent를 사용**하세요 — 다른 시각이 다른 이슈를 잡아냅니다.
