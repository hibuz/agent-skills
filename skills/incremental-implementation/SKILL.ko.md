---
name: incremental-implementation
description: 변경을 점진적으로 전달. 두 개 이상의 파일을 건드리는 기능이나 변경을 구현할 때 사용. 한 번에 많은 코드를 작성하려 할 때, 또는 작업이 한 단계에 land하기엔 너무 커 보일 때 사용.
---

# Incremental Implementation

## Overview

얇은 수직 슬라이스로 빌드하세요 — 한 조각을 구현하고, 테스트하고, 검증한 뒤, 확장하세요. 전체 기능을 한 번에 구현하는 것을 피하세요. 각 증분은 시스템을 동작하고 테스트 가능한 상태로 남겨야 합니다. 이것이 큰 기능을 관리 가능하게 만드는 실행 규율입니다.

## When to Use

- 모든 다중 파일 변경 구현
- 작업 분해로부터 새 기능 빌드
- 기존 코드 리팩터링
- 테스트 전에 ~100줄 이상을 작성하고 싶어질 때

**사용하지 말아야 할 때:** 범위가 이미 최소인 단일 파일, 단일 함수 변경.

## 증분 사이클

```
┌──────────────────────────────────────┐
│                                      │
│   Implement ──→ Test ──→ Verify ──┐  │
│       ▲                           │  │
│       └───── Commit ◄─────────────┘  │
│              │                       │
│              ▼                       │
│          다음 슬라이스               │
│                                      │
└──────────────────────────────────────┘
```

각 슬라이스에 대해:

1. 가장 작은 완전한 기능 조각을 **Implement**
2. **Test** — 테스트 스위트 실행 (없으면 테스트 작성)
3. **Verify** — 슬라이스가 예상대로 동작하는지 확인 (테스트 통과, 빌드 성공, 수동 확인)
4. **Commit** — 서술적 메시지로 진행 저장 (원자적 commit 가이드는 `git-workflow-and-versioning` 참고)
5. **다음 슬라이스로 이동** — 이어가기, 재시작하지 말 것

## Slicing 전략

### 수직 슬라이스 (권장)

스택을 통과하는 하나의 완전한 경로를 빌드:

```
Slice 1: task 생성 (DB + API + 기본 UI)
    → 테스트 통과, 사용자가 UI로 task 생성 가능

Slice 2: task 목록 (query + API + UI)
    → 테스트 통과, 사용자가 task를 볼 수 있음

Slice 3: task 편집 (update + API + UI)
    → 테스트 통과, 사용자가 task 수정 가능

Slice 4: task 삭제 (delete + API + UI + 확인)
    → 테스트 통과, 전체 CRUD 완료
```

각 슬라이스가 동작하는 end-to-end 기능을 전달합니다.

### Contract-First Slicing

백엔드와 프론트엔드가 병렬로 개발해야 할 때:

```
Slice 0: API contract 정의 (타입, 인터페이스, OpenAPI spec)
Slice 1a: contract에 대해 백엔드 구현 + API 테스트
Slice 1b: contract에 맞는 mock 데이터에 대해 프론트엔드 구현
Slice 2: 통합 및 end-to-end 테스트
```

### Risk-First Slicing

가장 위험하거나 가장 불확실한 조각을 먼저 처리:

```
Slice 1: WebSocket 연결이 동작함을 증명 (가장 높은 위험)
Slice 2: 증명된 연결 위에 실시간 task 업데이트 빌드
Slice 3: offline 지원과 재연결 추가
```

Slice 1이 실패하면, Slice 2와 3에 투자하기 전에 발견합니다.

## 구현 규칙

### Rule 0: 단순함 먼저

코드를 작성하기 전에 물어보세요: "동작할 수 있는 가장 단순한 것은?"

코드를 작성한 후, 이 검사들에 대해 리뷰:
- 더 적은 줄로 할 수 있나?
- 이 추상화가 그 복잡도만큼 값을 하나?
- staff 엔지니어가 이를 보고 "왜 그냥 ...하지 않았어?"라고 할까?
- 가상의 미래 요구사항을 위해 빌드 중인가, 현재 작업을 위해서인가?

```
SIMPLICITY CHECK:
✗ 알림 하나에 미들웨어 파이프라인을 가진 일반 EventBus
✓ 단순 함수 호출

✗ 비슷한 컴포넌트 두 개에 abstract factory 패턴
✓ 공유 유틸리티를 가진 단순한 컴포넌트 두 개

✗ form 세 개에 config 주도 form builder
✓ form 컴포넌트 세 개
```

비슷한 코드 세 줄이 성급한 추상화보다 낫습니다. 순진하고 명백히 정확한 버전을 먼저 구현하세요. 정확성이 테스트로 증명된 후에만 최적화하세요.

### Rule 0.5: 범위 규율

작업이 요구하는 것만 건드리세요.

하지 말 것:
- 변경에 인접한 코드를 "정리"
- 수정하지 않는 파일의 import를 리팩터
- 완전히 이해 못 하는 주석 제거
- "유용해 보여서" spec에 없는 기능 추가
- 읽기만 하는 파일의 syntax를 현대화

작업 범위 밖에 개선할 가치가 있는 것을 발견하면, 메모하되 — 고치지 말 것:

```
NOTICED BUT NOT TOUCHING:
- src/utils/format.ts has an unused import (이 작업과 무관)
- The auth middleware could use better error messages (별도 작업)
→ 이것들에 대한 task를 만들까요?
```

### Rule 1: 한 번에 한 가지

각 증분이 하나의 논리적인 것을 바꿉니다. 관심사를 섞지 마세요:

**나쁨:** 새 컴포넌트를 추가하고, 기존 것을 리팩터하고, 빌드 config를 업데이트하는 commit 하나.

**좋음:** 세 개의 별도 commit — 각 변경에 하나씩.

### Rule 2: 컴파일 가능하게 유지

각 증분 후, 프로젝트가 빌드되고 기존 테스트가 통과해야 합니다. 슬라이스 사이에 코드베이스를 깨진 상태로 두지 마세요.

### Rule 3: 미완성 기능에 Feature Flag

기능이 사용자에게 준비되지 않았지만 증분을 merge해야 한다면:

```typescript
// 진행 중 작업에 대한 feature flag
const ENABLE_TASK_SHARING = process.env.FEATURE_TASK_SHARING === 'true';

if (ENABLE_TASK_SHARING) {
  // 새 sharing UI
}
```

이는 미완성 작업을 노출하지 않고 작은 증분을 main branch에 merge하게 합니다.

### Rule 4: 안전한 기본값

새 코드는 안전하고 보수적인 동작을 기본으로 해야 합니다:

```typescript
// 안전: 기본 비활성화, opt-in
export function createTask(data: TaskInput, options?: { notify?: boolean }) {
  const shouldNotify = options?.notify ?? false;
  // ...
}
```

### Rule 5: Rollback 친화적

각 증분은 독립적으로 revert 가능해야 합니다:

- 추가적 변경(새 파일, 새 함수)은 revert하기 쉬움
- 기존 코드 수정은 최소이고 집중되어야 함
- 데이터베이스 migration은 대응하는 rollback migration을 가져야 함
- 같은 commit에서 무언가를 삭제하고 교체하는 것을 피하세요 — 분리하세요

## Agent와 작업하기

agent에게 점진적으로 구현하도록 지시할 때:

```
"plan의 Task 3을 구현하자.

데이터베이스 스키마 변경과 API endpoint만으로 시작.
UI는 아직 건드리지 마 — 다음 증분에서 할 거야.

구현 후, `npm test`와 `npm run build`를 실행해 아무것도
깨지지 않았는지 검증해."
```

각 증분에 무엇이 범위 안이고 무엇이 범위 밖인지 명시하세요.

## 증분 체크리스트

각 증분 후, 검증:

- [ ] 변경이 한 가지를 하고 완전히 함
- [ ] 모든 기존 테스트가 여전히 통과 (`npm test`)
- [ ] 빌드 성공 (`npm run build`)
- [ ] type checking 통과 (`npx tsc --noEmit`)
- [ ] linting 통과 (`npm run lint`)
- [ ] 새 기능이 예상대로 동작
- [ ] 변경이 서술적 메시지로 commit됨

**참고:** 각 검증 커맨드를 그것에 영향을 줄 수 있는 변경 후에 실행하세요. 성공한 실행 후, 코드가 그 이후 바뀌지 않았다면 같은 커맨드를 반복하지 마세요 — 바뀌지 않은 코드에 재실행은 정보를 더하지 않습니다.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "끝에 다 테스트할게" | 버그는 복리로 불어납니다. Slice 1의 버그가 Slice 2-5를 틀리게 만듭니다. 각 슬라이스를 테스트하세요. |
| "한 번에 다 하는 게 더 빨라" | 무언가 깨져 바뀐 500줄 중 어느 것이 원인인지 못 찾을 때까지 *더 빠르게 느껴질* 뿐입니다. |
| "이 변경은 별도 commit하기엔 너무 작아" | 작은 commit은 무료입니다. 큰 commit은 버그를 숨기고 rollback을 고통스럽게 합니다. |
| "feature flag는 나중에 추가할게" | 기능이 완성되지 않았으면, 사용자에게 보여서는 안 됩니다. 지금 flag를 추가하세요. |
| "이 refactor는 포함할 만큼 작아" | 기능과 섞인 refactor는 둘 다 리뷰하고 디버깅하기 어렵게 합니다. 분리하세요. |
| "확실히 하려고 빌드 커맨드를 다시 실행할게" | 성공한 실행 후, 코드가 그 이후 바뀌지 않았다면 같은 커맨드 반복은 아무것도 더하지 않습니다. 안심용이 아니라 이후 편집 후에 다시 실행하세요. |

## Red Flags

- 테스트 실행 없이 100줄 이상 코드 작성
- 단일 증분에 무관한 여러 변경
- "이것도 빠르게 추가하자" 범위 확장
- 더 빠르려고 test/verify 단계 건너뛰기
- 증분 사이 빌드나 테스트 깨짐
- 큰 미commit 변경 누적
- 세 번째 사용 사례가 요구하기 전에 추상화 빌드
- "온 김에" 작업 범위 밖 파일 건드리기
- 일회성 작업에 새 유틸리티 파일 생성
- 중간 코드 변경 없이 같은 build/test 커맨드를 연속 두 번 실행

## Verification

작업의 모든 증분 완료 후:

- [ ] 각 증분이 개별적으로 테스트되고 commit됨
- [ ] 전체 테스트 스위트 통과
- [ ] 빌드가 깨끗
- [ ] 기능이 명세대로 end-to-end로 동작
- [ ] 미commit 변경이 남지 않음
