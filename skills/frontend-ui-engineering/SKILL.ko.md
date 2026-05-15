---
name: frontend-ui-engineering
description: production 품질 UI를 빌드. 사용자 대면 인터페이스를 빌드하거나 수정할 때 사용. 컴포넌트 생성, layout 구현, 상태 관리, 또는 출력이 AI 생성이 아니라 production 품질로 보이고 느껴져야 할 때 사용.
---

# Frontend UI Engineering

## Overview

접근 가능하고, 성능이 좋고, 시각적으로 다듬어진 production 품질 사용자 인터페이스를 빌드하세요. 목표는 AI가 생성한 것이 아니라 일류 회사의 디자인을 아는 엔지니어가 빌드한 것처럼 보이는 UI입니다. 이는 진짜 디자인 시스템 준수, 적절한 접근성, 사려 깊은 상호작용 패턴, 그리고 일반적인 "AI 미학" 없음을 의미합니다.

## When to Use

- 새 UI 컴포넌트나 페이지 빌드
- 기존 사용자 대면 인터페이스 수정
- 반응형 layout 구현
- 상호작용이나 상태 관리 추가
- 시각 또는 UX 이슈 수정

## 컴포넌트 아키텍처

### 파일 구조

컴포넌트와 관련된 모든 것을 colocate:

```
src/components/
  TaskList/
    TaskList.tsx          # 컴포넌트 구현
    TaskList.test.tsx     # 테스트
    TaskList.stories.tsx  # Storybook story (사용 시)
    use-task-list.ts      # 커스텀 hook (복잡한 상태 시)
    types.ts              # 컴포넌트별 타입 (필요 시)
```

### 컴포넌트 패턴

**configuration보다 composition 선호:**

```tsx
// 좋음: composable
<Card>
  <CardHeader>
    <CardTitle>Tasks</CardTitle>
  </CardHeader>
  <CardBody>
    <TaskList tasks={tasks} />
  </CardBody>
</Card>

// 피하기: 과도하게 configure됨
<Card
  title="Tasks"
  headerVariant="large"
  bodyPadding="md"
  content={<TaskList tasks={tasks} />}
/>
```

**컴포넌트를 집중되게 유지:**

```tsx
// 좋음: 한 가지를 함
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

**데이터 fetching을 presentation에서 분리:**

```tsx
// Container: 데이터 처리
export function TaskListContainer() {
  const { tasks, isLoading, error } = useTasks();

  if (isLoading) return <TaskListSkeleton />;
  if (error) return <ErrorState message="Failed to load tasks" retry={refetch} />;
  if (tasks.length === 0) return <EmptyState message="No tasks yet" />;

  return <TaskList tasks={tasks} />;
}

// Presentation: 렌더링 처리
export function TaskList({ tasks }: { tasks: Task[] }) {
  return (
    <ul role="list" className="divide-y">
      {tasks.map(task => <TaskItem key={task.id} task={task} />)}
    </ul>
  );
}
```

## 상태 관리

**동작하는 가장 단순한 접근을 선택:**

```
로컬 상태 (useState)             → 컴포넌트별 UI 상태
끌어올린 상태                    → 2-3개 형제 컴포넌트 간 공유
Context                          → theme, auth, locale (read-heavy, write-rare)
URL 상태 (searchParams)          → 필터, pagination, 공유 가능 UI 상태
서버 상태 (React Query, SWR)     → 캐싱된 원격 데이터
글로벌 store (Zustand, Redux)    → 앱 전역 공유 복잡한 클라이언트 상태
```

**3 레벨보다 깊은 prop drilling을 피하세요.** 사용하지 않는 컴포넌트를 통해 props를 전달하고 있다면, context를 도입하거나 컴포넌트 트리를 재구조화하세요.

## 디자인 시스템 준수

### AI 미학 피하기

AI가 생성한 UI에는 알아볼 수 있는 패턴이 있습니다. 그 모두를 피하세요:

| AI 기본값 | 왜 문제인가 | Production 품질 |
|---|---|---|
| 모든 게 보라/남색 | 모델이 시각적으로 "안전한" 팔레트로 기본 설정해 모든 앱이 똑같아 보임 | 프로젝트의 실제 색상 팔레트 사용 |
| 과도한 gradient | gradient는 시각 노이즈를 더하고 대부분 디자인 시스템과 충돌 | 디자인 시스템에 맞는 flat 또는 미묘한 gradient |
| 모든 게 둥글둥글 (rounded-2xl) | 최대 rounding은 "친근함"을 신호하지만 실제 디자인의 corner radius 위계를 무시 | 디자인 시스템의 일관된 border-radius |
| 일반적인 hero 섹션 | 실제 콘텐츠나 사용자 필요와 연결 없는 템플릿 주도 layout | 콘텐츠 우선 layout |
| Lorem ipsum 스타일 카피 | placeholder 텍스트는 실제 콘텐츠가 드러내는 layout 문제(길이, 줄바꿈, overflow)를 숨김 | 현실적 placeholder 콘텐츠 |
| 곳곳에 과대한 padding | 동등하게 넉넉한 padding은 시각 위계를 파괴하고 화면 공간을 낭비 | 일관된 spacing scale |
| 스톡 카드 그리드 | 균일한 그리드는 정보 우선순위와 스캐닝 패턴을 무시하는 layout 지름길 | 목적 주도 layout |
| 그림자 무거운 디자인 | 레이어드 그림자는 콘텐츠와 경쟁하는 깊이를 더하고 저사양 기기에서 렌더링을 늦춤 | 디자인 시스템이 명시하지 않는 한 미묘하거나 그림자 없음 |

### Spacing과 Layout

일관된 spacing scale을 사용하세요. 값을 발명하지 마세요:

```css
/* scale 사용: 0.25rem 증분 (또는 프로젝트가 쓰는 것) */
/* Good */  padding: 1rem;      /* 16px */
/* Good */  gap: 0.75rem;       /* 12px */
/* Bad */   padding: 13px;      /* 어떤 scale에도 없음 */
/* Bad */   margin-top: 2.3rem; /* 어떤 scale에도 없음 */
```

### Typography

타입 위계를 존중하세요:

```
h1 → 페이지 제목 (페이지당 하나)
h2 → 섹션 제목
h3 → 하위 섹션 제목
body → 기본 텍스트
small → 보조/헬퍼 텍스트
```

heading 레벨을 건너뛰지 마세요. heading이 아닌 콘텐츠에 heading 스타일을 쓰지 마세요.

### 색상

- 시맨틱 색상 토큰 사용: `text-primary`, `bg-surface`, `border-default` — raw hex 값이 아니라
- 충분한 대비 보장 (일반 텍스트 4.5:1, 큰 텍스트 3:1)
- 정보 전달에 색상에만 의존하지 말 것 (아이콘, 텍스트, 또는 패턴도 사용)

## 접근성 (WCAG 2.1 AA)

모든 컴포넌트는 이 표준을 충족해야 합니다:

### 키보드 내비게이션

```tsx
// 모든 상호작용 요소는 키보드 접근 가능해야 함
<button onClick={handleClick}>Click me</button>        // ✓ 기본적으로 focus 가능
<div onClick={handleClick}>Click me</div>               // ✗ focus 불가
<div role="button" tabIndex={0} onClick={handleClick}    // ✓ 하지만 <button> 선호
     onKeyDown={e => {
       if (e.key === 'Enter') handleClick();
       if (e.key === ' ') e.preventDefault();
     }}
     onKeyUp={e => {
       if (e.key === ' ') handleClick();
     }}>
  Click me
</div>
```

### ARIA Label

```tsx
// 보이는 텍스트가 없는 상호작용 요소에 label
<button aria-label="Close dialog"><XIcon /></button>

// form input에 label
<label htmlFor="email">Email</label>
<input id="email" type="email" />

// 또는 보이는 label이 없을 때 aria-label 사용
<input aria-label="Search tasks" type="search" />
```

### Focus 관리

```tsx
// 콘텐츠가 바뀔 때 focus 이동
function Dialog({ isOpen, onClose }: DialogProps) {
  const closeRef = useRef<HTMLButtonElement>(null);

  useEffect(() => {
    if (isOpen) closeRef.current?.focus();
  }, [isOpen]);

  // 열려 있을 때 dialog 안에 focus trap
  return (
    <dialog open={isOpen}>
      <button ref={closeRef} onClick={onClose}>Close</button>
      {/* dialog content */}
    </dialog>
  );
}
```

### 의미 있는 빈 상태와 에러 상태

```tsx
// 빈 화면을 보이지 말 것
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

모바일 우선으로 설계한 뒤 확장:

```tsx
// Tailwind: 모바일 우선 반응형
<div className="
  grid grid-cols-1      /* 모바일: 단일 컬럼 */
  sm:grid-cols-2        /* Small: 2 컬럼 */
  lg:grid-cols-3        /* Large: 3 컬럼 */
  gap-4
">
```

이 breakpoint에서 테스트하세요: 320px, 768px, 1024px, 1440px.

## 로딩과 Transition

```tsx
// Skeleton 로딩 (콘텐츠에 spinner가 아니라)
function TaskListSkeleton() {
  return (
    <div className="space-y-3" aria-busy="true" aria-label="Loading tasks">
      {Array.from({ length: 3 }).map((_, i) => (
        <div key={i} className="h-12 bg-muted animate-pulse rounded" />
      ))}
    </div>
  );
}

// 체감 속도를 위한 optimistic update
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

## See Also

상세한 접근성 요구사항과 테스트 도구는 `references/accessibility-checklist.md`를 참고하세요.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "접근성은 있으면 좋은 것" | 많은 관할권에서 법적 요구사항이며 엔지니어링 품질 표준입니다. |
| "나중에 반응형으로 만들게" | 반응형 디자인을 retrofit하는 것은 처음부터 빌드하는 것보다 3배 어렵습니다. |
| "디자인이 최종이 아니니 styling 건너뛸게" | 디자인 시스템 기본값을 사용하세요. 스타일 없는 UI는 리뷰어에게 깨진 첫인상을 만듭니다. |
| "이건 그냥 프로토타입" | 프로토타입은 production 코드가 됩니다. 기초를 제대로 빌드하세요. |
| "AI 미학이 지금은 괜찮아" | 낮은 품질을 신호합니다. 처음부터 프로젝트의 실제 디자인 시스템을 사용하세요. |

## Red Flags

- 200줄 이상의 컴포넌트 (분할하세요)
- 인라인 스타일이나 임의의 픽셀 값
- 에러 상태, 로딩 상태, 또는 빈 상태 누락
- 키보드 내비게이션 테스트 없음
- 상태의 유일한 표시로서의 색상 (텍스트나 아이콘 없는 빨강/초록)
- 일반적인 "AI 룩" (보라 gradient, 과대 카드, 스톡 layout)

## Verification

UI 빌드 후:

- [ ] 컴포넌트가 console 에러 없이 렌더링
- [ ] 모든 상호작용 요소가 키보드 접근 가능 (페이지를 Tab으로 통과)
- [ ] 스크린 리더가 페이지의 콘텐츠와 구조를 전달 가능
- [ ] 반응형: 320px, 768px, 1024px, 1440px에서 동작
- [ ] 로딩, 에러, 빈 상태 모두 처리됨
- [ ] 프로젝트의 디자인 시스템을 따름 (spacing, 색상, typography)
- [ ] dev tools나 axe-core에 접근성 경고 없음
