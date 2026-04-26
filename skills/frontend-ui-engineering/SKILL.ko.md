---
name: frontend-ui-engineering
description: 프로덕션 품질의 UI를 구축합니다. 사용자 인터페이스를 만들거나 수정할 때 사용하세요. 컴포넌트를 생성하고, 레이아웃을 구현하고, 상태를 관리하거나, 결과물이 AI가 생성한 느낌이 아닌 프로덕션 품질의 룩앤필(look and feel)을 가져야 할 때 사용합니다.
---

# 프론트엔드 UI 엔지니어링 (Frontend UI Engineering)

## 개요 (Overview)

접근성(accessibility)이 뛰어나고, 성능이 좋으며, 시각적으로 세련된 프로덕션 품질의 사용자 인터페이스를 구축하세요. 목표는 AI가 생성한 것이 아니라 일류 기업의 디자인 감각이 있는 엔지니어가 만든 것처럼 보이는 UI를 만드는 것입니다. 이는 실제 디자인 시스템 준수, 적절한 접근성, 사려 깊은 상호작용 패턴을 의미하며, 전형적인 "AI 미학"을 배제하는 것을 의미합니다.

## 사용 시기 (When to Use)

- 새로운 UI 컴포넌트나 페이지를 구축할 때
- 기존의 사용자 인터페이스를 수정할 때
- 반응형 레이아웃을 구현할 때
- 상호작용이나 상태 관리를 추가할 때
- 시각적 또는 UX 문제를 해결할 때

## 컴포넌트 아키텍처 (Component Architecture)

### 파일 구조 (File Structure)

컴포넌트와 관련된 모든 것을 한곳에 두세요:

```
src/components/
  TaskList/
    TaskList.tsx          # 컴포넌트 구현
    TaskList.test.tsx     # 테스트
    TaskList.stories.tsx  # Storybook 스토리 (사용하는 경우)
    use-task-list.ts      # 커스텀 훅 (상태가 복잡한 경우)
    types.ts              # 컴포넌트 전용 타입 (필요한 경우)
```

### 컴포넌트 패턴 (Component Patterns)

**설정(configuration)보다 합성(composition)을 선호하세요:**

```tsx
// 좋음: 합성 가능함
<Card>
  <CardHeader>
    <CardTitle>Tasks</CardTitle>
  </CardHeader>
  <CardBody>
    <TaskList tasks={tasks} />
  </CardBody>
</Card>

// 피해야 함: 과도한 설정
<Card
  title="Tasks"
  headerVariant="large"
  bodyPadding="md"
  content={<TaskList tasks={tasks} />}
/>
```

**컴포넌트의 집중도를 유지하세요:**

```tsx
// 좋음: 한 가지 일만 수행
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

**데이터 페칭(data fetching)과 프레젠테이션을 분리하세요:**

```tsx
// 컨테이너: 데이터 처리
export function TaskListContainer() {
  const { tasks, isLoading, error } = useTasks();

  if (isLoading) return <TaskListSkeleton />;
  if (error) return <ErrorState message="Failed to load tasks" retry={refetch} />;
  if (tasks.length === 0) return <EmptyState message="No tasks yet" />;

  return <TaskList tasks={tasks} />;
}

// 프레젠테이션: 렌더링 처리
export function TaskList({ tasks }: { tasks: Task[] }) {
  return (
    <ul role="list" className="divide-y">
      {tasks.map(task => <TaskItem key={task.id} task={task} />)}
    </ul>
  );
}
```

## 상태 관리 (State Management)

**작동하는 가장 단순한 방식을 선택하세요:**

```
로컬 상태 (useState)           → 컴포넌트 전용 UI 상태
끌어올린 상태 (Lifted state)     → 2-3개의 형제 컴포넌트 간 공유
Context                       → 테마, 인증(auth), 로케일 (읽기 비중 높음, 쓰기 드묾)
URL 상태 (searchParams)        → 필터, 페이지네이션, 공유 가능한 UI 상태
서버 상태 (React Query, SWR)   → 캐싱이 포함된 원격 데이터
글로벌 스토어 (Zustand, Redux)  → 앱 전반에 공유되는 복잡한 클라이언트 상태
```

**3단계 이상의 프롭 드릴링(prop drilling)을 피하세요.** 해당 프롭을 사용하지 않는 컴포넌트를 거쳐 프롭을 전달하고 있다면, Context를 도입하거나 컴포넌트 트리를 재구성하세요.

## 디자인 시스템 준수 (Design System Adherence)

### AI 미학 피하기 (Avoid the AI Aesthetic)

AI가 생성한 UI에는 눈에 띄는 패턴이 있습니다. 다음을 모두 피하세요:

| AI 기본값 | 왜 문제인가 | 프로덕션 품질 |
|---|---|---|
| 모든 것에 보라/인디고색 사용 | 모델은 시각적으로 "안전한" 팔레트를 기본으로 선택하여 모든 앱을 똑같아 보이게 함 | 프로젝트의 실제 컬러 팔레트 사용 |
| 과도한 그라데이션 | 그라데이션은 시각적 노이즈를 더하고 대부분의 디자인 시스템과 충돌함 | 디자인 시스템에 맞는 플랫하거나 미묘한 그라데이션 사용 |
| 모든 것을 둥글게 처리 (rounded-2xl) | 최대치의 라운딩은 "친근함"을 주지만 실제 디자인의 모서리 반경(corner radii) 계층 구조를 무시함 | 디자인 시스템에 따른 일관된 border-radius 사용 |
| 일반적인 히어로 섹션 | 실제 콘텐츠나 사용자 요구와 연결되지 않은 템플릿 기반 레이아웃 | 콘텐츠 우선 레이아웃 |
| Lorem ipsum 스타일 카피 | 플레이스홀더 텍스트는 실제 콘텐츠가 드러내는 레이아웃 문제(길이, 줄바꿈, 넘침)를 가림 | 현실적인 플레이스홀더 콘텐츠 |
| 모든 곳에 과도한 패딩 | 일괄적인 넓은 패딩은 시각적 계층 구조를 파괴하고 화면 공간을 낭비함 | 일관된 간격 스케일(spacing scale) 사용 |
| 스톡 카드 그리드 | 획일적인 그리드는 정보 우선순위와 스캔 패턴을 무시하는 레이아웃 지름길임 | 목적 중심의 레이아웃 |
| 그림자가 짙은 디자인 | 겹쳐진 그림자는 콘텐츠와 충돌하는 깊이감을 주고 저사양 기기에서 렌더링을 늦춤 | 디자인 시스템에서 명시하지 않는 한 미묘하거나 그림자 없는 디자인 사용 |

### 간격 및 레이아웃 (Spacing and Layout)

일관된 간격 스케일을 사용하세요. 임의의 값을 만들어내지 마세요:

```css
/* 스케일 사용: 0.25rem 단위 (또는 프로젝트의 기준) */
/* 좋음 */  padding: 1rem;      /* 16px */
/* 좋음 */  gap: 0.75rem;       /* 12px */
/* 나쁨 */  padding: 13px;      /* 스케일에 없음 */
/* 나쁨 */  margin-top: 2.3rem; /* 스케일에 없음 */
```

### 타이포그래피 (Typography)

타이포그래피 계층 구조를 존중하세요:

```
h1 → 페이지 제목 (페이지당 하나)
h2 → 섹션 제목
h3 → 하위 섹션 제목
body → 기본 텍스트
small → 보조/도움말 텍스트
```

헤딩 레벨을 건너뛰지 마세요. 헤딩이 아닌 콘텐츠에 헤딩 스타일을 사용하지 마세요.

### 컬러 (Color)

- 시맨틱 컬러 토큰(semantic color tokens)을 사용하세요: `text-primary`, `bg-surface`, `border-default`. 가공되지 않은 헥스(hex) 값을 쓰지 마세요.
- 충분한 대비(contrast)를 확보하세요 (일반 텍스트 4.5:1, 큰 텍스트 3:1).
- 정보를 전달할 때 컬러에만 의존하지 마세요 (아이콘, 텍스트 또는 패턴도 함께 사용).

## 접근성 (Accessibility - WCAG 2.1 AA)

모든 컴포넌트는 다음 표준을 충족해야 합니다:

### 키보드 네비게이션 (Keyboard Navigation)

```tsx
// 모든 대화형 요소는 키보드로 접근 가능해야 함
<button onClick={handleClick}>Click me</button>        // ✓ 기본적으로 포커스 가능
<div onClick={handleClick}>Click me</div>               // ✗ 포커스 불가능
<div role="button" tabIndex={0} onClick={handleClick}    // ✓ 하지만 <button>을 선호
     onKeyDown={e => e.key === 'Enter' && handleClick()}>
  Click me
</div>
```

### ARIA 라벨 (ARIA Labels)

```tsx
// 시각적 텍스트가 없는 대화형 요소에 라벨 추가
<button aria-label="Close dialog"><XIcon /></button>

// 폼 입력 요소 라벨링
<label htmlFor="email">Email</label>
<input id="email" type="email" />

// 또는 시각적 라벨이 없을 때 aria-label 사용
<input aria-label="Search tasks" type="search" />
```

### 포커스 관리 (Focus Management)

```tsx
// 콘텐츠가 바뀔 때 포커스 이동
function Dialog({ isOpen, onClose }: DialogProps) {
  const closeRef = useRef<HTMLButtonElement>(null);

  useEffect(() => {
    if (isOpen) closeRef.current?.focus();
  }, [isOpen]);

  // 열려 있을 때 다이얼로그 내부에 포커스 가두기
  return (
    <dialog open={isOpen}>
      <button ref={closeRef} onClick={onClose}>Close</button>
      {/* 다이얼로그 콘텐츠 */}
    </dialog>
  );
}
```

### 의미 있는 빈 상태 및 에러 상태 (Meaningful Empty and Error States)

```tsx
// 빈 화면을 보여주지 마세요
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

## 반응형 디자인 (Responsive Design)

모바일 우선(mobile-first)으로 디자인한 후 확장하세요:

```tsx
// Tailwind: 모바일 우선 반응형
<div className="
  grid grid-cols-1      /* 모바일: 1열 */
  sm:grid-cols-2        /* 소형: 2열 */
  lg:grid-cols-3        /* 대형: 3열 */
  gap-4
">
```

다음 중단점(breakpoints)에서 테스트하세요: 320px, 768px, 1024px, 1440px.

## 로딩 및 트랜지션 (Loading and Transitions)

```tsx
// 스켈레톤 로딩 (콘텐츠 로딩 시 스피너 대신 사용)
function TaskListSkeleton() {
  return (
    <div className="space-y-3" aria-busy="true" aria-label="Loading tasks">
      {Array.from({ length: 3 }).map((_, i) => (
        <div key={i} className="h-12 bg-muted animate-pulse rounded" />
      ))}
    </div>
  );
}

// 체감 속도를 위한 낙관적 업데이트 (Optimistic updates)
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

상세한 접근성 요구 사항 및 테스트 도구는 `references/accessibility-checklist.ko.md`를 참조하세요.

## 흔한 자기합리화 (Common Rationalizations)

| 자기합리화 | 실제 상황 |
|---|---|
| "접근성은 있으면 좋은 기능이에요" | 많은 국가에서 법적 요구 사항이며 엔지니어링 품질 표준입니다. |
| "반응형은 나중에 작업할게요" | 반응형 디자인을 나중에 추가하는 것은 처음부터 구축하는 것보다 3배 더 어렵습니다. |
| "디자인이 확정 안 돼서 스타일링을 생략할게요" | 디자인 시스템 기본값을 사용하세요. 스타일링 안 된 UI는 리뷰어에게 깨진 첫인상을 줍니다. |
| "이건 그냥 프로토타입이에요" | 프로토타입이 프로덕션 코드가 됩니다. 처음부터 기초를 잘 다지세요. |
| "AI 미학도 지금은 괜찮아요" | 저품질이라는 신호를 줍니다. 처음부터 프로젝트의 실제 디자인 시스템을 사용하세요. |

## 위험 신호 (Red Flags)

- 200줄이 넘는 컴포넌트 (분할하세요)
- 인라인 스타일이나 임의의 픽셀 값 사용
- 에러 상태, 로딩 상태, 또는 빈 상태의 누락
- 키보드 네비게이션 테스트 부재
- 컬러가 상태의 유일한 지표인 경우 (텍스트나 아이콘 없이 빨강/초록만 사용)
- 전형적인 "AI 룩" (보라색 그라데이션, 과하게 큰 카드, 스톡 레이아웃)

## 검증 (Verification)

UI 구축 후:

- [ ] 컴포넌트가 콘솔 에러 없이 렌더링되는가
- [ ] 모든 대화형 요소가 키보드로 접근 가능한가 (Tab 키로 확인)
- [ ] 스크린 리더가 페이지 콘텐츠와 구조를 전달할 수 있는가
- [ ] 반응형: 320px, 768px, 1024px, 1440px에서 작동하는가
- [ ] 로딩, 에러, 빈 상태가 모두 처리되었는가
- [ ] 프로젝트의 디자인 시스템(간격, 컬러, 타이포그래피)을 따르는가
- [ ] 개발자 도구나 axe-core에서 접근성 경고가 없는가
