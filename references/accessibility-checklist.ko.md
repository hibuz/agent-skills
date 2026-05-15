# Accessibility Checklist

WCAG 2.1 AA 준수를 위한 빠른 참조. `frontend-ui-engineering` skill과 함께 사용하세요.

## 목차

- [필수 점검](#essential-checks)
- [흔한 HTML 패턴](#common-html-patterns)
- [테스트 도구](#testing-tools)
- [빠른 참조: ARIA Live Region](#quick-reference-aria-live-regions)
- [흔한 안티패턴](#common-anti-patterns)

## 필수 점검

### 키보드 내비게이션
- [ ] 모든 상호작용 요소가 Tab 키로 focus 가능
- [ ] focus 순서가 시각적/논리적 순서를 따름
- [ ] focus가 보임 (focus된 요소에 outline/ring)
- [ ] 커스텀 위젯에 키보드 지원 (Enter로 활성화, Escape로 닫기)
- [ ] 키보드 trap 없음 (사용자가 항상 컴포넌트에서 Tab으로 벗어날 수 있음)
- [ ] 페이지 상단에 skip-to-content 링크 - (최소한) 키보드 focus 시 보임
- [ ] 모달이 열려 있는 동안 focus를 trap하고, 닫을 때 focus 반환

### 스크린 리더
- [ ] 모든 이미지에 `alt` 텍스트 (장식용 이미지는 `alt=""`)
- [ ] 모든 form input에 연결된 label (`<label>` 또는 `aria-label`)
- [ ] 버튼과 링크에 서술적 텍스트 ("Click here"가 아님)
- [ ] 아이콘 전용 버튼에 `aria-label`
- [ ] 페이지에 하나의 `<h1>`, 제목이 레벨을 건너뛰지 않음
- [ ] 동적 콘텐츠 변경이 announce됨 (`aria-live` region)
- [ ] 테이블에 scope를 가진 `<th>` 헤더

### 시각
- [ ] 텍스트 대비 ≥ 4.5:1 (일반 텍스트) 또는 ≥ 3:1 (큰 텍스트, 18px+)
- [ ] UI 컴포넌트 대비 배경 대비 ≥ 3:1
- [ ] 색상이 정보를 전달하는 유일한 방법이 아님
- [ ] 레이아웃을 깨지 않고 텍스트를 200%로 resize 가능
- [ ] 초당 3회를 초과해 깜빡이는 콘텐츠 없음

### Form
- [ ] 모든 input에 보이는 label
- [ ] 필수 필드 표시 (색상만으로가 아님)
- [ ] 에러 메시지가 구체적이고 필드와 연결됨
- [ ] 에러 상태가 색상 이상으로 보임 (아이콘, 텍스트, 테두리)
- [ ] form 제출 에러가 요약되고 focus 가능
- [ ] 알려진 필드에 autocomplete 사용 (예: `type="email" autocomplete="email"`)

### 콘텐츠
- [ ] 언어 선언 (`<html lang="en">`)
- [ ] 페이지에 서술적 `<title>`
- [ ] 링크가 주변 텍스트와 구별됨 (색상만으로가 아님)
- [ ] 모바일에서 터치 타겟 ≥ 44x44px
- [ ] 의미 있는 빈 상태 (빈 화면이 아님)

## 흔한 HTML 패턴

### 버튼 vs. 링크

```html
<!-- 액션에는 <button> 사용 -->
<button onClick={handleDelete}>Delete Task</button>

<!-- 내비게이션에는 <a> 사용 -->
<a href="/tasks/123">View Task</a>

<!-- div/span을 버튼으로 절대 사용하지 말 것 -->
<div onClick={handleDelete}>Delete</div>  <!-- BAD -->
```

### Form Label

```html
<!-- 명시적 label 연결 -->
<label htmlFor="email">Email address</label>
<input id="email" type="email" required />

<!-- 암묵적 wrapping -->
<label>
  Email address
  <input type="email" required />
</label>

<!-- 숨겨진 label (보이는 label 권장) -->
<input type="search" aria-label="Search tasks" />
```

### ARIA Role

```html
<!-- 내비게이션 -->
<nav aria-label="Main navigation">...</nav>
<nav aria-label="Footer links">...</nav>

<!-- 상태 메시지 -->
<div role="status" aria-live="polite">Task saved</div>

<!-- 알림 메시지 -->
<div role="alert">Error: Title is required</div>

<!-- 모달 다이얼로그 -->
<dialog aria-modal="true" aria-labelledby="dialog-title">
  <h2 id="dialog-title">Confirm Delete</h2>
  ...
</dialog>

<!-- 로딩 상태 -->
<div aria-busy="true" aria-label="Loading tasks">
  <Spinner />
</div>
```

### 접근 가능한 List

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
# 자동 감사
npx axe-core          # 프로그래밍 방식 접근성 테스트
npx pa11y             # CLI 접근성 체커

# 브라우저에서
# Chrome DevTools → Lighthouse → Accessibility
# Chrome DevTools → Elements → Accessibility tree

# 스크린 리더 테스트
# macOS: VoiceOver (Cmd + F5)
# Windows: NVDA (무료) 또는 JAWS
# Linux: Orca
```

## 빠른 참조: ARIA Live Region

| 값 | 동작 | 사용 대상 |
|-------|----------|---------|
| `aria-live="polite"` | 다음 멈춤에서 announce | 상태 업데이트, 저장 확인 |
| `aria-live="assertive"` | 즉시 announce | 에러, 시간에 민감한 알림 |
| `role="status"` | `polite`와 동일 | 상태 메시지 |
| `role="alert"` | `assertive`와 동일 | 에러 메시지 |

## 흔한 안티패턴

| 안티패턴 | 문제 | 수정 |
|---|---|---|
| `div`를 버튼으로 | focus 불가, 키보드 지원 없음 | `<button>` 사용 |
| `alt` 텍스트 누락 | 스크린 리더에 이미지가 보이지 않음 | 서술적 `alt` 추가 |
| 색상 전용 상태 | 색맹 사용자에게 보이지 않음 | 아이콘, 텍스트, 또는 패턴 추가 |
| 자동 재생 미디어 | 혼란스럽고 멈출 수 없음 | 컨트롤 추가, 자동 재생 안 함 |
| ARIA 없는 커스텀 dropdown | 키보드/스크린 리더로 사용 불가 | 네이티브 `<select>` 또는 적절한 ARIA listbox 사용 |
| focus outline 제거 | 사용자가 자신의 위치를 볼 수 없음 | outline을 스타일링, 제거하지 말 것 |
| 빈 링크/버튼 | 설명 없이 "Link"라고 announce됨 | 텍스트나 `aria-label` 추가 |
| `tabindex > 0` | 자연스러운 tab 순서를 깨뜨림 | `tabindex="0"` 또는 `-1`만 사용 |
