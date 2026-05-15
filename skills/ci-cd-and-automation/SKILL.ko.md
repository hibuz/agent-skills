---
name: ci-cd-and-automation
description: CI/CD 파이프라인 설정을 자동화. 빌드 및 배포 파이프라인을 구성하거나 수정할 때 사용. 품질 게이트를 자동화하거나, CI에서 test runner를 구성하거나, 배포 전략을 수립할 필요가 있을 때 사용.
---

# CI/CD and Automation

## Overview

품질 게이트를 자동화해, 테스트, lint, type checking, build를 통과하지 않고는 어떤 변경도 production에 도달하지 못하게 하세요. CI/CD는 다른 모든 skill의 강제 메커니즘입니다 — 사람과 agent가 놓치는 것을 잡고, 모든 단일 변경에서 일관되게 그렇게 합니다.

**Shift Left:** 문제를 파이프라인에서 가능한 한 일찍 잡으세요. linting에서 잡힌 버그는 몇 분이 들지만; production에서 잡힌 같은 버그는 몇 시간이 듭니다. 검사를 상류로 옮기세요 — 테스트 전 정적 분석, staging 전 테스트, production 전 staging.

**Faster is Safer:** 더 작은 배치와 더 잦은 릴리스는 위험을 줄이지, 늘리지 않습니다. 변경 3개가 있는 배포가 30개가 있는 것보다 디버깅하기 쉽습니다. 잦은 릴리스는 릴리스 프로세스 자체에 대한 신뢰를 쌓습니다.

## When to Use

- 새 프로젝트의 CI 파이프라인 설정
- 자동 검사 추가 또는 수정
- 배포 파이프라인 구성
- 변경이 자동 검증을 트리거해야 할 때
- CI 실패 디버깅

## 품질 게이트 파이프라인

모든 변경은 merge 전 이 게이트들을 거칩니다:

```
Pull Request Opened
    │
    ▼
┌─────────────────┐
│   LINT CHECK     │  eslint, prettier
│   ↓ pass         │
│   TYPE CHECK     │  tsc --noEmit
│   ↓ pass         │
│   UNIT TESTS     │  jest/vitest
│   ↓ pass         │
│   BUILD          │  npm run build
│   ↓ pass         │
│   INTEGRATION    │  API/DB tests
│   ↓ pass         │
│   E2E (optional) │  Playwright/Cypress
│   ↓ pass         │
│   SECURITY AUDIT │  npm audit
│   ↓ pass         │
│   BUNDLE SIZE    │  bundlesize check
└─────────────────┘
    │
    ▼
  Ready for review
```

**어떤 게이트도 건너뛸 수 없습니다.** lint가 실패하면 lint를 수정하세요 — 규칙을 비활성화하지 마세요. 테스트가 실패하면 코드를 수정하세요 — 테스트를 건너뛰지 마세요.

## GitHub Actions 구성

### 기본 CI 파이프라인

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Type check
        run: npx tsc --noEmit

      - name: Test
        run: npm test -- --coverage

      - name: Build
        run: npm run build

      - name: Security audit
        run: npm audit --audit-level=high
```

### 데이터베이스 Integration 테스트 포함

```yaml
  integration:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: ci_user
          POSTGRES_PASSWORD: ${{ secrets.CI_DB_PASSWORD }}
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
      - run: npm ci
      - name: Run migrations
        run: npx prisma migrate deploy
        env:
          DATABASE_URL: postgresql://ci_user:${{ secrets.CI_DB_PASSWORD }}@localhost:5432/testdb
      - name: Integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://ci_user:${{ secrets.CI_DB_PASSWORD }}@localhost:5432/testdb
```

> **참고:** CI 전용 테스트 데이터베이스라도, 값을 하드코딩하기보다 자격 증명에 GitHub Secrets를 사용하세요. 이는 좋은 습관을 쌓고 다른 context에서 테스트 자격 증명의 우발적 재사용을 방지합니다.

### E2E 테스트

```yaml
  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
      - run: npm ci
      - name: Install Playwright
        run: npx playwright install --with-deps chromium
      - name: Build
        run: npm run build
      - name: Run E2E tests
        run: npx playwright test
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
```

## CI 실패를 Agent에게 피드백하기

AI agent와 함께하는 CI의 힘은 피드백 루프입니다. CI가 실패하면:

```
CI 실패
    │
    ▼
실패 출력 복사
    │
    ▼
agent에게 전달:
"The CI pipeline failed with this error:
[paste specific error]
Fix the issue and verify locally before pushing again."
    │
    ▼
agent가 수정 → push → CI 다시 실행
```

**핵심 패턴:**

```
Lint 실패 → agent가 `npm run lint --fix` 실행 후 commit
Type 에러  → agent가 에러 위치를 읽고 타입 수정
Test 실패 → agent가 debugging-and-error-recovery skill을 따름
Build 에러 → agent가 config와 의존성 확인
```

## 배포 전략

### Preview 배포

모든 PR은 수동 테스트를 위한 preview 배포를 받습니다:

```yaml
# PR에 preview 배포 (Vercel/Netlify/등)
deploy-preview:
  runs-on: ubuntu-latest
  if: github.event_name == 'pull_request'
  steps:
    - uses: actions/checkout@v4
    - name: Deploy preview
      run: npx vercel --token=${{ secrets.VERCEL_TOKEN }}
```

### Feature Flag

feature flag는 배포를 릴리스에서 분리합니다. 미완성이거나 위험한 기능을 flag 뒤에 배포해 다음을 할 수 있습니다:

- **활성화하지 않고 코드를 ship.** 일찍 main에 merge하고, 준비되면 활성화.
- **재배포 없이 roll back.** 코드를 revert하는 대신 flag를 비활성화.
- **새 기능을 canary.** 사용자 1%에 활성화, 그 다음 10%, 그 다음 100%.
- **A/B 테스트 실행.** 기능 유무에 따른 동작 비교.

```typescript
// 간단한 feature flag 패턴
if (featureFlags.isEnabled('new-checkout-flow', { userId })) {
  return renderNewCheckout();
}
return renderLegacyCheckout();
```

**Flag 라이프사이클:** 생성 → 테스트용 활성화 → Canary → 전체 rollout → flag와 죽은 코드 제거. 영원히 사는 flag는 기술 부채가 됩니다 — 생성할 때 cleanup 날짜를 설정하세요.

### 단계적 Rollout

```
PR이 main에 merge
    │
    ▼
  Staging 배포 (자동)
    │ 수동 검증
    ▼
  Production 배포 (수동 트리거 또는 staging 후 자동)
    │
    ▼
  에러 모니터링 (15분 윈도우)
    │
    ├── 에러 감지 → Rollback
    └── 깨끗함 → 완료
```

### Rollback 계획

모든 배포는 되돌릴 수 있어야 합니다:

```yaml
# 수동 rollback 워크플로
name: Rollback
on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Version to rollback to'
        required: true

jobs:
  rollback:
    runs-on: ubuntu-latest
    steps:
      - name: Rollback deployment
        run: |
          # 지정된 이전 버전 배포
          npx vercel rollback ${{ inputs.version }}
```

## 환경 관리

```
.env.example       → commit됨 (개발자용 템플릿)
.env                → commit 안 됨 (로컬 개발)
.env.test           → commit됨 (테스트 환경, 실제 secrets 없음)
CI secrets          → GitHub Secrets / vault에 저장
Production secrets  → 배포 플랫폼 / vault에 저장
```

CI는 production secrets를 절대 가져서는 안 됩니다. CI 테스트에는 별도의 secrets를 사용하세요.

## CI를 넘어선 자동화

### Dependabot / Renovate

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: npm
    directory: /
    schedule:
      interval: weekly
    open-pull-requests-limit: 5
```

### Build Cop 역할

CI를 green으로 유지할 책임자를 지정하세요. 빌드가 깨지면, Build Cop의 일은 수정하거나 revert하는 것입니다 — 깨짐을 일으킨 변경의 당사자가 아니라. 이는 다들 다른 누군가가 고칠 거라고 가정하는 동안 깨진 빌드가 누적되는 것을 방지합니다.

### PR 검사

- **필수 리뷰:** merge 전 최소 1개 승인
- **필수 status check:** merge 전 CI 통과 필수
- **branch protection:** main에 force-push 없음
- **Auto-merge:** 모든 검사 통과 및 승인 시 자동 merge

## CI 최적화

파이프라인이 10분을 초과하면, 영향 순서대로 다음 전략을 적용하세요:

```
느린 CI 파이프라인?
├── 의존성 캐시
│   └── node_modules에 actions/cache 또는 setup-node cache 옵션 사용
├── job을 병렬 실행
│   └── lint, typecheck, test, build를 별도 병렬 job으로 분할
├── 변경된 것만 실행
│   └── path 필터로 무관한 job 건너뛰기 (예: docs-only PR에 e2e 건너뛰기)
├── matrix build 사용
│   └── 여러 runner에 걸쳐 테스트 스위트 shard
├── 테스트 스위트 최적화
│   └── 느린 테스트를 critical path에서 제거, 대신 스케줄로 실행
└── 더 큰 runner 사용
    └── GitHub-hosted 더 큰 runner 또는 CPU 무거운 빌드에 self-hosted
```

**예시: 캐싱과 병렬성**
```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '22', cache: 'npm' }
      - run: npm ci
      - run: npm run lint

  typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '22', cache: 'npm' }
      - run: npm ci
      - run: npx tsc --noEmit

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '22', cache: 'npm' }
      - run: npm ci
      - run: npm test -- --coverage
```

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "CI가 너무 느려" | 건너뛰지 말고 파이프라인을 최적화하세요 (아래 CI 최적화 참고). 5분짜리 파이프라인이 몇 시간의 디버깅을 방지합니다. |
| "이 변경은 사소하니 CI 건너뛰자" | 사소한 변경이 빌드를 깨뜨립니다. 어차피 사소한 변경에는 CI가 빠릅니다. |
| "테스트가 불안정하니 그냥 재실행" | 불안정한 테스트는 실제 버그를 가리고 모두의 시간을 낭비합니다. 불안정성을 고치세요. |
| "CI는 나중에 추가할게" | CI 없는 프로젝트는 깨진 상태를 누적합니다. 첫날에 설정하세요. |
| "수동 테스트로 충분해" | 수동 테스트는 확장되지 않고 반복 가능하지 않습니다. 가능한 것을 자동화하세요. |

## Red Flags

- 프로젝트에 CI 파이프라인 없음
- CI 실패를 무시하거나 침묵시킴
- 파이프라인을 통과시키려고 CI에서 테스트 비활성화
- staging 검증 없는 production 배포
- rollback 메커니즘 없음
- secrets가 코드나 CI config 파일에 저장됨 (secrets manager가 아님)
- 최적화 노력 없는 긴 CI 시간

## Verification

CI를 설정하거나 수정한 후:

- [ ] 모든 품질 게이트가 존재 (lint, types, tests, build, audit)
- [ ] 모든 PR과 main push에 파이프라인 실행
- [ ] 실패가 merge를 차단 (branch protection 구성됨)
- [ ] CI 결과가 개발 루프로 피드백됨
- [ ] secrets가 코드가 아니라 secrets manager에 저장됨
- [ ] 배포에 rollback 메커니즘 있음
- [ ] 테스트 스위트에 대해 파이프라인이 10분 미만으로 실행
