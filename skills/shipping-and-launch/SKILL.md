---
name: shipping-and-launch
description: 생산 개시를 준비합니다. 프로덕션 배포를 준비할 때 사용합니다. pre-launch 체크리스트가 필요할 때, 모니터링을 설정할 때, 단계적 출시를 계획할 때 또는 rollback 전략이 필요할 때 사용하세요.
---

# 배송 및 출시

## 개요

자신있게 배송하세요. 목표는 단지 ​​배포하는 것이 아니라 모니터링을 실시하고 rollback 계획을 준비하며 성공이 어떤 것인지 명확하게 이해하면서 안전하게 배포하는 것입니다. 모든 출시는 되돌릴 수 있고 관찰 가능하며 점진적이어야 합니다.

## 사용 시기

- 처음으로 프로덕션에 기능 배포
- 사용자에게 중요한 변화를 알리는 것
- 데이터 또는 인프라 마이그레이션
- 베타 또는 조기 액세스 프로그램 개설
- 위험을 수반하는 배포(모두)

## 출시 전 체크리스트

### 코드 품질

- [ ] 모든 테스트 통과(단위, 통합, e2e)
- [ ] Build가 경고 없이 성공합니다.
- [ ] 린트 및 유형 검사 통과
- [ ] 코드 검토 및 승인
- [ ] 출시 전에 해결해야 할 TODO 의견이 없습니다.
- [ ] 프로덕션 코드에 `console.log` 디버깅 문이 없습니다.
- [ ] 오류 처리에는 예상되는 실패 모드가 포함됩니다.

### 보안

- [ ] 코드 또는 버전 관리에 비밀이 없습니다.
- [ ] `npm audit`에는 심각하거나 높은 취약점이 표시되지 않습니다.
- [ ] 모든 user-facing 엔드포인트에 대한 입력 검증
- [ ] 인증 및 승인 확인이 이루어지고 있습니다.
- [ ] 보안 헤더가 구성됨(CSP, HSTS 등)
- [ ] 인증 엔드포인트의 속도 제한
- [ ] 특정 원본으로 구성된 CORS(와일드카드 아님)

### Performance

- [ ] "양호" 임계값 내 Core Web Vitals
- [ ] 중요한 경로에 N+1 쿼리가 없습니다.
- [ ] 이미지 최적화(압축, 반응형 크기, 지연 로딩)
- [ ] 예산 내 번들 크기
- [ ] 데이터베이스 쿼리에 적절한 인덱스가 있음
- [ ] 정적 자산 및 반복 쿼리에 대해 구성된 캐싱

### 접근성

- [ ] 키보드 탐색은 모든 대화형 요소에 대해 작동합니다.
- [ ] 스크린 리더는 페이지 내용과 구조를 전달할 수 있습니다.
- [ ] 색상 대비는 WCAG 2.1 AA(텍스트의 경우 4.5:1)를 충족합니다.
- [ ] 모달 및 동적 콘텐츠에 대한 올바른 초점 관리
- [ ] 오류 메시지는 설명적이며 form 필드와 연관되어 있습니다.
- [ ] axe-core 또는 Lighthouse에 접근성 경고가 없습니다.

### 인프라

- [ ] 프로덕션 환경 변수 설정
- [ ] 데이터베이스 마이그레이션이 적용되었습니다(또는 적용 준비가 완료됨).
- [ ] DNS 및 SSL가 구성됨
- [ ] 정적 자산에 대해 구성된 CDN
- [ ] 로깅 및 오류 reporting 구성됨
- [ ] 상태 확인 엔드포인트가 존재하고 응답합니다.

### 문서

- [ ] README는 새로운 설정 requirements로 업데이트되었습니다.
- [ ] API 문서 최신
- [ ] 모든 아키텍처 결정을 위해 작성된 ADRs
- [ ] 변경 로그가 업데이트되었습니다.
- [ ] User-facing 문서 업데이트됨(해당되는 경우)

## Feature Flag 전략

릴리스에서 배포를 분리하기 위해 feature flags 뒤에 배송:

```typescript
// Feature flag check
const flags = await getFeatureFlags(userId);

if (flags.taskSharing) {
  // New feature: task sharing
  return <TaskSharingPanel task={task} />;
}

// Default: existing behavior
return null;
```

**Feature flag 수명주기:**

```
1. DEPLOY with flag OFF     → Code is in production but inactive
2. ENABLE for team/beta     → Internal testing in production environment
3. GRADUAL ROLLOUT          → 5% → 25% → 50% → 100% of users
4. MONITOR at each stage    → Watch error rates, performance, user feedback
5. CLEAN UP                 → Remove flag and dead code path after full rollout
```

**Rules:**
- 모든 feature flag에는 소유자와 만료일이 있습니다.
- 전체 출시 후 2주 이내에 플래그를 정리합니다.
- feature flags를 중첩하지 마세요(지수 조합 생성).
- CI에서 두 플래그 상태(켜기 및 끄기)를 모두 테스트합니다.

## 단계적 출시

### 출시 순서

```
1. DEPLOY to staging
   └── Full test suite in staging environment
   └── Manual smoke test of critical flows

2. DEPLOY to production (feature flag OFF)
   └── Verify deployment succeeded (health check)
   └── Check error monitoring (no new errors)

3. ENABLE for team (flag ON for internal users)
   └── Team uses the feature in production
   └── 24-hour monitoring window

4. CANARY rollout (flag ON for 5% of users)
   └── Monitor error rates, latency, user behavior
   └── Compare metrics: canary vs. baseline
   └── 24-48 hour monitoring window
   └── Advance only if all thresholds pass (see table below)

5. GRADUAL increase (25% -> 50% -> 100%)
   └── Same monitoring at each step
   └── Ability to roll back to previous percentage at any point

6. FULL rollout (flag ON for all users)
   └── Monitor for 1 week
   └── Clean up feature flag
```

### 출시 결정 임계값

다음 임계값을 사용하여 각 단계에서 진행, 보류 또는 롤백 여부를 결정합니다.

| 미터법 | 사전(녹색) | 잡고 조사하기(노란색) | 롤백(빨간색) |
|--------|-----------------|-------------------------------|-----------------|
| 오류율 | 기준치의 10% 이내 | 기준치보다 10-100% | >2배 기준 |
| P95 대기 시간 | 기준치의 20% 이내 | 기준치보다 20-50% | 기준치보다 >50% |
| Client JS 오류 | 새로운 오류 유형 없음 | 세션의 <0.1% of sessions | New errors at >0.1%에서 새로운 오류 |
| 비즈니스 지표 | 중립 또는 긍정적 | Decline <5% (may be noise) | Decline >5% |

### 롤백 시기

다음과 같은 경우 즉시 롤백하세요.
- 기준치 대비 오류율이 2배 이상 증가
- P95 대기 시간이 50% 이상 증가합니다.
- 사용자-reported 문제 급증
- 데이터 무결성 문제가 감지되었습니다.
- 보안 취약점 발견

## 모니터링 및 관찰 가능성

### 모니터링 대상

```
Application metrics:
├── Error rate (total and by endpoint)
├── Response time (p50, p95, p99)
├── Request volume
├── Active users
└── Key business metrics (conversion, engagement)

Infrastructure metrics:
├── CPU and memory utilization
├── Database connection pool usage
├── Disk space
├── Network latency
└── Queue depth (if applicable)

Client metrics:
├── Core Web Vitals (LCP, INP, CLS)
├── JavaScript errors
├── API error rates from client perspective
└── Page load time
```

### 오류 Reporting

```typescript
// Set up error boundary with reporting
class ErrorBoundary extends React.Component {
  componentDidCatch(error: Error, info: React.ErrorInfo) {
    // Report to error tracking service
    reportError(error, {
      componentStack: info.componentStack,
      userId: getCurrentUser()?.id,
      page: window.location.pathname,
    });
  }

  render() {
    if (this.state.hasError) {
      return <ErrorFallback onRetry={() => this.setState({ hasError: false })} />;
    }
    return this.props.children;
  }
}

// Server-side error reporting
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  reportError(err, {
    method: req.method,
    url: req.url,
    userId: req.user?.id,
  });

  // Don't expose internals to users
  res.status(500).json({
    error: { code: 'INTERNAL_ERROR', message: 'Something went wrong' },
  });
});
```

### 출시 후 확인

출시 후 첫 1시간 동안:

```
1. Check health endpoint returns 200
2. Check error monitoring dashboard (no new error types)
3. Check latency dashboard (no regression)
4. Test the critical user flow manually
5. Verify logs are flowing and readable
6. Confirm rollback mechanism works (dry run if possible)
```

## Rollback 전략

모든 배포에는 rollback 계획이 필요합니다.

```markdown
## Rollback Plan for [Feature/Release]

### Trigger Conditions
- Error rate > 2x baseline
- P95 latency > [X]ms
- User reports of [specific issue]

### Rollback Steps
1. Disable feature flag (if applicable)
   OR
1. Deploy previous version: `git revert <commit> && git push`
2. Verify rollback: health check, error monitoring
3. Communicate: notify team of rollback

### Database Considerations
- Migration [X] has a rollback: `npx prisma migrate rollback`
- Data inserted by new feature: [preserved / cleaned up]

### Time to Rollback
- Feature flag: < 1 minute
- Redeploy previous version: < 5 minutes
- Database rollback: < 15 minutes
```
## 참고 항목

- 보안 pre-launch 검사에 대해서는 `references/security-checklist.md`를 참조하세요.
- performance pre-launch 체크리스트는 `references/performance-checklist.md`를 참조하세요.
- 출시 전 접근성 검증은 `references/accessibility-checklist.md`를 참고하세요.

## 일반적인 합리화

| 합리화 | 현실 |
|---|---|
| "스테이징에서 작동하며 프로덕션에서도 작동합니다" | 프로덕션에는 데이터, 트래픽 패턴, 엣지 케이스가 다릅니다. 배포 후 모니터링합니다. |
| "이를 위해서는 feature flags가 필요하지 않습니다." | 모든 기능은 킬 스위치의 이점을 얻습니다. "간단한" 변경이라도 문제를 일으킬 수 있습니다. |
| "모니터링이 오버헤드입니다." | 모니터링이 없다는 것은 dashboards 대신 사용자 불만으로 문제를 발견한다는 의미입니다. |
| "나중에 모니터링을 추가하겠습니다" | 출시 전에 추가하세요. 볼 수 없는 것은 디버깅할 수 없습니다. |
| "롤백은 실패를 인정하는 것입니다" | 롤백은 책임 있는 엔지니어링입니다. 손상된 기능을 배송하는 것은 실패입니다. |

## 위험 신호

- rollback 계획 없이 배포
- 생산 중 모니터링이나 오류 reporting이 없습니다.
- 빅뱅 발매(모든 것을 한 번에, 준비 없음)
- 만료 또는 소유자가 없는 Feature flags
- 처음 한 시간 동안 배포를 모니터링하는 사람은 없습니다.
- 코드가 아닌 메모리로 프로덕션 환경 구성
- "금요일 오후야, 배송하자"

## 확인

배포하기 전에:

- [ ] 출시 전 체크리스트 완료(모든 섹션 녹색)
- [ ] Feature flag 구성됨(해당하는 경우)
- [ ] Rollback 계획이 문서화되었습니다.
- [ ] 모니터링 dashboards 설정
- [ ] 팀에 배포 알림이 전달됨

배포 후:

- [ ] 상태 확인이 200을 반환합니다.
- [ ] 오류율은 normal입니다.
- [ ] 지연 시간은 normal입니다.
- [ ] 중요한 사용자 흐름이 작동합니다.
- [ ] 로그가 흐르고 있습니다.
- [ ] Rollback 테스트 또는 검증 완료 준비
