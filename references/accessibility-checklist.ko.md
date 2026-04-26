# Accessibility Checklist

WCAG 2.1 AA 준수를 위한 빠른 참조 Checklist입니다. `frontend-ui-engineering` 기술과 함께 사용하세요.

## Table of Contents

- [Essential Checks](#essential-checks)
- [Common HTML Patterns](#common-html-patterns)
- [Testing Tools](#testing-tools)
- [Quick Reference: ARIA Live Regions](#quick-reference-aria-live-regions)
- [Common Anti-Patterns](#common-anti-patterns)

## Essential Checks

### Keyboard Navigation
- [ ] 모든 대화형 Element가 Tab 키로 Focus 가능함
- [ ] Focus 순서가 시각적/논리적 순서를 따름
- [ ] Focus 상태가 시각적으로 표시됨 (Focused Element에 Outline/Ring 표시)
- [ ] 커스텀 위젯이 키보드를 지원함 (Enter로 실행, Escape로 닫기)
- [ ] Keyboard Trap 없음 (사용자가 항상 Component에서 Tab으로 빠져나갈 수 있음)
- [ ] 페이지 상단에 Skip-to-content 링크 제공
- [ ] Modal이 열려 있는 동안 Focus를 가두고(Trap), 닫힐 때 Focus를 복원함

### Screen Readers
- [ ] 모든 이미지에 `alt` 텍스트 제공 (장식용 이미지의 경우 `alt=""`)
- [ ] 모든 Form Input에 연관된 Label이 있음 (`<label>` 또는 `aria-label`)
- [ ] 버튼과 링크에 설명적인 텍스트가 있음 ("여기 클릭" 금지)
- [ ] 아이콘 전용 버튼에 `aria-label` 제공
- [ ] 페이지에 하나의 `<h1>`이 있으며 Heading 레벨을 건너뛰지 않음
- [ ] 동적 콘텐츠 변경 사항이 안내됨 (`aria-live` 영역)
- [ ] Table에 Scope가 포함된 `<th>` Header가 있음

### Visual
- [ ] 텍스트 대비 ≥ 4.5:1 (일반 텍스트) 또는 ≥ 3:1 (큰 텍스트, 18px 이상)
- [ ] UI Component 대비가 배경색과 ≥ 3:1임
- [ ] 색상이 정보를 전달하는 유일한 방법이 아니어야 함
- [ ] Layout이 깨지지 않고 텍스트를 200%까지 확대 가능함
- [ ] 초당 3번 이상 깜빡이는 콘텐츠 없음

### Forms
- [ ] 모든 Input에 시각적인 Label이 있음
- [ ] 필수 필드가 표시됨 (색상만으로 표시하지 않음)
- [ ] Error 메시지가 구체적이며 해당 필드와 연관됨
- [ ] Error 상태가 색상 외의 수단(아이콘, 텍스트, 테두리)으로도 표시됨
- [ ] Form Submission Error가 요약되어 제공되며 Focus 가능함

### Content
- [ ] 언어 선언됨 (`<html lang="ko">`)
- [ ] 페이지에 설명적인 `<title>`이 있음
- [ ] 링크가 주변 텍스트와 구별됨 (색상만으로 구별하지 않음)
- [ ] 모바일에서 Touch Target ≥ 44x44px
- [ ] 의미 있는 Empty State 제공 (빈 화면 금지)

## Common HTML Patterns

### Buttons vs. Links

```html
<!-- 동작(Action)을 위해 <button> 사용 -->
<button onClick={handleDelete}>Delete Task</button>

<!-- 탐색(Navigation)을 위해 <a> 사용 -->
<a href="/tasks/123">View Task</a>

<!-- div/span을 버튼으로 절대 사용하지 마세요 -->
<div onClick={handleDelete}>Delete</div>  <!-- BAD -->
```

### Form Labels

```html
<!-- 명시적인 Label 연결 -->
<label htmlFor="email">Email address</label>
<input id="email" type="email" required />

<!-- 암시적인 래핑 -->
<label>
  Email address
  <input type="email" required />
</label>

<!-- 숨겨진 Label (시각적 Label을 선호함) -->
<input type="search" aria-label="Search tasks" />
```

### ARIA Roles

```html
<!-- Navigation -->
<nav aria-label="Main navigation">...</nav>
<nav aria-label="Footer links">...</nav>

<!-- 상태 메시지 -->
<div role="status" aria-live="polite">Task saved</div>

<!-- 경고 메시지 -->
<div role="alert">Error: Title is required</div>

<!-- Modal Dialog -->
<dialog aria-modal="true" aria-labelledby="dialog-title">
  <h2 id="dialog-title">Confirm Delete</h2>
  ...
</dialog>

<!-- Loading 상태 -->
<div aria-busy="true" aria-label="Loading tasks">
  <Spinner />
</div>
```

### Accessible Lists

```html
<ul role="list" aria-label="Tasks">
  <li>
    <input type="checkbox" id="task-1" aria-label="Complete: Buy groceries" />
    <label htmlFor="task-1">Buy groceries</label>
  </li>
</ul>
```

## Testing Tools

```bash
# Automated Audit
npx axe-core          # 프로그래밍 방식의 Accessibility 테스트
npx pa11y             # CLI Accessibility 체커

# 브라우저에서 확인
# Chrome DevTools → Lighthouse → Accessibility
# Chrome DevTools → Elements → Accessibility tree

# Screen Reader 테스트
# macOS: VoiceOver (Cmd + F5)
# Windows: NVDA (무료) 또는 JAWS
# Linux: Orca
```

## Quick Reference: ARIA Live Regions

| Value | Behavior | Use For |
|-------|----------|---------|
| `aria-live="polite"` | 다음 일시 중지 시 안내됨 | 상태 업데이트, 저장 확인 |
| `aria-live="assertive"` | 즉시 안내됨 | 오류, 시간에 민감한 알림 |
| `role="status"` | `polite`와 동일 | 상태 메시지 |
| `role="alert"` | `assertive`와 동일 | 오류 메시지 |

## Common Anti-Patterns

| Anti-Pattern | Problem | Fix |
|---|---|---|
| `div`를 버튼으로 사용 | Focus 불가, 키보드 지원 안 됨 | `<button>` 사용 |
| `alt` 텍스트 누락 | Screen Reader에서 이미지 인지 불가 | 설명적인 `alt` 추가 |
| 색상만으로 표시된 상태 | 색맹 사용자가 인지 불가 | 아이콘, 텍스트 또는 패턴 추가 |
| 미디어 자동 재생 | 혼란을 주며 중지할 수 없음 | 컨트롤 추가, 자동 재생 금지 |
| ARIA 없는 커스텀 Dropdown | 키보드/Screen Reader 사용 불가 | 네이티브 `<select>` 또는 적절한 ARIA listbox 사용 |
| Focus Outline 제거 | 사용자가 자신의 위치를 알 수 없음 | Outline에 Style을 적용하되 제거하지 말 것 |
| 내용 없는 링크/버튼 | 설명 없이 "Link"라고만 안내됨 | 텍스트 또는 `aria-label` 추가 |
| `tabindex > 0` | 자연스러운 Tab 순서를 해침 | `tabindex="0"` 또는 `-1`만 사용 |
