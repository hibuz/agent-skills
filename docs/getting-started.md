# agent-skills 시작하기

agent-skills는 Markdown instruction을 허용하는 모든 AI 코딩 agent와 함께 작동합니다. 이 guide는 범용 접근 방식을 다룹니다. tool-specific 설정은 전용 guide를 참조하세요.

## Skills 작동 방식

각 skill는 특정 엔지니어링 workflow를 설명하는 Markdown 파일(`SKILL.md`)입니다. agent의 컨텍스트에 로드되면 agent는 ​​확인 단계, 회피할 anti-patterns 및 종료 기준을 포함하여 workflow를 따릅니다.

**Skills는 참조 문서가 아닙니다.** step-by-step는 agent가 따르는 문서를 처리합니다.

## 빠른 시작(모든 Agent)

### 1. repository를 복제합니다.

```bash
git clone https://github.com/addyosmani/agent-skills.git
```

### 2. skill를 선택하세요

`skills/` 디렉터리를 찾아보세요. 각 하위 디렉터리에는 다음과 같은 `SKILL.md`가 포함되어 있습니다.
- **사용 시기** — 이를 나타내는 트리거 skill가 적용됩니다.
- **프로세스** — step-by-step workflow
- **확인** — 작업이 완료되었는지 확인하는 방법
- **일반적인 합리화** — agent가 단계를 건너뛰는 데 사용할 수 있는 변명
- **위험 신호** — skill가 위반되고 있음을 나타냅니다.

### 3. skill를 agent에 로드합니다.

관련 `SKILL.md` 콘텐츠를 agent의 시스템 프롬프트, 규칙 파일 또는 대화에 복사하세요. 가장 일반적인 접근 방식:

**시스템 프롬프트:** 세션 시작 시 skill 콘텐츠를 붙여넣습니다.

**규칙 파일:** 프로젝트의 규칙 파일(CLAUDE.md,.cursorrules 등)에 skill 콘텐츠를 추가합니다.

**대화:** 지침을 제공할 때 skill를 참조하세요. "이 변경에 대해서는 test-driven-development 프로세스를 따르세요."

### 4. 검색을 위해 meta-skill를 사용하세요

로드된 `using-agent-skills` skill로 시작합니다. 여기에는 작업 유형을 적절한 skill에 매핑하는 순서도가 포함되어 있습니다.

## 권장 설정

### 최소(여기서 시작)

세 가지 필수 skills를 규칙 파일에 로드합니다.

1. **spec-driven-development** — 무엇을 build할지 정의
2. **test-driven-development** — 작동 증명
3. **code-review-and-quality** — 병합 전 품질 확인용

이 세 가지는 AI 지원 개발에서 가장 중요한 품질 격차를 해소합니다.

### 전체 수명주기

포괄적인 적용 범위를 위해서는 단계별로 skills를 로드하세요.

```
Starting a project:  spec-driven-development → planning-and-task-breakdown
During development:  incremental-implementation + test-driven-development
Before merge:        code-review-and-quality + security-and-hardening
Before deploy:       shipping-and-launch
```

### 컨텍스트 인식 로딩

모든 skills를 한 번에 로드하지 마십시오. 이는 컨텍스트를 낭비합니다. 현재 작업과 관련된 skills를 로드합니다.

- UI를 작업 중이신가요? `frontend-ui-engineering` 로드
- 디버깅 중이신가요? `debugging-and-error-recovery` 로드
- CI를 설정하시겠습니까? `ci-cd-and-automation` 로드

## Skill 해부학

모든 skill는 동일한 구조를 따릅니다.

```
YAML frontmatter (name, description)
├── Overview — What this skill does
├── When to Use — Triggers and conditions
├── Core Process — Step-by-step workflow
├── Examples — Code samples and patterns
├── Common Rationalizations — Excuses and rebuttals
├── Red Flags — Signs the skill is being violated
└── Verification — Exit criteria checklist
```

전체 사양은 [skill-anatomy.md](skill-anatomy.md)를 참조하세요.

## Agents 사용

`agents/` 디렉터리에는 pre-configured agent 페르소나가 포함되어 있습니다.

| Agent | 목적 |
|-------|---------|
| `code-reviewer.md` | 5축 코드 검토 |
| `test-engineer.md` | 테스트 전략 및 글쓰기 |
| `security-auditor.md` | 취약점 감지 |

전문적인 검토가 필요한 경우 agent 정의를 로드하세요. 예를 들어 agent 코딩에 "code-reviewer agent 페르소나를 사용하여 이 변경 사항을 검토"하고 agent 정의를 제공하도록 요청하세요.

## 명령 사용

`.claude/commands/` 디렉터리에는 Claude Code에 대한 slash commands가 포함되어 있습니다.

| 명령 | Skill 호출 |
|---------|---------------|
| `/spec` | spec-driven-development |
| `/plan` | planning-and-task-breakdown |
| `/build` | incremental-implementation + test-driven-development |
| `/test` | test-driven-development |
| `/review` | code-review-and-quality |
| `/ship` | shipping-and-launch |

## 참조 사용

`references/` 디렉토리에는 보충 체크리스트가 포함되어 있습니다.

| 참고자료 | 함께 사용 |
|-----------|----------|
| `testing-patterns.md` | test-driven-development |
| `performance-checklist.md` | performance-optimization |
| `security-checklist.md` | security-and-hardening |
| `accessibility-checklist.md` | frontend-ui-engineering |

skill에서 다루는 것 이상으로 자세한 패턴이 필요한 경우 참조를 로드하세요.

## 사양 및 작업 아티팩트

`/spec` 및 `/plan` 명령은 작업 아티팩트(`SPEC.md`, `tasks/plan.md`, `tasks/todo.md`)를 생성합니다. 작업이 진행되는 동안 이를 **살아있는 문서**로 취급하십시오.

- 개발 중에 버전 제어를 유지하여 인간과 agent가 공유 소스를 갖도록 합니다.
- 범위나 결정이 변경되면 업데이트하세요.
- repo가 이러한 파일을 장기간 원하지 않는 경우 병합하기 전에 해당 파일을 삭제하거나 폴더를 `.gitignore`에 추가하세요. workflow는 해당 파일을 영구적으로 유지하도록 요구하지 않습니다.

## 팁

1. non-trivial 작업에 대해 **spec-driven-development로 시작**
2. **코드 작성 시 항상 test-driven-development를 로드**하세요.
3. **확인 단계를 건너뛰지 마세요** — 확인 단계가 핵심입니다.
4. **skills를 선택적으로 로드** — 더 많은 컨텍스트가 항상 더 나은 것은 아닙니다.
5. **검토를 위해 agents를 사용하세요** — 서로 다른 관점은 서로 다른 문제를 포착합니다
