# Agent Skills

**AI 코딩 agents를 위한 production-grade engineering skills.**

Skills는 senior engineer가 software를 build할 때 사용하는 workflows, quality gates, best practices를 담고 있습니다. 이 저장소는 AI agents가 개발의 각 단계에서 이를 일관되게 따르도록 패키지되어 있습니다.

```
  DEFINE          PLAN           BUILD          VERIFY         REVIEW          SHIP
 ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐
 │ Idea │ ───▶ │ Spec │ ───▶ │ Code │ ───▶ │ Test │ ───▶ │  QA  │ ───▶ │  Go  │
 │Refine│      │  PRD │      │ Impl │      │Debug │      │ Gate │      │ Live │
 └──────┘      └──────┘      └──────┘      └──────┘      └──────┘      └──────┘
  /spec          /plan          /build        /test         /review       /ship
```

---

## 명령

개발 라이프사이클에 매핑되는 7 slash commands. 각각은 오른쪽 skills를 자동으로 활성화합니다.

| 지금 무엇을 하고 있나요 | 명령 | 핵심원리 |
|-------------------|---------|---------------|
| 무엇을 build할지 정의 | `/spec` | 코드 전 사양 |
| 어떻게 build할지 계획 | `/plan` | 소규모, 원자적 작업 |
| 점진적으로 build | `/build` | 한 번에 한 조각 |
| 작동 증명 | `/test` | 테스트는 증거입니다 |
| 병합 전 검토 | `/review` | 코드 상태 개선 |
| 코드 단순화 | `/code-simplify` | 영리함보다 명확함 |
| 생산으로 배송 | `/ship` | 빠를수록 안전합니다 |

Skills는 작업 유형에 따라 자동으로도 활성화됩니다. API를 설계하면 `api-and-interface-design`가 트리거되고, UI를 build하면 `frontend-ui-engineering`가 트리거되는 식입니다.

---

## 빠른 시작

<details>
<summary><b>Claude Code(권장)</b></summary>

**Marketplace 설치:**

```
/plugin marketplace add addyosmani/agent-skills
/plugin install agent-skills@addy-agent-skills
```

> **SSH 오류?** Marketplace는 SSH를 통해 repos를 복제합니다. GitHub에 SSH 키가 설정되어 있지 않은 경우 [add your SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account) 또는 가져오기 전용 HTTPS로 전환하세요.
> ```bash
> git config --global url."https://github.com/".insteadOf "git@github.com:"
> ```

**로컬/개발:**

```bash
git clone https://github.com/addyosmani/agent-skills.git
claude --plugin-dir /path/to/agent-skills
```

</details>

<details>
<summary><b>Cursor</b></summary>

`SKILL.md`를 `.cursor/rules/`에 복사하거나 전체 `skills/` 디렉터리를 참조하세요. [docs/cursor-setup.md](docs/cursor-setup.md)를 참조하세요.

</details>

<details>
<summary><b>Gemini CLI</b></summary>

auto-discovery의 경우 기본 skills로 설치하거나 영구 컨텍스트의 경우 `GEMINI.md`에 추가하세요. [docs/gemini-cli-setup.md](docs/gemini-cli-setup.md)를 참조하세요.

**repo에서 설치:**

```bash
gemini skills install https://github.com/addyosmani/agent-skills.git --path skills
```

**로컬 클론에서 설치:**

```bash
gemini skills install ./agent-skills/skills/
```

</details>

<details>
<summary><b>Windsurf</b></summary>

Windsurf 규칙 구성에 skill 콘텐츠를 추가합니다. [docs/windsurf-setup.md](docs/windsurf-setup.md)를 참조하세요.

</details>

<details>
<summary><b>OpenCode</b></summary>

AGENTS.md 및 `skill` 도구를 통해 agent-driven skill 실행을 사용합니다.

[docs/opencode-setup.md](docs/opencode-setup.md)를 참조하세요.

</details>

<details>
<summary><b>GitHub Copilot</b></summary>

`agents/`의 agent 정의를 Copilot 페르소나로 사용하고 `.github/copilot-instructions.md`의 skill 콘텐츠를 사용하세요. [docs/copilot-setup.md](docs/copilot-setup.md)를 참조하세요.

</details>

<details>
<summary><b>Codex / 기타 Agents</b></summary>

Skills는 일반 Markdown입니다. 시스템 프롬프트나 지침 파일을 허용하는 모든 agent와 함께 작동합니다. [docs/getting-started.md](docs/getting-started.md)를 참조하세요.

</details>

---

## 전체 20개 Skills

위의 명령은 진입점입니다. 내부적으로는 20개의 skills를 활성화합니다. 각각은 단계, 검증 게이트 및 anti-rationalization 테이블이 포함된 구조화된 workflow입니다. skill를 직접 참조할 수도 있습니다.

### 정의 - build에 대한 내용을 명확히 합니다.

| Skill | 그것이 하는 일 | 사용 시기 |
|-------|-------------|----------|
| [idea-refine](skills/idea-refine/SKILL.md) | 모호한 아이디어를 구체적인 제안으로 전환하는 구조화된 발산적 /convergent 사고 | 탐구가 필요한 대략적인 개념이 있습니다 |
| [spec-driven-development](skills/spec-driven-development/SKILL.md) | 코드 앞에 목표, 명령, 구조, 코드 스타일, 테스트 및 경계를 다루는 PRD 작성 | 새 프로젝트, 기능 또는 중요한 변경 시작 |

### 계획 - 세분화

| Skill | 그것이 하는 일 | 사용 시기 |
|-------|-------------|----------|
| [planning-and-task-breakdown](skills/planning-and-task-breakdown/SKILL.md) | 수용 기준 및 종속성 순서를 사용하여 사양을 작고 검증 가능한 작업으로 분해 | 사양이 있고 구현 가능한 단위가 필요합니다 |

### Build - 코드 작성

| Skill | 그것이 하는 일 | 사용 시기 |
|-------|-------------|----------|
| [incremental-implementation](skills/incremental-implementation/SKILL.md) | 얇은 수직 조각 - 구현, 테스트, 확인, 커밋. Feature flags, 안전한 기본값, rollback-friendly 변경 | 두 개 이상의 파일을 수정하는 경우 |
| [test-driven-development](skills/test-driven-development/SKILL.md) | Red-Green-Refactor, 테스트 피라미드(80/15/5), 테스트 크기, DRY, Beyonce Rule에 대한 DAMP, 브라우저 테스트 | 논리 구현, 버그 수정 또는 동작 변경 |
| [context-engineering](skills/context-engineering/SKILL.md) | 적절한 시점에 agents에 올바른 정보를 공급 - 규칙 파일, context packing, MCP 통합 | 세션 시작, 작업 전환 또는 출력 품질이 저하되는 경우 |
| [source-driven-development](skills/source-driven-development/SKILL.md) | 모든 프레임워크 결정을 공식 문서에 근거 - 확인, 출처 인용, 확인되지 않은 항목 표시 | 모든 프레임워크나 라이브러리에 대해 신뢰할 수 있는 source-cited 코드를 원합니다 |
| [frontend-ui-engineering](skills/frontend-ui-engineering/SKILL.md) | 컴포넌트 아키텍처, 디자인 시스템, 상태 관리, 반응형 디자인, WCAG 2.1 AA 접근성 | user-facing 인터페이스를 build하거나 수정할 때 |
| [api-and-interface-design](skills/api-and-interface-design/SKILL.md) | 계약 우선 설계, Hyrum's Law, One-Version Rule, 오류 의미, 경계 검증 | APIs, 모듈 경계 또는 공용 인터페이스 설계 |

### 확인 - 작동하는지 증명

| Skill | 그것이 하는 일 | 사용 시기 |
|-------|-------------|----------|
| [browser-testing-with-devtools](skills/browser-testing-with-devtools/SKILL.md) | 실시간 런타임 데이터용 Chrome DevTools MCP - DOM 검사, 콘솔 로그, 네트워크 추적, performance 프로파일링 | Building 또는 브라우저에서 실행되는 항목 디버깅 |
| [debugging-and-error-recovery](skills/debugging-and-error-recovery/SKILL.md) | 5단계 분류: 재현, 위치화, 축소, 수정, 보호. Stop-the-line 규칙, 안전한 fallback | 테스트 실패, build 중단 또는 예상치 못한 동작 |

### 검토 - 병합 전 품질 게이트

| Skill | 그것이 하는 일 | 사용 시기 |
|-------|-------------|----------|
| [code-review-and-quality](skills/code-review-and-quality/SKILL.md) | 5축 review, change sizing(~100줄), severity label(Nit/Optional/FYI), review 속도 기준, 분할 전략 | 변경 사항을 병합하기 전에 |
| [code-simplification](skills/code-simplification/SKILL.md) | Chesterton's Fence, Rule of 500, 정확한 동작을 유지하면서 복잡성을 줄임 | 코드는 작동하지만 예상보다 읽거나 유지 관리하기가 더 어렵습니다. |
| [security-and-hardening](skills/security-and-hardening/SKILL.md) | OWASP 상위 10개 예방, 인증 패턴, 비밀 관리, 종속성 감사, three-tier 경계 시스템 | 사용자 입력, 인증, 데이터 저장 또는 외부 통합 처리 |
| [performance-optimization](skills/performance-optimization/SKILL.md) | 측정 우선 접근 방식 - Core Web Vitals 목표, profiling workflow, 번들 분석, anti-pattern 감지 | performance requirements가 있거나 regression이 의심될 때 |

### 배송 - 자신 있게 배포

| Skill | 그것이 하는 일 | 사용 시기 |
|-------|-------------|----------|
| [git-workflow-and-versioning](skills/git-workflow-and-versioning/SKILL.md) | Trunk-based development, 원자성 커밋, 크기 변경(~100줄), commit-as-save-point 패턴 | 코드 변경(항상) |
| [ci-cd-and-automation](skills/ci-cd-and-automation/SKILL.md) | Shift Left, 빠를수록 안전함, feature flags, 품질 게이트 파이프라인, 오류 피드백 루프 | build 설정 또는 수정 및 파이프라인 배포 |
| [deprecation-and-migration](skills/deprecation-and-migration/SKILL.md) | Code-as-liability 사고방식, 필수 vs 권고 지원 중단, 마이그레이션 패턴, 좀비 코드 제거 | 오래된 시스템 제거, 사용자 마이그레이션 또는 기능 종료 |
| [documentation-and-adrs](skills/documentation-and-adrs/SKILL.md) | Architecture Decision Records, API 문서, 인라인 문서 표준ards - *이유* 문서화 | 아키텍처 결정, APIs 변경 또는 기능 출시 |
| [shipping-and-launch](skills/shipping-and-launch/SKILL.md) | 출시 전 체크리스트, feature flag 수명 주기, 단계적 출시, rollback 절차, 모니터링 설정 | 프로덕션 배포 준비 |

---

## Agent 페르소나

타겟 검토를 위해 사전 구성된 전문가 페르소나:

| Agent | 역할 | 관점 |
|-------|------|-------------|
| [code-reviewer](agents/code-reviewer.md) | 수석 직원 엔지니어 | "직원 엔지니어가 이것을 승인할까요?"라는 5축 코드 검토 표준 |
| [test-engineer](agents/test-engineer.md) | QA 전문가 | 테스트 전략, 커버리지 분석 및 Prove-It 패턴 |
| [security-auditor](agents/security-auditor.md) | 보안 엔지니어 | 취약점 탐지, 위협 모델링, OWASP 평가 |

---

## 참조 체크리스트

필요할 때 skills가 가져오는 Quick 참조 자료:

| 참고자료 | 커버 |
|-----------|--------|
| [testing-patterns.md](references/testing-patterns.md) | 테스트 구조, 이름 지정, 조롱, React/API/E2E 예제, anti-patterns |
| [security-checklist.md](references/security-checklist.md) | 커밋 전 검사, 인증, 입력 유효성 검사, 헤더, CORS, OWASP 상위 10 |
| [performance-checklist.md](references/performance-checklist.md) | Core Web Vitals 대상, 프런트엔드/backend 체크리스트, 측정 명령 |
| [accessibility-checklist.md](references/accessibility-checklist.md) | 키보드 탐색, 스크린 리더, 시각 디자인, ARIA, 테스트 도구 |

---

## Skills 작동 방식

모든 skill는 일관된 구조를 따릅니다.

```
┌─────────────────────────────────────────────┐
│  SKILL.md                                   │
│                                             │
│  ┌─ Frontmatter ─────────────────────────┐  │
│  │ name: lowercase-hyphen-name           │  │
│  │ description: Use when [trigger]       │  │
│  └───────────────────────────────────────┘  │
│                                             │
│  Overview         → What this skill does    │
│  When to Use      → Triggering conditions   │
│  Process          → Step-by-step workflow   │
│  Rationalizations → Excuses + rebuttals     │
│  Red Flags        → Signs something's wrong │
│  Verification     → Evidence requirements   │
└─────────────────────────────────────────────┘
```

**주요 디자인 선택:**

- **산문이 아닌 프로세스.** Skills는 workflows agents를 따르며 읽은 참조 문서가 아닙니다. 각각에는 단계, 체크포인트 및 종료 기준이 있습니다.
- **반합리화.** 모든 skill에는 문서화된 counter-arguments와 함께 단계를 건너뛰기 위해 agents가 사용하는 일반적인 변명(예: "나중에 테스트를 추가하겠습니다") 표가 포함되어 있습니다.
- **검증은 non-negotiable입니다.** 모든 skill는 증거 requirements로 끝납니다 - 테스트 통과, build 출력, 런타임 데이터. "맞아 보인다"는 말만으로는 충분하지 않습니다.
- **점진적 공개.** `SKILL.md`가 진입점입니다. 필요한 경우에만 참조 로드를 지원하여 토큰 사용량을 최소화합니다.

---

## 프로젝트 구조

```
agent-skills/
├── skills/                            # 20 core skills (SKILL.md per directory)
│   ├── idea-refine/                   #   Define
│   ├── spec-driven-development/       #   Define
│   ├── planning-and-task-breakdown/   #   Plan
│   ├── incremental-implementation/    #   Build
│   ├── context-engineering/           #   Build
│   ├── source-driven-development/     #   Build
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
│   └── using-agent-skills/            #   Meta: how to use this pack
├── agents/                            # 3 specialist personas
├── references/                        # 4 supplementary checklists
├── hooks/                             # Session lifecycle hooks
├── .claude/commands/                  # 7 slash commands
└── docs/                              # Setup guides per tool
```

---

## 왜 Agent Skills인가요?

AI 코딩 agents는 기본적으로 최단 경로를 사용합니다. 이는 종종 소프트웨어를 안정적으로 만드는 사양, 테스트, 보안 검토 및 관행을 건너뛰는 것을 의미합니다. Agent Skills는 수석 엔지니어가 프로덕션 코드에 적용하는 것과 동일한 원칙을 적용하는 구조화된 agents workflows를 제공합니다.

각 skill는 hard-won 엔지니어링 판단(사양 작성 시기*, 테스트 대상*, 검토 방법*, 출시 시기*)을 인코딩합니다. 이는 일반적인 프롬프트가 아닙니다. production-quality 작업과 prototype-quality 작업을 구분하는 일종의 독선적인 process-driven workflows입니다.

Skills는 [Software Engineering at Google](https://abseil.io/resources/swe-book) 및 Google [engineering practices guide](https://google.github.io/eng-practices/)의 개념을 포함하여 Google 엔지니어링 문화의 모범 사례를 적용합니다. API 디자인에서 Hyrum's Law를 찾을 수 있으며, 테스트에서 Beyonce Rule 및 테스트 피라미드, 코드 검토에서 크기 조정 및 검토 속도 변경 norm, 단순화된 Chesterton's Fence, git workflow에서 trunk-based 개발, CI/CD의 Shift Left 및 feature flags 및 코드를 부채로 취급하는 전용 지원 중단 skill. 이는 추상적인 원칙이 아니며 step-by-step workflows agents 후속 항목에 직접 포함되어 있습니다.

---

## 기여

Skills는 **구체적**(모호한 조언이 아닌 실행 가능한 단계), **검증 가능**(요청 증거가 있는 명확한 종료 기준), **battle-tested**(실제 workflows 기반), **최소**(uide agent에 필요한 단계만)이어야 합니다.

format 사양은 [docs/skill-anatomy.md](docs/skill-anatomy.md)를, guideline은 [CONTRIBUTING.md](CONTRIBUTING.md)를 참조하세요.

---

## 라이센스

MIT - 프로젝트, 팀 및 도구에서 이러한 skills를 사용하세요.
