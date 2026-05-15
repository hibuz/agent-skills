# Agent Persona

단일 관점으로 단일 역할을 수행하는 전문 persona. 각 persona는 harness(Claude Code, Cursor, Copilot 등)가 시스템 프롬프트로 소비하는 Markdown 파일입니다.

| Persona | 역할 | 가장 적합한 용도 |
|---------|------|----------|
| [code-reviewer](code-reviewer.ko.md) | Senior Staff Engineer | merge 전 5축 리뷰 |
| [security-auditor](security-auditor.ko.md) | Security Engineer | 취약점 탐지, OWASP 스타일 감사 |
| [test-engineer](test-engineer.ko.md) | QA Engineer | 테스트 전략, 커버리지 분석, Prove-It 패턴 |

## persona가 skill 및 command와 어떻게 관계되는가

세 가지 레이어, 각각 별개의 역할:

| 레이어 | 무엇인가 | 예시 | 조합 역할 |
|-------|-----------|---------|------------------|
| **Skill** | 단계와 종료 기준을 가진 워크플로 | `code-review-and-quality` | *어떻게(how)* — persona나 command 내부에서 invoke됨 |
| **Persona** | 관점과 출력 형식을 가진 역할 | `code-reviewer` | *누가(who)* — 관점을 채택하고 보고서를 생성 |
| **Command** | 사용자 대면 진입점 | `/review`, `/ship` | *언제(when)* — persona와 skill을 조합 |

사용자(또는 슬래시 커맨드)가 오케스트레이터입니다. **persona는 다른 persona를 호출하지 않습니다.** skill은 persona 워크플로 내부의 필수 단계입니다.

## 각각 언제 사용하는가

### Persona 직접 호출
현재 변경에 대한 하나의 관점을 원하고 사용자가 루프 안에 있을 때 선택하세요.

- "Review this PR" → `code-reviewer`를 직접 invoke
- "Are there security issues in `auth.ts`?" → `security-auditor`를 직접 invoke
- "What tests are missing for the checkout flow?" → `test-engineer`를 직접 invoke

### 슬래시 커맨드 (배후에 단일 persona)
매번 다시 설명해야 할 반복 가능한 워크플로가 있을 때 선택하세요.

- `/review` → 프로젝트의 리뷰 skill로 `code-reviewer`를 래핑
- `/test` → TDD skill로 `test-engineer`를 래핑

### 슬래시 커맨드 (오케스트레이터 — fan-out)
**독립적인** 조사가 병렬로 실행되어 단일 agent가 이후 merge하는 보고서를 생성할 수 있을 때만 선택하세요.

- `/ship` → `code-reviewer` + `security-auditor` + `test-engineer`로 병렬 fan-out한 뒤, 그 보고서를 go/no-go 결정으로 종합

이것이 이 repo가 인정하는 유일한 오케스트레이션 패턴입니다. 전체 패턴 카탈로그와 안티패턴은 [references/orchestration-patterns.md](../references/orchestration-patterns.ko.md)를 참고하세요.

## 결정 매트릭스

```
작업이 단일 산출물에 대한 단일 관점인가?
├── 예 → Persona 직접 호출
└── 아니오  → 하위 작업이 독립적인가 (공유 가변 상태 없음, 순서 없음)?
         ├── 예 → 병렬 fan-out 슬래시 커맨드 (예: /ship)
         └── 아니오  → 사용자가 실행하는 순차 슬래시 커맨드 (/spec → /plan → /build → /test → /review)
```

## 동작 예시: 유효한 오케스트레이션

`/ship`은 이 repo의 표준 fan-out 오케스트레이터입니다:

```
/ship
  ├── (병렬) code-reviewer    → 리뷰 보고서
  ├── (병렬) security-auditor → 감사 보고서
  └── (병렬) test-engineer    → 커버리지 보고서
                  ↓
        merge 단계 (메인 agent)
                  ↓
        go/no-go 결정 + rollback 계획
```

왜 이것이 동작하는가:
- 각 sub-agent는 동일한 diff에서 작동하지만 **다른 관점**을 생성합니다
- 서로 의존성이 없음 → 진정한 병렬성, 실제 wall-clock 절약
- 각각 신규 context window에서 실행 → 메인 세션이 깔끔하게 유지됨
- merge 단계는 작고 전체 context의 이점을 받으므로 메인 agent에 남음

## 동작 예시: 무효한 오케스트레이션 (이것을 만들지 마세요)

"어떤 다른 persona를 호출할지 결정"하는 역할의 `meta-orchestrator` persona:

```
/work-on-pr → meta-orchestrator
                  ↓ ("리뷰가 필요하다"고 결정)
              code-reviewer
                  ↓ (반환)
              meta-orchestrator (결과를 의역)
                  ↓
              사용자
```

왜 이것이 실패하는가:
- 도메인 가치 없는 순수 라우팅 레이어
- 의역 단계 두 개 추가 → 정보 손실 + 2배 토큰 비용
- 사용자는 이미 리뷰를 원한다는 것을 알고 있음; `/review`를 직접 호출하게 하라
- 슬래시 커맨드와 `AGENTS.md` intent 매핑이 이미 하는 작업을 복제

## persona 규칙

1. persona는 단일 출력 형식을 가진 단일 역할입니다. 두 번째 역할을 추가하고 있다면, 두 번째 persona를 만드세요.
2. **persona는 다른 persona를 invoke하지 않습니다.** 조합은 슬래시 커맨드나 사용자의 역할입니다. Claude Code에서는 이것이 하드 플랫폼 제약이기도 합니다 — *"subagent는 다른 subagent를 스폰할 수 없다"* — 따라서 규칙이 강제됩니다.
3. persona는 skill(*어떻게*)을 invoke할 수 있습니다.
4. 모든 persona 파일은 자신이 어디에 들어맞는지 명시하는 "Composition" 블록으로 끝납니다.

## Claude Code 상호운용

이 repo의 persona는 수정 없이 Claude Code subagent와 Agent Teams 팀원으로 동작하도록 설계되었습니다:

- **subagent로서:** 이 플러그인이 활성화되면 자동 검색됨 (경로 설정 불필요). Agent 도구를 `subagent_type: code-reviewer`(또는 `security-auditor`, `test-engineer`)로 사용하세요. `/ship`이 표준 예시입니다.
- **Agent Teams 팀원으로서** (실험적, `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` 필요): 팀원을 스폰할 때 동일한 persona 이름을 참조하세요. persona의 본문은 팀원의 시스템 프롬프트에 추가 지침으로 **덧붙여집니다**(대체가 아님). 따라서 여러분의 persona 텍스트는 리드가 설치하는 팀 조율 지침(SendMessage, task-list 도구 등) 위에 놓입니다.

subagent는 결과를 메인 agent에만 보고합니다. Agent Teams는 팀원끼리 직접 메시지를 주고받게 합니다. 보고서로 충분할 때는 subagent를, sub-agent가 서로의 발견을 반박해야 할 때(예: 경합 가설 디버깅)는 Agent Teams를 사용하세요. 전체 매핑은 [references/orchestration-patterns.md](../references/orchestration-patterns.ko.md)를 참고하세요.

플러그인 agent는 `hooks`, `mcpServers`, `permissionMode` frontmatter를 지원하지 않습니다 — 이 필드들은 조용히 무시됩니다. 여기서 새 persona를 작성할 때 이것들에 의존하지 마세요.

## 새 persona 추가하기

1. 기존 persona가 사용하는 동일한 frontmatter 형식으로 `agents/<role>.md`를 만듭니다.
2. 역할, 범위, 출력 형식, 규칙을 정의합니다.
3. 하단에 **Composition** 블록을 추가합니다 (직접 invoke할 때 / ~를 통해 invoke / 다른 persona에서 invoke 금지).
4. 이 파일 상단의 표에 persona를 추가합니다.
5. persona가 새 오케스트레이션 패턴을 가능하게 한다면, persona 파일 자체에서 패턴을 만들어 내지 말고 `references/orchestration-patterns.md`에 문서화하세요.
