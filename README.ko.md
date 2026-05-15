# Agent Skills

**AI 코딩 agent를 위한 production 수준 엔지니어링 skill.**

skill은 senior 엔지니어가 소프트웨어를 만들 때 사용하는 워크플로, 품질 게이트, 모범 사례를 인코딩합니다. 이 skill들은 AI agent가 개발의 모든 단계에서 일관되게 따르도록 패키징되어 있습니다.

![Addy's Agent Skills](https://addyosmani.com/assets/images/addys-agent-skills.jpg)

---

## 커맨드

개발 라이프사이클에 매핑되는 7개의 슬래시 커맨드. 각 커맨드는 적절한 skill을 자동으로 활성화합니다.

| 무엇을 하는지 | 커맨드 | 핵심 원칙 |
|-------------------|---------|---------------|
| 무엇을 만들지 정의 | `/spec` | 코드보다 spec 먼저 |
| 어떻게 만들지 계획 | `/plan` | 작고 원자적인 작업 |
| 점진적으로 빌드 | `/build` | 한 번에 하나의 슬라이스 |
| 동작함을 증명 | `/test` | 테스트가 곧 증거 |
| merge 전 리뷰 | `/review` | 코드 건강성 향상 |
| 코드 단순화 | `/code-simplify` | 영리함보다 명료함 |
| production으로 ship | `/ship` | 빠른 것이 더 안전하다 |

skill은 무엇을 하느냐에 따라 자동으로 활성화되기도 합니다 — API를 설계하면 `api-and-interface-design`이, UI를 빌드하면 `frontend-ui-engineering`이 트리거되는 식입니다.

---

## 빠른 시작

<details>
<summary><b>Claude Code (권장)</b></summary>

**Marketplace 설치:**

```
/plugin marketplace add addyosmani/agent-skills
/plugin install agent-skills@addy-agent-skills
```

> **SSH 오류가 나나요?** marketplace는 repo를 SSH로 clone합니다. GitHub에 SSH 키가 설정되어 있지 않다면, [SSH 키를 추가](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)하거나 전체 HTTPS URL을 사용해 HTTPS clone을 강제하세요:
> ```bash
> /plugin marketplace add https://github.com/addyosmani/agent-skills.git
> /plugin install agent-skills@addy-agent-skills
> ```

**로컬 / 개발:**

```bash
git clone https://github.com/addyosmani/agent-skills.git
claude --plugin-dir /path/to/agent-skills
```

</details>

<details>
<summary><b>Cursor</b></summary>

아무 `SKILL.md`나 `.cursor/rules/`로 복사하거나 전체 `skills/` 디렉터리를 참조하세요. [docs/cursor-setup.ko.md](docs/cursor-setup.ko.md)를 참고하세요.

</details>

<details>
<summary><b>Gemini CLI</b></summary>

자동 검색을 위해 네이티브 skill로 설치하거나, 지속적인 context를 위해 `GEMINI.md`에 추가하세요. [docs/gemini-cli-setup.ko.md](docs/gemini-cli-setup.ko.md)를 참고하세요.

**repo에서 설치:**

```bash
gemini skills install https://github.com/addyosmani/agent-skills.git --path skills
```

**로컬 clone에서 설치:**

```bash
gemini skills install ./agent-skills/skills/
```

</details>

<details>
<summary><b>Windsurf</b></summary>

skill 내용을 Windsurf rules 설정에 추가하세요. [docs/windsurf-setup.ko.md](docs/windsurf-setup.ko.md)를 참고하세요.

</details>

<details>
<summary><b>OpenCode</b></summary>

AGENTS.md와 `skill` 도구를 통한 agent 주도 skill 실행을 사용합니다.

[docs/opencode-setup.ko.md](docs/opencode-setup.ko.md)를 참고하세요.

</details>

<details>
<summary><b>GitHub Copilot</b></summary>

`agents/`의 agent 정의를 Copilot persona로, skill 내용을 `.github/copilot-instructions.md`에 사용하세요. [docs/copilot-setup.ko.md](docs/copilot-setup.ko.md)를 참고하세요.

</details>

<details>
  <summary><b>Kiro IDE & CLI </b></summary>
  Kiro용 skill은 ".kiro/skills/" 아래에 위치하며 Project 또는 Global 레벨로 저장할 수 있습니다. Kiro는 Agents.md도 지원합니다. Kiro 문서는 https://kiro.dev/docs/skills/ 를 참고하세요
</details>

<details>
<summary><b>Codex / 기타 Agent</b></summary>

skill은 일반 Markdown입니다 - 시스템 프롬프트나 지침 파일을 받는 모든 agent에서 동작합니다. [docs/getting-started.ko.md](docs/getting-started.ko.md)를 참고하세요.

</details>



---

## 전체 23개 Skill

위 커맨드는 진입점입니다. 이 팩에는 총 23개의 skill이 포함됩니다 — 22개의 라이프사이클 skill과 `using-agent-skills` 메타 skill입니다. 각 skill은 단계, 검증 게이트, 합리화 방지 표를 갖춘 구조화된 워크플로입니다. 어떤 skill이든 직접 참조할 수도 있습니다.

### Meta - 어떤 skill을 적용할지 찾기

| Skill | 무엇을 하는가 | 언제 사용하는가 |
|-------|-------------|----------|
| [using-agent-skills](skills/using-agent-skills/SKILL.ko.md) | 들어오는 작업을 올바른 skill 워크플로에 매핑하고 공유 운영 규칙을 정의 | 세션을 시작하거나 어떤 skill을 적용할지 결정할 때 |

### Define - 무엇을 만들지 명확히 하기

| Skill | 무엇을 하는가 | 언제 사용하는가 |
|-------|-------------|----------|
| [interview-me](skills/interview-me/SKILL.ko.md) | 한 번에 한 질문씩 진행하는 인터뷰로, 사용자가 해야 한다고 생각하는 것이 아니라 실제로 원하는 것을 ~95% 확신에 이를 때까지 끌어냄 | 요청이 충분히 명세되지 않았거나, 사용자가 "interview me" / "grill me"를 호출할 때 |
| [idea-refine](skills/idea-refine/SKILL.ko.md) | 막연한 아이디어를 구체적 제안으로 바꾸는 구조화된 발산/수렴 사고 | 탐색이 필요한 거친 개념이 있을 때 |
| [spec-driven-development](skills/spec-driven-development/SKILL.ko.md) | 코드 작성 전에 목표, 커맨드, 구조, 코드 스타일, 테스트, 경계를 다루는 PRD 작성 | 새 프로젝트, 기능, 또는 중대한 변경을 시작할 때 |

### Plan - 잘게 나누기

| Skill | 무엇을 하는가 | 언제 사용하는가 |
|-------|-------------|----------|
| [planning-and-task-breakdown](skills/planning-and-task-breakdown/SKILL.ko.md) | spec을 수용 기준과 의존성 순서를 갖춘 작고 검증 가능한 작업으로 분해 | spec이 있고 구현 가능한 단위가 필요할 때 |

### Build - 코드 작성하기

| Skill | 무엇을 하는가 | 언제 사용하는가 |
|-------|-------------|----------|
| [incremental-implementation](skills/incremental-implementation/SKILL.ko.md) | 얇은 수직 슬라이스 - 구현, 테스트, 검증, commit. feature flag, 안전한 기본값, rollback 친화적 변경 | 두 개 이상의 파일을 건드리는 모든 변경 |
| [test-driven-development](skills/test-driven-development/SKILL.ko.md) | Red-Green-Refactor, 테스트 피라미드(80/15/5), 테스트 크기, DRY보다 DAMP, Beyonce Rule, browser 테스트 | 로직 구현, 버그 수정, 동작 변경 시 |
| [context-engineering](skills/context-engineering/SKILL.ko.md) | agent에게 올바른 정보를 올바른 시점에 제공 - rules 파일, context packing, MCP 연동 | 세션 시작, 작업 전환, 또는 출력 품질이 떨어질 때 |
| [source-driven-development](skills/source-driven-development/SKILL.ko.md) | 모든 프레임워크 결정을 공식 문서에 근거 - 검증, 출처 인용, 미검증 항목 표시 | 어떤 프레임워크나 라이브러리든 출처가 인용된 권위 있는 코드를 원할 때 |
| [doubt-driven-development](skills/doubt-driven-development/SKILL.ko.md) | 진행 중인 모든 비자명한 결정에 대한 적대적 신규-context 리뷰 - CLAIM → EXTRACT → DOUBT → RECONCILE → STOP, 선택적으로 사용자 승인 하의 교차 모델 escalation 포함 | 위험이 높거나(production, 보안, 비가역), 익숙하지 않은 코드에서 작업하거나, 확신에 찬 출력을 나중에 디버깅하는 것보다 지금 검증하는 것이 더 저렴할 때 |
| [frontend-ui-engineering](skills/frontend-ui-engineering/SKILL.ko.md) | 컴포넌트 아키텍처, 디자인 시스템, 상태 관리, 반응형 디자인, WCAG 2.1 AA 접근성 | 사용자 대면 인터페이스를 빌드하거나 수정할 때 |
| [api-and-interface-design](skills/api-and-interface-design/SKILL.ko.md) | contract-first 설계, Hyrum's Law, One-Version Rule, 에러 시맨틱, 경계 검증 | API, 모듈 경계, 또는 공개 인터페이스를 설계할 때 |

### Verify - 동작함을 증명하기

| Skill | 무엇을 하는가 | 언제 사용하는가 |
|-------|-------------|----------|
| [browser-testing-with-devtools](skills/browser-testing-with-devtools/SKILL.ko.md) | 라이브 런타임 데이터를 위한 Chrome DevTools MCP - DOM 검사, console 로그, network trace, performance profiling | browser에서 실행되는 무언가를 빌드하거나 디버깅할 때 |
| [debugging-and-error-recovery](skills/debugging-and-error-recovery/SKILL.ko.md) | 5단계 triage: 재현, 위치 파악, 축소, 수정, 가드. stop-the-line 규칙, 안전한 fallback | 테스트가 실패하거나, 빌드가 깨지거나, 동작이 예기치 않을 때 |

### Review - merge 전 품질 게이트

| Skill | 무엇을 하는가 | 언제 사용하는가 |
|-------|-------------|----------|
| [code-review-and-quality](skills/code-review-and-quality/SKILL.ko.md) | 5축 리뷰, 변경 크기 조정(~100줄), 심각도 레이블(Nit/Optional/FYI), 리뷰 속도 규범, 분할 전략 | 모든 변경을 merge하기 전에 |
| [code-simplification](skills/code-simplification/SKILL.ko.md) | Chesterton's Fence, Rule of 500, 정확한 동작을 보존하면서 복잡도 감소 | 코드는 동작하지만 필요 이상으로 읽거나 유지보수하기 어려울 때 |
| [security-and-hardening](skills/security-and-hardening/SKILL.ko.md) | OWASP Top 10 예방, 인증 패턴, secrets 관리, 의존성 감사, 3계층 경계 시스템 | 사용자 입력, 인증, 데이터 저장, 또는 외부 연동을 다룰 때 |
| [performance-optimization](skills/performance-optimization/SKILL.ko.md) | 측정 우선 접근 - Core Web Vitals 목표, profiling 워크플로, 번들 분석, 안티패턴 탐지 | 성능 요구사항이 있거나 회귀가 의심될 때 |

### Ship - 자신 있게 배포하기

| Skill | 무엇을 하는가 | 언제 사용하는가 |
|-------|-------------|----------|
| [git-workflow-and-versioning](skills/git-workflow-and-versioning/SKILL.ko.md) | trunk-based development, 원자적 commit, 변경 크기 조정(~100줄), commit-as-save-point 패턴 | 모든 코드 변경 시 (항상) |
| [ci-cd-and-automation](skills/ci-cd-and-automation/SKILL.ko.md) | Shift Left, Faster is Safer, feature flag, 품질 게이트 파이프라인, 실패 피드백 루프 | 빌드 및 배포 파이프라인을 구성하거나 수정할 때 |
| [deprecation-and-migration](skills/deprecation-and-migration/SKILL.ko.md) | 코드를 부채로 보는 사고방식, 강제 vs 권고 deprecation, migration 패턴, 좀비 코드 제거 | 오래된 시스템 제거, 사용자 migration, 또는 기능 종료 시 |
| [documentation-and-adrs](skills/documentation-and-adrs/SKILL.ko.md) | Architecture Decision Record, API 문서, 인라인 문서 표준 - *왜*를 문서화 | 아키텍처 결정, API 변경, 또는 기능 ship 시 |
| [shipping-and-launch](skills/shipping-and-launch/SKILL.ko.md) | 런칭 전 체크리스트, feature flag 라이프사이클, 단계적 rollout, rollback 절차, 모니터링 설정 | production 배포를 준비할 때 |

---

## Agent Persona

타겟화된 리뷰를 위한 사전 구성된 전문 persona:

| Agent | 역할 | 관점 |
|-------|------|-------------|
| [code-reviewer](agents/code-reviewer.ko.md) | Senior Staff Engineer | "staff 엔지니어가 이걸 승인할까?" 기준의 5축 코드 리뷰 |
| [test-engineer](agents/test-engineer.ko.md) | QA Specialist | 테스트 전략, 커버리지 분석, Prove-It 패턴 |
| [security-auditor](agents/security-auditor.ko.md) | Security Engineer | 취약점 탐지, 위협 모델링, OWASP 평가 |

---

## 참조 체크리스트

skill이 필요할 때 끌어오는 빠른 참조 자료:

| 참조 | 다루는 내용 |
|-----------|--------|
| [testing-patterns.ko.md](references/testing-patterns.ko.md) | 테스트 구조, 명명, mocking, React/API/E2E 예시, 안티패턴 |
| [security-checklist.ko.md](references/security-checklist.ko.md) | pre-commit 검사, 인증, 입력 검증, 헤더, CORS, OWASP Top 10 |
| [performance-checklist.ko.md](references/performance-checklist.ko.md) | Core Web Vitals 목표, 프론트엔드/백엔드 체크리스트, 측정 커맨드 |
| [accessibility-checklist.ko.md](references/accessibility-checklist.ko.md) | 키보드 내비게이션, 스크린 리더, 시각 디자인, ARIA, 테스트 도구 |

---

## Skill은 어떻게 동작하는가

모든 skill은 일관된 anatomy를 따릅니다:

```
┌─────────────────────────────────────────────────┐
│  SKILL.md                                       │
│                                                 │
│  ┌─ Frontmatter ─────────────────────────────┐  │
│  │ name: lowercase-hyphen-name               │  │
│  │ description: Guides agents through [task].│  │
│  │              Use when…                    │  │
│  └───────────────────────────────────────────┘  │                                                                                                
│  Overview         → 이 skill이 무엇을 하는가     │
│  When to Use      → 트리거 조건                  │
│  Process          → 단계별 워크플로              │
│  Rationalizations → 핑계 + 반박                  │
│  Red Flags        → 무언가 잘못됐다는 신호       │
│  Verification     → 증거 요구사항                │
└─────────────────────────────────────────────────┘
```

**핵심 설계 선택:**

- **산문이 아니라 프로세스.** skill은 agent가 읽는 참조 문서가 아니라 따르는 워크플로입니다. 각각 단계, 체크포인트, 종료 기준을 가집니다.
- **합리화 방지.** 모든 skill은 agent가 단계를 건너뛰는 데 쓰는 흔한 핑계(예: "테스트는 나중에 추가할게")와 문서화된 반론 표를 포함합니다.
- **검증은 타협 불가.** 모든 skill은 증거 요구사항으로 끝납니다 - 테스트 통과, 빌드 출력, 런타임 데이터. "맞는 것 같다"는 결코 충분하지 않습니다.
- **점진적 공개(Progressive disclosure).** `SKILL.md`가 진입점입니다. 보조 참조는 필요할 때만 로드되어 토큰 사용을 최소화합니다.

---

## 프로젝트 구조

```
agent-skills/
├── skills/                            # 23개 skill (22개 라이프사이클 + 1개 메타)
│   ├── interview-me/                  #   Define
│   ├── idea-refine/                   #   Define
│   ├── spec-driven-development/       #   Define
│   ├── planning-and-task-breakdown/   #   Plan
│   ├── incremental-implementation/    #   Build
│   ├── context-engineering/           #   Build
│   ├── source-driven-development/     #   Build
│   ├── doubt-driven-development/      #   Build
│   ├── frontend-ui-engineering/       #   Build
│   ├── test-driven-development/       #   Build
│   ├── api-and-interface-design/      #   Build
│   ├── browser-testing-with-devtools/ #   Verify
│   ├── debugging-and-error-recovery/  #   Verify
│   ├── code-review-and-quality/       #   Review
│   ├── code-simplification/          #   Review
│   ├── security-and-hardening/        #   Review
│   ├── performance-optimization/      #   Review
│   ├── git-workflow-and-versioning/   #   Ship
│   ├── ci-cd-and-automation/          #   Ship
│   ├── deprecation-and-migration/     #   Ship
│   ├── documentation-and-adrs/        #   Ship
│   ├── shipping-and-launch/           #   Ship
│   └── using-agent-skills/            #   Meta: 이 팩 사용법
├── agents/                            # 3개 전문 persona
├── references/                        # 4개 보조 체크리스트
├── hooks/                             # 세션 라이프사이클 hook
├── .claude/commands/                  # 7개 슬래시 커맨드 (Claude Code)
├── .gemini/commands/                  # 7개 슬래시 커맨드 (Gemini CLI)
└── docs/                              # 도구별 설정 가이드
```

---

## 왜 Agent Skills인가?

AI 코딩 agent는 기본적으로 가장 짧은 경로를 택합니다 - 이는 종종 spec, 테스트, 보안 리뷰, 그리고 소프트웨어를 신뢰할 수 있게 만드는 관행을 건너뛰는 것을 의미합니다. Agent Skills는 senior 엔지니어가 production 코드에 적용하는 것과 동일한 규율을 강제하는 구조화된 워크플로를 agent에게 제공합니다.

각 skill은 힘들게 얻은 엔지니어링 판단을 인코딩합니다: 언제 spec을 쓸지, 무엇을 테스트할지, 어떻게 리뷰할지, 언제 ship할지. 이것들은 일반적인 프롬프트가 아닙니다 - production 품질 작업을 프로토타입 품질 작업과 구분 짓는, 의견이 분명하고 프로세스 주도적인 워크플로입니다.

skill은 Google의 엔지니어링 문화에서 나온 모범 사례를 녹여냅니다 — [Software Engineering at Google](https://abseil.io/resources/swe-book)의 개념과 Google의 [엔지니어링 관행 가이드](https://google.github.io/eng-practices/)를 포함합니다. API 설계의 Hyrum's Law, 테스트의 Beyonce Rule과 테스트 피라미드, 코드 리뷰의 변경 크기 조정과 리뷰 속도 규범, 단순화의 Chesterton's Fence, git 워크플로의 trunk-based development, CI/CD의 Shift Left와 feature flag, 그리고 코드를 부채로 다루는 전용 deprecation skill을 찾을 수 있습니다. 이것들은 추상적 원칙이 아닙니다 — agent가 따르는 단계별 워크플로에 직접 내장되어 있습니다.

---

## 기여하기

skill은 **구체적**(막연한 조언이 아니라 실행 가능한 단계)이고, **검증 가능**(증거 요구사항을 갖춘 명확한 종료 기준)하며, **실전 검증됨**(실제 워크플로 기반)이고, **최소한**(agent를 안내하는 데 필요한 것만)이어야 합니다.

형식 명세는 [docs/skill-anatomy.ko.md](docs/skill-anatomy.ko.md)를, 가이드라인은 [CONTRIBUTING.ko.md](CONTRIBUTING.ko.md)를 참고하세요.

---

## 라이선스

MIT - 여러분의 프로젝트, 팀, 도구에서 이 skill을 사용하세요.
