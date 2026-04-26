# 에이전트 스킬 (Agent Skills)

**AI 코딩 에이전트를 위한 프로덕션급 엔지니어링 스킬.**

스킬(Skills)은 시니어 엔지니어가 소프트웨어를 구축할 때 사용하는 워크플로우(workflows), 품질 게이트(quality gates) 및 베스트 프랙티스(best practices)를 코드화한 것입니다. 이러한 스킬들은 패키지화되어 AI 에이전트가 개발의 모든 단계에서 일관되게 이를 따를 수 있도록 합니다.

```
  정의 (DEFINE)    계획 (PLAN)     구축 (BUILD)    검증 (VERIFY)    리뷰 (REVIEW)    배포 (SHIP)
 ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐
 │ 아이디어│ ───▶ │ 스펙 │ ───▶ │ 코드 │ ───▶ │ 테스트│ ───▶ │  QA  │ ───▶ │ 실행 │
 │ 정교화 │      │ PRD  │      │ 구현 │      │ 디버깅│      │ 게이트│      │  라이브│
 └──────┘      └──────┘      └──────┘      └──────┘      └──────┘      └──────┘
  /spec          /plan          /build        /test         /review       /ship
```

---

## 명령어 (Commands)

개발 생명주기(development lifecycle)에 매핑되는 7가지 슬래시 명령어입니다. 각 명령어는 적절한 스킬을 자동으로 활성화합니다.

| 작업 내용 | 명령어 | 핵심 원칙 |
|-----------|---------|---------------|
| 구축할 내용 정의 | `/spec` | 코드 작성 전 스펙(Spec) 정의 |
| 구축 방법 계획 | `/plan` | 작고 원자적인(atomic) 작업 단위 |
| 점진적 구축 | `/build` | 한 번에 하나의 슬라이스씩 |
| 작동 증명 | `/test` | 테스트가 곧 증명 |
| 머지 전 리뷰 | `/review` | 코드 건강성 향상 |
| 코드 단순화 | `/code-simplify` | 기교보다 명확성 |
| 프로덕션 배포 | `/ship` | 빠를수록 안전함 |

스킬은 수행 중인 작업에 따라 자동으로 활성화되기도 합니다. 예를 들어, API를 설계할 때는 `api-and-interface-design`이, UI를 구축할 때는 `frontend-ui-engineering`이 트리거됩니다.

---

## 빠른 시작 (Quick Start)

<details>
<summary><b>Claude Code (권장)</b></summary>

**마켓플레이스 설치:**

```
/plugin marketplace add addyosmani/agent-skills
/plugin install agent-skills@addy-agent-skills
```

> **SSH 오류가 발생하나요?** 마켓플레이스는 SSH를 통해 저장소를 클론(clone)합니다. GitHub에 SSH 키가 설정되어 있지 않다면, [SSH 키를 추가](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)하거나 페치(fetch) 시에만 HTTPS를 사용하도록 전환하세요:
> ```bash
> git config --global url."https://github.com/".insteadOf "git@github.com:"
> ```

**로컬 / 개발용:**

```bash
git clone https://github.com/addyosmani/agent-skills.git
claude --plugin-dir /path/to/agent-skills
```

</details>

<details>
<summary><b>Cursor</b></summary>

`SKILL.md` 파일을 `.cursor/rules/`에 복사하거나 전체 `skills/` 디렉토리를 참조하세요. [docs/cursor-setup.ko.md](docs/cursor-setup.ko.md)를 참조하세요.

</details>

<details>
<summary><b>Gemini CLI</b></summary>

자동 감지를 위해 네이티브 스킬로 설치하거나, 지속적인 컨텍스트(context)를 위해 `GEMINI.md`에 추가하세요. [docs/gemini-cli-setup.ko.md](docs/gemini-cli-setup.ko.md)를 참조하세요.

**저장소에서 설치:**

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

Windsurf 규칙 설정에 스킬 내용을 추가하세요. [docs/windsurf-setup.ko.md](docs/windsurf-setup.ko.md)를 참조하세요.

</details>

<details>
<summary><b>OpenCode</b></summary>

AGENTS.ko.md와 `skill` 도구를 통한 에이전트 주도 스킬 실행을 사용합니다.

[docs/opencode-setup.ko.md](docs/opencode-setup.ko.md)를 참조하세요.

</details>

<details>
<summary><b>GitHub Copilot</b></summary>

`agents/`의 에이전트 정의를 Copilot 페르소나로 사용하고, 스킬 내용을 `.github/copilot-instructions.md`에 넣으세요. [docs/copilot-setup.ko.md](docs/copilot-setup.ko.md)를 참조하세요.

</details>

<details>
<summary><b>Codex / 기타 에이전트</b></summary>

스킬은 일반 마크다운(Markdown) 파일이므로 시스템 프롬프트(system prompts)나 지침 파일을 수용하는 모든 에이전트에서 작동합니다. [docs/getting-started.ko.md](docs/getting-started.ko.md)를 참조하세요.

</details>

---

## 20가지 전체 스킬

위의 명령어들은 진입점입니다. 내부적으로 이 명령어들은 20가지 스킬을 활성화하며, 각 스킬은 단계별 워크플로우, 검증 게이트(verification gates), 안티-합리화(anti-rationalization) 테이블로 구성된 구조화된 워크플로우입니다. 원하는 스킬을 직접 참조할 수도 있습니다.

### 정의 (Define) - 구축할 내용 명확화

| 스킬 | 역할 | 사용 시점 |
|-------|-------------|----------|
| [idea-refine](skills/idea-refine/SKILL.ko.md) | 모호한 아이디어를 구체적인 제안으로 바꾸기 위한 구조화된 발산적/수렴적 사고 | 탐색이 필요한 대략적인 컨셉이 있을 때 |
| [spec-driven-development](skills/spec-driven-development/SKILL.ko.md) | 코드 작성 전 목표, 명령어, 구조, 코드 스타일, 테스트 및 경계를 포함하는 PRD 작성 | 새로운 프로젝트, 기능 또는 중요한 변경을 시작할 때 |

### 계획 (Plan) - 세부 분해

| 스킬 | 역할 | 사용 시점 |
|-------|-------------|----------|
| [planning-and-task-breakdown](skills/planning-and-task-breakdown/SKILL.ko.md) | 스펙을 수락 기준(acceptance criteria)과 의존성 순서가 있는 작고 검증 가능한 작업으로 분해 | 스펙이 있고 구현 가능한 단위가 필요할 때 |

### 구축 (Build) - 코드 작성

| 스킬 | 역할 | 사용 시점 |
|-------|-------------|----------|
| [incremental-implementation](skills/incremental-implementation/SKILL.ko.md) | 얇은 수직 슬라이스 - 구현, 테스트, 검증, 커밋. 피처 플래그(feature flags), 안전한 기본값, 롤백 친화적 변경 | 둘 이상의 파일에 영향을 주는 모든 변경 |
| [test-driven-development](skills/test-driven-development/SKILL.ko.md) | Red-Green-Refactor, 테스트 피라미드 (80/15/5), 테스트 크기, DRY보다 DAMP, 비욘세 규칙(Beyonce Rule), 브라우저 테스트 | 로직 구현, 버그 수정 또는 동작 변경 시 |
| [context-engineering](skills/context-engineering/SKILL.ko.md) | 에이전트에게 적절한 시점에 적절한 정보 제공 - 규칙 파일, 컨텍스트 패킹(context packing), MCP 통합 | 세션 시작, 작업 전환 또는 출력 품질이 떨어질 때 |
| [source-driven-development](skills/source-driven-development/SKILL.ko.md) | 모든 프레임워크 결정을 공식 문서에 기반함 - 검증, 출처 인용, 미검증 항목 표시 | 프레임워크나 라이브러리에 대해 권위 있고 출처가 명시된 코드가 필요할 때 |
| [frontend-ui-engineering](skills/frontend-ui-engineering/SKILL.ko.md) | 컴포넌트 아키텍처, 디자인 시스템, 상태 관리, 반응형 디자인, WCAG 2.1 AA 접근성 | 사용자 인터페이스(UI)를 구축하거나 수정할 때 |
| [api-and-interface-design](skills/api-and-interface-design/SKILL.ko.md) | 컨트랙트 우선(contract-first) 설계, 하이럼의 법칙(Hyrum's Law), 단일 버전 규칙, 오류 시맨틱(error semantics), 경계 검증 | API, 모듈 경계 또는 공용 인터페이스 설계 시 |

### 검증 (Verify) - 작동 증명

| 스킬 | 역할 | 사용 시점 |
|-------|-------------|----------|
| [browser-testing-with-devtools](skills/browser-testing-with-devtools/SKILL.ko.md) | 라이브 런타임 데이터를 위한 Chrome DevTools MCP - DOM 조사, 콘솔 로그, 네트워크 트레이스, 성능 프로파일링 | 브라우저에서 실행되는 항목을 구축하거나 디버깅할 때 |
| [debugging-and-error-recovery](skills/debugging-and-error-recovery/SKILL.ko.md) | 5단계 분류: 재현, 국소화, 축소, 수정, 방어. 라인 중단(stop-the-line) 규칙, 안전한 폴백(fallbacks) | 테스트 실패, 빌드 중단 또는 예상치 못한 동작 발생 시 |

### 리뷰 (Review) - 머지 전 품질 게이트

| 스킬 | 역할 | 사용 시점 |
|-------|-------------|----------|
| [code-review-and-quality](skills/code-review-and-quality/SKILL.ko.md) | 5축 리뷰, 변경 크기(~100줄), 심각도 레이블(Nit/Optional/FYI), 리뷰 속도 규범, 분할 전략 | 변경 사항을 머지하기 전 |
| [code-simplification](skills/code-simplification/SKILL.ko.md) | 체스터턴의 울타리(Chesterton's Fence), 500의 규칙(Rule of 500), 정확한 동작을 유지하면서 복잡성 감소 | 코드는 작동하지만 읽기나 유지보수가 필요 이상으로 어려울 때 |
| [security-and-hardening](skills/security-and-hardening/SKILL.ko.md) | OWASP Top 10 방지, 인증 패턴, 비밀 정보 관리(secrets management), 의존성 감사, 3계층 경계 시스템 | 사용자 입력, 인증, 데이터 저장 또는 외부 통합을 다룰 때 |
| [performance-optimization](skills/performance-optimization/SKILL.ko.md) | 측정 우선 접근 방식 - Core Web Vitals 목표, 프로파일링 워크플로우, 번들 분석, 안티 패턴 탐지 | 성능 요구 사항이 있거나 성능 저하가 의심될 때 |

### 배포 (Ship) - 자신 있게 배포

| 스킬 | 역할 | 사용 시점 |
|-------|-------------|----------|
| [git-workflow-and-versioning](skills/git-workflow-and-versioning/SKILL.ko.md) | 트렁크 기반 개발(trunk-based development), 원자적 커밋, 변경 크기(~100줄), 커밋-저장 지점 패턴 | 모든 코드 변경 시 (항상) |
| [ci-cd-and-automation](skills/ci-cd-and-automation/SKILL.ko.md) | 시프트 레프트(Shift Left), 빠를수록 안전함, 피처 플래그, 품질 게이트 파이프라인, 실패 피드백 루프 | 빌드 및 배포 파이프라인을 설정하거나 수정할 때 |
| [deprecation-and-migration](skills/deprecation-and-migration/SKILL.ko.md) | 코드를 부채(liability)로 보는 사고방식, 강제적 vs 권고적 지원 중단, 마이그레이션 패턴, 좀비 코드 제거 | 오래된 시스템 제거, 사용자 마이그레이션 또는 기능 중단 시 |
| [documentation-and-adrs](skills/documentation-and-adrs/SKILL.ko.md) | 아키텍처 결정 기록(ADR), API 문서, 인라인 문서화 표준 - *이유(why)*를 문서화 | 아키텍처 결정, API 변경 또는 기능 배포 시 |
| [shipping-and-launch](skills/shipping-and-launch/SKILL.ko.md) | 출시 전 체크리스트, 피처 플래그 생명주기, 단계적 출시(staged rollouts), 롤백 절차, 모니터링 설정 | 프로덕션 배포를 준비할 때 |

---

## 에이전트 페르소나 (Agent Personas)

타겟팅된 리뷰를 위해 사전 설정된 전문가 페르소나입니다:

| 에이전트 | 역할 | 관점 |
|-------|------|-------------|
| [code-reviewer](agents/code-reviewer.ko.md) | 시니어 스태프 엔지니어 | "스태프 엔지니어가 이것을 승인할 것인가?" 기준의 5축 코드 리뷰 |
| [test-engineer](agents/test-engineer.ko.md) | QA 전문가 | 테스트 전략, 커버리지 분석 및 Prove-It 패턴 |
| [security-auditor](agents/security-auditor.ko.md) | 보안 엔지니어 | 취약점 탐지, 위협 모델링, OWASP 평가 |

---

## 참조 체크리스트 (Reference Checklists)

필요할 때 스킬이 불러와 사용하는 빠른 참조 자료입니다:

| 참조 자료 | 내용 |
|-----------|--------|
| [testing-patterns.ko.md](references/testing-patterns.ko.md) | 테스트 구조, 명명 규칙, 모킹(mocking), React/API/E2E 예시, 안티 패턴 |
| [security-checklist.ko.md](references/security-checklist.ko.md) | 커밋 전 체크, 인증, 입력 검증, 헤더, CORS, OWASP Top 10 |
| [performance-checklist.ko.md](references/performance-checklist.ko.md) | Core Web Vitals 목표, 프론트엔드/백엔드 체크리스트, 측정 명령어 |
| [accessibility-checklist.ko.md](references/accessibility-checklist.ko.md) | 키보드 탐색, 스크린 리더, 비주얼 디자인, ARIA, 테스트 도구 |

---

## 스킬 작동 방식

모든 스킬은 일관된 구조를 따릅니다:

```
┌─────────────────────────────────────────────┐
│  SKILL.md                                   │
│                                             │
│  ┌─ Frontmatter ─────────────────────────┐  │
│  │ name: lowercase-hyphen-name           │  │
│  │ description: Use when [trigger]       │  │
│  └───────────────────────────────────────┘  │
│                                             │
│  개요 (Overview)    → 이 스킬의 역할          │
│  사용 시점         → 트리거 조건             │
│  프로세스          → 단계별 워크플로우        │
│  합리화           → 변명 + 반박              │
│  레드 플래그       → 문제의 징후              │
│  검증             → 증거 요구 사항           │
└─────────────────────────────────────────────┘
```

**주요 설계 선택:**

- **산문이 아닌 프로세스.** 스킬은 에이전트가 읽는 참조 문서가 아니라 따르는 워크플로우입니다. 각 스킬에는 단계, 체크포인트 및 종료 기준이 있습니다.
- **안티-합리화(Anti-rationalization).** 모든 스킬에는 에이전트가 단계를 건너뛰기 위해 사용하는 일반적인 변명(예: "나중에 테스트를 추가할게요")과 이에 대한 문서화된 반론 테이블이 포함되어 있습니다.
- **검증은 타협할 수 없습니다.** 모든 스킬은 테스트 통과, 빌드 결과물, 런타임 데이터와 같은 증거 요구 사항으로 끝납니다. "맞는 것 같다"는 결코 충분하지 않습니다.
- **점진적 공개(Progressive disclosure).** `SKILL.md`는 진입점입니다. 지원 참조 자료는 필요할 때만 로드되어 토큰(token) 사용을 최소화합니다.

---

## 프로젝트 구조

```
agent-skills/
├── skills/                            # 20가지 핵심 스킬 (디렉토리당 SKILL.md)
│   ├── idea-refine/                   #   정의 (Define)
│   ├── spec-driven-development/       #   정의 (Define)
│   ├── planning-and-task-breakdown/   #   계획 (Plan)
│   ├── incremental-implementation/    #   구축 (Build)
│   ├── context-engineering/           #   구축 (Build)
│   ├── source-driven-development/     #   구축 (Build)
│   ├── frontend-ui-engineering/       #   구축 (Build)
│   ├── test-driven-development/       #   구축 (Build)
│   ├── api-and-interface-design/      #   구축 (Build)
│   ├── browser-testing-with-devtools/ #   검증 (Verify)
│   ├── debugging-and-error-recovery/  #   검증 (Verify)
│   ├── code-review-and-quality/       #   리뷰 (Review)
│   ├── code-simplification/          #   리뷰 (Review)
│   ├── security-and-hardening/        #   리뷰 (Review)
│   ├── performance-optimization/      #   리뷰 (Review)
│   ├── git-workflow-and-versioning/   #   배포 (Ship)
│   ├── ci-cd-and-automation/          #   배포 (Ship)
│   ├── deprecation-and-migration/     #   배포 (Ship)
│   ├── documentation-and-adrs/        #   배포 (Ship)
│   ├── shipping-and-launch/           #   배포 (Ship)
│   └── using-agent-skills/            #   메타: 이 팩을 사용하는 방법
├── agents/                            # 3가지 전문가 페르소나
├── references/                        # 4가지 보충 체크리스트
├── hooks/                             # 세션 생명주기 훅
├── .claude/commands/                  # 7가지 슬래시 명령어
└── docs/                              # 도구별 설정 가이드
```

---

## 왜 에이전트 스킬인가?

AI 코딩 에이전트는 기본적으로 가장 짧은 경로를 택합니다. 이는 종종 스펙, 테스트, 보안 리뷰 및 소프트웨어를 신뢰할 수 있게 만드는 관행들을 건너뛰는 것을 의미합니다. 에이전트 스킬은 에이전트에게 시니어 엔지니어가 프로덕션 코드에 적용하는 것과 동일한 규율을 강제하는 구조화된 워크플로우를 제공합니다.

각 스킬에는 어렵게 얻은 엔지니어링 판단이 담겨 있습니다: *언제* 스펙을 작성할지, *무엇을* 테스트할지, *어떻게* 리뷰할지, 그리고 *언제* 배포할지 등입니다. 이것들은 일반적인 프롬프트가 아니라 프로덕션 품질의 작업과 프로토타입 품질의 작업을 구분 짓는 독창적이고 프로세스 중심적인 워크플로우입니다.

스킬에는 [Software Engineering at Google](https://abseil.io/resources/swe-book) 및 Google의 [엔지니어링 프랙티스 가이드](https://google.github.io/eng-practices/)의 개념을 포함하여 Google의 엔지니어링 문화에서 검증된 베스트 프랙티스가 녹아 있습니다. API 설계의 하이럼의 법칙(Hyrum's Law), 테스트의 비욘세 규칙(Beyonce Rule) 및 테스트 피라미드, 코드 리뷰의 변경 크기 및 리뷰 속도 규범, 단순화의 체스터턴의 울타리(Chesterton's Fence), git 워크플로우의 트렁크 기반 개발, CI/CD의 시프트 레프트 및 피처 플래그, 그리고 코드를 부채로 다루는 전용 지원 중단 스킬 등을 찾을 수 있습니다. 이것들은 추상적인 원칙이 아니라 에이전트가 따르는 단계별 워크플로우에 직접 내장되어 있습니다.

---

## 기여하기

스킬은 **구체적**(모호한 조언이 아닌 실행 가능한 단계), **검증 가능**(증거 요구 사항이 있는 명확한 종료 기준), **전투 검증됨**(실제 워크플로우 기반), **최소한**(에이전트를 가이드하는 데 필요한 것만 포함)이어야 합니다.

형식 사양은 [docs/skill-anatomy.ko.md](docs/skill-anatomy.ko.md)를, 가이드라인은 [CONTRIBUTING.ko.md](CONTRIBUTING.ko.md)를 참조하세요.

---

## 라이선스 (License)

MIT - 프로젝트, 팀 및 도구에서 이 스킬들을 자유롭게 사용하세요.
