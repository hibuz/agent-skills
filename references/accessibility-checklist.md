# 접근성 체크리스트

WCAG 2.1 AA 규정 준수를 위한 Quick 참조입니다. `frontend-ui-engineering` skill와 함께 사용하세요.

## 목차

- [Essential Checks](#essential-checks)
- [Common HTML Patterns](#common-html-patterns)
- [Testing Tools](#testing-tools)
- [Quick Reference: ARIA Live Regions](#quick-reference-aria-live-regions)
- [Common Anti-Patterns](#common-anti-patterns)

## 필수 점검사항

### 키보드 탐색
- [ ] Tab 키를 통해 초점을 맞출 수 있는 모든 대화형 요소
- [ ] 초점 순서는 Visual/logical 순서를 따릅니다.
- [ ] 포커스가 표시됩니다(포커스된 요소의 개요/ring).
- [ ] 사용자 정의 위젯에는 키보드 지원이 있습니다(활성화하려면 Enter, 닫으려면 Esc).
- [ ] No keyboard traps (user can always Tab away from a component)
- [ ] 페이지 상단의 to-content 링크 건너뛰기
- [ ] 열려 있는 동안 모달 트랩 포커스, 닫혀 있는 동안 포커스 반환

### 스크린 리더
- [ ] 모든 이미지에는 `alt` 텍스트가 있습니다(또는 장식용 이미지의 경우 `alt=""`).
- [ ] 모든 form 입력에는 관련 레이블이 있습니다(`<label>` 또는 `aria-label`).
- [ ] 버튼과 링크에 설명 텍스트가 있습니다("Click here" 아님).
- [ ] 아이콘 전용 버튼에는 `aria-label`가 있습니다.
- [ ] 페이지에 하나의 `<h1>`가 있고 제목이 수준을 건너뛰지 않습니다.
- [ ] 동적 콘텐츠 변경 사항 발표(`aria-live` 지역)
- [ ] 테이블에는 범위가 있는 `<th>` 헤더가 있습니다.

### 비주얼
- [ ] 텍스트 대비 ≥ 4.5:1(normal 텍스트) 또는 ≥ 3:1(큰 텍스트, 18px+)
- [ ] UI 구성요소 대비 배경 대비 ≥ 3:1
- [ ] 색상만이 information을 전달하는 유일한 방법은 아닙니다
- [ ] 레이아웃을 깨지 않고 200%까지 크기 조정 가능한 텍스트
- [ ] 초당 3회 이상 깜박이는 콘텐츠 없음

### Forms
- [ ] 모든 입력에는 눈에 보이는 레이블이 있습니다.
- [ ] Requi 빨간색 필드가 표시됨(색상만으로 표시되지 않음)
- [ ] 해당 필드와 관련된 구체적인 오류 메시지
- [ ] 색상(아이콘, 텍스트, 테두리) 이상으로 보이는 오류 상태
- [ ] Form 제출 오류가 요약되고 집중 가능

### 내용
- [ ] 언어 선언(`<html lang="en">`)
- [ ] 페이지에 설명적인 `<title>`가 있습니다.
- [ ] 주변 텍스트와 uish를 구분하는 링크(색상만으로 표시되지 않음)
- [ ] 모바일에서 터치 대상 ≥ 44x44px
- [ ] 의미 있는 빈 상태(빈 화면 아님)

## 일반적인 HTML 패턴

### 버튼과 링크

```html
<!-- Use <button> for actions -->
<button onClick={handleDelete}>Delete Task</button>

<!-- Use <a> for navigation -->
<a href="/tasks/123">View Task</a>

<!-- NEVER use div/span as buttons -->
<div onClick={handleDelete}>Delete</div>  <!-- BAD -->
```

### Form 라벨

```html
<!-- Explicit label association -->
<label htmlFor="email">Email address</label>
<input id="email" type="email" required />

<!-- Implicit wrapping -->
<label>
  Email address
  <input type="email" required />
</label>

<!-- Hidden label (visible label preferred) -->
<input type="search" aria-label="Search tasks" />
```

### ARIA 역할

```html
<!-- Navigation -->
<nav aria-label="Main navigation">...</nav>
<nav aria-label="Footer links">...</nav>

<!-- Status messages -->
<div role="status" aria-live="polite">Task saved</div>

<!-- Alert messages -->
<div role="alert">Error: Title is required</div>

<!-- Modal dialogs -->
<dialog aria-modal="true" aria-labelledby="dialog-title">
  <h2 id="dialog-title">Confirm Delete</h2>
  ...
</dialog>

<!-- Loading states -->
<div aria-busy="true" aria-label="Loading tasks">
  <Spinner />
</div>
```

### 접근 가능한 목록

```html
<ul role="list" aria-label="Tasks">
  <li>
    <input type="checkbox" id="task-1" aria-label="Complete: Buy groceries" />
    <label htmlFor="task-1">Buy groceries</label>
  </li>
</ul>
```

## 테스트 도구

```bash
# Automated audit
npx axe-core          # Programmatic accessibility testing
npx pa11y             # CLI accessibility checker

# In browser
# Chrome DevTools → Lighthouse → Accessibility
# Chrome DevTools → Elements → Accessibility tree

# Screen reader testing
# macOS: VoiceOver (Cmd + F5)
# Windows: NVDA (free) or JAWS
# Linux: Orca
```

## Quick 참조: ARIA 라이브 지역

| 가치 | 행동 | 사용 대상 |
|-------|----------|---------|
| `aria-live="polite"` | 다음 일시에 발표됨 | 상태 업데이트, 저장된 확인 |
| `aria-live="assertive"` | 즉시 발표 | 오류, time-sensitive 경고 |
| `role="status"` | `polite`와 동일 | 상태 메시지 |
| `role="alert"` | `assertive`와 동일 | 오류 메시지 |

## 공통 Anti-Patterns

| Anti-Pattern | 문제 | 수정 |
|---|---|---|
| `div` 버튼으로 | 초점을 맞출 수 없으며 키보드가 지원되지 않습니다 | `<button>` 사용 |
| `alt` 텍스트 누락 | 스크린 리더에 표시되지 않는 이미지 | 설명적인 `alt` 추가 |
| 색상 전용 상태 | color-blind 사용자에게 표시되지 않음 | 아이콘, 텍스트 또는 패턴 추가 |
| 미디어 자동재생 | Disorienting, can't be stopped | 컨트롤 추가, 자동 재생 안 함 |
| ARIA가 없는 사용자 정의 드롭다운 | 키보드/screen 리더에서 사용할 수 없음 | 기본 `<select>` 또는 적절한 ARIA 목록 상자 사용 |
| 초점 윤곽선 제거 | 사용자는 자신이 어디에 있는지 볼 수 없습니다 | 윤곽선 스타일을 지정하고 제거하지 마세요 |
| 빈 링크/buttons | 설명 없이 발표된 "링크" | 텍스트 추가 또는 `aria-label` |
| `tabindex > 0` | 자연스러운 탭 순서가 깨집니다 | `tabindex="0"` 또는 `-1`만 사용 |
