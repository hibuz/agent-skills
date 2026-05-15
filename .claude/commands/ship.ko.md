---
description: 전문 persona로 병렬 fan-out하여 런칭 전 체크리스트를 실행한 뒤 go/no-go 결정을 종합
---

agent-skills:shipping-and-launch skill을 invoke하세요.

`/ship`은 **fan-out 오케스트레이터**입니다. 현재 변경에 대해 세 개의 전문 persona를 병렬로 실행한 뒤, 그 보고서를 rollback 계획을 갖춘 단일 go/no-go 결정으로 merge합니다. persona들은 독립적으로 작동합니다 — 공유 상태 없음, 순서 없음 — 이것이 여기서 병렬 실행을 안전하고 유용하게 만드는 요소입니다.

## Phase A — 병렬 fan-out

Agent 도구를 사용해 세 개의 subagent를 동시에 스폰합니다. **세 개의 Agent 도구 호출을 한 assistant 턴에서 발행해 병렬로 실행되게 하세요** — 순차 호출은 이 커맨드의 목적을 무산시킵니다.

Claude Code에서 각 호출은 persona의 `name` 필드와 일치하는 `subagent_type`을 전달합니다:

1. **`code-reviewer`** — staged 변경이나 최근 commit에 대해 5축 리뷰(정확성, 가독성, 아키텍처, 보안, 성능)를 실행. 표준 리뷰 템플릿을 출력.
2. **`security-auditor`** — 취약점 및 위협 모델 검토를 실행. OWASP Top 10, secrets 처리, auth/authz, 의존성 CVE 확인. 표준 감사 보고서를 출력.
3. **`test-engineer`** — 변경에 대한 테스트 커버리지를 분석. happy path, 엣지 케이스, 에러 경로, 동시성 시나리오의 공백 식별. 표준 커버리지 분석을 출력.

Agent 도구가 없는 다른 harness에서는, 각 persona의 시스템 프롬프트를 순차적으로 invoke하고 그 출력을 병렬로 반환된 것처럼 다루세요 — merge 단계는 여전히 동작합니다.

제약 (Claude Code의 subagent 모델에서):
- subagent는 다른 subagent를 스폰할 수 없습니다 — 한 persona가 다른 persona에 위임하게 하지 마세요.
- 각 subagent는 자체 context window를 받고 자신의 보고서만 이 메인 세션에 반환합니다.
- 단순히 보고만 하는 것이 아니라 서로 대화하는 팀원이 필요하다면, Claude Code Agent Teams를 사용하고 이 persona들을 팀원 타입으로 참조하세요 (`references/orchestration-patterns.md` 참고).

**Persona 해석.** `.claude/agents/`나 `~/.claude/agents/`에 자신만의 `code-reviewer`, `security-auditor`, `test-engineer`를 정의했다면, 그것이 이 플러그인 버전보다 우선합니다 — `/ship`은 여러분의 커스터마이징을 자동으로 가져옵니다. 이는 의도된 것입니다: 플러그인 subagent는 Claude Code의 스코프 우선순위 표 맨 아래에 있으므로, 사용자 레벨 정의가 설계상 이깁니다.

## Phase B — 메인 context에서 merge

세 보고서가 모두 돌아오면, 메인 agent(하위 persona가 아님)가 이를 종합합니다:

1. **Code Quality** — `code-reviewer`의 Critical/Important 발견과 실패한 테스트, lint, build 출력을 집계. 리뷰어 간 중복 해소.
2. **Security** — `security-auditor`의 Critical/High 발견을 런칭 blocker로 승격. `code-reviewer`의 보안 축과 교차 참조.
3. **Performance** — `code-reviewer`의 성능 축에서 가져옴; 해당되면 Core Web Vitals 교차 확인.
4. **Accessibility** — 키보드 내비게이션, 스크린 리더 지원, 대비 검증 (세 persona가 다루지 않음 — 여기서 직접 처리하거나 accessibility 체크리스트 invoke).
5. **Infrastructure** — env var, migration, 모니터링, feature flag. 직접 검증.
6. **Documentation** — README, ADR, changelog. 직접 검증.

## Phase C — 결정과 rollback

단일 출력을 생성하세요:

```markdown
## Ship Decision: GO | NO-GO

### Blockers (must fix before ship)
- [Source persona: Critical finding + file:line]

### Recommended fixes (should fix before ship)
- [Source persona: Important finding + file:line]

### Acknowledged risks (shipping anyway)
- [Risk + mitigation]

### Rollback plan
- Trigger conditions: [어떤 신호가 rollback을 촉발하는지]
- Rollback procedure: [정확한 단계]
- Recovery time objective: [목표]

### Specialist reports (full)
- [code-reviewer report]
- [security-auditor report]
- [test-engineer report]
```

## 규칙

1. Phase A의 세 persona는 병렬로 실행됩니다 — 절대 순차적으로가 아닙니다.
2. persona는 서로 호출하지 않습니다. 메인 agent가 Phase B에서 merge합니다.
3. rollback 계획은 어떤 GO 결정 전에도 필수입니다.
4. 어떤 persona든 Critical 발견을 반환하면, 사용자가 명시적으로 위험을 수용하지 않는 한 기본 평결은 NO-GO입니다.
5. **다음이 모두 참일 때만 fan-out을 건너뛰세요:** 변경이 2개 이하 파일을 건드리고, diff가 50줄 미만이며, auth, payments, 데이터 접근, 또는 config/env를 건드리지 않음. 그렇지 않으면 fan-out을 기본으로 하세요. `/ship`은 production 대상 변경을 위해 설계되었습니다 — blast radius가 비자명할 때는 diff가 작아 보여도 병렬 리뷰를 실행하세요.
