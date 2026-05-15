---
name: source-driven-development
description: 모든 구현 결정을 공식 문서에 근거. 구식 패턴 없는 출처 인용된 권위 있는 코드를 원할 때 사용. 정확성이 중요한 어떤 프레임워크나 라이브러리로 빌드할 때 사용.
---

# Source-Driven Development

## Overview

모든 프레임워크별 코드 결정은 공식 문서로 뒷받침되어야 합니다. 기억으로 구현하지 마세요 — 검증하고, 인용하고, 사용자가 출처를 보게 하세요. 학습 데이터는 오래되고, API는 deprecate되고, 모범 사례는 진화합니다. 이 skill은 모든 패턴이 사용자가 확인할 수 있는 권위 있는 출처로 추적되기 때문에 사용자가 신뢰할 수 있는 코드를 받도록 보장합니다.

## When to Use

- 사용자가 주어진 프레임워크의 현재 모범 사례를 따르는 코드를 원함
- 프로젝트 전반에 복사될 boilerplate, starter 코드, 또는 패턴 빌드
- 사용자가 문서화되고, 검증되고, "올바른" 구현을 명시적으로 요청
- 프레임워크의 권장 접근이 중요한 기능 구현 (form, routing, 데이터 fetching, 상태 관리, auth)
- 프레임워크별 패턴을 쓰는 코드 리뷰나 개선
- 프레임워크별 코드를 기억으로 작성하려 할 때 언제든

**사용하지 말아야 할 때:**

- 정확성이 특정 버전에 의존하지 않음 (변수 이름 변경, 오타 수정, 파일 이동)
- 모든 버전에서 동일하게 작동하는 순수 로직 (루프, conditional, 자료구조)
- 사용자가 검증보다 속도를 명시적으로 원함 ("그냥 빨리 해")

## 프로세스

```
DETECT ──→ FETCH ──→ IMPLEMENT ──→ CITE
  │          │           │            │
  ▼          ▼           ▼            ▼
 어떤       관련        문서화된     출처를
 스택?      문서        패턴을       보여줌
            가져오기    따름
```

### Step 1: 스택과 버전 감지

프로젝트의 의존성 파일을 읽어 정확한 버전 식별:

```
package.json    → Node/React/Vue/Angular/Svelte
composer.json   → PHP/Symfony/Laravel
requirements.txt / pyproject.toml → Python/Django/Flask
go.mod          → Go
Cargo.toml      → Rust
Gemfile         → Ruby/Rails
```

발견한 것을 명시적으로 진술:

```
STACK DETECTED:
- React 19.1.0 (from package.json)
- Vite 6.2.0
- Tailwind CSS 4.0.3
→ Fetching official docs for the relevant patterns.
```

버전이 없거나 모호하면, **사용자에게 물으세요**. 추측하지 마세요 — 버전이 어떤 패턴이 올바른지 결정합니다.

### Step 2: 공식 문서 Fetch

구현하는 기능에 대한 특정 문서 페이지를 fetch하세요. 홈페이지가 아니라, 전체 문서가 아니라 — 관련 페이지입니다.

**출처 위계 (권위 순):**

| 우선순위 | 출처 | 예시 |
|----------|--------|---------|
| 1 | 공식 문서 | react.dev, docs.djangoproject.com, symfony.com/doc |
| 2 | 공식 블로그 / changelog | react.dev/blog, nextjs.org/blog |
| 3 | 웹 표준 참조 | MDN, web.dev, html.spec.whatwg.org |
| 4 | browser/런타임 호환성 | caniuse.com, node.green |

**권위 없음 — 절대 주 출처로 인용하지 말 것:**

- Stack Overflow 답변
- 블로그 게시물이나 튜토리얼 (인기 있는 것도)
- AI 생성 문서나 요약
- 자신의 학습 데이터 (그게 핵심 — 검증하세요)

**fetch하는 것에 정확하세요:**

```
나쁨:  React 홈페이지 fetch
좋음:  react.dev/reference/react/useActionState fetch

나쁨:  "django authentication best practices" 검색
좋음:  docs.djangoproject.com/en/6.0/topics/auth/ fetch
```

fetch 후, 핵심 패턴을 추출하고 deprecation 경고나 migration 가이드를 메모하세요.

공식 출처들이 서로 충돌할 때(예: migration 가이드가 API 참조와 모순), 불일치를 사용자에게 표면화하고 어느 패턴이 감지된 버전에 대해 실제로 동작하는지 검증하세요.

### Step 3: 문서화된 패턴을 따라 구현

문서가 보여주는 것에 맞는 코드를 작성:

- 기억이 아니라 문서의 API 시그니처 사용
- 문서가 무언가를 하는 새 방법을 보이면, 새 방법 사용
- 문서가 패턴을 deprecate하면, deprecate된 버전을 쓰지 말 것
- 문서가 무언가를 다루지 않으면, 미검증으로 표시

**문서가 기존 프로젝트 코드와 충돌할 때:**

```
CONFLICT DETECTED:
The existing codebase uses useState for form loading state,
but React 19 docs recommend useActionState for this pattern.
(Source: react.dev/reference/react/useActionState)

Options:
A) Use the modern pattern (useActionState) — consistent with current docs
B) Match existing code (useState) — consistent with codebase
→ Which approach do you prefer?
```

충돌을 표면화하세요. 조용히 하나를 고르지 마세요.

### Step 4: 출처 인용

모든 프레임워크별 패턴은 인용을 받습니다. 사용자가 모든 결정을 검증할 수 있어야 합니다.

**코드 주석에:**

```typescript
// React 19 form handling with useActionState
// Source: https://react.dev/reference/react/useActionState#usage
const [state, formAction, isPending] = useActionState(submitOrder, initialState);
```

**대화에서:**

```
I'm using useActionState instead of manual useState for the
form submission state. React 19 replaced the manual
isPending/setIsPending pattern with this hook.

Source: https://react.dev/blog/2024/12/05/react-19#actions
"useTransition now supports async functions [...] to handle
pending states automatically"
```

**인용 규칙:**

- 단축이 아니라 전체 URL
- 가능한 곳에 anchor가 있는 deep link 선호 (예: `/useActionState`보다 `/useActionState#usage`) — anchor는 문서 재구조화에서 top-level 페이지보다 잘 살아남음
- 명백하지 않은 결정을 뒷받침할 때 관련 구절 인용
- 플랫폼 기능을 권장할 때 browser/런타임 지원 데이터 포함
- 패턴에 대한 문서를 못 찾으면, 명시적으로 그렇다고 말하세요:

```
UNVERIFIED: I could not find official documentation for this
pattern. This is based on training data and may be outdated.
Verify before using in production.
```

검증 못 한 것에 대한 정직함이 거짓 확신보다 더 가치 있습니다.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "이 API에 확신해" | 확신은 증거가 아닙니다. 학습 데이터는 올바르게 보이지만 현재 버전에 깨지는 구식 패턴을 포함합니다. 검증하세요. |
| "문서 fetch는 토큰 낭비야" | API를 hallucinate하는 게 더 낭비입니다. 사용자가 한 시간 디버깅한 뒤 함수 시그니처가 바뀐 것을 발견합니다. fetch 한 번이 몇 시간의 재작업을 방지합니다. |
| "문서에 필요한 게 없을 거야" | 문서가 다루지 않으면, 그것은 가치 있는 정보입니다 — 패턴이 공식 권장이 아닐 수 있습니다. |
| "그냥 구식일 수 있다고 언급할게" | 면책 조항은 도움이 안 됩니다. 검증하고 인용하거나, 명확히 미검증으로 표시하세요. hedging이 최악의 옵션입니다. |
| "단순한 작업이라 확인할 필요 없어" | 잘못된 패턴의 단순한 작업이 템플릿이 됩니다. 사용자가 현대적 접근이 존재함을 발견하기 전에 deprecate된 form 핸들러를 열 개 컴포넌트에 복사합니다. |

## Red Flags

- 그 버전의 문서를 확인하지 않고 프레임워크별 코드 작성
- API에 대해 출처를 인용하는 대신 "I believe"나 "I think" 사용
- 어느 버전에 적용되는지 모르고 패턴 구현
- 공식 문서 대신 Stack Overflow나 블로그 게시물 인용
- 학습 데이터에 나온다고 deprecate된 API 사용
- 구현 전 `package.json` / 의존성 파일을 읽지 않음
- 프레임워크별 결정에 출처 인용 없이 코드 전달
- 한 페이지만 관련 있는데 전체 문서 사이트 fetch

## Verification

source-driven development로 구현 후:

- [ ] 프레임워크와 라이브러리 버전이 의존성 파일에서 식별됨
- [ ] 프레임워크별 패턴에 대해 공식 문서가 fetch됨
- [ ] 모든 출처가 블로그 게시물이나 학습 데이터가 아니라 공식 문서
- [ ] 코드가 현재 버전 문서가 보이는 패턴을 따름
- [ ] 비자명한 결정이 전체 URL과 함께 출처 인용을 포함
- [ ] deprecate된 API가 사용되지 않음 (migration 가이드에 대해 확인)
- [ ] 문서와 기존 코드 간 충돌이 사용자에게 표면화됨
- [ ] 검증할 수 없는 모든 것이 명시적으로 미검증으로 표시됨
