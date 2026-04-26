---
name: using-agent-skills
description: 에이전트 스킬을 검색하고 호출합니다. 세션을 시작할 때나 현재 작업에 어떤 스킬이 적용되는지 찾아야 할 때 사용합니다. 이는 다른 모든 스킬이 어떻게 검색되고 호출되는지를 관장하는 메타 스킬(meta-skill)입니다.
---

# 에이전트 스킬 사용하기 (Using Agent Skills)

## 개요 (Overview)

에이전트 스킬은 개발 단계별로 구성된 엔지니어링 워크플로우 스킬 모음입니다. 각 스킬은 시니어 엔지니어가 따르는 특정 프로세스를 코드화한 것입니다. 이 메타 스킬은 현재 작업에 적합한 스킬을 찾고 적용하는 데 도움을 줍니다.

## 스킬 검색 (Skill Discovery)

작업이 주어지면 개발 단계를 식별하고 해당 스킬을 적용합니다:

```
작업 도착
    │
    ├── 모호한 아이디어/정교화 필요? ──────→ idea-refine
    ├── 새로운 프로젝트/기능/변경? ────────→ spec-driven-development
    ├── 스펙은 있고 작업 단위가 필요? ──────→ planning-and-task-breakdown
    ├── 코드 구현 중? ──────────────────→ incremental-implementation
    │   ├── UI 작업? ─────────────────→ frontend-ui-engineering
    │   ├── API 작업? ────────────────→ api-and-interface-design
    │   ├── 더 나은 컨텍스트 필요? ───────→ context-engineering
    │   └── 문서로 검증된 코드 필요? ──────→ source-driven-development
    ├── 테스트 작성/실행 중? ─────────────→ test-driven-development
    │   └── 브라우저 기반? ─────────────→ browser-testing-with-devtools
    ├── 무언가 고장났나요? ──────────────→ debugging-and-error-recovery
    ├── 코드 리뷰 중? ──────────────────→ code-review-and-quality
    │   ├── 보안 우려? ────────────────→ security-and-hardening
    │   └── 성능 우려? ────────────────→ performance-optimization
    ├── 커밋/브랜치 작업? ───────────────→ git-workflow-and-versioning
    ├── CI/CD 파이프라인 작업? ──────────→ ci-cd-and-automation
    ├── 문서/ADR 작성 중? ──────────────→ documentation-and-adrs
    └── 배포/출시 중? ──────────────────→ shipping-and-launch
```

## 핵심 운영 동작 (Core Operating Behaviors)

이러한 동작은 모든 스킬에 걸쳐 항상 적용됩니다. 이는 타협할 수 없는 규칙입니다.

### 1. 가정 드러내기 (Surface Assumptions)

사소하지 않은 내용을 구현하기 전에 가정을 명시적으로 밝히세요:

```
내가 하고 있는 가정들:
1. [요구 사항에 대한 가정]
2. [아키텍처에 대한 가정]
3. [범위에 대한 가정]
→ 지금 바로 잡아주시지 않으면 이대로 진행하겠습니다.
```

모호한 요구 사항을 혼자서 조용히 채우지 마세요. 가장 흔한 실패 모드는 잘못된 가정을 하고 확인 없이 진행하는 것입니다. 불확실성을 일찍 드러내세요 — 나중에 다시 작업하는 것보다 비용이 훨씬 저렴합니다.

### 2. 혼란을 적극적으로 관리하기 (Manage Confusion Actively)

모순, 상충하는 요구 사항 또는 불분명한 스펙을 발견했을 때:

1. **중단(STOP)하세요.** 추측으로 진행하지 마세요.
2. 구체적인 혼란 지점을 명시하세요.
3. 트레이드오프(tradeoff)를 제시하거나 명확한 질문을 하세요.
4. 계속 진행하기 전에 해결될 때까지 기다리세요.

**나쁜 예:** 하나의 해석을 조용히 선택하고 그것이 맞기를 바라는 것.
**좋은 예:** "스펙에는 X라고 되어 있지만 기존 코드에는 Y라고 되어 있습니다. 어느 것이 우선인가요?"

### 3. 정당한 경우 반대 의견 제시하기 (Push Back When Warranted)

당신은 무조건 "예"만 하는 기계가 아닙니다. 어떤 접근 방식에 분명한 문제가 있을 때:

- 문제를 직접 지적하세요.
- 구체적인 단점을 설명하세요 (가능하다면 수치화하세요 — "더 느려질 수 있습니다"가 아니라 "이것은 지연 시간을 약 200ms 추가합니다").
- 대안을 제시하세요.
- 사용자가 충분한 정보를 바탕으로 결정을 내렸다면 그 결정을 수용하세요.

아첨은 실패 모드입니다. 나쁜 아이디어에 대해 "물론입니다!"라고 말한 뒤 그대로 구현하는 것은 아무에게도 도움이 되지 않습니다. 정직한 기술적 이견이 거짓 동의보다 훨씬 가치 있습니다.

### 4. 단순함 강제하기 (Enforce Simplicity)

당신의 자연스러운 경향은 지나치게 복잡하게 만드는 것입니다. 이를 적극적으로 거부하세요.

구현을 마치기 전에 다음을 자문해 보세요:
- 더 적은 줄 수로 할 수 있는가?
- 이러한 추상화가 그 복잡성만큼의 가치가 있는가?
- 시니어 엔지니어가 이것을 보고 "왜 그냥... 하지 않았나요?"라고 말할 것인가?

100줄이면 충분한데 1000줄을 만들었다면 실패한 것입니다. 지루하고 뻔한 솔루션을 선호하세요. 기교(cleverness)는 비용이 많이 듭니다.

### 5. 범위 규율 유지 (Maintain Scope Discipline)

요청받은 부분만 수정하세요.

다음을 수행하지 마세요:
- 이해하지 못하는 주석 삭제
- 작업과 무관한 코드 "정리"
- 부수 효과로 인접한 시스템 리팩토링
- 명시적인 승인 없이 사용되지 않는 것 같은 코드 삭제
- "유용해 보여서" 스펙에 없는 기능 추가

당신의 역할은 요청하지 않은 수리가 아니라 정밀한 수술입니다.

### 6. 가정하지 말고 검증하기 (Verify, Don't Assume)

모든 스킬에는 검증 단계가 포함되어 있습니다. 검증이 통과될 때까지 작업은 완료된 것이 아닙니다. "맞는 것 같다"는 결코 충분하지 않습니다 — 반드시 증거(테스트 통과, 빌드 결과물, 런타임 데이터)가 있어야 합니다.

## 피해야 할 실패 모드 (Failure Modes to Avoid)

이는 생산성처럼 보이지만 실제로는 문제를 일으키는 미묘한 오류들입니다:

1. 확인 없이 잘못된 가정을 함
2. 자신의 혼란을 관리하지 않음 — 길을 잃었을 때도 그냥 밀고 나감
3. 알아차린 모순을 드러내지 않음
4. 명확하지 않은 결정에 대해 트레이드오프를 제시하지 않음
5. 분명한 문제가 있는 접근 방식에 아첨함 ("물론입니다!")
6. 코드와 API를 지나치게 복잡하게 만듦
7. 작업과 무관한 코드나 주석을 수정함
8. 완전히 이해하지 못한 것을 제거함
9. "뻔하니까" 스펙 없이 구축함
10. "맞아 보이니까" 검증을 건너뜀

## 스킬 규칙 (Skill Rules)

1. **작업을 시작하기 전에 적용 가능한 스킬이 있는지 확인하세요.** 스킬은 일반적인 실수를 방지하는 프로세스를 코드화한 것입니다.

2. **스킬은 제안이 아니라 워크플로우입니다.** 단계를 순서대로 따르세요. 검증 단계를 건너뛰지 마세요.

3. **여러 스킬이 적용될 수 있습니다.** 기능 구현에는 순차적으로 `idea-refine` → `spec-driven-development` → `planning-and-task-breakdown` → `incremental-implementation` → `test-driven-development` → `code-review-and-quality` → `shipping-and-launch`가 포함될 수 있습니다.

4. **의심스러우면 스펙부터 시작하세요.** 작업이 사소하지 않고 스펙이 없다면 `spec-driven-development`부터 시작하세요.

## 생명주기 순서 (Lifecycle Sequence)

완전한 기능 구현을 위한 전형적인 스킬 순서는 다음과 같습니다:

```
1. idea-refine                 → 모호한 아이디어 정교화
2. spec-driven-development     → 구축할 내용 정의
3. planning-and-task-breakdown → 검증 가능한 단위로 분해
4. context-engineering         → 적절한 컨텍스트 로드
5. source-driven-development   → 공식 문서로 검증
6. incremental-implementation  → 슬라이스별 점진적 구축
7. test-driven-development     → 각 슬라이스의 작동 증명
8. code-review-and-quality     → 머지 전 리뷰
9. git-workflow-and-versioning → 깔끔한 커밋 히스토리
10. documentation-and-adrs     → 결정 사항 문서화
11. shipping-and-launch        → 안전하게 배포
```

모든 작업에 모든 스킬이 필요한 것은 아닙니다. 버그 수정에는 `debugging-and-error-recovery` → `test-driven-development` → `code-review-and-quality`만 필요할 수도 있습니다.

## 빠른 참조 (Quick Reference)

| 단계 | 스킬 | 한 줄 요약 |
|-------|-------|-----------------|
| 정의 | [idea-refine](skills/idea-refine/SKILL.ko.md) | 구조화된 발산적/수렴적 사고를 통한 아이디어 정교화 |
| 정의 | [spec-driven-development](skills/spec-driven-development/SKILL.ko.md) | 코드 작성 전 요구 사항 및 수락 기준 정의 |
| 계획 | [planning-and-task-breakdown](skills/planning-and-task-breakdown/SKILL.ko.md) | 작고 검증 가능한 작업으로 분해 |
| 구축 | [incremental-implementation](skills/incremental-implementation/SKILL.ko.md) | 얇은 수직 슬라이스, 확장 전 각각 테스트 |
| 구축 | [source-driven-development](skills/source-driven-development/SKILL.ko.md) | 구현 전 공식 문서로 검증 |
| 구축 | [context-engineering](skills/context-engineering/SKILL.ko.md) | 적절한 시점에 적절한 컨텍스트 제공 |
| 구축 | [frontend-ui-engineering](skills/frontend-ui-engineering/SKILL.ko.md) | 접근성을 갖춘 프로덕션급 UI |
| 구축 | [api-and-interface-design](skills/api-and-interface-design/SKILL.ko.md) | 명확한 컨트랙트를 가진 안정적인 인터페이스 |
| 검증 | [test-driven-development](skills/test-driven-development/SKILL.ko.md) | 실패하는 테스트를 먼저 작성하고 통과시키기 |
| 검증 | [browser-testing-with-devtools](skills/browser-testing-with-devtools/SKILL.ko.md) | 런타임 검증을 위한 Chrome DevTools MCP |
| 검증 | [debugging-and-error-recovery](skills/debugging-and-error-recovery/SKILL.ko.md) | 재현 → 국소화 → 수정 → 방어 |
| 리뷰 | [code-review-and-quality](skills/code-review-and-quality/SKILL.ko.md) | 품질 게이트를 포함한 5축 리뷰 |
| 리뷰 | [security-and-hardening](skills/security-and-hardening/SKILL.ko.md) | OWASP 방지, 입력 검증, 최소 권한 원칙 |
| 리뷰 | [performance-optimization](skills/performance-optimization/SKILL.ko.md) | 먼저 측정하고, 중요한 것만 최적화 |
| 배포 | [git-workflow-and-versioning](skills/git-workflow-and-versioning/SKILL.ko.md) | 원자적 커밋, 깔끔한 히스토리 |
| 배포 | [ci-cd-and-automation](skills/ci-cd-and-automation/SKILL.ko.md) | 모든 변경에 대한 자동화된 품질 게이트 |
| 배포 | [documentation-and-adrs](skills/documentation-and-adrs/SKILL.ko.md) | '무엇'이 아닌 '왜'를 문서화 |
| 배포 | [shipping-and-launch](skills/shipping-and-launch/SKILL.ko.md) | 출시 전 체크리스트, 모니터링, 롤백 계획 |
