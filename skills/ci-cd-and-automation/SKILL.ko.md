---
name: ci-cd-and-automation
description: CI/CD Pipeline 설정을 자동화합니다. Build 및 Deployment Pipeline을 설정하거나 수정할 때 사용하세요. Quality Gate를 자동화하거나, CI에서 Test Runner를 구성하거나, Deployment Strategy를 수립해야 할 때 사용합니다.
---

# CI/CD and Automation

## Overview

Test, Lint, Type check, Build를 통과하지 않은 변경 사항이 Production에 도달하지 않도록 Quality Gate를 자동화하세요. CI/CD는 다른 모든 기술을 위한 강제 메커니즘입니다. 인간과 Agent가 놓치는 것을 잡아내며, 모든 단일 변경 사항에 대해 일관되게 수행합니다.

**Shift Left:** 문제를 Pipeline 가능한 한 앞 단계에서 포착하세요. Linting에서 발견된 버그는 해결하는 데 몇 분이면 충분하지만, Production에서 발견된 동일한 버그는 몇 시간이 걸립니다. 체크 사항을 Upstream으로 이동시키세요 — Test 전에 Static Analysis를, Staging 전에 Test를, Production 전에 Staging을 수행하세요.

**Faster is Safer:** 더 작은 배치와 더 빈번한 Release는 위험을 증가시키는 것이 아니라 감소시킵니다. 3개의 변경 사항이 포함된 Deployment는 30개가 포함된 것보다 디버깅하기 쉽습니다. 빈번한 Release는 Release Process 자체에 대한 신뢰를 구축합니다.

## When to Use

- 새로운 프로젝트의 CI Pipeline 설정 시
- 자동화된 체크 사항을 추가하거나 수정할 때
- Deployment Pipeline 구성 시
- 변경 사항이 자동화된 검증을 트리거해야 할 때
- CI 실패를 디버깅할 때

## The Quality Gate Pipeline

모든 변경 사항은 Merge 전 다음 Gate를 통과해야 합니다:

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

**어떤 Gate도 건너뛸 수 없습니다.** Lint가 실패하면 Lint를 수정하세요 — 규칙을 비활성화하지 마세요. Test가 실패하면 코드를 수정하세요 — Test를 건너뛰지 마세요.

## GitHub Actions Configuration

### Basic CI Pipeline

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

### With Database Integration Tests

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

> **Note:** CI 전용 Test Database의 경우에도 값들을 하드코딩하지 말고 GitHub Secrets를 사용하여 Credential을 관리하세요. 이는 좋은 습관을 기르고 Test Credential이 다른 Context에서 우발적으로 재사용되는 것을 방지합니다.

### E2E Tests

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

## Feeding CI Failures Back to Agents

Agent와 함께 CI를 사용할 때의 강력함은 Feedback Loop에 있습니다. CI가 실패했을 때:

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

**Key patterns:**

```
Lint failure → Agent가 `npm run lint --fix`를 실행하고 Commit함
Type error  → Agent가 Error 위치를 읽고 Type을 수정함
Test failure → Agent가 debugging-and-error-recovery 기술을 따름
Build error → Agent가 Config 및 Dependency를 확인함
```

## Deployment Strategies

### Preview Deployments

모든 PR은 수동 Test를 위해 Preview Deployment를 생성합니다:

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

Feature Flag는 Deployment를 Release로부터 분리합니다. 미완성이거나 위험한 기능을 Flag 뒤에 Deploy하여 다음을 수행할 수 있습니다:

- **기능을 활성화하지 않고 코드를 배포함.** 일찍 Main에 Merge하고 준비가 되었을 때 활성화하세요.
- **Redeploy 없이 Rollback함.** 코드를 Revert하는 대신 Flag를 비활성화하세요.
- **새로운 기능을 Canary 배포함.** 사용자의 1%에게 활성화한 다음 10%, 100%로 확대하세요.
- **A/B test 실행.** 기능이 있을 때와 없을 때의 동작을 비교하세요.

```typescript
// Simple feature flag pattern
if (featureFlags.isEnabled('new-checkout-flow', { userId })) {
  return renderNewCheckout();
}
return renderLegacyCheckout();
```

**Flag lifecycle:** 생성 → Test를 위해 활성화 → Canary → 전체 Rollout → Flag 및 Dead code 제거. 영원히 남아있는 Flag는 Technical Debt(기술 부채)가 됩니다 — 생성 시 Cleanup 날짜를 설정하세요.

### Staged Rollouts

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

### Rollback Plan

모든 Deployment는 되돌릴 수 있어야 합니다:

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

## Environment Management

```
.env.example       → Committed (개발자를 위한 템플릿)
.env                → NOT committed (로컬 개발용)
.env.test           → Committed (Test 환경용, 실제 Secret 없음)
CI secrets          → GitHub Secrets / Vault에 저장
Production secrets  → Deployment Platform / Vault에 저장
```

CI는 절대 Production Secret을 가져서는 안 됩니다. CI Test를 위해 별도의 Secret을 사용하세요.

## Automation Beyond CI

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

### Build Cop Role

CI를 Green 상태로 유지할 책임자를 지정하세요. Build가 깨졌을 때 Build Cop의 역할은 수정하거나 Revert하는 것이지, Build를 깨뜨린 변경 사항을 만든 사람의 몫이 아닙니다. 이는 누군가 수정하겠지라고 가정하는 동안 깨진 Build가 쌓이는 것을 방지합니다.

### PR Checks

- **Required reviews:** Merge 전 최소 1명의 승인 필요
- **Required status checks:** Merge 전 CI 통과 필수
- **Branch protection:** Main 브랜치로의 Force-push 금지
- **Auto-merge:** 모든 Check를 통과하고 승인되면 자동으로 Merge

## CI Optimization

Pipeline이 10분을 초과하면 영향도가 큰 순서대로 다음 전략들을 적용하세요:

```
Slow CI pipeline?
├── Cache dependencies
│   └── node_modules를 위해 actions/cache 또는 setup-node cache 옵션 사용
├── Run jobs in parallel
│   └── Lint, Typecheck, Test, Build를 별도의 병렬 Job으로 분리
├── Only run what changed
│   └── Path filter를 사용하여 관련 없는 Job 건너뛰기 (예: Docs만 수정된 PR에서 E2E 건너뛰기)
├── Use matrix builds
│   └── Test Suite를 여러 Runner에 분산(Shard)
├── Optimize the test suite
│   └── Critical Path에서 느린 Test 제거, 대신 스케줄에 따라 실행
└── Use larger runners
    └── GitHub에서 호스팅하는 더 큰 Runner 또는 CPU 사용량이 많은 Build를 위한 Self-hosted Runner 사용
```

**Example: caching and parallelism**
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
| "CI가 너무 느려요" | Pipeline을 최적화하세요(아래 CI Optimization 참조). 건너뛰지 마세요. 5분의 Pipeline이 몇 시간의 디버깅을 방지합니다. |
| "사소한 변경이니 CI를 건너뛸게요" | 사소한 변경이 Build를 깨뜨립니다. 사소한 변경에 대해 CI는 어차피 빠릅니다. |
| "Test가 Flaky(간헐적 실패)하니 그냥 다시 돌리죠" | Flaky Test는 실제 버그를 가리고 모든 사람의 시간을 낭비합니다. Flakiness를 수정하세요. |
| "나중에 CI를 추가할게요" | CI가 없는 프로젝트는 깨진 상태가 누적됩니다. 첫날부터 설정하세요. |
| "수동 Test면 충분해요" | 수동 Test는 Scale되지 않으며 반복 가능하지 않습니다. 가능한 한 자동화하세요. |

## Red Flags

- 프로젝트에 CI Pipeline이 없음
- CI 실패가 무시되거나 침묵됨
- Pipeline 통과를 위해 CI에서 Test를 비활성화함
- Staging 검증 없이 Production Deploy 수행
- Rollback 메커니즘 없음
- Secret이 코드나 CI Config 파일에 저장됨 (Secret Manager가 아님)
- 최적화 노력 없이 CI 시간이 길어짐

## Verification

CI 설정 또는 수정 후:

- [ ] 모든 Quality Gate가 존재함 (Lint, Types, Tests, Build, Audit)
- [ ] 모든 PR 및 Main 브랜치 Push 시 Pipeline이 실행됨
- [ ] 실패 시 Merge가 차단됨 (Branch Protection 설정됨)
- [ ] CI 결과가 Development Loop로 피드백됨
- [ ] Secret이 코드가 아닌 Secret Manager에 저장됨
- [ ] Deployment에 Rollback 메커니즘이 있음
- [ ] Test Suite에 대해 Pipeline이 10분 이내에 실행됨
