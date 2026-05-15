# Orchestration Patterns

이 repo가 인정하는 agent 오케스트레이션 패턴의 참조 카탈로그와 피해야 할 안티패턴. 여러 persona를 조율하는 새 슬래시 커맨드를 추가하거나, 기존 persona를 "감싸는" 새 persona를 도입하기 전에 이것을 읽으세요.

지배 규칙: **사용자(또는 슬래시 커맨드)가 오케스트레이터입니다. persona는 다른 persona를 invoke하지 않습니다.** skill은 persona 워크플로 내부의 필수 단계입니다.

---

## 인정된 패턴

### 1. 직접 호출 (오케스트레이션 없음)

단일 persona, 단일 관점, 단일 산출물. 기본이자 가장 저렴한 옵션.

```
user → code-reviewer → report → user
```

**사용 시점:** 작업이 하나의 산출물에 대한 하나의 관점이고 한 문장으로 기술할 수 있을 때.

**예시:**
- "Review this PR" → `code-reviewer`
- "Find security issues in `auth.ts`" → `security-auditor`
- "What tests are missing for the checkout flow?" → `test-engineer`

**비용:** 한 번의 왕복. 항상 오케스트레이션 패턴과 비교해야 할 기준선.

---

### 2. 단일 persona 슬래시 커맨드

프로젝트의 skill로 하나의 persona를 감싸는 슬래시 커맨드. 사용자가 매번 워크플로를 다시 설명하지 않아도 됩니다.

```
/review → code-reviewer (with code-review-and-quality skill) → report
```

**사용 시점:** 같은 단일 persona 호출이 같은 설정으로 반복적으로 일어날 때.

**이 repo의 예시:** `/review`, `/test`, `/code-simplify`.

**비용:** 직접 호출과 동일. 슬래시 커맨드는 저장된 프롬프트일 뿐.

**안티 신호:** 슬래시 커맨드의 본문이 대부분 "어떤 persona를 호출할지 결정"이라면, 삭제하고 사용자가 persona를 직접 호출하게 하세요.

---

### 3. merge를 동반한 병렬 fan-out

여러 persona가 같은 입력에 대해 동시에 작동하며, 각각 독립적인 보고서를 생성합니다. (메인 agent의 context에서 일어나는) merge 단계가 이를 단일 결정으로 종합합니다.

```
                    ┌─→ code-reviewer    ─┐
/ship → fan out  ───┼─→ security-auditor ─┤→ merge → go/no-go + rollback
                    └─→ test-engineer    ─┘
```

**사용 시점:**
- 하위 작업이 진정으로 독립적임 (공유 가변 상태 없음, 순서 의존성 없음)
- 각 sub-agent가 자체 context window의 이점을 받음
- merge 단계가 메인 context에 머물 만큼 충분히 작음
- wall-clock 지연 시간이 중요함

**이 repo의 예시:** `/ship`.

**비용:** N개의 병렬 sub-agent context + 한 번의 merge 턴. 직접 호출보다 높지만, wall-clock이 더 빠르고 각 sub-agent가 단일 관점에 집중하므로 더 나은 보고서를 생성함.

**이 패턴을 채택하기 전 검증 체크리스트:**
- [ ] 순서 이슈 없이 모든 sub-agent를 동시에 실행할 수 있는가?
- [ ] 각 persona가 같은 발견을 다른 각도에서 내는 게 아니라, 다른 *종류*의 발견을 생성하는가?
- [ ] merge 단계가 메인 agent의 남은 context에 들어가는가?
- [ ] 사용자의 대기 시간이 병렬성이 실제로 체감될 만큼 긴가?

답이 하나라도 "아니오"라면, 직접 호출이나 단일 persona 커맨드로 후퇴하세요.

---

### 4. 사용자 주도 슬래시 커맨드로서의 순차 파이프라인

사용자가 정의된 순서로 슬래시 커맨드를 실행하며, 그 사이에 context(또는 commit 히스토리)를 전달합니다. 오케스트레이터 agent가 없습니다 — 사용자가 오케스트레이터입니다.

```
사용자 실행:  /spec  →  /plan  →  /build  →  /test  →  /review  →  /ship
```

**사용 시점:** 워크플로에 의존성이 있고(각 단계가 이전 단계의 출력을 필요로 함) 단계 사이의 인간 판단이 가치를 더할 때.

**이 repo의 예시:** 전체 DEFINE → PLAN → BUILD → VERIFY → REVIEW → SHIP 라이프사이클.

**비용:** 단계당 하나의 sub-agent context. 오케스트레이터 agent가 없으므로 오케스트레이션 레이어는 무료.

**자동화하지 않는 이유:** LLM "라이프사이클 오케스트레이터"는 (a) hand-off를 위해 요약해야 하므로 단계 간 뉘앙스를 잃고, (b) 잘못된 방향의 작업을 일찍 잡는 인간 체크포인트를 건너뛰고, (c) 의역 턴을 통해 토큰 비용을 두 배로 만듭니다.

---

### 5. 연구 격리 (context 보존)

작업이 메인 context를 오염시켜서는 안 되는 대량의 자료 읽기를 요구할 때, 요약만 반환하는 연구 sub-agent를 스폰합니다.

```
main agent → research sub-agent (50개 파일 읽기) → 요약 → main agent 계속
```

**사용 시점:**
- 메인 세션이 다운스트림 작업에 집중해야 함
- 조사 결과가 소비하는 입력보다 훨씬 작음
- 이후 메인 agent가 생각할 여유를 가질 때 결정 품질이 향상됨

**예시:** "monorepo 전체에서 이 deprecated API의 모든 호출 지점 찾기", "이 30개 ADR이 캐싱에 대해 뭐라고 하는지 요약".

**비용:** 격리된 sub-agent context 하나. 대안이 메인 context에 수백 개 파일을 로드하는 것일 때면 언제든 가치가 있음.

**Claude Code에서는 커스텀 연구 persona를 정의하기보다 내장 `Explore` subagent를 사용하세요.** `Explore`는 Haiku에서 실행되고, write/edit 도구가 거부되며, 이 패턴을 위해 특별히 만들어졌습니다. `Explore`가 맞지 않을 때(예: 모델이 추론하지 못할 도메인 특화 시스템 프롬프트가 필요할 때)만 커스텀 연구 subagent를 정의하세요.

---

## Claude Code 호환성

이 카탈로그는 harness 중립적이지만, 대부분의 독자는 Claude Code에서 실행할 것입니다. 각 패턴이 Claude Code의 기본 요소에 어떻게 매핑되는지 — 그리고 플랫폼이 우리를 위해 규칙을 강제하는 곳은 어디인지.

### persona가 사는 곳

플러그인 subagent는 플러그인 루트의 `agents/`에 들어갑니다. 이 repo는 플러그인이므로(`.claude-plugin/plugin.json`), `agents/code-reviewer.md`, `agents/security-auditor.md`, `agents/test-engineer.md`는 플러그인이 활성화되면 자동 검색됩니다. 경로 설정이 필요 없습니다.

### Subagent vs. Agent Teams

Claude Code에는 두 가지 병렬성 기본 요소가 있습니다. 패턴 3(merge를 동반한 병렬 fan-out)은 **subagent**에 매핑됩니다. 서로 대화하는 팀원이 필요하다면 대신 **Agent Teams**를 사용하세요.

| | Subagent | Agent Teams |
|--|-----------|-------------|
| 조율 | 메인 agent가 fan-out, sub-agent는 보고만 함 | 팀원이 서로 메시지, 작업 목록 공유 |
| Context | subagent당 자체 context window | 팀원당 자체 context window |
| 사용 시점 | 보고서를 생성하는 독립 작업 | 논의가 필요한 협업 |
| 상태 | 안정적 | 실험적 — `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` 필요 |
| 비용 | 낮음 | 높음 — 각 팀원이 별도 Claude 인스턴스 |

**이 repo의 persona는 두 모드 모두에서 동작합니다.** subagent로 스폰되면(예: `/ship`에 의해) 발견을 메인 세션에 보고합니다. 팀원으로 스폰되면(`Spawn a teammate using the security-auditor agent type…`) 서로의 발견을 직접 반박할 수 있습니다. persona 정의는 동일하며; 스폰 context만 바뀝니다.

한 가지 미묘함: persona의 `skills`와 `mcpServers` frontmatter 필드는 subagent로 실행될 때는 존중되지만 **팀원으로 실행될 때는 무시됩니다** — 팀원은 일반 세션과 마찬가지로 프로젝트 및 사용자 설정에서 skill과 MCP 서버를 로드합니다. persona가 특정 skill이나 MCP 서버 로드에 의존한다면, 두 모드 모두에서 사용 가능하도록 세션 레벨에서 구성하세요.

### 플랫폼이 강제하는 규칙

이 카탈로그의 두 규칙은 단순한 관례가 아닙니다 — Claude Code가 강제합니다:

- **"subagent는 다른 subagent를 스폰할 수 없다"** (문서 그대로). 안티패턴 B(persona가 persona 호출)와 안티패턴 D(깊은 persona 트리)는 Claude Code에서 구조적으로 존재할 수 없습니다.
- **"중첩 팀 없음"** — 팀원은 자체 팀을 스폰할 수 없습니다. 같은 안티패턴이 팀 레벨에서 차단됩니다.

이는 기여자가 실수로 안티패턴을 만드는 것을 걱정하지 않고 이 카탈로그의 패턴을 채택할 수 있다는 뜻입니다. 그것들은 그냥 로드에 실패할 것입니다.

### 알아둘 내장 subagent

커스텀 subagent를 정의하기 전에, 다음 중 하나가 그 역할을 커버하는지 확인하세요:

| 내장 | 목적 |
|----------|---------|
| `Explore` | 읽기 전용 코드베이스 검색 및 분석. 패턴 5(연구 격리)에 사용. |
| `Plan` | plan 모드 중 읽기 전용 연구. |
| `general-purpose` | 탐색과 수정이 모두 필요한 다단계 작업. |

이것들을 재정의하지 마세요. 전문 persona(code-reviewer, security-auditor, test-engineer)를 그 위에 레이어링하세요.

### 플러그인 agent의 frontmatter 제약

플러그인 subagent는 `hooks`, `mcpServers`, `permissionMode` frontmatter 필드를 **지원하지 않습니다** — 이것들은 조용히 무시됩니다. 향후 persona가 이 중 하나를 필요로 한다면, 사용자가 파일을 `.claude/agents/`나 `~/.claude/agents/`로 복사해야 합니다.

플러그인 agent에서 동작하는 필드는: `name`, `description`, `tools`, `disallowedTools`, `model`, `maxTurns`, `skills`, `memory`, `background`, `effort`, `isolation`, `color`, `initialPrompt`입니다. 비용을 최적화하고 싶으면 persona별로 `model`을 사용하세요 (예: `test-engineer` 커버리지 스캔은 Haiku, `code-reviewer`는 Sonnet, `security-auditor`는 Opus).

### 여러 subagent를 병렬로 스폰하기

Claude Code에서 병렬 fan-out(패턴 3)은 **한 assistant 턴에서 여러 Agent 도구 호출을 발행**해야 합니다. 순차 턴은 실행을 직렬화합니다. `/ship`은 이를 명시적으로 언급합니다. 어떤 새 오케스트레이터 커맨드든 동일하게 해야 합니다.

---

## 동작 예시: 경합 가설 디버깅을 위한 Agent Teams

이 예시는 `/ship`의 subagent fan-out 대신 **Agent Teams**를 선택할 때를 보여줍니다. 두 패턴은 멀리서 보면 비슷합니다 — 둘 다 같은 세 persona를 스폰 — 하지만 가치는 다른 곳에서 옵니다.

### 시나리오

> *Checkout이 가끔 완료 전 ~30초 동안 멈춥니다. 대략 50세션마다 한 번 발생합니다. 로그에 에러 없음. 지난주 릴리스 이후 시작됨.*

그럴듯한 근본 원인 (상호 배타적, 모두 증상에 부합):

1. 새 payment-confirmation 흐름의 race condition
2. 가끔 느린 동기 network 호출로 떨어지는 auth 검사
3. cart 크기에 따라 스케일되는 쿼리의 인덱스 누락
4. SDK가 timeout 전 조용히 재시도하는 불안정한 서드파티 API

단일 agent는 첫 번째 그럴듯한 이론을 골라 조사를 멈출 것입니다. `/ship` 스타일 subagent fan-out은 각 persona가 독립적으로 보고하게 하지만 — 그들의 보고서는 절대 만나지 않으므로, 잘못된 이론을 배제하는 것이 없습니다.

이것이 바로 Agent Teams 문서가 기술하는 경우입니다: *"여러 독립 조사자가 적극적으로 서로를 반증하려 할 때, 살아남는 이론이 실제 근본 원인일 가능성이 훨씬 높다."*

### 왜 이것이 `/ship` 작업이 *아닌가*

| | `/ship` (subagent) | Agent Teams |
|--|--------------------|-------------|
| sub-agent가 보는 것 | 같은 diff, 다른 렌즈 | 공유 작업 목록, 서로의 메시지 |
| 출력 | 세 개의 독립 보고서 → 하나의 merge | 적대적 토론 → 합의된 근본 원인 |
| 적절한 때 | 알려진 산출물에 대한 평결을 원할 때 | 가설들 사이에서 산출물을 *찾고* 싶을 때 |

`/ship`은 평결이고; Agent Teams는 조사입니다.

### 설정 (환경별 1회)

Agent Teams는 실험적입니다. `~/.claude/settings.json`에:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Claude Code v2.1.32 이상 필요. 이 repo의 persona는 자동으로 가져와집니다 — 손으로 작성할 team-config 파일이 없습니다.

### 트리거 프롬프트

lead 세션에 자연어로 입력하세요:

```
Users report checkout hangs for ~30 seconds intermittently after last
week's release. No errors in logs.

Create an agent team to debug this with competing hypotheses. Spawn
three teammates using the existing agent types:

  - code-reviewer  — investigate race conditions and blocking calls
                     in the checkout code path
  - security-auditor — investigate auth checks, session handling,
                       and any synchronous network calls added recently
  - test-engineer  — propose tests that would distinguish between the
                     hypotheses and check coverage gaps in checkout

Have them message each other directly to challenge each other's
theories. Update findings as consensus emerges. Only converge when
two teammates agree they can disprove the others'.
```

lead는 기존 persona 이름을 참조해 세 팀원을 스폰합니다. persona 본문은 (lead가 설치하는 팀 조율 지침 위에) 각 팀원의 시스템 프롬프트에 추가 지침으로 **덧붙여집니다**; 위 트리거 프롬프트가 그들의 작업이 됩니다.

### 무슨 일이 일어나는가

1. 각 팀원은 자체 context window에서 실행되며, 자신의 렌즈로 코드베이스를 탐색합니다.
2. 팀원은 `message`를 사용해 발견을 서로에게 직접 보냅니다. lead가 중계할 필요가 없습니다.
3. 공유 작업 목록은 누가 무엇을 조사 중인지 보여줍니다 — `Ctrl+T`(in-process 모드)나 tmux 창(split 모드)에서 언제든 볼 수 있습니다.
4. `code-reviewer`가 순차적이어야 할 `Promise.all`을 발견하면, `security-auditor`에게 메시지를 보내 auth 호출이 race의 일부가 아닌지 확인합니다. `security-auditor`가 확인하고 답합니다 — race가 진짜 문제임을 확인하거나 반대 증거를 생성합니다.
5. `test-engineer`는 이기고 있는 이론에 대한 집중된 integration test를 제안하고, 팀은 합의를 선언하기 전에 이를 사용해 검증합니다.
6. lead가 수렴된 발견을 종합해 여러분에게 제시합니다.

`Shift+Down`으로 순환하며 입력해 어느 팀원이든 중단할 수 있습니다 — 잘못된 경로로 간 조사자를 방향 전환시킬 때 유용합니다.

### 언제 정리하는가

조사가 근본 원인에 도달하면, lead에게 말하세요:

```
Clean up the team
```

항상 팀원이 아니라 lead를 통해 정리하세요 (문서에 따르면: 팀원은 정리를 위한 전체 팀 context가 부족함).

### 비용 기대

세 Sonnet 팀원이 ~10–15분의 조사를 실행하면 `/ship`이 subagent로 스폰한 같은 세 persona보다 눈에 띄게 더 많은 비용이 듭니다. 정당화는 *결론의 품질*입니다 — 잘못된 수정이 비싼 production 디버깅에서는, 추가 토큰이 헐값입니다. 일상적인 PR 리뷰에는 `/ship`을 고수하세요.

### 이 시나리오의 안티패턴

이것을 subagent를 fan-out하는 `/debug` 슬래시 커맨드로 다시 만들지 **마세요**. subagent는 서로 메시지를 주고받을 수 없습니다 — 이 패턴을 작동시키는 적대적 토론을 잃게 됩니다. 어떤 워크플로가 계속 나온다면, 위 트리거 프롬프트를 subagent를 오용하는 슬래시 커맨드로 감싸기보다 스니펫으로 문서화하세요.

### Agent Teams를 사용하지 *않을* 때

- 알려진 diff에 대한 production 대상 평결 → `/ship` 사용 (subagent).
- 하나의 산출물에 대한 한 전문가 관점 → persona 직접 호출.
- 순차 라이프사이클 (spec → plan → build) → 사용자 주도 슬래시 커맨드 (패턴 4).
- 작은 요약을 동반한 읽기 중심 연구 → 내장 `Explore` subagent.

올바른 답을 내기 위해 팀원이 서로를 반박할 **필요**가 있을 때만 Agent Teams를 선택하세요.

---

## 안티패턴

### A. Router persona ("meta-orchestrator")

어떤 다른 persona를 호출할지 결정하는 역할의 persona.

```
/work → router-persona → "this needs a review" → code-reviewer → router (의역) → user
```

**왜 실패하는가:**
- 도메인 가치 없는 순수 라우팅 레이어
- 의역 단계 두 개 추가 → 정보 손실 + 약 2배 토큰 비용
- 사용자는 이미 리뷰를 원한다는 것을 알고 있었음; `/review`를 직접 호출할 수 있었음
- 슬래시 커맨드와 `AGENTS.md`의 intent 매핑이 이미 하는 작업을 복제

**대신 할 것:** 슬래시 커맨드를 추가하거나 개선. `AGENTS.md`에 intent → command 매핑을 문서화.

---

### B. 다른 persona를 호출하는 persona

auth 코드를 보면 내부적으로 `security-auditor`를 invoke하는 `code-reviewer`.

**왜 실패하는가:**
- persona는 단일 관점을 생성하도록 설계됨; 체이닝은 그것을 무산시킴
- 호출하는 persona가 전달하는 요약은 호출되는 persona가 필요로 하는 context를 잃음
- 실패 모드가 늘어남 (어느 persona의 출력 형식이 이기는가? 누구의 규칙이 적용되는가?)
- 사용자에게 비용을 숨김

**대신 할 것:** 호출하는 persona가 보고서에 후속 감사를 *권장*하게 하세요. 사용자나 슬래시 커맨드가 두 번째 검토를 실행합니다.

---

### C. 의역하는 순차 오케스트레이터

사용자를 대신해 `/spec`, 그 다음 `/plan`, 그 다음 `/build` 등을 호출하는 agent.

**왜 실패하는가:**
- 잘못된 방향의 작업을 잡는 인간 체크포인트를 잃음
- 각 hand-off가 context를 요약 — 긴 파이프라인에 걸쳐 누적된 drift
- 토큰 비용 두 배: 모든 단계마다 오케스트레이터 턴 + sub-agent 턴
- 판단이 가장 중요한 바로 그 지점에서 사용자 주체성을 제거

**대신 할 것:** 사용자를 오케스트레이터로 유지. `README.md`에 권장 순서를 문서화하고 사용자가 invoke하게 하세요.

---

### D. 깊은 persona 트리

`/ship`이 `pre-ship-coordinator`를 호출하고, 그것이 `quality-coordinator`를 호출하고, 그것이 `code-reviewer`를 호출.

**왜 실패하는가:**
- 각 레이어가 결정 가치 없이 지연 시간과 토큰을 더함
- 디버깅이 다단계 조사가 됨
- leaf persona가 여러 요약 단계로 context를 잃음

**대신 할 것:** 오케스트레이션 깊이를 최대 1로 유지 (슬래시 커맨드 → persona). merge는 메인 agent에서 일어납니다.

---

## 결정 흐름

새 오케스트레이션 워크플로를 고려할 때, 이 흐름을 따르세요:

```
작업이 하나의 산출물에 대한 하나의 관점인가?
├── 예 → 직접 호출. 끝.
└── 아니오  → 같은 조합이 반복되는가?
         ├── 아니오  → 직접 호출, 임시. 끝.
         └── 예 → 하위 작업이 독립적인가?
                  ├── 아니오  → 사용자가 실행하는 순차 슬래시 커맨드 (패턴 4).
                  └── 예 → merge를 동반한 병렬 fan-out (패턴 3).
                           위 체크리스트에 대해 검증.
                           어떤 검사든 실패하면 → 단일 persona 커맨드로 후퇴 (패턴 2).
```

---

## 이 카탈로그에 새 패턴을 언제 추가하는가

다음 후에만 새 항목을 추가하세요:

1. 실제 작업에서 그 패턴을 최소 두 번 사용함
2. 그것을 시연하는 이 repo의 구체적 산출물을 지목할 수 있음
3. 기존 패턴이 왜 동작하지 않았을지 설명할 수 있음
4. 그것의 안티패턴 그림자를 기술할 수 있음 (사람들이 대신 잘못 만들 것)

성급한 카탈로그 항목은 아무도 따르지 않는 희망적 문서가 됩니다.
