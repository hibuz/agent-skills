---
name: source-driven-development
description: 공식 문서의 모든 구현 결정을 근거로 삼습니다. 오래된 패턴이 없는 신뢰할 수 있는 source-cited 코드를 원할 때 사용하세요. 정확성이 중요한 프레임워크나 라이브러리와 함께 building을 수행할 때 사용하세요.
---

# 소스 중심 개발

## 개요

모든 framework-specific 코드 결정은 공식 문서의 뒷받침을 받아야 합니다. 외워서 구현하지 마세요. 확인하고, 인용하고, 사용자가 소스를 볼 수 있도록 하세요. 훈련 데이터는 오래되고 APIs는 더 이상 사용되지 않으며 모범 사례가 발전합니다. 이 skill는 모든 패턴이 확인할 수 있는 신뢰할 수 있는 소스를 추적하므로 사용자가 신뢰할 수 있는 코드를 얻을 수 있도록 보장합니다.

## 사용 시기

- 사용자는 특정 프레임워크에 대한 현재 모범 사례를 따르는 코드를 원합니다.
- 프로젝트 전체에 복사될 Building 상용구, 시작 코드 또는 패턴
- 사용자가 명시적으로 문서화, 검증 또는 "올바른" 구현을 요구합니다.
- 프레임워크의 권장 접근 방식이 중요한 기능 구현(forms, 라우팅, 데이터 가져오기, 상태 관리, 인증)
- framework-specific 패턴을 사용하는 코드 검토 또는 개선
- 메모리에서 framework-specific 코드를 작성하려고 할 때마다

**사용하지 말아야 할 때:**

- 정확성은 특정 버전에 의존하지 않습니다(변수 이름 바꾸기, 오타 수정, 파일 이동).
- 모든 버전(루프, 조건부, 데이터 구조)에서 동일하게 작동하는 순수 논리
- 사용자는 검증보다 속도를 명시적으로 원합니다("그냥 quickly로 하세요").

## 프로세스

```
DETECT ──→ FETCH ──→ IMPLEMENT ──→ CITE
  │          │           │            │
  ▼          ▼           ▼            ▼
 What       Get the    Follow the   Show your
 stack?     relevant   documented   sources
            docs       patterns
```

### 1단계: 스택 및 버전 감지

정확한 버전을 식별하려면 프로젝트의 종속성 파일을 읽으십시오.

```
package.json    → Node/React/Vue/Angular/Svelte
composer.json   → PHP/Symfony/Laravel
requirements.txt / pyproject.toml → Python/Django/Flask
go.mod          → Go
Cargo.toml      → Rust
Gemfile         → Ruby/Rails
```

발견한 내용을 명시적으로 기술하세요.

```
STACK DETECTED:
- React 19.1.0 (from package.json)
- Vite 6.2.0
- Tailwind CSS 4.0.3
→ Fetching official docs for the relevant patterns.
```

버전이 누락되었거나 모호한 경우 **사용자에게 문의**하세요. 추측하지 마십시오. 버전에 따라 어떤 패턴이 올바른지 결정됩니다.

### 2단계: 공식 문서 가져오기

구현 중인 기능에 대한 특정 문서 페이지를 가져옵니다. 홈페이지나 전체 문서가 아닌 관련 페이지입니다.

**소스 계층 구조(권한 순서):**

| 우선순위 | 소스 | 예 |
|----------|--------|---------|
| 1 | 공식 문서 | react.dev, docs.djangoproject.com, Symfony.com/doc |
| 2 | 공식 블로그 / 변경 로그 | react.dev/blog, nextjs.org/blog |
| 3 | 웹 표준rds 참조 | MDN, web.dev, html.spec.whatwg.org |
| 4 | 브라우저/runtime 호환성 | caniuse.com, node.green |

**신뢰할 수 있는 정보가 아닙니다. 주요 출처로 인용하지 마세요.**

- 스택 오버플로 답변
- 블로그 게시물 또는 튜토리얼(인기 있는 게시물도 포함)
- AI에서 생성된 문서 또는 요약
- 자신만의 훈련 데이터(즉, 확인하세요)

**가져오는 내용을 정확하게 입력하세요.**

```
BAD:  Fetch the React homepage
GOOD: Fetch react.dev/reference/react/useActionState

BAD:  Search "django authentication best practices"
GOOD: Fetch docs.djangoproject.com/en/6.0/topics/auth/
```

가져온 후 주요 패턴을 추출하고 지원 중단 경고 또는 마이그레이션 guidance를 기록해 둡니다.

공식 소스가 서로 충돌하는 경우(예: 마이그레이션 guide가 API 참조와 모순되는 경우) 사용자에게 불일치를 알리고 감지된 버전에 대해 실제로 작동하는 패턴이 무엇인지 확인하세요.

### 3단계: 문서화된 다음 패턴 구현

문서에 표시된 내용과 일치하는 코드를 작성하세요.

- 메모리가 아닌 문서의 API 서명을 사용하세요.
- 문서에서 무언가를 수행하는 새로운 방법을 보여주면 새로운 방법을 사용하세요.
- 문서에서 패턴을 더 이상 사용하지 않는 경우 더 이상 사용되지 않는 버전을 사용하지 마세요.
- 문서에서 다루지 않는 내용이 있으면 확인되지 않은 것으로 표시하세요.

**문서가 기존 프로젝트 코드와 충돌하는 경우:**

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

갈등을 표면화하세요. 조용히 하나를 선택하지 마십시오.

### 4단계: 출처 인용

모든 framework-specific 패턴은 인용을 받습니다. 사용자는 모든 결정을 확인할 수 있어야 합니다.

**코드 주석에서:**

```typescript
// React 19 form handling with useActionState
// Source: https://react.dev/reference/react/useActionState#usage
const [state, formAction, isPending] = useActionState(submitOrder, initialState);
```

**대화 중:**

```
I'm using useActionState instead of manual useState for the
form submission state. React 19 replaced the manual
isPending/setIsPending pattern with this hook.

Source: https://react.dev/blog/2024/12/05/react-19#actions
"useTransition now supports async functions [...] to handle
pending states automatically"
```

**인용 규칙:**

- 단축되지 않은 전체 URLs
- 가능한 경우 앵커가 있는 딥 링크를 선호합니다(예: `/useActionState`보다 `/useActionState#usage`) — 앵커는 top-level 페이지보다 문서 rest 구조에서 더 잘 살아남습니다.
- non-obvious 결정을 뒷받침하는 경우 관련 구절을 인용하세요.
- platform 기능 추천 시 browser/runtime 지원 데이터 포함
- 패턴에 대한 문서를 찾을 수 없다면 명시적으로 말하세요.

```
UNVERIFIED: I could not find official documentation for this
pattern. This is based on training data and may be outdated.
Verify before using in production.
```

검증할 수 없는 것에 대한 솔직함이 잘못된 확신보다 더 가치 있습니다.

## 일반적인 합리화

| 합리화 | 현실 |
|---|---|
| "나는 이 API에 대해 확신합니다" | 자신감은 증거가 아닙니다. 학습 데이터에는 올바르게 보이지만 현재 버전과 맞지 않는 오래된 패턴이 포함되어 있습니다. 확인하다. |
| "문서를 가져오는 것은 토큰을 낭비합니다" | API를 환각시키는 것은 더 많은 것을 낭비합니다. 사용자는 한 시간 동안 디버깅한 후 함수 서명이 변경된 것을 발견합니다. 한 번의 가져오기로 몇 시간의 재작업을 방지할 수 있습니다. |
| "문서에는 내가 필요한 것이 없습니다." | 문서에서 다루지 않으면 information이 중요합니다. 해당 패턴은 공식적으로 권장되지 않을 수 있습니다. |
| "나는 그것이 구식일 수도 있다고 언급하겠습니다" | 면책 조항은 도움이 되지 않습니다. 확인하고 인용하거나 확인되지 않은 항목으로 명확하게 표시하세요. 헤징은 최악의 선택입니다. |
| "이것은 간단한 작업이므로 확인할 필요가 없습니다." | 잘못된 패턴의 간단한 작업은 템플릿이 됩니다. 사용자는 최신 접근 방식이 존재한다는 사실을 발견하기 전에 더 이상 사용되지 않는 form 처리기를 10개의 구성 요소에 복사합니다. |

## 위험 신호

- 해당 버전에 대한 문서를 확인하지 않고 framework-specific 코드 작성
- 출처를 인용하는 대신 API에 대해 "나는 믿는다" 또는 "나는 생각한다"를 사용합니다.
- 어떤 버전에 적용되는지 모르고 패턴을 구현하는 경우
- 공식 문서 대신 Stack Overflow나 블로그 게시물을 인용합니다.
- 훈련 데이터에 나타나기 때문에 더 이상 사용되지 않는 APIs 사용
- 구현하기 전에 `package.json`/종속성 파일을 읽지 않음
- framework-specific 결정에 대해 소스 인용 없이 코드 제공
- 단 한 페이지만 관련된 경우 전체 문서 사이트를 가져옵니다.

## 확인

source-driven 개발로 구현한 후:

- [ ] 프레임워크 및 라이브러리 버전이 종속 파일에서 식별되었습니다.
- [ ] framework-specific 패턴에 대한 공식 문서를 가져왔습니다.
- [ ] 모든 소스는 블로그 게시물이나 교육 데이터가 아닌 공식 문서입니다.
- [ ] 코드는 현재 버전의 문서에 표시된 패턴을 따릅니다.
- [ ] 중요하지 않은 결정에는 전체 URLs가 포함된 소스 인용이 포함됩니다.
- [ ] 더 이상 사용되지 않는 APIs가 사용되지 않습니다(마이그레이션 guides와 비교하여 확인).
- [ ] 문서와 기존 코드 간의 충돌이 사용자에게 표시되었습니다.
- [ ] 확인할 수 없는 모든 항목은 명시적으로 확인되지 않은 것으로 표시됩니다.
