---
name: git-workflow-and-versioning
description: git workflow 방식의 구조. 코드를 변경할 때 사용하세요. 커밋, 분기, 충돌 해결 시 또는 여러 병렬 스트림에 걸쳐 작업을 구성해야 할 때 사용하세요.
---

# Git Workflow 및 버전 관리

## 개요

Git은 안전망입니다. 커밋을 저장 지점으로, 분기를 샌드박스로, 기록을 문서로 처리합니다. AI agents가 고속으로 코드를 생성하는 방식을 통해 엄격한 버전 제어는 변경 사항을 관리, 검토 및 되돌릴 수 있도록 유지하는 메커니즘입니다.

## 사용 시기

항상. 모든 코드 변경은 git을 통해 이루어집니다.

## 핵심 원칙

### Trunk-Based Development(권장)

`main`를 항상 배포 가능하게 유지하세요. 1~3일 이내에 다시 병합되는 short-lived 기능 분기에서 작업하세요. 수명이 긴 개발 브랜치는 숨겨진 비용입니다. 분기하고, 병합 충돌을 일으키고, 통합을 지연시킵니다. DORA 연구는 trunk-based 개발이 high-performing 엔지니어링 팀과 상관관계가 있음을 일관되게 보여줍니다.

```
main ──●──●──●──●──●──●──●──●──●──  (always deployable)
        ╲      ╱  ╲    ╱
         ●──●─╱    ●──╱    ← short-lived feature branches (1-3 days)
```

이는 권장되는 기본값입니다. gitflow 또는 long-lived 분기를 사용하는 팀은 원칙(원자적 커밋, 작은 변경, 설명 메시지)을 분기 모델에 적용할 수 있습니다. 커밋 원칙은 특정 분기 전략보다 중요합니다.

- **개발 브랜치는 비용입니다.** 브랜치가 매일 존재하면서 병합 위험이 누적됩니다.
- **릴리스 분기가 허용됩니다.** 메인이 진행되는 동안 릴리스를 안정화해야 하는 경우.
- **Feature flags > 긴 분기.** 몇 주 동안 분기에 보관하는 것보다 플래그 뒤에 불완전한 작업을 배포하는 것을 선호합니다.

### 1. 일찍 커밋하고 자주 커밋하세요.

성공적인 각 증분에는 자체 커밋이 적용됩니다. 커밋되지 않은 대규모 변경 사항을 누적하지 마세요.

```
Work pattern:
  Implement slice → Test → Verify → Commit → Next slice

Not this:
  Implement everything → Hope it works → Giant commit
```

커밋은 저장 포인트입니다. 다음 변경으로 인해 문제가 발생하면 즉시 마지막 known-good 상태로 되돌릴 수 있습니다.

### 2. 원자적 커밋

각 커밋은 하나의 논리적 작업을 수행합니다.

```
# Good: Each commit is self-contained
git log --oneline
a1b2c3d Add task creation endpoint with validation
d4e5f6g Add task creation form component
h7i8j9k Connect form to API and add loading state
m1n2o3p Add task creation tests (unit + integration)

# Bad: Everything mixed together
git log --oneline
x1y2z3a Add task feature, fix sidebar, update deps, refactor utils
```

### 3. 설명 메시지

커밋 메시지는 *무엇*뿐만 아니라 *이유*를 설명합니다.

```
# Good: Explains intent
feat: add email validation to registration endpoint

Prevents invalid email formats from reaching the database.
Uses Zod schema validation at the route handler level,
consistent with existing validation patterns in auth.ts.

# Bad: Describes what's obvious from the diff
update auth.ts
```

**Format:**
```
<type>: <short description>

<optional body explaining why, not what>
```

**유형:**
- `feat` — 새로운 기능
- `fix` — 버그 수정
- `refactor` — 버그를 수정하지도 않고 기능을 추가하지도 않는 코드 변경
- `test` — 테스트 추가 또는 업데이트
- `docs` — 문서 전용
- `chore` — 도구, 종속성, 구성

### 4. 우려 사항을 별도로 유지

formatting 변경 사항과 동작 변경 사항을 결합하지 마십시오. 리팩터링을 기능과 결합하지 마세요. 각 변경 유형은 별도의 커밋이어야 하며 이상적으로는 별도의 PR여야 합니다.

```
# Good: Separate concerns
git commit -m "refactor: extract validation logic to shared utility"
git commit -m "feat: add phone number validation to registration"

# Bad: Mixed concerns
git commit -m "refactor validation and add phone number field"
```

**기능 작업과 별도의 리팩토링.** 리팩토링 변경과 기능 변경은 서로 다른 두 가지 변경 사항이므로 별도로 제출하세요. 이렇게 하면 기록에서 각 변경 사항을 더 쉽게 검토하고, 되돌리고, 이해할 수 있습니다. 검토자의 재량에 따라 작은 정리(변수 이름 바꾸기)가 기능 커밋에 포함될 수 있습니다.

### 5. 변경 사항의 크기를 조정하세요.

commit/PR당 최대 100줄을 목표로 합니다. 1000줄 이상의 변경 사항은 분할되어야 합니다. 큰 변경 사항을 분석하는 방법은 `code-review-and-quality`의 분할 전략을 참조하세요.

```
~100 lines  → Easy to review, easy to revert
~300 lines  → Acceptable for a single logical change
~1000 lines → Split into smaller changes
```

## 분기 전략

### 기능 분기

```
main (always deployable)
  │
  ├── feature/task-creation    ← One feature per branch
  ├── feature/user-settings    ← Parallel work
  └── fix/duplicate-tasks      ← Bug fixes
```

- `main`에서 분기(또는 팀의 기본 분기)
- 지점 short-lived 유지(1~3일 이내에 병합) — long-lived 지점은 숨겨진 비용입니다.
- 병합 후 분기 삭제
- 불완전한 기능을 위해 long-lived 분기보다 feature flags를 선호합니다.

### 지점 이름 지정

```
feature/<short-description>   → feature/task-creation
fix/<short-description>       → fix/duplicate-tasks
chore/<short-description>     → chore/update-deps
refactor/<short-description>  → refactor/auth-module
```

## 작업 트리 작업

병렬 AI agent 작업의 경우 git worktrees를 사용하여 여러 분기를 동시에 실행합니다.

```bash
# Create a worktree for a feature branch
git worktree add ../project-feature-a feature/task-creation
git worktree add ../project-feature-b feature/user-settings

# Each worktree is a separate directory with its own branch
# Agents can work in parallel without interfering
ls ../
  project/              ← main branch
  project-feature-a/    ← task-creation branch
  project-feature-b/    ← user-settings branch

# When done, merge and clean up
git worktree remove ../project-feature-a
```

혜택:
- 여러 agents는 서로 다른 기능을 동시에 작동할 수 있습니다.
- 분기 전환이 필요하지 않습니다(각 디렉터리에는 자체 분기가 있음).
- 하나의 실험이 실패하면 작업 트리를 삭제하세요. 아무것도 손실되지 않습니다.
- 변경 사항은 명시적으로 병합될 때까지 격리됩니다.

## 저장 포인트 패턴

```
Agent starts work
    │
    ├── Makes a change
    │   ├── Test passes? → Commit → Continue
    │   └── Test fails? → Revert to last commit → Investigate
    │
    ├── Makes another change
    │   ├── Test passes? → Commit → Continue
    │   └── Test fails? → Revert to last commit → Investigate
    │
    └── Feature complete → All commits form a clean history
```

이 패턴은 작업 증가분을 두 번 이상 잃지 않는다는 것을 의미합니다. agent가 레일을 벗어나면 `git reset --hard HEAD`가 마지막 성공 상태로 돌아갑니다.

## 변경 요약

수정 후에는 구조화된 요약을 제공하세요. 이렇게 하면 검토가 더 쉬워지고 문서 범위 규율이 향상되며 의도하지 않은 변경 사항이 드러납니다.

```
CHANGES MADE:
- src/routes/tasks.ts: Added validation middleware to POST endpoint
- src/lib/validation.ts: Added TaskCreateSchema using Zod

THINGS I DIDN'T TOUCH (intentionally):
- src/routes/auth.ts: Has similar validation gap but out of scope
- src/middleware/error.ts: Error format could be improved (separate task)

POTENTIAL CONCERNS:
- The Zod schema is strict — rejects extra fields. Confirm this is desired.
- Added zod as a dependency (72KB gzipped) — already in package.json
```

이 패턴은 잘못된 가정을 조기에 파악하고 검토자에게 변경 사항에 대한 명확한 지도를 제공합니다. "DIDN'T TOUCH" 섹션은 특히 중요합니다. 이는 범위 규율을 연습했고 원치 않는 개조 작업을 진행하지 않았음을 보여줍니다.

## 사전 커밋 위생

모든 커밋 전:

```bash
# 1. Check what you're about to commit
git diff --staged

# 2. Ensure no secrets
git diff --staged | grep -i "password\|secret\|api_key\|token"

# 3. Run tests
npm test

# 4. Run linting
npm run lint

# 5. Run type checking
npx tsc --noEmit
```

git Hooks를 사용하여 이를 자동화하세요:

```json
// package.json (using lint-staged + husky)
{
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{json,md}": ["prettier --write"]
  }
}
```

## 생성된 파일 처리

- **생성된 파일을 커밋** 프로젝트에서 예상하는 경우에만(예: `package-lock.json`, Prisma 마이그레이션)
- **커밋하지 마세요** build 출력(`dist/`, `.next/`), 환경 파일(`.env`) 또는 IDE 구성(공유되지 않는 경우 `.vscode/settings.json`)
- **`node_modules/`, `dist/`, `.env`, `.env.local`, `*.pem`를 포함하는 `.gitignore`**를 보유하십시오.

## 디버깅을 위해 Git 사용하기

```bash
# Find which commit introduced a bug
git bisect start
git bisect bad HEAD
git bisect good <known-good-commit>
# Git checkouts midpoints; run your test at each to narrow down

# View what changed recently
git log --oneline -20
git diff HEAD~5..HEAD -- src/

# Find who last changed a specific line
git blame src/services/task.ts

# Search commit messages for a keyword
git log --grep="validation" --oneline
```

## 일반적인 합리화

| 합리화 | 현실 |
|---|---|
| "기능이 완료되면 커밋하겠습니다" | 하나의 거대한 커밋은 검토, 디버그 또는 되돌리기가 불가능합니다. 각 슬라이스를 커밋합니다. |
| "메시지는 중요하지 않습니다" | 메시지는 문서입니다. 미래의 귀하(및 미래의 agents)는 변경된 내용과 이유를 이해해야 합니다. |
| "나중에 모두 없애겠습니다" | 스쿼싱은 개발 내러티브를 파괴합니다. 처음부터 깨끗한 증분 커밋을 선호합니다. |
| "분기는 오버헤드를 추가합니다" | 단기 분기는 무료이며 충돌하는 작업이 충돌하는 것을 방지합니다. 수명이 긴 브랜치가 문제입니다. 1~3일 이내에 병합됩니다. |
| "이 변경 사항은 나중에 분할하겠습니다." | 큰 변경 사항은 검토하기 어렵고 배포하기 위험하며 되돌리기가 더 어렵습니다. 분할은 제출 후가 아니라 제출 전에 분할됩니다. |
| ".gitignore가 필요하지 않습니다." | 생산 비밀이 포함된 `.env`가 커밋될 때까지. 즉시 설정하세요. |

## 위험 신호

- 커밋되지 않은 대규모 변경 사항이 누적됩니다.
- "수정", "업데이트", "기타"와 같은 커밋 메시지
- Formatting 변경 사항과 동작 변경 사항이 혼합됨
- 프로젝트에 `.gitignore`가 없습니다.
- `node_modules/`, `.env` 또는 build 아티팩트 커밋
- 메인에서 크게 갈라지는 수명이 긴 가지
- 공유 브랜치로 강제 푸시

## 확인

모든 커밋에 대해:

- [ ] 커밋은 하나의 논리적인 작업을 수행합니다.
- [ ] 메시지는 유형 규칙에 따라 이유를 설명합니다.
- [ ] 커밋하기 전에 테스트를 통과했습니다.
- [ ] 차이점에 비밀이 없습니다.
- [ ] 동작 변경과 혼합된 formatting-only 변경 사항 없음
- [ ] `.gitignore`는 표준 제외 사항을 포함합니다.
