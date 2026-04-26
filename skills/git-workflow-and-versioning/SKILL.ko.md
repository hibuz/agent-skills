---
name: git-workflow-and-versioning
description: Git 워크플로우 관행을 구조화합니다. 코드 변경을 할 때마다 사용하세요. 커밋, 브랜치 생성, 충돌 해결 또는 여러 병렬 스트림에 걸쳐 작업을 정리해야 할 때 사용합니다.
---

# Git 워크플로우 및 버전 관리 (Git Workflow and Versioning)

## 개요 (Overview)

Git은 당신의 안전망입니다. 커밋(Commit)은 저장 지점으로, 브랜치(Branch)는 샌드박스로, 히스토리(History)는 문서로 취급하세요. AI 에이전트가 빠른 속도로 코드를 생성하는 환경에서, 절제된 버전 관리는 변경 사항을 관리 가능하고 리뷰 가능하며 되돌릴 수 있게 유지하는 메커니즘입니다.

## 사용 시기 (When to Use)

항상 사용하세요. 모든 코드 변경은 Git을 통해 흐릅니다.

## 핵심 원칙 (Core Principles)

### 트렁크 기반 개발 (Trunk-Based Development - 권장)

`main` 브랜치를 항상 배포 가능한 상태로 유지하세요. 1~3일 이내에 다시 머지(Merge)되는 수명이 짧은 기능 브랜치(feature branches)에서 작업하세요. 수명이 긴 개발 브랜치는 숨겨진 비용입니다. 이들은 메인 브랜치와 멀어지고, 머지 충돌을 일으키며, 통합을 지연시킵니다. DORA 연구 결과에 따르면 트렁크 기반 개발은 고성능 엔지니어링 팀과 일관된 상관관계를 보입니다.

```
main ──●──●──●──●──●──●──●──●──●──  (항상 배포 가능)
        ╲      ╱  ╲    ╱
         ●──●─╱    ●──╱    ← 수명이 짧은 기능 브랜치 (1-3일)
```

이것이 권장되는 기본 방식입니다. Gitflow나 수명이 긴 브랜치를 사용하는 팀도 핵심 원칙(원자적 커밋, 작은 변경, 설명적인 메시지)을 그들의 브랜칭 모델에 적용할 수 있습니다. 특정 브랜칭 전략보다 커밋 규율이 더 중요합니다.

- **개발 브랜치는 비용입니다.** 브랜치가 유지되는 매일 머지 위험이 축적됩니다.
- **릴리스 브랜치(Release branches)는 허용됩니다.** `main`이 계속 진행되는 동안 릴리스를 안정화해야 할 때 사용하세요.
- **피처 플래그(Feature flags) > 긴 브랜치.** 작업을 수주 동안 브랜치에 두는 것보다 완료되지 않은 작업을 플래그 뒤에 숨겨서 배포하는 편을 선호하세요.

### 1. 일찍, 자주 커밋하기 (Commit Early, Commit Often)

성공적인 각 증분(increment)은 각각의 커밋을 가집니다. 커밋되지 않은 대규모 변경 사항을 쌓아두지 마세요.

```
작업 패턴:
  조각 구현 → 테스트 → 검증 → 커밋 → 다음 조각

잘못된 패턴:
  전부 구현 → 잘 되길 기도함 → 거대한 커밋
```

커밋은 저장 지점입니다. 다음 변경 사항이 무언가를 망가뜨리면, 즉시 마지막으로 알려진 양호한 상태로 되돌릴 수 있습니다.

### 2. 원자적 커밋 (Atomic Commits)

각 커밋은 하나의 논리적인 작업만 수행합니다.

```
# 좋음: 각 커밋이 독립적임
git log --oneline
a1b2c3d 검증 기능을 포함한 태스크 생성 엔드포인트 추가
d4e5f6g 태스크 생성 폼 컴포넌트 추가
h7i8j9k 폼을 API에 연결하고 로딩 상태 추가
m1n2o3p 태스크 생성 테스트 추가 (유닛 + 통합)

# 나쁨: 모든 것이 뒤섞임
git log --oneline
x1y2z3a 태스크 기능 추가, 사이드바 수정, 의존성 업데이트, 유틸리티 리팩토링
```

### 3. 설명적인 메시지 (Descriptive Messages)

커밋 메시지는 '무엇(what)'뿐만 아니라 '왜(why)'를 설명해야 합니다.

```
# 좋음: 의도를 설명함
feat: 등록 엔드포인트에 이메일 검증 추가

유효하지 않은 이메일 형식이 데이터베이스에 도달하는 것을 방지합니다.
auth.ts의 기존 검증 패턴과 일관되게
라우트 핸들러 수준에서 Zod 스키마 검증을 사용합니다.

# 나쁨: diff에서 뻔히 보이는 내용을 설명함
update auth.ts
```

**형식:**
```
<type>: <short description>

<optional body explaining why, not what>
```

**유형(Types):**
- `feat` — 새로운 기능
- `fix` — 버그 수정
- `refactor` — 버그 수정이나 기능 추가가 아닌 코드 변경
- `test` — 테스트 추가 또는 업데이트
- `docs` — 문서 작업만 해당
- `chore` — 도구, 의존성, 설정 변경

### 4. 관심사 분리 (Keep Concerns Separate)

포맷팅 변경과 동작 변경을 합치지 마세요. 리팩토링과 기능 추가를 합치지 마세요. 각 유형의 변경은 별도의 커밋이어야 하며, 이상적으로는 별도의 PR이어야 합니다.

```
# 좋음: 관심사 분리
git commit -m "refactor: 검증 로직을 공통 유틸리티로 추출"
git commit -m "feat: 등록 시 전화번호 검증 추가"

# 나쁨: 관심사 혼재
git commit -m "검증 리팩토링 및 전화번호 필드 추가"
```

**리팩토링과 기능 작업을 분리하세요.** 리팩토링 변경과 기능 변경은 서로 다른 두 가지 변경 사항입니다. 이들을 별도로 제출하세요. 이렇게 하면 각 변경 사항을 리뷰하고, 되돌리고, 히스토리에서 이해하기가 훨씬 쉬워집니다. 변수 이름 변경과 같은 작은 정리 작업은 리뷰어의 재량에 따라 기능 커밋에 포함될 수 있습니다.

### 5. 변경 규모 조절 (Size Your Changes)

커밋/PR당 약 100줄을 목표로 하세요. 약 1000줄이 넘는 변경 사항은 분할해야 합니다. 대규모 변경 사항을 나누는 방법은 `code-review-and-quality.ko.md`의 분할 전략을 참조하세요.

```
~100줄  → 리뷰하기 쉽고, 되돌리기 쉬움
~300줄  → 하나의 논리적 변경으로 수용 가능함
~1000줄 → 더 작은 변경 사항으로 분할 필요
```

## 브랜칭 전략 (Branching Strategy)

### 기능 브랜치 (Feature Branches)

```
main (항상 배포 가능)
  │
  ├── feature/task-creation    ← 브랜치당 하나의 기능
  ├── feature/user-settings    ← 병렬 작업
  └── fix/duplicate-tasks      ← 버그 수정
```

- `main`(또는 팀의 기본 브랜치)에서 브랜치를 생성합니다.
- 브랜치 수명을 짧게 유지하세요(1~3일 이내 머지). 수명이 긴 브랜치는 숨겨진 비용입니다.
- 머지 후 브랜치를 삭제하세요.
- 완료되지 않은 기능에 대해서는 긴 브랜치보다 피처 플래그(feature flags)를 선호하세요.

### 브랜치 명명 규칙 (Branch Naming)

```
feature/<short-description>   → feature/task-creation
fix/<short-description>       → fix/duplicate-tasks
chore/<short-description>     → chore/update-deps
refactor/<short-description>  → refactor/auth-module
```

## 워크트리 활용 (Working with Worktrees)

병렬 AI 에이전트 작업을 위해, Git 워크트리(worktrees)를 사용하여 여러 브랜치를 동시에 실행하세요.

```bash
# 기능 브랜치를 위한 워크트리 생성
git worktree add ../project-feature-a feature/task-creation
git worktree add ../project-feature-b feature/user-settings

# 각 워크트리는 자체 브랜치를 가진 별도의 디렉토리입니다.
# 에이전트들은 서로 방해받지 않고 병렬로 작업할 수 있습니다.
ls ../
  project/              ← main 브랜치
  project-feature-a/    ← task-creation 브랜치
  project-feature-b/    ← user-settings 브랜치

# 완료되면 머지하고 정리합니다.
git worktree remove ../project-feature-a
```

장점:
- 여러 에이전트가 서로 다른 기능을 동시에 작업할 수 있습니다.
- 브랜치 전환이 필요 없습니다(각 디렉토리가 고유한 브랜치를 가짐).
- 실험 하나가 실패하면 워크트리를 삭제하면 됩니다. 잃는 것이 없습니다.
- 명시적으로 머지하기 전까지 변경 사항이 격리됩니다.

## 저장 지점 패턴 (The Save Point Pattern)

```
에이전트 작업 시작
    │
    ├── 변경 수행
    │   ├── 테스트 통과? → 커밋 → 계속 진행
    │   └── 테스트 실패? → 마지막 커밋으로 리셋 → 조사
    │
    ├── 또 다른 변경 수행
    │   ├── 테스트 통과? → 커밋 → 계속 진행
    │   └── 테스트 실패? → 마지막 커밋으로 리셋 → 조사
    │
    └── 기능 완료 → 모든 커밋이 깔끔한 히스토리를 형성함
```

이 패턴은 작업의 한 증분 이상을 절대 잃지 않음을 의미합니다. 에이전트가 잘못된 방향으로 가면 `git reset --hard HEAD`를 통해 마지막으로 성공한 상태로 되돌아갈 수 있습니다.

## 변경 요약 (Change Summaries)

수정 후에는 구조화된 요약을 제공하세요. 이는 리뷰를 쉽게 만들고, 범위 준수를 문서화하며, 의도하지 않은 변경 사항을 드러냅니다.

```
변경 사항 요약:
- src/routes/tasks.ts: POST 엔드포인트에 검증 미들웨어 추가
- src/lib/validation.ts: Zod를 사용한 TaskCreateSchema 추가

수정하지 않은 사항 (의도적):
- src/routes/auth.ts: 유사한 검증 공백이 있으나 범위 밖임
- src/middleware/error.ts: 에러 형식 개선 가능 (별도 작업)

잠재적 우려 사항:
- Zod 스키마가 엄격하여 추가 필드를 거부함. 이것이 의도된 바인지 확인 필요.
- zod를 의존성에 추가함 (72KB gzipped) — 이미 package.json에 있음
```

이 패턴은 잘못된 가정을 조기에 발견하고 리뷰어에게 변경 사항에 대한 명확한 지도를 제공합니다. "수정하지 않은 사항" 섹션은 특히 중요합니다. 당신이 범위 규율을 준수했으며 요청하지 않은 리모델링을 하지 않았음을 보여줍니다.

## 커밋 전 위생 점검 (Pre-commit Hygiene)

모든 커밋 전에:

```bash
# 1. 커밋할 내용 확인
git diff --staged

# 2. 비밀 정보(secrets)가 없는지 확인
git diff --staged | grep -i "password\|secret\|api_key\|token"

# 3. 테스트 실행
npm test

# 4. 린트(Lint) 실행
npm run lint

# 5. 타입 체크 실행
npx tsc --noEmit
```

Git 훅(hooks)을 사용하여 이를 자동화하세요.

```json
// package.json (lint-staged + husky 사용 시)
{
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{json,md}": ["prettier --write"]
  }
}
```

## 생성된 파일 처리 (Handling Generated Files)

- **생성된 파일 커밋:** 프로젝트에서 요구하는 경우에만 커밋하세요(예: `package-lock.json`, Prisma 마이그레이션 파일).
- **커밋 금지:** 빌드 결과물(`dist/`, `.next/`), 환경 변수 파일(`.env`), IDE 설정(`.vscode/settings.json`, 공유가 필요한 경우는 제외).
- **`.gitignore` 작성:** `node_modules/`, `dist/`, `.env`, `.env.local`, `*.pem` 등을 포함해야 합니다.

## 디버깅을 위한 Git 활용

```bash
# 어떤 커밋에서 버그가 발생했는지 찾기
git bisect start
git bisect bad HEAD
git bisect good <known-good-commit>
# Git이 중간 지점을 체크아웃합니다. 각 지점에서 테스트를 실행하여 범위를 좁히세요.

# 최근 변경 사항 보기
git log --oneline -20
git diff HEAD~5..HEAD -- src/

# 특정 라인을 마지막으로 수정한 사람 찾기
git blame src/services/task.ts

# 커밋 메시지에서 키워드 검색
git log --grep="validation" --oneline
```

## 흔한 자기합리화 (Common Rationalizations)

| 자기합리화 | 실제 상황 |
|---|---|
| "기능이 다 끝나면 커밋할게요" | 거대한 커밋 하나는 리뷰, 디버그, 되돌리기가 불가능합니다. 각 조각마다 커밋하세요. |
| "메시지는 중요하지 않아요" | 메시지는 문서입니다. 미래의 당신(그리고 미래의 에이전트)은 무엇이 왜 바뀌었는지 이해해야 합니다. |
| "나중에 다 스쿼시(squash)할 거예요" | 스쿼시는 개발 과정을 파괴합니다. 처음부터 깔끔한 증분 커밋을 지향하세요. |
| "브랜치는 오버헤드를 유발해요" | 짧은 수명의 브랜치는 비용이 들지 않으며 작업 충돌을 방지합니다. 문제는 긴 브랜치입니다. 1~3일 내에 머지하세요. |
| "이 변경 사항은 나중에 나눌게요" | 큰 변경 사항은 리뷰하기 어렵고, 배포하기 위험하며, 되돌리기 힘듭니다. 제출 후가 아니라 전에 나누세요. |
| ".gitignore는 필요 없어요" | 운영 비밀 정보가 담긴 `.env`가 커밋되기 전까지만 그렇습니다. 즉시 설정하세요. |

## 위험 신호 (Red Flags)

- 커밋되지 않은 대규모 변경 사항이 쌓임
- "fix", "update", "misc" 같은 성의 없는 커밋 메시지
- 포맷팅 변경과 동작 변경이 혼재됨
- 프로젝트에 `.gitignore`가 없음
- `node_modules/`, `.env` 또는 빌드 아티팩트를 커밋함
- `main` 브랜치와 심하게 멀어진 수명이 긴 브랜치
- 공유 브랜치에 강제 푸시(force-push)함

## 검증 (Verification)

모든 커밋에 대해:

- [ ] 커밋이 하나의 논리적인 작업을 수행하는가
- [ ] 메시지가 '왜'를 설명하고 유형(type) 관례를 따르는가
- [ ] 커밋 전에 테스트를 통과했는가
- [ ] diff에 비밀 정보(secrets)가 없는가
- [ ] 동작 변경 없이 포맷팅만 바뀐 내용이 섞여 있지 않은가
- [ ] `.gitignore`가 표준 제외 항목들을 포함하는가
