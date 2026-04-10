---
name: frontend-ui-engineering
description: Builds production-quality UIs. building 또는 user-facing 인터페이스를 수정할 때 사용합니다. 구성 요소 생성, 레이아웃 구현, 상태 관리 또는 출력이 AI 생성이 아닌 production-quality 모양과 느낌이 필요한 경우에 사용합니다.
---

# 프런트엔드 UI 엔지니어링

## 개요

액세스 가능하고 성능이 우수하며 시각적으로 세련된 Build production-quality 사용자 인터페이스입니다. 목표는 UI입니다. 이는 일류 회사의 design-aware 엔지니어가 built처럼 보이지만 AI에 의해 생성된 것과는 다릅니다. 이는 실제 디자인 시스템 준수, 적절한 접근성, 사려 깊은 상호 작용 패턴 및 일반적인 "AI 미학"이 없음을 의미합니다.

## 사용 시기

- Building 새로운 UI 구성 요소 또는 페이지
- 기존 user-facing 인터페이스 수정
- 반응형 레이아웃 구현
- 상호작용성 또는 상태 관리 추가
- 시각적 또는 UX 문제 해결

## 구성 요소 아키텍처

### 파일 구조

구성 요소와 관련된 모든 것을 같은 위치에 배치하십시오.

```
src/components/
  TaskList/
    TaskList.tsx          # Component implementation
    TaskList.test.tsx     # Tests
    TaskList.stories.tsx  # Storybook stories (if using)
    use-task-list.ts      # Custom hook (if complex state)
    types.ts              # Component-specific types (if needed)
```

### 구성요소 패턴

**구성보다 구성을 선호합니다.**

```tsx
// Good: Composable
<Card>
  <CardHeader>
    <CardTitle>Tasks</CardTitle>
  </CardHeader>
  <CardBody>
    <TaskList tasks={tasks} />
  </CardBody>
</Card>

// Avoid: Over-configured
<Card
  title="Tasks"
  headerVariant="large"
  bodyPadding="md"
  content={<TaskList tasks={tasks} />}
/>
```

**구성요소에 집중하세요:**

```tsx
// Good: Does one thing
export function TaskItem({ task, onToggle, onDelete }: TaskItemProps) {
  return (
    <li className="flex items-center gap-3 p-3">
      <Checkbox checked={task.done} onChange={() => onToggle(task.id)} />
      <span className={task.done ? 'line-through text-muted' : ''}>{task.title}</span>
      <Button variant="ghost" size="sm" onClick={() => onDelete(task.id)}>
        <TrashIcon />
      </Button>
    </li>
  );
}
```

**프레젠테이션에서 별도의 데이터 가져오기:**

```tsx
// Container: handles data
export function TaskListContainer() {
  const { tasks, isLoading, error } = useTasks();

  if (isLoading) return <TaskListSkeleton />;
  if (error) return <ErrorState message="Failed to load tasks" retry={refetch} />;
  if (tasks.length === 0) return <EmptyState message="No tasks yet" />;

  return <TaskList tasks={tasks} />;
}

// Presentation: handles rendering
export function TaskList({ tasks }: { tasks: Task[] }) {
  return (
    <ul role="list" className="divide-y">
      {tasks.map(task => <TaskItem key={task.id} task={task} />)}
    </ul>
  );
}
```

## 상태 관리

**효과적인 가장 간단한 접근 방식을 선택하세요.**

```
Local state (useState)           → Component-specific UI state
Lifted state                     → Shared between 2-3 sibling components
Context                          → Theme, auth, locale (read-heavy, write-rare)
URL state (searchParams)         → Filters, pagination, shareable UI state
Server state (React Query, SWR)  → Remote data with caching
Global store (Zustand, Redux)    → Complex client state shared app-wide
```

**3개 수준보다 더 깊은 prop 드릴링을 피하세요.** prop을 사용하지 않는 구성 요소를 통해 prop을 전달하는 경우 컨텍스트를 도입하거나 구성 요소 트리를 rest구조화하세요.

## 디자인 시스템 준수

### AI 미적 요소를 피하세요

AI에서 생성된 UI에는 인식 가능한 패턴이 있습니다. 모두 피하십시오:

| AI 기본값 | 왜 문제가 되는가 | 생산 품질 |
|---|---|---|
| Purple/indigo 모든 것 | 모델은 기본적으로 시각적으로 "안전한" 팔레트를 사용하므로 모든 앱이 동일하게 보입니다. | 프로젝트의 실제 색상 팔레트 사용 |
| 과도한 그라데이션 | 그라디언트는 시각적 노이즈를 추가하고 대부분의 디자인 시스템과 충돌합니다 | 디자인 시스템에 맞는 평면적이거나 미묘한 그라데이션 |
| 모든 것을 반올림했습니다(rounded-2xl) | 최대 반올림은 "친화적"이라는 신호를 보내지만 실제 설계에서는 모서리 반경의 계층 구조를 무시합니다 | 디자인 시스템의 일관된 border-radius |
| 일반 영웅 섹션 | 실제 콘텐츠나 사용자 요구와 연결되지 않은 템플릿 기반 레이아웃 | 콘텐츠 우선 레이아웃 |
| 로렘 ipsum-style 복사 | 자리 표시자 텍스트는 실제 콘텐츠에서 드러나는 레이아웃 문제(길이, 줄 바꿈, 오버플로)를 숨깁니다 | 현실적인 자리표시자 콘텐츠 |
| 곳곳에 오버사이즈 패딩 | 동일한 넉넉한 패딩은 시각적 계층 구조를 파괴하고 화면 공간을 낭비합니다 | 일관된 간격 규모 |
| 주식 카드 그리드 | Uniform 그리드는 information 우선순위 및 스캔 패턴을 무시하는 레이아웃 바로가기입니다. 목적 중심 레이아웃 |
| 그림자가 많은 디자인 | 계층화된 그림자는 콘텐츠와 경쟁하고 low-end 장치에서 렌더링 속도를 늦추는 깊이를 추가합니다 | 디자인 시스템에서 지정하지 않는 한 그림자가 미묘하거나 없음 |

### 간격 및 레이아웃

일관된 간격 척도를 사용하십시오. 가치를 만들어내지 마세요:

```css
/* Use the scale: 0.25rem increments (or whatever the project uses) */
/* Good */  padding: 1rem;      /* 16px */
/* Good */  gap: 0.75rem;       /* 12px */
/* Bad */   padding: 13px;      /* Not on any scale */
/* Bad */   margin-top: 2.3rem; /* Not on any scale */
```

### 타이포그래피

유형 계층 구조를 존중하십시오.

```
h1 → Page title (one per page)
h2 → Section title
h3 → Subsection title
body → Default text
small → Secondary/helper text
```

제목 수준을 건너뛰지 마십시오. non-heading 콘텐츠에는 제목 스타일을 사용하지 마세요.

### 색상

- 의미론적 색상 토큰 사용: `text-primary`, `bg-surface`, `border-default` - 원시 16진수 값 아님
- 충분한 대비를 보장합니다(normal 텍스트의 경우 4.5:1, 큰 텍스트의 경우 3:1).
- information을 전달하기 위해 색상에만 의존하지 마십시오(아이콘, 텍스트 또는 패턴도 사용).

## 접근성 (WCAG 2.1 AA)

모든 구성요소는 다음 표준을 충족해야 합니다. rds:

### 키보드 탐색

```tsx
// Every interactive element must be keyboard accessible
<button onClick={handleClick}>Click me</button>        // ✓ Focusable by default
<div onClick={handleClick}>Click me</div>               // ✗ Not focusable
<div role="button" tabIndex={0} onClick={handleClick}    // ✓ But prefer <button>
     onKeyDown={e => e.key === 'Enter' && handleClick()}>
  Click me
</div>
```

### ARIA 라벨

```tsx
// Label interactive elements that lack visible text
<button aria-label="Close dialog"><XIcon /></button>

// Label form inputs
<label htmlFor="email">Email</label>
<input id="email" type="email" />

// Or use aria-label when no visible label exists
<input aria-label="Search tasks" type="search" />
```

### 집중 관리

```tsx
// Move focus when content changes
function Dialog({ isOpen, onClose }: DialogProps) {
  const closeRef = useRef<HTMLButtonElement>(null);

  useEffect(() => {
    if (isOpen) closeRef.current?.focus();
  }, [isOpen]);

  // Trap focus inside dialog when open
  return (
    <dialog open={isOpen}>
      <button ref={closeRef} onClick={onClose}>Close</button>
      {/* dialog content */}
    </dialog>
  );
}
```

### 의미 있는 비어 있음 및 오류 상태

```tsx
// Don't show blank screens
function TaskList({ tasks }: { tasks: Task[] }) {
  if (tasks.length === 0) {
    return (
      <div role="status" className="text-center py-12">
        <TasksEmptyIcon className="mx-auto h-12 w-12 text-muted" />
        <h3 className="mt-2 text-sm font-medium">No tasks</h3>
        <p className="mt-1 text-sm text-muted">Get started by creating a new task.</p>
        <Button className="mt-4" onClick={onCreateTask}>Create Task</Button>
      </div>
    );
  }

  return <ul role="list">...</ul>;
}
```

## 반응형 디자인

먼저 모바일용으로 디자인한 후 확장하세요.

```tsx
// Tailwind: mobile-first responsive
<div className="
  grid grid-cols-1      /* Mobile: single column */
  sm:grid-cols-2        /* Small: 2 columns */
  lg:grid-cols-3        /* Large: 3 columns */
  gap-4
">
```

320px, 768px, 1024px, 1440px 중단점에서 테스트하세요.

## 로딩 및 전환

```tsx
// Skeleton loading (not spinners for content)
function TaskListSkeleton() {
  return (
    <div className="space-y-3" aria-busy="true" aria-label="Loading tasks">
      {Array.from({ length: 3 }).map((_, i) => (
        <div key={i} className="h-12 bg-muted animate-pulse rounded" />
      ))}
    </div>
  );
}

// Optimistic updates for perceived speed
function useToggleTask() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: toggleTask,
    onMutate: async (taskId) => {
      await queryClient.cancelQueries({ queryKey: ['tasks'] });
      const previous = queryClient.getQueryData(['tasks']);

      queryClient.setQueryData(['tasks'], (old: Task[]) =>
        old.map(t => t.id === taskId ? { ...t, done: !t.done } : t)
      );

      return { previous };
    },
    onError: (_err, _taskId, context) => {
      queryClient.setQueryData(['tasks'], context?.previous);
    },
  });
}
```

## 참고 항목

자세한 접근성 요구 사항 및 테스트 도구는 `references/accessibility-checklist.md`를 참조하세요.

## 일반적인 합리화

| 합리화 | 현실 |
|---|---|
| "접근성은 nice-to-have입니다" | 이는 많은 관할권 및 엔지니어링 품질 표준에서 법적 요구 사항입니다. |
| "나중에 반응형으로 만들겠습니다" | 반응형 디자인을 개조하는 것은 처음부터 bui를 적용하는 것보다 3배 더 어렵습니다. |
| "디자인이 확정되지 않아 스타일링은 생략하겠습니다" | 디자인 시스템 기본값을 사용합니다. 스타일이 지정되지 않은 UI는 리뷰어에게 깨진 첫인상을 줍니다. |
| "이것은 단지 프로토타입일 뿐입니다" | 프로토타입은 생산 코드가 됩니다. Build 기초가 맞습니다. |
| "AI 미학은 현재로서는 괜찮습니다." | 품질이 낮다는 신호입니다. 처음부터 프로젝트의 실제 디자인 시스템을 사용하세요. |

## 위험 신호

- 200개 이상의 라인으로 구성된 컴포넌트(분할)
- 인라인 스타일 또는 임의의 픽셀 값
- 누락된 오류 상태, 로드 상태 또는 빈 상태
- 키보드 탐색 테스트 없음
- 상태를 나타내는 유일한 색상(텍스트나 아이콘이 없는 red/green)
- 일반 "AI 모양"(보라색 그라데이션, 대형 cards, 스톡 레이아웃)

## 확인

building UI 이후:

- [ ] 콘솔 오류 없이 구성 요소 렌더링
- [ ] 모든 대화형 요소는 키보드로 접근 가능합니다(페이지를 탭하여 이동).
- [ ] 스크린 리더는 페이지의 내용과 구조를 전달할 수 있습니다.
- [ ] 반응형: 320px, 768px, 1024px, 1440px에서 작동
- [ ] 로드, 오류 및 빈 상태가 모두 처리됨
- [ ] 프로젝트의 디자인 시스템(간격, 색상, 타이포그래피)을 따릅니다.
- [ ] 개발 도구 또는 axe-core에 접근성 경고가 없습니다.
