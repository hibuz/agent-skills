---
name: incremental-implementation
description: 변경 사항을 점진적으로 제공합니다. 둘 이상의 파일에 영향을 미치는 기능이나 변경을 구현할 때 사용합니다. 한 번에 많은 양의 코드를 작성하려고 하거나 작업이 너무 커서 한 단계로 완료할 수 없을 때 사용하세요.
---

# 증분 구현

## 개요

얇은 수직 조각의 Build — 한 부분을 구현하고 테스트하고 확인한 다음 확장하십시오. 한 번에 전체 기능을 구현하지 마세요. 각 증분은 시스템을 작동하고 테스트 가능한 상태로 유지해야 합니다. 이는 대규모 기능을 관리 가능하게 만드는 실행 원칙입니다.

## 사용 시기

- multi-file 변경 구현
- Building 작업 분석의 새로운 기능
- 기존 코드 리팩토링
- 테스트하기 전에 100줄 이상 쓰고 싶은 유혹이 생길 때마다

**사용하지 말아야 할 때:** 단일 파일, single-function는 범위가 이미 최소인 경우 변경됩니다.

## 증분주기

```
┌──────────────────────────────────────┐
│                                      │
│   Implement ──→ Test ──→ Verify ──┐  │
│       ▲                           │  │
│       └───── Commit ◄─────────────┘  │
│              │                       │
│              ▼                       │
│          Next slice                  │
│                                      │
└──────────────────────────────────────┘
```

각 조각에 대해 다음을 수행합니다.

1. 가장 작은 전체 기능 **구현**
2. **테스트** — suite 테스트를 실행합니다(또는 테스트가 없는 경우 테스트 작성).
3. **확인** — 슬라이스가 예상대로 작동하는지 확인합니다(테스트 통과, build 성공, 수동 확인).
4. **커밋** -- 설명 메시지와 함께 진행 상황을 저장합니다(원자적 커밋 guidance는 `git-workflow-and-versioning` 참조).
5. **다음 슬라이스로 이동** — restart를 사용하지 말고 앞으로 이월하세요.

## 슬라이싱 전략

### 수직 슬라이스(선호)

Build 스택을 통한 하나의 완전한 경로:

```
Slice 1: Create a task (DB + API + basic UI)
    → Tests pass, user can create a task via the UI

Slice 2: List tasks (query + API + UI)
    → Tests pass, user can see their tasks

Slice 3: Edit a task (update + API + UI)
    → Tests pass, user can modify tasks

Slice 4: Delete a task (delete + API + UI + confirmation)
    → Tests pass, full CRUD complete
```

각 슬라이스는 작동하는 end-to-end 기능을 제공합니다.

### 계약 우선 슬라이싱

백엔드와 프런트엔드를 동시에 개발해야 하는 경우:

```
Slice 0: Define the API contract (types, interfaces, OpenAPI spec)
Slice 1a: Implement backend against the contract + API tests
Slice 1b: Implement frontend against mock data matching the contract
Slice 2: Integrate and test end-to-end
```

### 위험 우선 슬라이싱

가장 위험하거나 가장 불확실한 부분을 먼저 해결하세요.

```
Slice 1: Prove the WebSocket connection works (highest risk)
Slice 2: Build real-time task updates on the proven connection
Slice 3: Add offline support and reconnection
```

슬라이스 1이 실패하면 슬라이스 2와 3에 투자하기 전에 이를 발견합니다.

## 구현 규칙

### 규칙 0: 단순성이 최우선

코드를 작성하기 전에 "작동할 수 있는 가장 간단한 것은 무엇입니까?"라고 질문해 보십시오.

코드를 작성한 후 다음 사항을 확인하여 검토하세요.
- 더 적은 줄로 이 작업을 수행할 수 있습니까?
- 이러한 추상화로 인해 복잡성이 발생합니까?
- 담당 엔지니어가 이것을 보고 "왜 그냥..."이라고 말할까요?
- 가상의 미래 요청에 대해 building인가요, 아니면 현재 작업인가요?

```
SIMPLICITY CHECK:
✗ Generic EventBus with middleware pipeline for one notification
✓ Simple function call

✗ Abstract factory pattern for two similar components
✓ Two straightforward components with shared utilities

✗ Config-driven form builder for three forms
✓ Three form components
```

유사한 세 줄의 코드가 조기 추상화보다 낫습니다. 순진한 obviously-correct 버전을 먼저 구현하세요. 테스트를 통해 정확성이 입증된 후에만 최적화하십시오.

### 규칙 0.5: 범위 규율

작업에 필요한 것만 터치하세요.quires.

NOT를 수행합니다.
- 변경 사항 옆에 있는 "정리" 코드
- 수정하지 않는 파일에서 가져오기를 리팩터링합니다.
- 완전히 이해하지 못하는 댓글은 삭제하세요.
- "유용해 보이기 때문에" 사양에 없는 기능을 추가합니다.
- 읽기만 하는 파일의 구문을 현대화합니다.

작업 범위 밖에서 개선할 가치가 있는 사항을 발견한 경우 이를 기록하고 수정하지 마세요.

```
NOTICED BUT NOT TOUCHING:
- src/utils/format.ts has an unused import (unrelated to this task)
- The auth middleware could use better error messages (separate task)
→ Want me to create tasks for these?
```

### 규칙 1: 한 번에 하나씩

각 증분은 하나의 논리적인 사항을 변경합니다. 우려 사항을 혼합하지 마십시오.

**나쁜:** 새 구성요소를 추가하고, 기존 구성요소를 리팩터링하고, build 구성을 업데이트하는 하나의 커밋입니다.

**좋음:** 3개의 개별 커밋(각 변경마다 하나씩)

### 규칙 2: 컴파일 가능하게 유지

각 증분 후에 프로젝트는 build를 수행해야 하며 기존 테스트를 통과해야 합니다. 슬라이스 사이에 코드베이스를 손상된 상태로 두지 마십시오.

### 규칙 3: 불완전한 기능에 대한 Feature Flags

기능이 사용자를 위해 준비되지 않았지만 증분을 병합해야 하는 경우:

```typescript
// Feature flag for work-in-progress
const ENABLE_TASK_SHARING = process.env.FEATURE_TASK_SHARING === 'true';

if (ENABLE_TASK_SHARING) {
  // New sharing UI
}
```

이를 통해 불완전한 작업을 노출시키지 않고 작은 증분을 기본 분기에 병합할 수 있습니다.

### 규칙 4: 안전한 기본값

새 코드는 안전하고 보수적인 동작을 기본으로 해야 합니다.

```typescript
// Safe: disabled by default, opt-in
export function createTask(data: TaskInput, options?: { notify?: boolean }) {
  const shouldNotify = options?.notify ?? false;
  // ...
}
```

### 규칙 5: Rollback-Friendly

각 증분은 독립적으로 되돌릴 수 있어야 합니다.

- 추가 변경 사항(새 파일, 새 기능)을 쉽게 되돌릴 수 있습니다.
- 기존 코드에 대한 수정은 최소화하고 집중해야 합니다.
- 데이터베이스 마이그레이션에는 해당 rollback 마이그레이션이 있어야 합니다.
- 한 커밋에서 무언가를 삭제하고 같은 커밋에서 교체하는 것을 피하세요. 분리하세요.

## Agents로 작업하기

점진적으로 구현하도록 agent에 지시하는 경우:

```
"Let's implement Task 3 from the plan.

Start with just the database schema change and the API endpoint.
Don't touch the UI yet — we'll do that in the next increment.

After implementing, run `npm test` and `npm run build` to verify
nothing is broken."
```

각 증분의 범위에 무엇이 있고 범위에 NOT가 무엇인지 명시하십시오.

## 증분 체크리스트

각 증분 후에 다음을 확인하십시오.

- [ ] 변경은 한 가지만 수행하고 완전히 수행합니다.
- [ ] 기존 테스트는 모두 통과합니다(`npm test`).
- [ ] build가 성공합니다(`npm run build`).
- [ ] 유형 확인 통과(`npx tsc --noEmit`)
- [ ] 린팅 패스(`npm run lint`)
- [ ] 새로운 기능이 예상대로 작동합니다.
- [ ] 변경 사항이 설명 메시지와 함께 커밋됩니다.

## 일반적인 합리화

| 합리화 | 현실 |
|---|---|
| "마지막에 다 테스트해보겠습니다" | 버그 화합물. 슬라이스 1의 버그로 인해 슬라이스 2-5가 잘못되었습니다. 각 슬라이스를 테스트합니다. |
| "한 번에 하는 것이 더 빠릅니다." | 뭔가가 깨지고 500개의 변경된 줄 중 어느 줄이 그 원인인지 찾을 수 없을 때까지는 *더 빠르게 느껴집니다*. |
| "이러한 변경 사항은 너무 작아서 별도로 커밋할 수 없습니다." | 작은 커밋은 무료입니다. 대규모 커밋은 버그를 숨기고 rollback를 고통스럽게 만듭니다. |
| "나중에 feature flag를 추가하겠습니다" | 기능이 완전하지 않은 경우 user-visible가 아니어야 합니다. 지금 플래그를 추가하세요. |
| "이 리팩터링은 포함할 수 있을 만큼 작습니다." | 기능과 혼합된 리팩터링은 검토 및 디버깅을 더욱 어렵게 만듭니다. 그들을 분리하십시오. |

## 위험 신호

- 테스트를 실행하지 않고 작성된 100줄 이상의 코드
- 단일 증분 내에서 관련되지 않은 여러 변경 사항
- "quickly 이것도 추가하겠습니다" 범위 확장
- 더 빠르게 이동하려면 test/verify 단계를 건너뛰세요.
- Build 또는 증분 사이에서 테스트가 중단됨
- 커밋되지 않은 대규모 변경 사항이 누적됩니다.
- 세 번째 사용 사례에서 요구하기 전에 Building 추상화
- "여기 있는 동안" 작업 범위 밖의 파일을 터치하는 경우
- one-time 작업을 위한 새 유틸리티 파일 생성

## 확인

작업에 대한 모든 증분을 완료한 후:

- [ ] 각 증분은 개별적으로 테스트되고 커밋되었습니다.
- [ ] 전체 테스트 suite를 통과했습니다.
- [ ] build가 깨끗합니다.
- [ ] 이 기능은 지정된 대로 end-to-end를 작동합니다.
- [ ] 커밋되지 않은 변경 사항이 남아 있지 않습니다.
