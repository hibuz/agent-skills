---
name: git-workflow-and-versioning
description: git 워크플로 관행을 구조화. 모든 코드 변경 시 사용. commit, branching, 충돌 해결, 또는 여러 병렬 스트림에 걸쳐 작업을 조직할 필요가 있을 때 사용.
---

# Git Workflow and Versioning

## Overview

git은 당신의 안전망입니다. commit을 save point로, branch를 sandbox로, history를 문서로 다루세요. AI agent가 고속으로 코드를 생성하는 상황에서, 규율 있는 version control은 변경을 관리 가능하고, 리뷰 가능하고, 되돌릴 수 있게 유지하는 메커니즘입니다.

## When to Use

항상. 모든 코드 변경은 git을 통해 흐릅니다.

## Core Principles

### Trunk-Based Development (권장)

`main`을 항상 배포 가능하게 유지하세요. 1-3일 내에 다시 merge되는 단명 feature branch에서 작업하세요. 장수 개발 branch는 숨은 비용입니다 — 발산하고, merge 충돌을 만들고, 통합을 지연시킵니다. DORA 연구는 trunk-based development가 고성과 엔지니어링 팀과 상관됨을 일관되게 보여줍니다.

```
main ──●──●──●──●──●──●──●──●──●──  (항상 배포 가능)
        ╲      ╱  ╲    ╱
         ●──●─╱    ●──╱    ← 단명 feature branch (1-3일)
```

이것이 권장 기본값입니다. gitflow나 장수 branch를 쓰는 팀은 원칙(원자적 commit, 작은 변경, 서술적 메시지)을 자신의 branching 모델에 적용할 수 있습니다 — 특정 branching 전략보다 commit 규율이 더 중요합니다.

- **Dev branch는 비용입니다.** branch가 사는 매일, merge 위험을 누적합니다.
- **Release branch는 허용됩니다.** main이 앞으로 나아가는 동안 릴리스를 안정화해야 할 때.
- **Feature flag > 장수 branch.** 미완성 작업을 몇 주간 branch에 두기보다 flag 뒤에 배포하는 것을 선호하세요.

### 1. 일찍 Commit, 자주 Commit

각 성공한 증분이 자체 commit을 받습니다. 큰 미commit 변경을 누적하지 마세요.

```
작업 패턴:
  슬라이스 구현 → Test → Verify → Commit → 다음 슬라이스

이게 아니라:
  전부 구현 → 동작하길 바람 → 거대한 commit
```

commit은 save point입니다. 다음 변경이 무언가를 깨뜨리면, 즉시 마지막 known-good 상태로 revert할 수 있습니다.

### 2. 원자적 Commit

각 commit은 하나의 논리적 일을 합니다:

```
# 좋음: 각 commit이 자기 완결적
git log --oneline
a1b2c3d Add task creation endpoint with validation
d4e5f6g Add task creation form component
h7i8j9k Connect form to API and add loading state
m1n2o3p Add task creation tests (unit + integration)

# 나쁨: 모든 게 섞임
git log --oneline
x1y2z3a Add task feature, fix sidebar, update deps, refactor utils
```

### 3. 서술적 메시지

commit 메시지는 *무엇*만이 아니라 *왜*를 설명합니다:

```
# 좋음: 의도 설명
feat: add email validation to registration endpoint

Prevents invalid email formats from reaching the database.
Uses Zod schema validation at the route handler level,
consistent with existing validation patterns in auth.ts.

# 나쁨: diff에서 명백한 것을 기술
update auth.ts
```

**형식:**
```
<type>: <짧은 설명>

<무엇이 아니라 왜를 설명하는 선택적 본문>
```

**Type:**
- `feat` — 새 기능
- `fix` — 버그 수정
- `refactor` — 버그 수정도 기능 추가도 아닌 코드 변경
- `test` — 테스트 추가나 업데이트
- `docs` — 문서만
- `chore` — 도구, 의존성, config

### 4. 관심사를 분리하라

포맷팅 변경을 동작 변경과 결합하지 마세요. 리팩터를 기능과 결합하지 마세요. 각 유형의 변경은 별도 commit — 이상적으로는 별도 PR이어야 합니다:

```
# 좋음: 관심사 분리
git commit -m "refactor: extract validation logic to shared utility"
git commit -m "feat: add phone number validation to registration"

# 나쁨: 섞인 관심사
git commit -m "refactor validation and add phone number field"
```

**리팩터링을 기능 작업에서 분리하세요.** 리팩터링 변경과 기능 변경은 두 개의 다른 변경입니다 — 별도로 제출하세요. 이는 각 변경을 리뷰, revert, 히스토리에서 이해하기 쉽게 만듭니다. 작은 정리(변수 이름 변경)는 리뷰어 재량으로 기능 commit에 포함될 수 있습니다.

### 5. 변경 크기 조정

commit/PR당 ~100줄을 목표로 하세요. ~1000줄 초과 변경은 분할되어야 합니다. 큰 변경을 분해하는 방법은 `code-review-and-quality`의 분할 전략을 참고하세요.

```
~100줄  → 리뷰 쉬움, revert 쉬움
~300줄  → 단일 논리적 변경에 허용 가능
~1000줄 → 더 작은 변경으로 분할
```

## Branching 전략

### Feature Branch

```
main (항상 배포 가능)
  │
  ├── feature/task-creation    ← branch당 하나의 기능
  ├── feature/user-settings    ← 병렬 작업
  └── fix/duplicate-tasks      ← 버그 수정
```

- `main`(또는 팀의 기본 branch)에서 branch
- branch를 단명하게 유지 (1-3일 내 merge) — 장수 branch는 숨은 비용
- merge 후 branch 삭제
- 미완성 기능에는 장수 branch보다 feature flag 선호

### Branch 명명

```
feature/<짧은-설명>   → feature/task-creation
fix/<짧은-설명>       → fix/duplicate-tasks
chore/<짧은-설명>     → chore/update-deps
refactor/<짧은-설명>  → refactor/auth-module
```

## Worktree로 작업하기

병렬 AI agent 작업을 위해, git worktree를 사용해 여러 branch를 동시에 실행하세요:

```bash
# feature branch용 worktree 생성
git worktree add ../project-feature-a feature/task-creation
git worktree add ../project-feature-b feature/user-settings

# 각 worktree는 자체 branch를 가진 별도 디렉터리
# agent가 간섭 없이 병렬 작업 가능
ls ../
  project/              ← main branch
  project-feature-a/    ← task-creation branch
  project-feature-b/    ← user-settings branch

# 끝나면 merge하고 정리
git worktree remove ../project-feature-a
```

이점:
- 여러 agent가 다른 기능을 동시에 작업 가능
- branch 전환 불필요 (각 디렉터리가 자체 branch)
- 한 실험이 실패하면, worktree 삭제 — 아무것도 잃지 않음
- 명시적으로 merge될 때까지 변경이 격리됨

## Save Point 패턴

```
agent가 작업 시작
    │
    ├── 변경
    │   ├── 테스트 통과? → Commit → 계속
    │   └── 테스트 실패? → 마지막 commit으로 revert → 조사
    │
    ├── 또 다른 변경
    │   ├── 테스트 통과? → Commit → 계속
    │   └── 테스트 실패? → 마지막 commit으로 revert → 조사
    │
    └── 기능 완료 → 모든 commit이 깨끗한 히스토리를 형성
```

이 패턴은 한 증분 이상의 작업을 절대 잃지 않음을 의미합니다. agent가 탈선하면, `git reset --hard HEAD`가 마지막 성공 상태로 되돌립니다.

## 변경 요약

모든 수정 후, 구조화된 요약을 제공하세요. 이는 리뷰를 쉽게 하고, 범위 규율을 문서화하고, 의도치 않은 변경을 표면화합니다:

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

이 패턴은 잘못된 가정을 일찍 잡고 리뷰어에게 변경의 명확한 지도를 줍니다. "DIDN'T TOUCH" 섹션이 특히 중요합니다 — 범위 규율을 발휘했고 요청받지 않은 개보수를 하지 않았음을 보여줍니다.

## Pre-Commit 위생

모든 commit 전에:

```bash
# 1. commit할 것을 확인
git diff --staged

# 2. secrets 없음 보장
git diff --staged | grep -i "password\|secret\|api_key\|token"

# 3. 테스트 실행
npm test

# 4. linting 실행
npm run lint

# 5. type checking 실행
npx tsc --noEmit
```

git hook으로 이를 자동화:

```json
// package.json (lint-staged + husky 사용)
{
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{json,md}": ["prettier --write"]
  }
}
```

## 생성된 파일 처리

- 프로젝트가 기대할 때만 **생성된 파일 commit** (예: `package-lock.json`, Prisma migration)
- 빌드 출력(`dist/`, `.next/`), 환경 파일(`.env`), 또는 IDE config(공유되지 않는 한 `.vscode/settings.json`)는 **commit 안 함**
- 다음을 커버하는 **`.gitignore` 보유**: `node_modules/`, `dist/`, `.env`, `.env.local`, `*.pem`

## 디버깅에 Git 사용

```bash
# 어느 commit이 버그를 도입했는지 찾기
git bisect start
git bisect bad HEAD
git bisect good <known-good-commit>
# git이 중간 지점을 checkout; 각각에서 테스트 실행해 좁히기

# 최근 변경 보기
git log --oneline -20
git diff HEAD~5..HEAD -- src/

# 특정 줄을 마지막으로 누가 바꿨는지 찾기
git blame src/services/task.ts

# commit 메시지에서 키워드 검색
git log --grep="validation" --oneline
```

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "기능이 끝나면 commit할게" | 거대한 commit 하나는 리뷰, 디버깅, revert가 불가능합니다. 각 슬라이스를 commit하세요. |
| "메시지는 중요하지 않아" | 메시지는 문서입니다. 미래의 당신(과 미래 agent)은 무엇이 왜 바뀌었는지 이해해야 합니다. |
| "나중에 다 squash할게" | squash는 개발 서사를 파괴합니다. 처음부터 깨끗한 점진적 commit을 선호하세요. |
| "branch는 오버헤드를 더해" | 단명 branch는 무료이고 충돌하는 작업의 충돌을 방지합니다. 장수 branch가 문제입니다 — 1-3일 내 merge하세요. |
| "이 변경은 나중에 분할할게" | 큰 변경은 리뷰하기 어렵고, 배포가 위험하고, revert가 어렵습니다. 제출 후가 아니라 전에 분할하세요. |
| ".gitignore는 필요 없어" | production secrets가 든 `.env`가 commit될 때까지는. 즉시 설정하세요. |

## Red Flags

- 큰 미commit 변경 누적
- "fix", "update", "misc" 같은 commit 메시지
- 포맷팅 변경이 동작 변경과 섞임
- 프로젝트에 `.gitignore` 없음
- `node_modules/`, `.env`, 또는 빌드 artifact commit
- main에서 크게 발산하는 장수 branch
- 공유 branch에 force-push

## Verification

모든 commit에 대해:

- [ ] commit이 하나의 논리적 일을 함
- [ ] 메시지가 왜를 설명하고, type 규약을 따름
- [ ] commit 전 테스트 통과
- [ ] diff에 secrets 없음
- [ ] 포맷팅만의 변경이 동작 변경과 섞이지 않음
- [ ] `.gitignore`가 표준 제외를 커버
