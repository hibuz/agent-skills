---
name: code-simplification
description: 명료성을 위해 코드를 단순화. 동작을 바꾸지 않고 명료성을 위해 코드를 리팩터링할 때 사용. 코드가 동작하지만 필요 이상으로 읽거나, 유지보수하거나, 확장하기 어려울 때 사용. 불필요한 복잡도가 누적된 코드를 리뷰할 때 사용.
---

# Code Simplification

> [Claude Code Simplifier 플러그인](https://github.com/anthropics/claude-plugins-official/blob/main/plugins/code-simplifier/agents/code-simplifier.md)에서 영감을 받음. 여기서는 모든 AI 코딩 agent를 위한 모델 중립적, 프로세스 주도적 skill로 각색됨.

## Overview

정확한 동작을 보존하면서 복잡도를 줄여 코드를 단순화하세요. 목표는 더 적은 줄이 아닙니다 — 읽고, 이해하고, 수정하고, 디버깅하기 더 쉬운 코드입니다. 모든 단순화는 간단한 테스트를 통과해야 합니다: "새 팀원이 원본보다 이것을 더 빨리 이해할까?"

## When to Use

- 기능이 동작하고 테스트가 통과하지만, 구현이 필요 이상으로 무겁게 느껴질 때
- 코드 리뷰 중 가독성이나 복잡도 이슈가 표시될 때
- 깊게 중첩된 로직, 긴 함수, 또는 불명확한 이름을 마주칠 때
- 시간 압박 하에 작성된 코드를 리팩터링할 때
- 파일 전반에 흩어진 관련 로직을 통합할 때
- 중복이나 불일치를 도입한 변경을 merge한 후

**사용하지 말아야 할 때:**

- 코드가 이미 깨끗하고 읽기 쉬움 — 단순화 자체를 위해 단순화하지 말 것
- 코드가 무엇을 하는지 아직 이해 못 함 — 단순화 전에 이해할 것
- 코드가 성능에 중요하고 "더 단순한" 버전이 측정 가능하게 더 느릴 것
- 모듈을 통째로 다시 쓸 참 — 버릴 코드를 단순화하는 것은 노력 낭비

## 다섯 가지 원칙

### 1. 동작을 정확히 보존

코드가 무엇을 하는지 바꾸지 마세요 — 어떻게 표현하는지만. 모든 입력, 출력, 부수효과, 에러 동작, 엣지 케이스가 동일하게 유지되어야 합니다. 단순화가 동작을 보존하는지 확신하지 못한다면, 하지 마세요.

```
모든 변경 전에 물어보기:
→ 이것이 모든 입력에 대해 같은 출력을 생성하는가?
→ 이것이 같은 에러 동작을 유지하는가?
→ 이것이 같은 부수효과와 순서를 보존하는가?
→ 모든 기존 테스트가 수정 없이 여전히 통과하는가?
```

### 2. 프로젝트 규약 따르기

단순화는 외부 선호를 강요하는 것이 아니라 코드를 코드베이스와 더 일관되게 만드는 것입니다. 단순화 전에:

```
1. CLAUDE.md / 프로젝트 규약 읽기
2. 이웃 코드가 유사한 패턴을 어떻게 다루는지 학습
3. 다음에 대해 프로젝트 스타일에 맞추기:
   - import 순서와 모듈 시스템
   - 함수 선언 스타일
   - 명명 규약
   - 에러 처리 패턴
   - 타입 annotation 깊이
```

프로젝트 일관성을 깨뜨리는 단순화는 단순화가 아닙니다 — churn입니다.

### 3. 영리함보다 명료함 선호

compact 버전이 파싱하는 데 정신적 멈춤이 필요하다면, 명시적 코드가 compact 코드보다 낫습니다.

```typescript
// 불명확: 빽빽한 ternary 체인
const label = isNew ? 'New' : isUpdated ? 'Updated' : isArchived ? 'Archived' : 'Active';

// 명확: 읽기 쉬운 매핑
function getStatusLabel(item: Item): string {
  if (item.isNew) return 'New';
  if (item.isUpdated) return 'Updated';
  if (item.isArchived) return 'Archived';
  return 'Active';
}
```

```typescript
// 불명확: 인라인 로직이 있는 체인된 reduce
const result = items.reduce((acc, item) => ({
  ...acc,
  [item.id]: { ...acc[item.id], count: (acc[item.id]?.count ?? 0) + 1 }
}), {});

// 명확: 이름 붙인 중간 단계
const countById = new Map<string, number>();
for (const item of items) {
  countById.set(item.id, (countById.get(item.id) ?? 0) + 1);
}
```

### 4. 균형 유지

단순화에는 실패 모드가 있습니다: 과도한 단순화. 다음 함정을 주의하세요:

- **너무 공격적으로 inline** — 개념에 이름을 준 헬퍼를 제거하면 호출 지점이 읽기 어려워짐
- **무관한 로직 결합** — 단순한 두 함수를 하나의 복잡한 함수로 merge하는 것은 더 단순하지 않음
- **"불필요한" 추상화 제거** — 일부 추상화는 복잡도가 아니라 확장성이나 테스트 가능성을 위해 존재
- **줄 수 최적화** — 더 적은 줄이 목표가 아님; 더 쉬운 이해가 목표

### 5. 변경된 것에 한정

기본적으로 최근 수정된 코드를 단순화하세요. 명시적으로 범위 확대를 요청받지 않는 한 무관한 코드의 drive-by 리팩터를 피하세요. 범위 없는 단순화는 diff에 노이즈를 만들고 의도치 않은 회귀를 위험에 빠뜨립니다.

## 단순화 프로세스

### Step 1: 손대기 전에 이해 (Chesterton's Fence)

무언가를 바꾸거나 제거하기 전에, 왜 존재하는지 이해하세요. 이것이 Chesterton's Fence입니다: 길을 가로지르는 울타리를 보고 왜 거기 있는지 모른다면, 허물지 마세요. 먼저 이유를 이해하고, 그 이유가 여전히 적용되는지 결정하세요.

```
단순화 전에 답하기:
- 이 코드의 책임은 무엇인가?
- 무엇이 이를 호출하는가? 이것이 무엇을 호출하는가?
- 엣지 케이스와 에러 경로는 무엇인가?
- 예상 동작을 정의하는 테스트가 있는가?
- 왜 이렇게 작성되었을까? (성능? 플랫폼 제약? 역사적 이유?)
- git blame 확인: 이 코드의 원래 context는 무엇이었나?
```

이것들에 답할 수 없다면, 단순화할 준비가 안 된 것입니다. 먼저 더 많은 context를 읽으세요.

### Step 2: 단순화 기회 식별

다음 패턴을 스캔하세요 — 각각은 막연한 냄새가 아니라 구체적 신호입니다:

**구조적 복잡도:**

| 패턴 | 신호 | 단순화 |
|---------|--------|----------------|
| 깊은 중첩 (3+ 레벨) | 제어 흐름 따라가기 어려움 | 조건을 guard clause나 헬퍼 함수로 추출 |
| 긴 함수 (50+ 줄) | 여러 책임 | 서술적 이름의 집중된 함수로 분할 |
| 중첩 ternary | 파싱에 정신적 stack 필요 | if/else 체인, switch, 또는 lookup 객체로 교체 |
| Boolean 파라미터 flag | `doThing(true, false, true)` | options 객체나 별도 함수로 교체 |
| 반복 conditional | 여러 곳에 같은 `if` 검사 | 잘 명명된 predicate 함수로 추출 |

**명명과 가독성:**

| 패턴 | 신호 | 단순화 |
|---------|--------|----------------|
| 일반적 이름 | `data`, `result`, `temp`, `val`, `item` | 내용을 설명하도록 rename: `userProfile`, `validationErrors` |
| 축약 이름 | `usr`, `cfg`, `btn`, `evt` | 축약이 보편적(`id`, `url`, `api`)이 아니면 전체 단어 사용 |
| 오해를 부르는 이름 | 상태도 변경하는 `get` 이름의 함수 | 실제 동작을 반영하도록 rename |
| "무엇"을 설명하는 주석 | `count++` 위의 `// increment counter` | 주석 삭제 — 코드가 충분히 명확 |
| "왜"를 설명하는 주석 | `// Retry because the API is flaky under load` | 유지 — 코드가 표현 못 하는 의도를 담음 |

**중복:**

| 패턴 | 신호 | 단순화 |
|---------|--------|----------------|
| 중복 로직 | 여러 곳에 같은 5+ 줄 | 공유 함수로 추출 |
| 죽은 코드 | 도달 불가 분기, 사용 안 된 변수, 주석 처리된 블록 | 제거 (정말 죽었는지 확인 후) |
| 불필요한 추상화 | 값을 더하지 않는 wrapper | wrapper를 inline, 하위 함수를 직접 호출 |
| 과도하게 엔지니어링된 패턴 | factory-for-a-factory, 전략 하나뿐인 strategy | 단순한 직접 접근으로 교체 |
| 중복 타입 assertion | 이미 추론된 타입으로 cast | assertion 제거 |

### Step 3: 변경을 점진적으로 적용

한 번에 하나의 단순화를 하세요. 각 변경 후 테스트를 실행하세요. **리팩터링 변경을 기능이나 버그 수정 변경에서 별도로 제출하세요.** 리팩터하고 기능을 추가하는 PR은 두 개의 PR입니다 — 분할하세요.

```
각 단순화에 대해:
1. 변경
2. 테스트 스위트 실행
3. 테스트 통과 → commit (또는 다음 단순화로 계속)
4. 테스트 실패 → revert하고 재고
```

여러 단순화를 테스트되지 않은 단일 변경으로 batch하지 마세요. 무언가 깨지면, 어떤 단순화가 일으켰는지 알아야 합니다.

**Rule of 500:** 리팩터링이 500줄을 넘게 건드릴 것이라면, 손으로 변경하기보다 자동화(codemod, sed 스크립트, AST 변환)에 투자하세요. 그 규모의 수동 편집은 에러가 나기 쉽고 리뷰하기 지칩니다.

### Step 4: 결과 검증

모든 단순화 후, 한 발 물러나 전체를 평가하세요:

```
BEFORE와 AFTER 비교:
- 단순화된 버전이 진정으로 이해하기 더 쉬운가?
- 코드베이스와 일관성 없는 새 패턴을 도입했는가?
- diff가 깨끗하고 리뷰 가능한가?
- 팀원이 이 변경을 승인할까?
```

"단순화된" 버전이 이해하거나 리뷰하기 더 어렵다면, revert하세요. 모든 단순화 시도가 성공하는 것은 아닙니다.

## 언어별 가이드

### TypeScript / JavaScript

```typescript
// 단순화: 불필요한 async wrapper
// Before
async function getUser(id: string): Promise<User> {
  return await userService.findById(id);
}
// After
function getUser(id: string): Promise<User> {
  return userService.findById(id);
}

// 단순화: 장황한 conditional 할당
// Before
let displayName: string;
if (user.nickname) {
  displayName = user.nickname;
} else {
  displayName = user.fullName;
}
// After
const displayName = user.nickname || user.fullName;

// 단순화: 수동 배열 빌드
// Before
const activeUsers: User[] = [];
for (const user of users) {
  if (user.isActive) {
    activeUsers.push(user);
  }
}
// After
const activeUsers = users.filter((user) => user.isActive);

// 단순화: 중복 boolean return
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

### Python

```python
# 단순화: 장황한 dictionary 빌드
# Before
result = {}
for item in items:
    result[item.id] = item.name
# After
result = {item.id: item.name for item in items}

# 단순화: early return으로 중첩 conditional
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
// 단순화: 장황한 conditional 렌더링
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

// 단순화: 중간 컴포넌트를 통한 prop drilling
// Before — context나 composition이 이를 더 잘 해결하는지 고려하세요.
// 이것은 판단의 영역입니다 — 표시하되 자동 리팩터하지 마세요.
```

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "동작하니, 손댈 필요 없어" | 읽기 어려운 동작 코드는 깨질 때 고치기 어렵습니다. 지금 단순화하면 모든 미래 변경에서 시간을 절약합니다. |
| "더 적은 줄이 항상 더 단순해" | 1줄 중첩 ternary는 5줄 if/else보다 단순하지 않습니다. 단순함은 줄 수가 아니라 이해 속도에 관한 것입니다. |
| "이 무관한 코드도 빠르게 단순화할게" | 범위 없는 단순화는 노이즈 diff를 만들고 바꾸려 하지 않은 코드에 회귀를 위험에 빠뜨립니다. 집중하세요. |
| "타입이 자체 문서화해줘" | 타입은 의도가 아니라 구조를 문서화합니다. 잘 명명된 함수가 *왜*를 설명하는 것이 타입 시그니처가 *무엇*을 설명하는 것보다 낫습니다. |
| "이 추상화가 나중에 유용할 수도" | 추측성 추상화를 보존하지 마세요. 지금 안 쓰이면, 값 없는 복잡도입니다. 제거하고 필요할 때 다시 추가하세요. |
| "원래 작성자에게 이유가 있었을 거야" | 그럴 수도. git blame을 확인 — Chesterton's Fence를 적용. 하지만 누적된 복잡도는 종종 이유가 없습니다; 압박 하 반복의 잔여물일 뿐입니다. |
| "이 기능을 추가하면서 리팩터할게" | 리팩터링을 기능 작업에서 분리하세요. 섞인 변경은 리뷰, revert, 히스토리에서 이해하기 더 어렵습니다. |

## Red Flags

- 통과하려면 테스트 수정이 필요한 단순화 (동작을 바꿨을 가능성)
- 원본보다 길고 따라가기 어려운 "단순화된" 코드
- 프로젝트 규약이 아니라 자신의 선호에 맞춰 이름 변경
- "코드가 깨끗해진다"고 에러 처리 제거
- 완전히 이해 못 하는 코드를 단순화
- 많은 단순화를 리뷰하기 어려운 큰 commit 하나로 batch
- 요청받지 않고 현재 작업 범위 밖의 코드를 리팩터

## Verification

단순화 패스를 완료한 후:

- [ ] 모든 기존 테스트가 수정 없이 통과
- [ ] 빌드가 새 경고 없이 성공
- [ ] Linter/formatter 통과 (스타일 회귀 없음)
- [ ] 각 단순화가 리뷰 가능한 점진적 변경
- [ ] diff가 깨끗 — 무관한 변경이 섞이지 않음
- [ ] 단순화된 코드가 프로젝트 규약을 따름 (CLAUDE.md 또는 동등물에 대해 확인)
- [ ] 어떤 에러 처리도 제거되거나 약화되지 않음
- [ ] 죽은 코드가 남지 않음 (사용 안 된 import, 도달 불가 분기)
- [ ] 팀원이나 리뷰 agent가 순 개선으로 변경을 승인할 것
