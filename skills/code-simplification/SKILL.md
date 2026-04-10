---
name: code-simplification
description: 명확성을 위해 코드를 단순화합니다. 동작을 변경하지 않고 명확성을 위해 코드를 리팩터링할 때 사용합니다. 코드가 작동하지만 읽기, 유지 관리 또는 확장이 예상보다 어려운 경우에 사용하세요. 불필요하게 복잡해진 코드를 검토할 때 사용합니다.
---

# 코드 단순화

> [Claude Code Simplifier plugin](https://github.com/anthropics/claude-plugins-official/blob/main/plugins/code-simplifier/agents/code-simplifier.md)에서 영감을 얻었습니다. 여기에서는 AI 코딩 agent에 대해 model-agnostic, process-driven skill로 적용되었습니다.

## 개요

정확한 동작을 유지하면서 복잡성을 줄여 코드를 단순화합니다. 목표는 줄 수를 줄이는 것이 아니라 읽고, 이해하고, 수정하고, 디버깅하기 쉬운 코드입니다. 모든 단순화는 다음과 같은 간단한 테스트를 통과해야 합니다. "새로운 팀원이 원래 팀원보다 이것을 더 빨리 이해할 수 있을까요?"

## 사용 시기

- 기능이 작동하고 테스트를 통과했지만 구현이 필요 이상으로 무겁게 느껴지는 경우
- 코드 검토 중 가독성이나 복잡성 문제가 표시된 경우
- 깊게 중첩된 논리, 긴 함수 또는 불분명한 이름을 접하는 경우
- 시간적 압박 속에서 작성된 코드를 리팩토링할 때
- 파일에 흩어져 있는 관련 로직을 통합하는 경우
- 중복이나 불일치가 발생한 변경 사항을 병합한 후

**사용하지 말아야 할 때:**

- 코드는 이미 깨끗하고 읽기 쉽습니다. 코드를 단순화하지 마세요.
- 아직 코드의 기능을 이해하지 못합니다. 단순화하기 전에 먼저 이해하세요.
- 코드는 performance-critical이며 "간단한" 버전은 상당히 느릴 것입니다.
- 모듈을 완전히 다시 작성하려고 합니다. 일회용 코드를 단순화하면 노력이 낭비됩니다.

## 5가지 원칙

### 1. 동작을 정확하게 유지

코드의 기능을 변경하지 말고 코드가 표현하는 방식만 변경하세요. 모든 입력, 출력, 부작용, 오류 동작 및 예외 사례는 동일하게 유지되어야 합니다. 단순화로 동작이 유지되는지 확신할 수 없다면 단순화하지 마세요.

```
ASK BEFORE EVERY CHANGE:
→ Does this produce the same output for every input?
→ Does this maintain the same error behavior?
→ Does this preserve the same side effects and ordering?
→ Do all existing tests still pass without modification?
```

### 2. 프로젝트 규칙을 따르세요.

단순화란 외부 환경 설정을 강요하지 않고 코드를 코드베이스와 더욱 일관성 있게 만드는 것을 의미합니다. 단순화하기 전에:

```
1. Read CLAUDE.md / project conventions
2. Study how neighboring code handles similar patterns
3. Match the project's style for:
   - Import ordering and module system
   - Function declaration style
   - Naming conventions
   - Error handling patterns
   - Type annotation depth
```

프로젝트 일관성을 깨뜨리는 단순화는 단순화가 아니라 이탈입니다.

### 3. 영리함보다 명확성을 선호하세요

압축 버전에서 구문 분석을 위해 잠시 휴식을 취해야 하는 경우 명시적 코드가 압축 코드보다 낫습니다.

```typescript
// UNCLEAR: Dense ternary chain
const label = isNew ? 'New' : isUpdated ? 'Updated' : isArchived ? 'Archived' : 'Active';

// CLEAR: Readable mapping
function getStatusLabel(item: Item): string {
  if (item.isNew) return 'New';
  if (item.isUpdated) return 'Updated';
  if (item.isArchived) return 'Archived';
  return 'Active';
}
```

```typescript
// UNCLEAR: Chained reduces with inline logic
const result = items.reduce((acc, item) => ({
  ...acc,
  [item.id]: { ...acc[item.id], count: (acc[item.id]?.count ?? 0) + 1 }
}), {});

// CLEAR: Named intermediate step
const countById = new Map<string, number>();
for (const item of items) {
  countById.set(item.id, (countById.get(item.id) ?? 0) + 1);
}
```

### 4. 균형 유지

단순화에는 실패 모드가 있습니다: over-simplification. 다음 함정을 조심하세요.

- **너무 공격적으로 인라인** — 개념에 이름을 부여한 도우미를 제거하면 호출 사이트를 읽기가 더 어려워집니다.
- **관련되지 않은 로직 결합** — 두 개의 간단한 함수를 하나의 복잡한 함수로 병합하는 것은 더 간단하지 않습니다.
- **"불필요한" 추상화 제거** — 일부 추상화는 복잡성이 아닌 확장성 또는 테스트 가능성을 위해 존재합니다.
- **줄 수 최적화** — 줄 수를 줄이는 것이 목표는 아닙니다. 더 쉬운 이해는

### 5. 변경된 범위

최근 수정된 코드를 단순화하는 것이 기본값입니다. 범위를 넓히도록 명시적으로 요청하지 않는 한 관련 없는 코드의 drive-by 리팩터링을 피하세요. 범위가 지정되지 않은 단순화는 차이점에 노이즈를 발생시키고 의도하지 않은 회귀 위험을 초래합니다.

## 단순화 프로세스

### 1단계: 만지기 전에 이해하기(체스터튼의 울타리)

무엇이든 변경하거나 제거하기 전에 그것이 존재하는 이유를 이해하십시오. 이것이 Chesterton의 울타리입니다. 길 건너편에 울타리가 있는데 왜 거기 있는지 이해하지 못한다면, 그것을 허물지 마십시오. 먼저 이유를 이해한 다음 그 이유가 여전히 적용되는지 결정하십시오.

```
BEFORE SIMPLIFYING, ANSWER:
- What is this code's responsibility?
- What calls it? What does it call?
- What are the edge cases and error paths?
- Are there tests that define the expected behavior?
- Why might it have been written this way? (Performance? Platform constraint? Historical reason?)
- Check git blame: what was the original context for this code?
```

이에 답할 수 없다면 단순화할 준비가 되지 않은 것입니다. 먼저 자세한 내용을 읽어보세요.

### 2단계: 단순화 기회 식별

다음 패턴을 검색하세요. 각 패턴은 막연한 냄새가 아닌 구체적인 신호입니다.

**구조적 복잡성:**

| 패턴 | 신호 | 단순화 |
|---------|--------|----------------|
| 깊은 중첩(3개 이상의 레벨) | 제어 흐름을 따르기 어려움 | 조건을 보호 절이나 도우미 함수로 추출 |
| 긴 기능(50줄 이상) | 다양한 책임 | 설명적인 이름을 사용하여 집중된 함수로 분할 |
| 중첩된 삼항 | 구문 분석할 Requires 정신 스택 | if/else 체인, 스위치 또는 조회 객체로 바꾸기 |
| 부울 매개변수 플래그 | `doThing(true, false, true)` | 옵션 개체 또는 별도의 함수로 바꾸기 |
| 반복되는 조건 | 여러 곳에서 동일한 `if` 확인 | well-named 술어 함수로 추출 |

**이름 지정 및 가독성:**

| 패턴 | 신호 | 단순화 |
|---------|--------|----------------|
| 일반 이름 | `data`, `result`, `temp`, `val`, `item` | 내용을 설명하기 위해 이름을 바꿉니다. `userProfile`, `validationErrors` |
| 약칭 | `usr`, `cfg`, `btn`, `evt` | 약어가 보편적인 경우가 아니면 전체 words를 사용하십시오(`id`, `url`, `api`) |
| 오해의 소지가 있는 이름 | 상태를 변경하는 `get`라는 함수 | 실제 동작을 반영하도록 이름 바꾸기 |
| "무엇"을 설명하는 댓글 | `count++` 위의 `// increment counter` | 주석 삭제 - 코드가 충분히 명확합니다 |
| "이유"를 설명하는 댓글 | `// Retry because the API is flaky under load` | 이를 유지하십시오 - 코드가 표현할 수 없는 의도를 담고 있습니다 |

**중복성:**

| 패턴 | 신호 | 단순화 |
|---------|--------|----------------|
| 중복된 논리 | 여러 위치에 동일한 5줄 이상 | 공유 기능으로 추출 |
| 데드 코드 | 도달할 수 없는 분기, 사용되지 않는 변수, commented-out 블록 | 제거(정말로 죽었는지 확인한 후) |
| 불필요한 추상화 | 가치를 추가하지 않는 래퍼 | 래퍼를 인라인하고 기본 함수를 직접 호출 |
| 과도하게 설계된 패턴 | 공장-for-a-factory, strategy-with-one-strategy | 간단한 직접 접근 방식으로 대체 |
| 중복 유형 어설션 | 이미 추론된 유형으로 캐스팅 | 어설션 제거 |

### 3단계: 변경 사항을 점진적으로 적용

한 번에 하나씩 단순화하십시오. 각 변경 후에 테스트를 실행합니다. **기능 또는 버그 수정 변경 사항과 별도로 리팩토링 변경 사항을 제출하세요.** 기능을 리팩터링하고 추가하는 PR는 두 개의 PRs — 분할됩니다.

```
FOR EACH SIMPLIFICATION:
1. Make the change
2. Run the test suite
3. If tests pass → commit (or continue to next simplification)
4. If tests fail → revert and reconsider
```

여러 단순화를 테스트되지 않은 단일 변경 사항으로 일괄 처리하지 마세요. 문제가 발생한 경우 어떤 단순화로 인해 문제가 발생했는지 알아야 합니다.

**500의 법칙:** 리팩토링이 500줄을 초과하는 경우 수동으로 변경하는 대신 자동화(codemods, sed 스크립트, AST transforms)에 투자하세요. 해당 규모의 수동 편집은 error-prone이며 검토하기가 어렵습니다.

### 4단계: 결과 확인

모든 단순화를 마친 후에는 한 걸음 물러나 전체를 평가해 보십시오.

```
COMPARE BEFORE AND AFTER:
- Is the simplified version genuinely easier to understand?
- Did you introduce any new patterns inconsistent with the codebase?
- Is the diff clean and reviewable?
- Would a teammate approve this change?
```

"단순화된" 버전이 이해하거나 검토하기 더 어려운 경우 되돌리세요. 모든 단순화 시도가 성공하는 것은 아닙니다.

## 언어별 Guidance

### TypeScript / JavaScript

```typescript
// SIMPLIFY: Unnecessary async wrapper
// Before
async function getUser(id: string): Promise<User> {
  return await userService.findById(id);
}
// After
function getUser(id: string): Promise<User> {
  return userService.findById(id);
}

// SIMPLIFY: Verbose conditional assignment
// Before
let displayName: string;
if (user.nickname) {
  displayName = user.nickname;
} else {
  displayName = user.fullName;
}
// After
const displayName = user.nickname || user.fullName;

// SIMPLIFY: Manual array building
// Before
const activeUsers: User[] = [];
for (const user of users) {
  if (user.isActive) {
    activeUsers.push(user);
  }
}
// After
const activeUsers = users.filter((user) => user.isActive);

// SIMPLIFY: Redundant boolean return
// Before
function isValid(input: string): boolean {
  if (input.length > 0 && input.length < 100) {
    return true;
  }
  return false;
}
// After
function isValid(input: string): boolean {
  return input.length > 0 && input.length < 100;
}
```

### 파이썬

```python
# SIMPLIFY: Verbose dictionary building
# Before
result = {}
for item in items:
    result[item.id] = item.name
# After
result = {item.id: item.name for item in items}

# SIMPLIFY: Nested conditionals with early return
# Before
def process(data):
    if data is not None:
        if data.is_valid():
            if data.has_permission():
                return do_work(data)
            else:
                raise PermissionError("No permission")
        else:
            raise ValueError("Invalid data")
    else:
        raise TypeError("Data is None")
# After
def process(data):
    if data is None:
        raise TypeError("Data is None")
    if not data.is_valid():
        raise ValueError("Invalid data")
    if not data.has_permission():
        raise PermissionError("No permission")
    return do_work(data)
```

### React / JSX

```tsx
// SIMPLIFY: Verbose conditional rendering
// Before
function UserBadge({ user }: Props) {
  if (user.isAdmin) {
    return <Badge variant="admin">Admin</Badge>;
  } else {
    return <Badge variant="default">User</Badge>;
  }
}
// After
function UserBadge({ user }: Props) {
  const variant = user.isAdmin ? 'admin' : 'default';
  const label = user.isAdmin ? 'Admin' : 'User';
  return <Badge variant={variant}>{label}</Badge>;
}

// SIMPLIFY: Prop drilling through intermediate components
// Before — consider whether context or composition solves this better.
// This is a judgment call — flag it, don't auto-refactor.
```

## 일반적인 합리화

| 합리화 | 현실 |
|---|---|
| "작동 중입니다. 만질 필요가 없습니다." | 읽기 어려운 작업 코드는 깨졌을 때 수정하기가 어렵습니다. 이제 단순화하면 향후 모든 변경에 소요되는 시간이 절약됩니다. |
| "줄이 적을수록 항상 더 간단합니다" | 1줄 중첩 삼항은 5줄 if/else보다 간단하지 않습니다. 단순성은 줄 수가 아니라 이해 속도에 관한 것입니다. |
| "이 관련 없는 코드도 quickly 단순화하겠습니다." | 범위가 지정되지 않은 단순화는 시끄러운 차이를 만들고 변경하지 않으려는 코드에서 회귀 위험을 초래합니다. 집중하세요. |
| "The types make it self-documenting" | 의도가 아닌 문서 구조를 입력합니다. well-named 함수는 *무엇*을 설명하는 형식 서명보다 *이유*를 더 잘 설명합니다. |
| "이 추상화는 나중에 유용할 수 있습니다." | 추측성 추상화를 유지하지 마세요. 지금 사용하지 않으면 가치가 없는 복잡성입니다. 필요한 경우 이를 제거하고 re-add를 제거하십시오. |
| "원저자에게도 이유가 있었을 거에요" | 아마도. 자식 비난을 확인하세요. Chesterton's Fence를 적용하세요. 그러나 축적된 복잡성에는 이유가 없는 경우가 많습니다. 그것은 단지 압력을 받는 반복의 잔여물일 뿐입니다. |
| "이 기능을 추가하면서 리팩토링하겠습니다" | 기능 작업과 리팩토링을 분리하세요. 혼합된 변경 사항은 기록에서 검토, 되돌리기 및 이해하기가 더 어렵습니다. |

## 위험 신호

- 통과하기 위해 테스트 수정을 요구하는 단순화(동작이 변경되었을 가능성이 높음)
- 원본보다 더 길고 따라하기 어려운 "단순화된" 코드
- 프로젝트 규칙보다는 선호도에 맞게 이름을 바꿉니다.
- "코드를 더 깔끔하게 만들기" 때문에 오류 처리 제거
- 완전히 이해하지 못하는 코드 단순화
- 많은 단순화를 하나의 대규모 hard-to-review 커밋으로 일괄 처리
- 요청 없이 현재 작업 범위 밖의 코드 리팩토링

## 확인

단순화 패스를 완료한 후:

- [ ] 수정 없이 기존 테스트 모두 통과
- [ ] Build succeeds with no new warnings
- [ ] Linter/formatter 통과(스타일 회귀 없음)
- [ ] 각 단순화는 검토 가능한 점진적 변경입니다.
- [ ] diff가 깨끗합니다. 관련 없는 변경 사항이 섞여 있지 않습니다.
- [ ] 단순화된 코드는 프로젝트 규칙을 따릅니다(CLAUDE.md 또는 equivalent에 대해 확인).
- [ ] 오류 처리가 제거되거나 약화되지 않았습니다.
- [ ] 데드 코드가 남지 않았습니다(사용하지 않은 가져오기, 도달할 수 없는 분기).
- [ ] 팀 동료 또는 검토자 agent가 변경 사항을 순 개선으로 승인할 것입니다.
