---
name: code-review-and-quality
description: multi-axis 코드 검토를 수행합니다. 변경 사항을 병합하기 전에 사용하세요. 자신, 다른 agent 또는 사람이 작성한 코드를 검토할 때 사용합니다. 기본 분기에 들어가기 전에 여러 차원에서 코드 품질을 평가해야 할 때 사용합니다.
---

# 코드 검토 및 품질

## 개요

품질 게이트를 통한 다차원 코드 검토. 모든 변경 사항은 병합 전에 검토되며 예외는 없습니다. 검토에서는 정확성, 가독성, 아키텍처, 보안 및 성능ormance의 5개 축을 다룹니다.

**승인 표준:** 완벽하지 않더라도 전반적인 코드 상태가 확실히 개선되면 변경을 승인합니다. 완벽한 코드는 존재하지 않습니다. 목표는 지속적인 개선입니다. 변경 사항이 정확히 여러분이 작성한 방식이 아니기 때문에 변경 사항을 차단하지 마십시오. 코드베이스를 개선하고 프로젝트 규칙을 따른다면 승인하세요.

## 사용 시기

- PR를 병합하거나 변경하기 전
- 기능 구현을 완료한 후
- 다른 agent 또는 모델이 코드를 생성한 경우 평가해야 합니다.
- 기존 코드를 리팩토링할 때
- 버그 수정 후(수정 사항과 회귀 테스트를 모두 검토)

## 5축 검토

모든 검토는 다음 차원에서 코드를 평가합니다.

### 1. 정확성

코드가 주장하는 대로 수행됩니까?

- 요구되는 사양이나 작업과 일치합니까?
- 엣지 케이스(null, 비어 있음, 경계 값)가 처리됩니까?
- 오류 경로가 처리됩니까(해피 경로뿐만 아니라)?
- 모든 테스트를 통과했나요? 테스트가 실제로 올바른 것을 테스트하고 있습니까?
- off-by-one 오류, 경쟁 조건 또는 상태 불일치가 있습니까?

### 2. 가독성 및 단순성

작성자의 설명 없이 다른 엔지니어(또는 agent)가 이 코드를 이해할 수 있습니까?

- 이름은 설명적이고 프로젝트 규칙과 일치합니까? (컨텍스트 없이 `temp`, `data`, `result` 없음)
- 제어 흐름이 간단합니까(중첩된 삼항, 심층 콜백 방지)?
- Is the code organized logically (related code grouped, clear module boundaries)?
- 단순화해야 할 "영리한" 트릭이 있나요?
- **더 적은 줄로 이 작업을 수행할 수 있습니까?** (100줄이면 충분할 경우 1000줄은 실패)
- **추상화로 인해 복잡성이 발생합니까?** (세 번째 사용 사례까지 일반화하지 마세요)
- 의견이 non-obvious 의도를 명확히 하는 데 도움이 됩니까? (단, 뻔한 코드는 주석 처리하지 마세요.)
- 데드 코드 아티팩트(no-op 변수(`_unused`), backwards-compat shim 또는 `// removed` 주석)가 있습니까?

### 3. 아키텍처

변경 사항이 시스템 설계에 적합합니까?

- 기존 패턴을 따르는가, 아니면 새로운 패턴을 도입하는가? 새로운 것이라면 정당한가?
- 깨끗한 모듈 경계를 유지하는가?
- 공유해야 할 코드 중복이 있나요?
- 종속성이 올바른 방향으로 흐르고 있습니까(순환 종속성이 없음)?
- 추상화 수준이 적절한가요(over-engineered도 아니고 너무 결합되지도 않음)?

### 4. 보안

자세한 보안 guidance는 `security-and-hardening`를 참조하세요. 변경으로 인해 취약점이 발생합니까?

- 사용자 입력이 검증되고 삭제됩니까?
- 코드, 로그 및 버전 관리에서 비밀이 유지됩니까?
- 필요한 곳에 인증/authorization를 체크하는가?
- SQL 쿼리가 매개변수화되어 있습니까(문자열 연결 없음)?
- XSS를 방지하기 위해 출력이 인코딩되어 있습니까?
- 알려진 취약점 없이 신뢰할 수 있는 소스로부터의 종속성이 있습니까?
- 외부 소스(APIs, 로그, 사용자 콘텐츠, 구성 파일)의 데이터는 신뢰할 수 없는 것으로 처리됩니까?
- 로직이나 렌더링에 사용하기 전에 시스템 경계에서 외부 데이터 흐름이 검증됩니까?

### 5. 성능ormance

자세한 프로파일링 및 최적화는 `performance-optimization`를 참조하세요. 변경으로 인해 performance 문제가 발생합니까?

- N+1 쿼리 패턴이 있나요?
- 무제한 루프 또는 무제한 데이터 가져오기가 있습니까?
- 비동기식이어야 하는 동기 작업이 있나요?
- UI 구성 요소에 불필요한 re-renders가 있습니까?
- 목록 끝점에 누락된 페이지 매김이 있습니까?
- 핫 패스에 대형 개체가 생성되었습니까?

## 크기 변경

작고 집중된 변경 사항은 검토하기 쉽고, 병합하기 쉽고, 배포하기 더 안전합니다. 다음 크기를 타겟팅하세요.

```
~100 lines changed   → Good. Reviewable in one sitting.
~300 lines changed   → Acceptable if it's a single logical change.
~1000 lines changed  → Too large. Split it.
```

**"한 가지 변경 사항"으로 간주되는 것:** 한 가지 사항을 해결하고 관련 테스트를 포함하며 제출 후 시스템 기능을 유지하는 단일 self-contained 수정입니다. 전체 기능이 아닌 기능의 한 부분입니다.

**변경 사항이 너무 큰 경우 분할 전략:**

| 전략 | 어떻게 | 언제 |
|----------|-----|------|
| **스택** | 작은 변경사항을 제출하고 이를 기반으로 다음 변경사항을 시작하세요 | 순차적 종속성 |
| **파일 그룹별** | 다른 검토자가 필요한 그룹을 위한 개별 변경 | 교차적 우려 |
| **수평** | 먼저 공유 코드/stubs를 생성한 다음 소비자 | 계층화된 아키텍처 |
| **수직** | 기능의 더 작은 full-stack 조각으로 나누기 | 특집 작품 |

**대규모 변경이 허용되는 경우:** 검토자가 모든 줄이 아닌 의도만 확인하면 되는 완전한 파일 삭제 및 자동화된 리팩토링.

**기능 작업과 별도의 리팩토링.** 기존 코드를 리팩터링하고 새로운 동작을 추가하는 변경 사항은 두 가지 변경 사항입니다. 별도로 제출하세요. 검토자의 재량에 따라 소규모 정리(변수 이름 변경)가 포함될 수 있습니다.

## 변경 설명

모든 변경에는 버전 제어 기록에 독립적인 설명이 필요합니다.

**첫 번째 줄:** 짧고, 명령적이며, 독립형입니다. "FizzBuzz RPC 삭제"가 아니라 "FizzBuzz RPC를 삭제"합니다. 기록을 검색하는 사람이 diff를 읽지 않고도 변경 사항을 이해할 수 있을 만큼 충분히 informative여야 합니다.

**본문:** 변경되는 사항과 그 이유 코드 자체에는 보이지 않는 컨텍스트, 결정 및 추론을 포함합니다. 관련된 경우 버그 번호, 벤치마크 결과 또는 디자인 문서에 대한 링크입니다. 접근 방식의 단점이 있는 경우 이를 인정합니다.

**Anti-patterns:** "버그 수정", "build 수정", "패치 추가", "A에서 B로 코드 이동", "1단계", "편의 기능 추가"

## 검토 프로세스

### 1단계: 맥락 이해

코드를 보기 전에 의도를 이해하세요.

```
- What is this change trying to accomplish?
- What spec or task does it implement?
- What is the expected behavior change?
```

### 2단계: 먼저 테스트 검토

테스트를 통해 의도와 적용 범위가 드러납니다.

```
- Do tests exist for the change?
- Do they test behavior (not implementation details)?
- Are edge cases covered?
- Do tests have descriptive names?
- Would the tests catch a regression if the code changed?
```

### 3단계: 구현 검토

5개 축을 염두에 두고 코드를 살펴보세요.

```
For each file changed:
1. Correctness: Does this code do what the test says it should?
2. Readability: Can I understand this without help?
3. Architecture: Does this fit the system?
4. Security: Any vulnerabilities?
5. Performance: Any bottlenecks?
```

### 4단계: 조사 결과 분류

작성자가 required와 선택 사항을 알 수 있도록 모든 댓글에 심각도 라벨을 지정하세요.

| 접두사 | 의미 | 작성자 작업 |
|--------|---------|---------------|
| *(접두사 없음)* | Requi빨간색 변경 | 병합 전에 주소를 지정해야 함 |
| **긴급:** | 블록 병합 | 보안 취약성, 데이터 손실, 기능 손상 |
| **니트:** | 미성년자, 선택사항 | 작성자는 무시할 수 있습니다 — formatting, 스타일 기본 설정 |
| **선택 사항:** / **고려 사항:** | 제안 | 고려해 볼 가치는 있지만 required |
| **FYI** | Informational 전용 | 조치가 필요하지 않습니다. 향후 참조를 위한 컨텍스트 |

이렇게 하면 작성자가 모든 피드백을 필수로 처리하고 선택적 제안에 시간을 낭비하는 것을 방지할 수 있습니다.

### 5단계: 인증 확인

저자의 검증사례를 확인해보세요:

```
- What tests were run?
- Did the build pass?
- Was the change tested manually?
- Are there screenshots for UI changes?
- Is there a before/after comparison?
```

## 다중 모델 검토 패턴

다양한 검토 관점에 대해 다양한 모델을 사용하세요.

```
Model A writes the code
    │
    ▼
Model B reviews for correctness and architecture
    │
    ▼
Model A addresses the feedback
    │
    ▼
Human makes the final call
```

이는 단일 모델이 놓칠 수 있는 문제를 포착합니다. 모델마다 사각지대가 다릅니다.

**검토를 위한 예시 프롬프트 agent:**
```
Review this code change for correctness, security, and adherence to
our project conventions. The spec says [X]. The change should [Y].
Flag any issues as Critical, Important, or Suggestion.
```

## 데드 코드 위생

리팩토링 또는 구현 변경 후 분리된 코드가 있는지 확인하세요.

1. 현재 연결할 수 없거나 사용되지 않는 코드 식별
2. 명시적으로 나열하세요.
3. **삭제하기 전에 확인하십시오.** "이 now-unused 요소: [목록]을(를) 제거해야 합니까?"

데드 코드를 방치하지 마십시오. 이는 미래의 독자와 agents를 혼란스럽게 합니다. 하지만 확실하지 않은 내용을 조용히 삭제하지 마세요. 의심스러우면 물어보세요.

```
DEAD CODE IDENTIFIED:
- formatLegacyDate() in src/utils/date.ts — replaced by formatDate()
- OldTaskCard component in src/components/ — replaced by TaskCard
- LEGACY_API_URL constant in src/config.ts — no remaining references
→ Safe to remove these?
```

## 검토 속도

느린 검토는 전체 팀을 차단합니다. context-switching의 검토 비용은 다른 사람에게 부과되는 대기 비용보다 적습니다.

- **영업일 기준 1일 이내에 응답** - 이는 목표가 아닌 최대치입니다.
- **이상적인 흐름:** 집중적인 코딩에 깊이 관여하지 않는 한 검토 요청이 도착한 직후에 응답합니다. 일반적인 변경은 하루에 여러 번의 검토를 완료해야 합니다.
- quick 최종 승인보다 **빠른 개별 응답을 우선**합니다. Quick 피드백은 여러 라운드가 필요한 경우에도 불만을 줄여줍니다.
- **대규모 변경:** 하나의 대규모 변경 세트를 검토하기보다는 작성자에게 분할하도록 요청하세요.

## 불일치 처리

리뷰 분쟁을 해결할 때 다음 계층 구조를 적용하세요.

1. **기술적 사실과 데이터**는 의견과 선호도보다 우선합니다.
2. **스타일 guides**는 스타일 문제에 대한 절대적인 권위입니다.
3. **소프트웨어 설계**는 개인 취향이 아닌 엔지니어링 원칙에 따라 평가되어야 합니다.
4. **코드베이스 일관성**은 전반적인 상태를 저하시키지 않는 한 허용됩니다.

**"나중에 정리하겠습니다."라는 말을 받아들이지 마세요.** 경험에 따르면 정리가 지연되는 경우는 거의 발생하지 않습니다. genuine 긴급 상황이 아닌 이상 제출하기 전에 Require 정리를 수행하세요. 이 변경으로 주변 문제를 해결할 수 없는 경우, require가 self-assignment로 버그를 신고하세요.

## 정직성 검토

귀하, 다른 agent 또는 사람이 작성한 코드를 검토할 때:

- **rubber-stamp를 사용하지 마세요.** 검토 증거가 없는 "LGTM"는 누구에게도 도움이 되지 않습니다.
- **실제 문제를 완화하지 마십시오.** 프로덕션에 영향을 미칠 버그인 경우 "이것은 사소한 문제일 수 있습니다"는 부정직합니다.
- **가능한 경우 문제를 정량화합니다.** "이 N+1 쿼리는 목록의 항목당 ~50ms를 추가합니다."가 "느릴 수 있습니다."보다 낫습니다.
- **명백한 문제가 있는 접근 방식은 뒤로 미루세요.** 아첨은 검토에서 실패 모드입니다. 구현에 문제가 있으면 직접적으로 말하고 대안을 제안하세요.
- **재정의를 우아하게 수락하세요.** 작성자가 전체 맥락을 알고 동의하지 않는 경우 작성자의 판단을 따르세요. 사람이 아닌 코드에 댓글을 다세요. 개인적인 비평의 틀을 재구성하여 코드 자체에 초점을 맞추세요.

## 의존성 규율

코드 검토의 일부는 종속성 검토입니다.

**종속성을 추가하기 전에:**
1. 기존 스택이 이 문제를 해결합니까? (종종 그렇습니다.)
2. 종속성은 얼마나 큽니까? (번들이 미치는 영향을 확인하세요.)
3. 적극적으로 유지관리되고 있나요? (마지막 커밋, 미해결 문제를 확인하세요.)
4. 알려진 취약점이 있습니까? (`npm audit`)
5. 라이센스는 무엇입니까? (프로젝트와 호환되어야 합니다.)

**규칙:** 새로운 종속성보다 표준 라이브러리와 기존 유틸리티를 선호하세요. 모든 의존성은 책임입니다.

## 검토 체크리스트

```markdown
## Review: [PR/Change title]

### Context
- [ ] I understand what this change does and why

### Correctness
- [ ] Change matches spec/task requirements
- [ ] Edge cases handled
- [ ] Error paths handled
- [ ] Tests cover the change adequately

### Readability
- [ ] Names are clear and consistent
- [ ] Logic is straightforward
- [ ] No unnecessary complexity

### Architecture
- [ ] Follows existing patterns
- [ ] No unnecessary coupling or dependencies
- [ ] Appropriate abstraction level

### Security
- [ ] No secrets in code
- [ ] Input validated at boundaries
- [ ] No injection vulnerabilities
- [ ] Auth checks in place
- [ ] External data sources treated as untrusted

### Performance
- [ ] No N+1 patterns
- [ ] No unbounded operations
- [ ] Pagination on list endpoints

### Verification
- [ ] Tests pass
- [ ] Build succeeds
- [ ] Manual verification done (if applicable)

### Verdict
- [ ] **Approve** — Ready to merge
- [ ] **Request changes** — Issues must be addressed
```
## 참고 항목

- 자세한 보안 검토는 guidance를 참조하세요. `references/security-checklist.md`
- performance 검토 확인은 `references/performance-checklist.md`를 참조하세요.

## 일반적인 합리화

| 합리화 | 현실 |
|---|---|
| "작동합니다. 그것만으로도 충분합니다." | 읽을 수 없거나 안전하지 않거나 구조적으로 잘못된 작업 코드는 복합적인 부채를 생성합니다. |
| "내가 썼기 때문에 그것이 맞다는 것을 압니다" | 저자는 자신의 가정에 눈이 멀었습니다. 모든 변화는 다른 관점에서 이익을 얻습니다. |
| "나중에 정리하겠습니다" | 나중에는 절대 오지 않습니다. 리뷰는 품질의 관문입니다. 이를 활용하세요. 병합 후가 아닌 병합 전에 quire 정리가 필요합니다. |
| "AI 생성 코드는 아마도 괜찮을 것입니다." | AI 코드는 더 많은 조사가 필요합니다. 틀렸을 때에도 자신감 있고 그럴듯합니다. |
| "테스트를 통과했으니 괜찮아요" | 테스트는 필요하지만 충분하지는 않습니다. 아키텍처 문제, 보안 문제 또는 가독성 문제를 파악하지 못합니다. |

## 위험 신호

- PRs는 검토 없이 병합되었습니다.
- 테스트 통과 여부만 확인하는 검토 (다른 축은 무시)
- 실제 리뷰의 증거가 없는 "LGTM"
- security-focused 검토 없이 보안에 민감한 변경
- "너무 커서 제대로 검토할 수 없는" 대형 PRs(분할)
- 버그 수정 PRs로 회귀 테스트 없음
- 심각도 라벨 없이 주석을 검토합니다. 필수 항목과 선택 항목이 무엇인지 명확하지 않게 만듭니다.
- "나중에 고칠게요"를 받아들이는 것 — 그런 일은 절대 일어나지 않습니다

## 확인

검토가 완료된 후:

- [ ] 모든 중요한 문제가 해결되었습니다.
- [ ] 모든 중요한 문제가 해결되었거나 정당한 사유로 명시적으로 연기되었습니다.
- [ ] 테스트 통과
- [ ] Build 성공
- [ ] 검증 스토리가 문서화되어 있습니다(변경된 내용, 어떻게 검증되었는지).
