---
name: doubt-driven-development
description: 모든 비자명한 결정이 확정되기 전에 신규-context 적대적 리뷰를 거치게 함. 정확성이 속도보다 중요할 때, 익숙하지 않은 코드에서 작업할 때, 위험이 높을 때(production, 보안 민감 로직, 비가역 작업), 또는 확신에 찬 출력을 나중에 디버깅하는 것보다 지금 검증하는 것이 더 저렴할 때 사용.
---

# Doubt-Driven Development

## Overview

확신에 찬 답이 옳은 답은 아닙니다. 긴 세션은 아무도 눈치채지 못한 채 가정을 조용히 "사실"로 바꾸는 context를 누적합니다. doubt-driven development는 어떤 비자명한 출력이 확정되기 전에 — 승인이 아니라 **반증**으로 편향된 — 신규-context 리뷰어를 구체화하는 규율입니다.

이것은 `/review`가 아닙니다. `/review`는 완성된 산출물에 대한 평결입니다. 이것은 진행 중 자세입니다: 비자명한 결정이 방향 수정이 아직 저렴할 때 교차 검문됩니다.

## When to Use

결정이 다음 중 적어도 하나가 참일 때 **비자명**합니다:

- branching 로직을 도입하거나 수정함
- 모듈이나 서비스 경계를 넘음
- 타입 시스템이나 컴파일러가 검증할 수 없는 속성(thread safety, 멱등성, 순서, 불변식)을 주장함
- 그 정확성이 미래 독자가 볼 수 없는 context에 의존함
- blast radius가 비가역적임 (production 배포, 데이터 migration, 공개 API 변경)

다음일 때 skill을 적용하세요:

- 불확실성 하에 아키텍처 결정을 내릴 참
- 비자명한 코드를 commit할 참
- 명백하지 않은 사실을 주장할 참 ("이건 안전해", "이건 스케일돼", "이건 spec과 일치해")
- 완전히 이해하지 못하는 코드에서 작업 중

**사용하지 말아야 할 때:**

- 기계적 작업 (이름 변경, 포맷팅, 파일 이동)
- 명확하고 모호하지 않은 사용자 지시를 따름
- 기존 코드 읽기나 요약
- 명백히 정확한 한 줄 변경
- 순수 도구 작업 (테스트 실행, 파일 나열)
- 사용자가 검증보다 속도를 명시적으로 요청함

모든 키 입력을 의심하면, 아무것도 ship하지 못합니다. skill은 위에 정의된 비자명한 결정에만 적용됩니다.

## 로딩 제약

이 skill은 Step 3(DOUBT, 아래 상세)이 신규-context 리뷰어를 스폰할 수 있는 **메인 세션 오케스트레이터**용으로 설계되었습니다.

- **이 skill을 persona의 `skills:` frontmatter에 추가하지 마세요.** Step 3을 따르는 persona는 다른 persona를 스폰하게 됩니다 — `references/orchestration-patterns.md`가 명시적으로 금지하는 오케스트레이션 안티패턴 ("persona는 다른 persona를 invoke하지 않는다").
- **subagent context 안에서 이 skill을 적용하고 있는 자신을 발견하면**(Claude Code가 중첩 subagent 스폰을 막는 곳): 선호 경로는 doubt-driven이 중첩 실행될 수 없음을 사용자에게 표면화하고 메인 세션이 처리하게 하는 것입니다. 최후의 수단으로만, 저하된 자기 질문 fallback이 존재합니다 — ARTIFACT + CONTRACT를 이전 추론과 단단한 정신적 분리선을 둔 새 self-prompt로 다시 쓰고, Step 1–5를 진행. 이것은 **신규-context 리뷰가 아니므로**(자신의 context를 지니고 다님), 결과를 저하된 것으로 표시하고 사용자에게 도달 가능할 때면 언제든 escalation을 선호하세요.

## 프로세스

skill을 적용할 때 이 체크리스트를 복사하세요:

```
Doubt cycle:
- [ ] Step 1: CLAIM — 주장 + 왜 중요한지 작성
- [ ] Step 2: EXTRACT — artifact + contract 분리, 추론 제거
- [ ] Step 3: DOUBT — 적대적 프롬프트로 신규-context 리뷰어 invoke
- [ ] Step 4: RECONCILE — 모든 발견을 artifact 텍스트에 대해 분류
- [ ] Step 5: STOP — 정지 조건 충족 (사소한 발견, 3 cycle, 또는 사용자 override)
```

### Step 1: CLAIM — 확정되는 것을 표면화

결정을 두세 줄로 명명:

```
CLAIM: "The new caching layer is thread-safe under the
        read-heavy workload described in the spec."
WHY THIS MATTERS: a race here corrupts user data and is
                  hard to detect in QA.
```

그렇게 압축적으로 주장을 쓸 수 없다면, 결정이 아니라 분위기를 가진 것입니다. 면밀히 살피기 전에 표면화하세요.

### Step 2: EXTRACT — 가장 작은 리뷰 가능 단위

신규-context 리뷰어는 여정이 아니라 **artifact**와 **contract**가 필요합니다.

- 코드: diff 또는 함수 — 전체 파일이 아니라
- 결정: 제안을 3–5문장 + 충족해야 할 제약
- 주장(assertion): 주장 + 그것을 뒷받침한다고 하는 증거 (Step 1 CLAIM 블록과 구별되게 유지; 그것은 면밀히 살펴지는 오케스트레이터의 가설)

당신의 추론을 제거하세요. 결론을 넘겨주면, 결론의 검증을 돌려받습니다. 단위는 리뷰어가 한 번 읽고 머릿속에 담을 수 있을 만큼 작아야 합니다 — 500줄 PR이라면, 먼저 분해하세요.

### Step 3: DOUBT — 신규-context 리뷰어 invoke

리뷰어의 프롬프트는 **반드시 적대적이어야** 합니다. 프레이밍이 답을 결정합니다.

```
Adversarial review. Find what is wrong with this artifact.
Assume the author is overconfident. Look for:
- Unstated assumptions
- Edge cases not handled
- Hidden coupling or shared state
- Ways the contract could be violated
- Existing conventions this might break
- Failure modes under unexpected input

Do NOT validate. Do NOT summarize. Find issues, or state
explicitly that you cannot find any after thorough examination.

ARTIFACT: <paste artifact>
CONTRACT: <paste contract>
```

**ARTIFACT + CONTRACT만 전달. CLAIM을 전달하지 마세요.** 리뷰어에게 결론을 건네면 동의 쪽으로 편향됩니다. 리뷰어는 artifact가 contract를 충족하는지 독립적으로 판단해야 합니다.

Claude Code에서, `agents/`의 role 기반 리뷰어는 설계상 격리된 context로 시작하며 여기서 사용 가능합니다 — 명단과 도메인별 매칭은 `agents/`를 참고하세요.

**위 적대적 프롬프트는 persona의 기본 응답 형태보다 우선합니다.** `code-reviewer` 같은 persona는 강점과 약점을 모두 갖춘 균형 잡힌 평결을 생성하도록 작성되어 있습니다; doubt-driven은 이슈만 있는 출력이 필요합니다. 적대적 프롬프트를 호출에 그대로 붙여넣어 persona의 기본을 override하세요. persona의 응답 형태를 깔끔하게 override할 수 없다면, 적대적 프롬프트를 가진 일반 subagent로 후퇴하세요.

#### 교차 모델 escalation

단일 모델 리뷰어는 원작자와 맹점을 공유합니다 — 더 차갑고 다른 아키텍처의 모델이 그것을 잡습니다. doubt-driven은 이미 비자명한 결정에 대해 opt-in이므로, 그 범위 내에서 교차 모델을 제안하는 것은 선택적 마찰이 아니라 skill 가치의 일부입니다.

**상호작용 세션: 항상 제안. 절대 조용히 건너뛰지 말 것.**

**Step 1: 사용자에게 묻기**

위 Step 3의 단일 모델 리뷰 후, 하지만 RECONCILE 전에, 멈추고 물으세요:

> *"Single-model review complete. Want a cross-model second opinion? Options: Gemini CLI, Codex CLI, manual external review (you paste it elsewhere), or skip."*

이 질문은 모든 상호작용 doubt cycle에서 필수입니다 — 위험이 낮게 느껴지는 artifact라도. agent가 아니라 사용자가 비용이 값어치 있는지 결정합니다. agent의 일은 선택지를 표면화하는 것입니다.

**Step 2: 사용자가 CLI를 고르면 — 검증 후 invoke**

1. 도구가 PATH에 있는지 확인 (`which gemini`, `which codex`).
2. 전체 프롬프트를 전달하기 전에 동작하는지 테스트 (`gemini --version` 또는 동등물) — 오래되거나 깨진 binary는 `which`는 통과해도 실제 입력에서 실패할 수 있음.
3. 필요한 flag, auth, env var(예: API 키)를 포함해 정확한 호출을 사용자와 확인. 구현이 다양함; 절대 가정하지 말 것.
4. ARTIFACT + CONTRACT + 적대적 프롬프트**만** 전달. 세션 context 없음, CLAIM 없음.
5. shell escaping에 유의. artifact에 따옴표, `$(...)`, 또는 backtick이 있으면, 인라인 `-p "…"`보다 stdin(`echo … | gemini`)이나 heredoc을 선호. 의심스러우면, 실행 전에 사용자에게 호출을 확인 요청.
6. 출력을 Step 4(RECONCILE)로 가져가기.

**artifact를 shell-quoted 인자로 절대 interpolate하지 마세요.** 코드, markdown, 리뷰 프롬프트는 일상적으로 backtick, `$(...)`, 따옴표 문자를 포함하며, 이는 프롬프트를 truncate하거나 임베드된 shell을 실행합니다. 전체 프롬프트를 파일에 쓰고 stdin으로 pipe하세요.

예시 형태 (설치된 도구에 대해 flag를 검증하세요 — 구현과 버전에 따라 문법이 다름):

```bash
# 적대적 프롬프트 + ARTIFACT + CONTRACT를 먼저 temp 파일에 작성.
# 그 다음 stdin으로 pipe해 artifact의 shell metacharacter가 비활성으로 유지되게.

# Codex (read-only sandbox가 CLI의 workspace 쓰기를 막음):
codex exec --sandbox read-only -C <repo-path> - < /tmp/doubt-prompt.md

# Gemini ('--approval-mode plan'은 read-only; '-p ""'는 non-interactive
# 모드를 트리거하고 프롬프트는 stdin에서 읽힘):
gemini --approval-mode plan -p "" < /tmp/doubt-prompt.md
```

read-only sandbox가 핵심 세부사항입니다: doubt artifact 자체가 (의도적이거나 우발적인 prompt injection) 지시문을 포함할 수 있으며, 그렇지 않으면 교차 모델 CLI가 그것을 당신의 workspace에 대해 실행할 것입니다.

**Step 3: CLI가 사용 불가하거나 실패하면**

실패를 명시적으로 표면화하세요. 제안: 수동으로 실행, 다른 도구 시도, 또는 건너뛰기. 조용히 단일 모델로 후퇴하지 마세요 — 사용자는 교차 모델이 일어나지 않았음을 알아야 합니다.

**Step 4: 사용자가 건너뛰면**

출력에 건너뛰기를 인정(*"Proceeding with single-model findings only"*)하고 RECONCILE로 계속하세요. 건너뛰기는 괜찮습니다; 조용한 건너뛰기는 아닙니다.

**비상호작용 context** (CI, `/loop`, autonomous-loop, 스케줄 실행):

- 교차 모델은 **건너뜀**, 그리고 건너뛰기는 출력에 **공지**되어야 함: *"Cross-model skipped: non-interactive context."*
- **명시적 사용자 승인 없이 외부 CLI를 절대 invoke하지 마세요** — 이것은 핵심 안전 속성입니다.

교차 모델은 비용, 지연, 도구 취약성을 더합니다. agent는 매 cycle 선택지를 표면화하고; 사용자가 이 artifact가 그것을 정당화하는지 결정합니다.

### Step 4: RECONCILE — 발견을 다시 접기

리뷰어의 출력은 데이터이지 평결이 아닙니다. **당신이 여전히 오케스트레이터입니다.** 분류 전에 각 발견에 대해 artifact 텍스트를 다시 읽으세요 — 리뷰어를 고무 도장 찍는 것은 그것을 무시하는 것과 같은 실패 모드입니다.

각 발견에 대해, 이 **우선순위 순서**로 분류 (첫 번째 매칭 클래스가 이김):

1. **Contract 오독** — 제공한 CONTRACT가 불명확하거나 불완전했기 때문에 리뷰어가 특정 무언가를 표시. contract를 먼저 고치고, 다음 cycle에 재분류.
2. **유효 + 실행 가능** — artifact 변경을 요구하는 실제 이슈. 변경하고, 다시 loop.
3. **유효한 trade-off** — 이슈는 실제지만 고치는 비용이 받아들이는 비용을 초과. 사용자가 보도록 trade-off를 명시적으로 문서화.
4. **노이즈** — 리뷰어가 갖지 못한 context 하에서 실제로 맞는 무언가를 표시. 기록하고, 넘어가고, 묻기: 그 context를 contract에 추가했다면 false flag를 방지했을까?

신규 리뷰어는 context가 부족해 틀릴 수 있습니다. "신규"라는 이유만으로 양보하지 마세요.

### Step 5: STOP — 재귀가 아니라 경계된 loop

다음일 때 멈추세요:

- 다음 반복이 사소하거나 이미 고려된 발견만 반환, **또는**
- 3 cycle 완료 (사용자에게 escalate, 혼자 네 번째를 갈지 말 것), **또는**
- 사용자가 명시적으로 "ship it"이라고 함

3 cycle 후에도 리뷰어가 여전히 실질적 이슈를 표면화하면, artifact가 준비되지 않았을 수 있습니다. 이를 사용자에게 표면화하세요 — 해결되지 않은 세 cycle은 계속 loop할 이유가 아니라 artifact에 대한 정보입니다.

artifact가 커서 3 cycle이 "명백히 불충분"하다면: artifact가 너무 큽니다 — Step 2로 돌아가 분해하세요. 경계를 들어 올리지 마세요.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "확신하니, doubt 단계를 건너뛰자" | 확신은 새로운 문제에서 정확성과 상관이 낮습니다. 확신의 순간이 바로 맹점이 숨는 때입니다. |
| "리뷰어 스폰은 비싸" | production의 잘못된 commit을 디버깅하는 게 더 비쌉니다. 검사는 경계됨; 버그는 아님. |
| "리뷰어는 그냥 nitpick할 거야" | 범위가 없을 때만. 프롬프트를 "contract 하에서 이를 실패시킬 이슈"로 제약하세요. |
| "doubt는 끝에 `/review`로 할게" | `/review`는 최종 게이트입니다. doubt-driven은 방향 수정이 저렴할 때 잘못된 방향을 일찍 잡습니다. PR 시점이면 너무 늦습니다. |
| "모든 단계를 의심하면 ship 못 해" | skill은 모든 키 입력이 아니라 비자명한 결정에 적용됩니다. "When NOT to Use"를 다시 읽으세요. |
| "두 의견이 항상 하나보다 나아" | 두 번째가 context가 적고 노이즈를 생성할 때는 아닙니다. 양보하지 말고 reconcile하세요. |
| "리뷰어가 동의하지 않았으니 내가 틀렸어" | 리뷰어는 당신의 context가 부족함 — 불일치는 평결이 아니라 정보입니다. artifact를 다시 읽고, 분류하고, 결정하세요. |
| "교차 모델은 항상 더 나아" | 교차 모델은 단일 모델이 자신과 공유하는 맹점을 잡지만, 비용과 도구 취약성을 더합니다. 매 상호작용 doubt cycle에 제안 — artifact가 그것을 정당화하는지 사용자가 결정. agent의 일은 선택지를 표면화하는 것이지 gate하는 것이 아닙니다. |
| "사용자가 한 번 yes 했으니, CLI를 계속 invoke해도 돼" | 각 호출은 자체 승인입니다. artifact, 프롬프트, flag가 호출 간 바뀝니다 — 매 실행 전 정확한 커맨드를 사용자와 재확인하세요. |

## Red Flags

- 한 줄 이름 변경이나 포맷팅 변경에 신규-context 리뷰어 스폰
- artifact 텍스트를 다시 읽지 않고 리뷰어 출력을 권위 있게 취급
- 사용자에게 escalate하지 않고 3 cycle 초과 loop
- 리뷰어에게 "이게 좋아?"가 아니라 "이슈를 찾아"로 프롬프트
- 위험이 높은 결정에 시간 압박으로 doubt 건너뛰기
- 변경되지 않은 artifact에 신규-context 재스폰 (같은 발견을 받음; 지연 중)
- **Doubt theater (확인 가능한 신호)**: 리뷰어가 실질적 발견을 표면화한 2개 이상 cycle에 걸쳐, 어떤 발견도 실행 가능으로 분류되지 않음. 의심이 아니라 검증 중입니다. 멈추고 escalate.
- commit 후에만 의심 — 그것은 doubt-driven development가 아니라 `/review`
- 도구가 존재하고, 구성되었고, 그 정확한 문법을 받는지 사용자와 확인하지 않고 외부 CLI 호출을 하드코딩
- **상호작용 doubt cycle에서 교차 모델을 조용히 건너뛰기.** 권장하지 않을 때라도, 제안은 보여야 합니다. 건너뛰기는 괜찮음; 조용한 건너뛰기는 아님.
- 외부 CLI가 에러나거나 없을 때 조용히 후퇴 — 실패를 표면화하고 사용자가 방향을 바꾸게 하세요
- 리뷰어 입력에서 contract 제거
- CLAIM을 리뷰어에 전달 (동의 쪽으로 편향)

## 다른 Skill과의 상호작용

- **`code-review-and-quality` / `/review`**: 상호 보완적. `/review`는 사후 PR 평결; doubt-driven은 진행 중 결정별. 둘 다 사용.
- **`source-driven-development`**: SDD는 *프레임워크에 대한 사실*을 공식 문서에 대해 검증. doubt-driven은 *artifact에 대한 당신의 추론*을 검증. SDD는 API가 존재하는지 확인; doubt-driven은 contract 하에서 그것을 올바르게 썼는지 확인.
- **`test-driven-development`**: TDD의 RED 단계는 구체화된 의심입니다 — 실패하는 테스트는 반증 시도. TDD가 적용될 때, 그 실패하는 테스트가 행동적 주장에 대한 doubt 단계 *그 자체*입니다.
- **`debugging-and-error-recovery`**: 리뷰어가 실제 실패 모드를 표면화하면, 디버깅 skill로 들어가 위치를 파악하고 수정.
- **Repo 오케스트레이션 규칙** (`references/orchestration-patterns.md`): 이 skill은 메인 세션에서 오케스트레이션합니다. persona가 다른 persona를 호출하는 것은 안티패턴 B입니다 — 위 로딩 제약 참고.

## Verification

doubt-driven development 적용 후:

- [ ] 모든 비자명한 결정(위 정의에 따라)이 확정 전 CLAIM으로 명시적으로 명명됨
- [ ] 비자명한 artifact당 최소 한 번의 신규-context 리뷰 (TDD의 RED 단계가 생성한 실패하는 테스트는 다른 Skill과의 상호작용에 따라 행동적 주장에 대해 이를 충족)
- [ ] 리뷰어가 ARTIFACT + CONTRACT를 받음 — CLAIM 아님, 당신의 추론 아님
- [ ] 리뷰어의 프롬프트가 적대적("이슈를 찾아")이었음, 검증("좋아?")이 아니라
- [ ] 발견이 우선순위(contract 오독 / 실행 가능 / trade-off / 노이즈)를 사용해 artifact 텍스트에 대해 분류됨 (고무 도장 아님)
- [ ] 정지 조건 충족 (사소한 발견, 3 cycle, 또는 사용자 override)
- [ ] 상호작용 모드에서, 교차 모델이 사용자에게 **명시적으로 제안**됨 (artifact 위험과 무관) 그리고 응답이 출력에 인정됨
- [ ] 비상호작용 모드에서, 교차 모델이 건너뛰어졌고 건너뛰기가 공지됨
- [ ] 모든 외부 CLI 호출이 PATH 확인, 동작 binary 테스트, 사용자와의 문법 확인, 실행에 대한 명시적 승인을 앞서 거침
