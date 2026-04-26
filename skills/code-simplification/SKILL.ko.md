---
name: code-simplification
description: 명확성을 위해 코드를 단순화합니다. 동작을 변경하지 않고 명확성을 위해 코드를 리팩토링할 때 사용하세요. 코드는 작동하지만 읽기, 유지보수 또는 확장하기가 필요 이상으로 어려울 때 사용합니다. 불필요한 복잡성이 쌓인 코드를 리뷰할 때 사용합니다.
---

# 코드 단순화 (Code Simplification)

> [Claude Code Simplifier 플러그인](https://github.com/anthropics/claude-plugins-official/blob/main/plugins/code-simplifier/agents/code-simplifier.md)에서 영감을 얻었습니다. 여기서는 모든 AI 코딩 에이전트를 위한 모델 독립적이고 프로세스 중심적인 스킬로 개정되었습니다.

## 개요 (Overview)

정확한 동작을 보존하면서 복잡성을 줄여 코드를 단순화하세요. 목표는 라인 수를 줄이는 것이 아니라, 읽고 이해하고 수정하고 디버깅하기 쉬운 코드를 만드는 것입니다. 모든 단순화 작업은 다음의 간단한 테스트를 통과해야 합니다. "새로운 팀원이 원래 코드보다 이 코드를 더 빨리 이해할 수 있는가?"

## 사용 시기 (When to Use)

- 기능이 작동하고 테스트를 통과했지만, 구현이 필요 이상으로 무겁게 느껴질 때
- 코드 리뷰 중에 가독성이나 복잡성 문제가 지적되었을 때
- 깊게 중첩된 로직, 너무 긴 함수, 또는 불명확한 이름을 발견했을 때
- 시간 압박 속에서 작성된 코드를 리팩토링할 때
- 여러 파일에 흩어져 있는 관련 로직을 통합할 때
- 중복이나 일관성 없는 변경 사항이 머지된 후

**사용하지 말아야 할 때:**

- 코드가 이미 깨끗하고 가독성이 좋을 때 — 단순화를 위한 단순화는 하지 마세요.
- 코드가 무엇을 하는지 아직 이해하지 못했을 때 — 단순화하기 전에 먼저 이해하세요.
- 코드가 성능에 민감하며, "단순한" 버전이 측정 가능할 정도로 더 느릴 때
- 모듈 전체를 완전히 다시 작성할 예정일 때 — 버려질 코드를 단순화하는 것은 노력을 낭비하는 것입니다.

## 5대 원칙 (The Five Principles)

### 1. 동작을 정확하게 보존하기 (Preserve Behavior Exactly)

코드가 무엇을 하는지는 바꾸지 말고, 어떻게 표현하는지만 바꾸세요. 모든 입력, 출력, 부작용(side effects), 에러 동작, 예외 케이스는 동일하게 유지되어야 합니다. 단순화가 동작을 보존하는지 확신할 수 없다면 변경하지 마세요.

```
모든 변경 전에 자문하세요:
→ 이것이 모든 입력에 대해 동일한 출력을 내는가?
→ 이것이 동일한 에러 동작을 유지하는가?
→ 이것이 동일한 부작용과 순서를 보존하는가?
→ 수정 없이도 기존의 모든 테스트를 통과하는가?
```

### 2. 프로젝트 관례 따르기 (Follow Project Conventions)

단순화란 코드베이스와 더 일관되게 만드는 것을 의미하며, 외부의 선호도를 강요하는 것이 아닙니다. 단순화하기 전에:

```
1. CLAUDE.md / 프로젝트 관례를 읽으세요.
2. 주변 코드가 유사한 패턴을 어떻게 처리하는지 공부하세요.
3. 다음 사항에 대해 프로젝트의 스타일을 맞추세요:
   - 임포트 순서 및 모듈 시스템
   - 함수 선언 스타일
   - 명명 규칙 (Naming conventions)
   - 에러 처리 패턴
   - 타입 어노테이션 깊이
```

프로젝트의 일관성을 깨는 단순화는 단순화가 아니라 소음(churn)일 뿐입니다.

### 3. 기교보다 명확성을 선호하기 (Prefer Clarity Over Cleverness)

간결한 버전이 해석을 위해 멈칫하게 만든다면, 명시적인 코드가 더 낫습니다.

```typescript
// 불명확: 밀집된 삼항 연산자 체인
const label = isNew ? 'New' : isUpdated ? 'Updated' : isArchived ? 'Archived' : 'Active';

// 명확: 가독성 좋은 매핑
function getStatusLabel(item: Item): string {
  if (item.isNew) return 'New';
  if (item.isUpdated) return 'Updated';
  if (item.isArchived) return 'Archived';
  return 'Active';
}
```

```typescript
// 불명확: 인라인 로직이 포함된 체이닝된 reduce
const result = items.reduce((acc, item) => ({
  ...acc,
  [item.id]: { ...acc[item.id], count: (acc[item.id]?.count ?? 0) + 1 }
}), {});

// 명확: 이름 붙은 중간 단계
const countById = new Map<string, number>();
for (const item of items) {
  countById.set(item.id, (countById.get(item.id) ?? 0) + 1);
}
```

### 4. 균형 유지하기 (Maintain Balance)

단순화에는 '과도한 단순화'라는 실패 모드가 있습니다. 다음 함정들을 주의하세요:

- **지나친 인라이닝 (Inlining)** — 개념에 이름을 부여한 헬퍼를 제거하면 호출 지점의 가독성이 떨어집니다.
- **관련 없는 로직 결합** — 두 개의 단순한 함수를 하나의 복잡한 함수로 합치는 것은 더 단순한 것이 아닙니다.
- **"불필요한" 추상화 제거** — 어떤 추상화는 복잡성을 위해서가 아니라 확장성이나 테스트 가능성을 위해 존재합니다.
- **라인 수 최적화** — 라인 수가 적은 것이 목표가 아닙니다. 더 이해하기 쉬운 것이 목표입니다.

### 5. 변경 범위에 집중하기 (Scope to What Changed)

최근에 수정된 코드를 단순화하는 것을 기본으로 하세요. 범위를 넓혀달라는 명시적인 요청이 없는 한, 관련 없는 코드를 리팩토링하지 마세요. 범위를 벗어난 단순화는 diff에 노이즈를 만들고 의도치 않은 회귀(regression) 위험을 초래합니다.

## 단순화 프로세스 (The Simplification Process)

### 1단계: 건드리기 전에 이해하기 (체스터턴의 울타리 - Chesterton's Fence)

무엇인가를 변경하거나 제거하기 전에 그것이 왜 존재하는지 이해하세요. 이것이 '체스터턴의 울타리'입니다. 길을 가로막는 울타리가 있는데 왜 거기 있는지 모르겠다면, 그것을 철거하지 마세요. 먼저 이유를 이해하고, 그 이유가 여전히 유효한지 결정하세요.

```
단순화하기 전에 답하세요:
- 이 코드의 책임은 무엇인가?
- 무엇이 이것을 호출하는가? 이것은 무엇을 호출하는가?
- 예외 케이스와 에러 경로는 무엇인가?
- 예상되는 동작을 정의하는 테스트가 있는가?
- 왜 이렇게 작성되었을까? (성능? 플랫폼 제약? 역사적 이유?)
- git blame 확인: 이 코드의 원래 컨텍스트는 무엇인가?
```

이 질문들에 답할 수 없다면 단순화할 준비가 되지 않은 것입니다. 먼저 더 많은 컨텍스트를 읽으세요.

### 2단계: 단순화 기회 식별하기 (Identify Simplification Opportunities)

다음 패턴들을 스캔하세요. 각각은 모호한 느낌이 아니라 구체적인 신호입니다.

**구조적 복잡성 (Structural complexity):**

| 패턴 | 신호 | 단순화 방법 |
|---------|--------|----------------|
| 깊은 중첩 (3단계 이상) | 제어 흐름을 따라가기 어려움 | 가드 클로즈(guard clauses)나 헬퍼 함수로 조건 추출 |
| 긴 함수 (50줄 이상) | 여러 책임을 가짐 | 설명적인 이름을 가진 집중된 함수들로 분할 |
| 중첩된 삼항 연산자 | 해석하는 데 인지적 부하 발생 | if/else 체인, switch, 또는 룩업 객체로 교체 |
| 불리언 파라미터 플래그 | `doThing(true, false, true)` | 옵션 객체나 별도의 함수로 교체 |
| 반복되는 조건문 | 여러 곳에서 동일한 `if` 체크 | 이름을 잘 지은 서술형(predicate) 함수로 추출 |

**명명 및 가독성 (Naming and readability):**

| 패턴 | 신호 | 단순화 방법 |
|---------|--------|----------------|
| 일반적인 이름 | `data`, `result`, `temp`, `val`, `item` | 내용을 설명하는 이름으로 변경: `userProfile`, `validationErrors` |
| 축약된 이름 | `usr`, `cfg`, `btn`, `evt` | 축약어가 보편적이지 않다면(`id`, `url`, `api`) 풀 네임 사용 |
| 오해의 소지가 있는 이름 | `get`이라는 이름인데 상태도 변경함 | 실제 동작을 반영하도록 이름 변경 |
| "무엇"을 설명하는 주석 | `count++` 위에 `// 카운터 증가` | 주석 삭제 — 코드로 충분히 명확함 |
| "왜"를 설명하는 주석 | `// 부하 상황에서 API가 불안정하여 재시도함` | 유지 — 코드가 표현할 수 없는 의도를 담고 있음 |

**중복 (Redundancy):**

| 패턴 | 신호 | 단순화 방법 |
|---------|--------|----------------|
| 중복된 로직 | 여러 곳에서 동일한 5줄 이상의 코드 | 공유 함수로 추출 |
| 죽은 코드 (Dead code) | 도달할 수 없는 분기, 미사용 변수, 주석 처리된 블록 | 삭제 (진짜 죽은 코드인지 확인 후) |
| 불필요한 추상화 | 가치를 더하지 않는 래퍼(wrapper) | 래퍼를 인라이닝하고 내부 함수를 직접 호출 |
| 과하게 설계된 패턴 | 팩토리를 위한 팩토리, 전략이 하나뿐인 전략 패턴 | 단순하고 직접적인 접근 방식으로 교체 |
| 불필요한 타입 단언 | 이미 추론된 타입으로 캐스팅 | 단언 제거 |

### 3단계: 점진적으로 변경 적용하기 (Apply Changes Incrementally)

한 번에 하나씩 단순화하세요. 변경마다 테스트를 실행하세요. **리팩토링 변경 사항은 기능 추가나 버그 수정 변경 사항과 별도로 제출하세요.** 리팩토링과 기능 추가가 섞인 PR은 두 개의 PR로 나누어야 합니다.

```
각 단순화 작업마다:
1. 변경 수행
2. 테스트 슈트 실행
3. 테스트 통과 → 커밋 (또는 다음 단순화 진행)
4. 테스트 실패 → 되돌리고 재검토
```

테스트되지 않은 여러 단순화 작업을 한꺼번에 묶지 마세요. 무언가 고장 났을 때 어떤 작업 때문인지 알아야 합니다.

**500줄의 법칙:** 리팩토링이 500줄 이상을 건드린다면, 수동으로 수정하기보다는 자동화 도구(codemods, sed 스크립트, AST transforms)를 활용하세요. 그 규모의 수동 편집은 에러가 발생하기 쉽고 리뷰하기 매우 힘듭니다.

### 4단계: 결과 검증하기 (Verify the Result)

모든 단순화 작업이 끝나면 한 걸음 물러나 전체를 평가하세요:

```
전후 비교:
- 단순화된 버전이 진정으로 이해하기 더 쉬운가?
- 코드베이스와 일관되지 않은 새로운 패턴을 도입하지 않았는가?
- diff가 깨끗하고 리뷰 가능한가?
- 동료가 이 변경을 승인할 것인가?
```

단순화된 버전이 이해하기 더 어렵거나 리뷰하기 힘들다면 되돌리세요. 모든 단순화 시도가 성공하는 것은 아닙니다.

## 언어별 가이드라인 (Language-Specific Guidance)

### TypeScript / JavaScript

```typescript
// 단순화: 불필요한 async 래퍼
// 전
async function getUser(id: string): Promise<User> {
  return await userService.findById(id);
}
// 후
function getUser(id: string): Promise<User> {
  return userService.findById(id);
}

// 단순화: 장황한 조건부 할당
// 전
let displayName: string;
if (user.nickname) {
  displayName = user.nickname;
} else {
  displayName = user.fullName;
}
// 후
const displayName = user.nickname || user.fullName;

// 단순화: 수동 배열 구축
// 전
const activeUsers: User[] = [];
for (const user of users) {
  if (user.isActive) {
    activeUsers.push(user);
  }
}
// 후
const activeUsers = users.filter((user) => user.isActive);

// 단순화: 중복된 불리언 반환
// 전
function isValid(input: string): boolean {
  if (input.length > 0 && input.length < 100) {
    return true;
  }
  return false;
}
// 후
function isValid(input: string): boolean {
  return input.length > 0 && input.length < 100;
}
```

### Python

```python
# 단순화: 장황한 딕셔너리 구축
# 전
result = {}
for item in items:
    result[item.id] = item.name
# 후
result = {item.id: item.name for item in items}

# 단순화: 조기 반환을 통한 중첩 조건문 제거
# 전
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
# 후
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
// 단순화: 장황한 조건부 렌더링
// 전
function UserBadge({ user }: Props) {
  if (user.isAdmin) {
    return <Badge variant="admin">Admin</Badge>;
  } else {
    return <Badge variant="default">User</Badge>;
  }
}
// 후
function UserBadge({ user }: Props) {
  const variant = user.isAdmin ? 'admin' : 'default';
  const label = user.isAdmin ? 'Admin' : 'User';
  return <Badge variant={variant}>{label}</Badge>;
}

// 단순화: 중간 컴포넌트를 거치는 프롭 드릴링(Prop drilling)
// 전 — Context나 합성이 이 문제를 더 잘 해결하는지 고려하세요.
// 이것은 판단이 필요한 영역입니다. 자동으로 리팩토링하지 말고 문제를 제기하세요.
```

## 흔한 자기합리화 (Common Rationalizations)

| 자기합리화 | 실제 상황 |
|---|---|
| "작동하고 있으니 건드릴 필요 없어요" | 읽기 어려운 코드는 고장 났을 때 고치기도 어렵습니다. 지금 단순화하면 미래의 모든 변경 시간을 아낄 수 있습니다. |
| "라인 수가 적은 게 항상 더 단순한 거예요" | 1줄짜리 중첩 삼항 연산자는 5줄짜리 if/else보다 단순하지 않습니다. 단순함은 라인 수가 아니라 이해 속도에 관한 것입니다. |
| "관련 없는 이 코드도 같이 빨리 단순화할게요" | 범위를 벗어난 단순화는 diff에 노이즈를 만들고 의도치 않은 회귀 위험을 초래합니다. 집중하세요. |
| "타입이 있으니 스스로 설명이 돼요" | 타입은 구조를 설명하지만 의도를 설명하지는 않습니다. 이름을 잘 지은 함수가 타입 시그니처보다 '왜'를 더 잘 설명합니다. |
| "이 추상화는 나중에 유용할 수 있어요" | 추측에 근거한 추상화를 남겨두지 마세요. 지금 사용되지 않는다면 가치 없는 복잡성일 뿐입니다. 제거하고 필요할 때 다시 추가하세요. |
| "원래 작성자가 이유가 있었겠죠" | 그럴 수도 있습니다. git blame을 확인하고 체스터턴의 울타리를 적용하세요. 하지만 쌓인 복잡성은 대개 이유가 없습니다. 압박 속에서 반복된 수정의 잔재일 뿐입니다. |
| "기능을 추가하면서 리팩토링도 할게요" | 리팩토링과 기능 작업을 분리하세요. 혼합된 변경 사항은 리뷰, 되돌리기, 히스토리 파악이 더 어렵습니다. |

## 위험 신호 (Red Flags)

- 통과를 위해 테스트를 수정해야 하는 단순화 (동작을 변경했을 가능성이 높음)
- 원래 코드보다 더 길고 따라가기 어려운 "단순화된" 코드
- 프로젝트 관례가 아닌 자신의 선호도에 맞춰 이름을 변경함
- "코드가 더 깨끗해지니까"라는 이유로 에러 처리를 제거함
- 완전히 이해하지 못한 코드를 단순화함
- 리뷰하기 어려운 대규모 커밋 하나에 많은 단순화 작업을 묶음
- 요청받지 않고 현재 태스크의 범위를 벗어난 코드를 리팩토링함

## 검증 (Verification)

단순화 작업을 마친 후:

- [ ] 수정 없이 기존의 모든 테스트를 통과하는가
- [ ] 새로운 경고 없이 빌드가 성공하는가
- [ ] 린터/포매터가 통과하는가 (스타일 회귀 없음)
- [ ] 각 단순화 작업이 리뷰 가능한 증분 변경 사항인가
- [ ] diff가 깨끗하며 관련 없는 변경 사항이 섞여 있지 않은가
- [ ] 단순화된 코드가 프로젝트 관례를 따르는가 (CLAUDE.md 등 확인)
- [ ] 제거되거나 약화된 에러 처리가 없는가
- [ ] 미사용 임포트나 도달할 수 없는 분기 등 죽은 코드가 남지 않았는가
- [ ] 동료나 리뷰 에이전트가 이 변경을 순수한 개선으로 승인할 것인가
