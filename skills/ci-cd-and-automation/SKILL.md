---
name: ci-cd-and-automation
description: CI/CD 파이프라인 설정을 자동화합니다. build 및 배포 파이프라인을 설정하거나 수정할 때 사용합니다. 품질 게이트를 자동화하거나, CI에서 테스트 실행기를 구성하거나, 배포 전략을 수립해야 할 때 사용하세요.
---

# CI/CD 및 자동화

## 개요

테스트, 린트, 유형 확인 및 build를 통과하지 않고 변경 사항이 생산에 도달하지 않도록 품질 게이트를 자동화합니다. CI/CD는 다른 모든 skill에 대한 적용 메커니즘입니다. 이는 인간과 agents가 놓친 부분을 포착하고 모든 단일 변경 사항에서 일관되게 수행합니다.

**Shift Left:** 파이프라인에서 가능한 한 빨리 문제를 파악하세요. Linting에 걸린 버그는 몇 분만 소요됩니다. 생산 과정에서 발견된 동일한 버그로 인해 몇 시간이 소요됩니다. 검사 업스트림 이동 - 테스트 전 정적 분석, 준비 전 테스트, 생산 전 준비.

**빠를수록 안전합니다.** 배치 규모가 작고 릴리스 빈도가 높을수록 위험이 커지는 것이 아니라 줄어듭니다. 3개의 변경 사항이 있는 배포는 30개의 변경 사항이 있는 배포보다 디버그하기가 더 쉽습니다. 빈번한 릴리스 build 릴리스 프로세스 자체에 대한 확신이 있습니다.

## 사용 시기

- 새 프로젝트의 CI 파이프라인 설정
- 자동 검사 추가 또는 수정
- 배포 파이프라인 구성
- 변경사항으로 인해 자동 확인이 실행되어야 하는 경우
- CI 오류 디버깅

## 품질 게이트 파이프라인

모든 변경 사항은 병합 전에 다음 게이트를 통과합니다.

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

**어떤 게이트도 건너뛸 수 없습니다.** Lint가 실패하면 Lint를 수정하세요. 규칙을 비활성화하지 마세요. 테스트가 실패하면 코드를 수정하세요. 테스트를 건너뛰지 마세요.

## GitHub 작업 구성

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

### 데이터베이스 통합 테스트 포함

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

> **참고:** CI 전용 테스트 데이터베이스의 경우에도 자격 증명에 값을 하드코딩하는 대신 GitHub 비밀을 사용하세요. 이 builds는 좋은 습관을 들이고 다른 상황에서 테스트 자격 증명이 실수로 재사용되는 것을 방지합니다.

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

## CI 오류를 Agents로 다시 공급

AI agents가 포함된 CI의 성능은 피드백 루프입니다. CI가 실패하는 경우:

```
CI fails
    │
    ▼
Copy the failure output
    │
    ▼
Feed it to the agent:
"The CI pipeline failed with this error:
[paste specific error]
Fix the issue and verify locally before pushing again."
    │
    ▼
Agent fixes → pushes → CI runs again
```

**주요 패턴:**

```
Lint failure → Agent runs `npm run lint --fix` and commits
Type error  → Agent reads the error location and fixes the type
Test failure → Agent follows debugging-and-error-recovery skill
Build error → Agent checks config and dependencies
```

## 배포 전략

### 배포 미리보기

모든 PR에는 수동 테스트를 위한 미리 보기 배포가 제공됩니다.

```yaml
# Deploy preview on PR (Vercel/Netlify/etc.)
deploy-preview:
  runs-on: ubuntu-latest
  if: github.event_name == 'pull_request'
  steps:
    - uses: actions/checkout@v4
    - name: Deploy preview
      run: npx vercel --token=${{ secrets.VERCEL_TOKEN }}
```

### Feature Flags

Feature flags는 릴리스에서 배포를 분리합니다. 불완전하거나 위험한 기능을 플래그 뒤에 배포하여 다음을 수행할 수 있습니다.

- **활성화하지 않고 코드를 배송합니다.** 초기에 메인에 병합하고 준비가 되면 활성화합니다.
- **재배포 없이 롤백합니다.** 코드를 되돌리는 대신 플래그를 비활성화합니다.
- **카나리아의 새로운 기능.** 사용자의 1%, 10%, 100%에 대해 활성화됩니다.
- **A/B 테스트를 실행합니다.** 기능이 있는 경우와 없는 경우의 동작을 비교합니다.

```typescript
// Simple feature flag pattern
if (featureFlags.isEnabled('new-checkout-flow', { userId })) {
  return renderNewCheckout();
}
return renderLegacyCheckout();
```

**플래그 수명 주기:** 생성 → 테스트용 활성화 → Canary → 전체 출시 → 플래그 및 데드 코드 제거. 영원히 지속되는 플래그는 기술적 부채가 됩니다. 플래그를 만들 때 정리 날짜를 설정하세요.

### 단계적 출시

```
PR merged to main
    │
    ▼
  Staging deployment (auto)
    │ Manual verification
    ▼
  Production deployment (manual trigger or auto after staging)
    │
    ▼
  Monitor for errors (15-minute window)
    │
    ├── Errors detected → Rollback
    └── Clean → Done
```

### Rollback 계획

모든 배포는 되돌릴 수 있어야 합니다.

```yaml
# Manual rollback workflow
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
          # Deploy the specified previous version
          npx vercel rollback ${{ inputs.version }}
```

## 환경관리

```
.env.example       → Committed (template for developers)
.env                → NOT committed (local development)
.env.test           → Committed (test environment, no real secrets)
CI secrets          → Stored in GitHub Secrets / vault
Production secrets  → Stored in deployment platform / vault
```

CI에는 프로덕션 비밀이 있어서는 안 됩니다. CI 테스트에는 별도의 비밀을 사용하세요.

## CI를 넘어서는 자동화

### 의존봇 / 혁신

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

### Build 경찰 역할

CI를 녹색으로 유지할 책임이 있는 사람을 지정하십시오. build가 중단되면 Build 경찰의 임무는 중단을 초래한 변경 사항을 적용한 사람이 아니라 문제를 해결하거나 되돌리는 것입니다. 이렇게 하면 모두가 다른 사람이 문제를 고칠 것이라고 가정하는 동안 손상된 builds가 누적되는 것을 방지할 수 있습니다.

### PR 확인

- **Required 검토 필요:** 병합 전 최소 1번의 승인
- **Required 상태 확인 필요:** CI는 병합 전에 통과해야 합니다.
- **분기 보호:** 메인에 force-pushes 없음
- **자동 병합:** 모든 검사를 통과하고 승인되면 자동으로 병합됩니다.

## CI 최적화

When the pipeline exceeds 10 minutes, apply these strategies in order of impact:

```
Slow CI pipeline?
├── Cache dependencies
│   └── Use actions/cache or setup-node cache option for node_modules
├── Run jobs in parallel
│   └── Split lint, typecheck, test, build into separate parallel jobs
├── Only run what changed
│   └── Use path filters to skip unrelated jobs (e.g., skip e2e for docs-only PRs)
├── Use matrix builds
│   └── Shard test suites across multiple runners
├── Optimize the test suite
│   └── Remove slow tests from the critical path, run them on a schedule instead
└── Use larger runners
    └── GitHub-hosted larger runners or self-hosted for CPU-heavy builds
```

**예: 캐싱 및 병렬 처리**
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

## 일반적인 합리화

| 합리화 | 현실 |
|---|---|
| "CI가 너무 느립니다." | Optimize the pipeline (see CI Optimization below), don't skip it. 5분 파이프라인으로 인해 몇 시간의 디버깅 시간이 단축됩니다. |
| "이 변경 사항은 간단합니다. CI를 건너뛰세요." | 사소한 변경으로 인해 builds가 중단됩니다. 어쨌든 CI는 사소한 변경에도 빠릅니다. |
| "테스트가 불안정합니다. 단지 re-run" | 불안정한 테스트는 실제 버그를 가리고 모든 사람의 시간을 낭비합니다. 벗겨짐을 수정합니다. |
| "나중에 CI를 추가하겠습니다" | CI가 없는 프로젝트는 손상된 상태를 누적합니다. 첫날부터 설정하세요. |
| "수동 테스트로 충분합니다" | 수동 테스트는 확장되지 않으며 반복 가능하지 않습니다. 가능한 것을 자동화하세요. |

## 위험 신호

- 프로젝트에 CI 파이프라인이 없습니다.
- CI 오류가 무시되거나 침묵되었습니다.
- Tests disabled in CI to make the pipeline pass
- 스테이징 검증 없이 프로덕션 배포
- rollback 메커니즘 없음
- 코드 또는 CI 구성 파일에 저장된 비밀(비밀 관리자 아님)
- 최적화 노력 없이 긴 CI 시간

## 확인

CI를 설정하거나 수정한 후:

- [ ] 모든 품질 게이트가 존재합니다(린트, 유형, 테스트, build, 감사).
- [ ] 파이프라인은 모든 PR에서 실행되고 기본으로 푸시됩니다.
- [ ] 실패 블록 병합(분기 보호 구성됨)
- [ ] CI 결과는 개발 루프에 다시 피드백됩니다.
- [ ] 비밀은 코드가 아닌 비밀 관리자에 저장됩니다.
- [ ] 배포에는 rollback 메커니즘이 있습니다.
- [ ] suite 테스트를 위해 파이프라인이 10분 이내에 실행됩니다.
