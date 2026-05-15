---
name: shipping-and-launch
description: production 런칭을 준비. production 배포를 준비할 때 사용. 런칭 전 체크리스트가 필요할 때, 모니터링을 설정할 때, 단계적 rollout을 계획할 때, 또는 rollback 전략이 필요할 때 사용.
---

# Shipping and Launch

## Overview

자신 있게 ship하세요. 목표는 단지 배포가 아닙니다 — 모니터링이 갖춰지고, rollback 계획이 준비되고, 성공이 어떻게 보이는지 명확히 이해한 채 안전하게 배포하는 것입니다. 모든 런칭은 되돌릴 수 있고, 관찰 가능하고, 점진적이어야 합니다.

## When to Use

- 기능을 production에 처음 배포
- 사용자에게 중대한 변경 릴리스
- 데이터나 인프라 migration
- beta나 early access 프로그램 개시
- 위험을 수반하는 모든 배포 (모든 배포)

## 런칭 전 체크리스트

### 코드 품질

- [ ] 모든 테스트 통과 (unit, integration, e2e)
- [ ] 빌드가 경고 없이 성공
- [ ] lint와 type checking 통과
- [ ] 코드 리뷰되고 승인됨
- [ ] 런칭 전 해결해야 할 TODO 주석 없음
- [ ] production 코드에 `console.log` 디버깅 문장 없음
- [ ] 에러 처리가 예상 실패 모드를 커버

### 보안

- [ ] 코드나 version control에 secret 없음
- [ ] `npm audit`가 critical이나 high 취약점을 안 보임
- [ ] 모든 사용자 대면 endpoint에 입력 검증
- [ ] 인증과 인가 검사 갖춰짐
- [ ] security header 구성됨 (CSP, HSTS 등)
- [ ] 인증 endpoint에 rate limiting
- [ ] CORS가 특정 origin으로 구성됨 (wildcard 아님)

### 성능

- [ ] Core Web Vitals가 "Good" 임계값 내
- [ ] critical path에 N+1 쿼리 없음
- [ ] 이미지 최적화됨 (압축, 반응형 크기, lazy loading)
- [ ] 번들 크기가 예산 내
- [ ] 데이터베이스 쿼리에 적절한 인덱스
- [ ] 정적 asset과 반복 쿼리에 캐싱 구성됨

### 접근성

- [ ] 모든 상호작용 요소에 키보드 내비게이션 동작
- [ ] 스크린 리더가 페이지 콘텐츠와 구조를 전달 가능
- [ ] 색상 대비가 WCAG 2.1 AA 충족 (텍스트 4.5:1)
- [ ] 모달과 동적 콘텐츠에 focus 관리 정확
- [ ] 에러 메시지가 서술적이고 form 필드와 연결됨
- [ ] axe-core나 Lighthouse에 접근성 경고 없음

### 인프라

- [ ] production에 환경 변수 설정됨
- [ ] 데이터베이스 migration 적용됨 (또는 적용 준비됨)
- [ ] DNS와 SSL 구성됨
- [ ] 정적 asset에 CDN 구성됨
- [ ] logging과 에러 리포팅 구성됨
- [ ] health check endpoint가 존재하고 응답함

### 문서

- [ ] 새 설정 요구사항으로 README 업데이트됨
- [ ] API 문서가 최신
- [ ] 모든 아키텍처 결정에 ADR 작성됨
- [ ] Changelog 업데이트됨
- [ ] 사용자 대면 문서 업데이트됨 (해당 시)

## Feature Flag 전략

배포를 릴리스에서 분리하기 위해 feature flag 뒤에 ship:

```typescript
// feature flag 확인
const flags = await getFeatureFlags(userId);

if (flags.taskSharing) {
  // 새 기능: task sharing
  return <TaskSharingPanel task={task} />;
}

// 기본: 기존 동작
return null;
```

**Feature flag 라이프사이클:**

```
1. flag OFF로 DEPLOY     → 코드는 production에 있지만 비활성
2. 팀/beta에 ENABLE      → production 환경에서 내부 테스트
3. GRADUAL ROLLOUT       → 사용자 5% → 25% → 50% → 100%
4. 각 단계에서 MONITOR   → 에러율, 성능, 사용자 피드백 관찰
5. CLEAN UP              → 전체 rollout 후 flag와 죽은 코드 경로 제거
```

**규칙:**
- 모든 feature flag에 소유자와 만료 날짜 있음
- 전체 rollout 2주 이내에 flag 정리
- feature flag를 중첩하지 말 것 (지수적 조합 생성)
- CI에서 flag 양쪽 상태(on과 off) 테스트

## 단계적 Rollout

### Rollout 순서

```
1. staging에 DEPLOY
   └── staging 환경에서 전체 테스트 스위트
   └── critical 흐름의 수동 smoke test

2. production에 DEPLOY (feature flag OFF)
   └── 배포 성공 검증 (health check)
   └── 에러 모니터링 확인 (새 에러 없음)

3. 팀에 ENABLE (내부 사용자에 flag ON)
   └── 팀이 production에서 기능 사용
   └── 24시간 모니터링 윈도우

4. CANARY rollout (사용자 5%에 flag ON)
   └── 에러율, 지연, 사용자 행동 모니터링
   └── 메트릭 비교: canary vs. baseline
   └── 24-48시간 모니터링 윈도우
   └── 모든 임계값 통과 시에만 진행 (아래 표 참고)

5. GRADUAL 증가 (25% -> 50% -> 100%)
   └── 각 단계에서 동일한 모니터링
   └── 어느 시점에서든 이전 퍼센트로 roll back 가능

6. FULL rollout (모든 사용자에 flag ON)
   └── 1주간 모니터링
   └── feature flag 정리
```

### Rollout 결정 임계값

각 단계에서 진행, 보류, 또는 roll back을 결정하는 데 이 임계값을 사용:

| 메트릭 | 진행 (green) | 보류하고 조사 (yellow) | Roll back (red) |
|--------|-----------------|-------------------------------|-----------------|
| 에러율 | baseline의 10% 이내 | baseline 대비 10-100% 위 | baseline의 >2배 |
| P95 지연 | baseline의 20% 이내 | baseline 대비 20-50% 위 | baseline 대비 >50% 위 |
| 클라이언트 JS 에러 | 새 에러 유형 없음 | 세션의 <0.1%에 새 에러 | 세션의 >0.1%에 새 에러 |
| 비즈니스 메트릭 | 중립 또는 긍정 | 하락 <5% (노이즈일 수 있음) | 하락 >5% |

### 언제 Roll Back

다음일 때 즉시 roll back:
- 에러율이 baseline의 2배 이상 증가
- P95 지연이 50% 이상 증가
- 사용자 보고 이슈 급증
- 데이터 무결성 이슈 감지
- 보안 취약점 발견

## 모니터링과 Observability

### 무엇을 모니터링하나

```
애플리케이션 메트릭:
├── 에러율 (총계 및 endpoint별)
├── 응답 시간 (p50, p95, p99)
├── 요청 볼륨
├── 활성 사용자
└── 핵심 비즈니스 메트릭 (전환, engagement)

인프라 메트릭:
├── CPU 및 메모리 사용률
├── 데이터베이스 connection pool 사용
├── 디스크 공간
├── network 지연
└── 큐 깊이 (해당 시)

클라이언트 메트릭:
├── Core Web Vitals (LCP, INP, CLS)
├── JavaScript 에러
├── 클라이언트 관점의 API 에러율
└── 페이지 로드 시간
```

### 에러 리포팅

```typescript
// 리포팅을 갖춘 error boundary 설정
class ErrorBoundary extends React.Component {
  componentDidCatch(error: Error, info: React.ErrorInfo) {
    // 에러 추적 서비스에 보고
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

// 서버 측 에러 리포팅
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  reportError(err, {
    method: req.method,
    url: req.url,
    userId: req.user?.id,
  });

  // 사용자에게 내부 정보를 노출하지 말 것
  res.status(500).json({
    error: { code: 'INTERNAL_ERROR', message: 'Something went wrong' },
  });
});
```

### 런칭 후 검증

런칭 후 첫 시간에:

```
1. health endpoint가 200 반환하는지 확인
2. 에러 모니터링 대시보드 확인 (새 에러 유형 없음)
3. 지연 대시보드 확인 (회귀 없음)
4. critical 사용자 흐름 수동 테스트
5. 로그가 흐르고 읽을 수 있는지 검증
6. rollback 메커니즘이 동작하는지 확인 (가능하면 dry run)
```

## Rollback 전략

모든 배포는 일어나기 전에 rollback 계획이 필요합니다:

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
## See Also

- 보안 런칭 전 검사는 `references/security-checklist.md`를 참고
- 성능 런칭 전 체크리스트는 `references/performance-checklist.md`를 참고
- 런칭 전 접근성 검증은 `references/accessibility-checklist.md`를 참고

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "staging에서 동작하니 production에서도 동작할 거야" | production은 다른 데이터, 트래픽 패턴, 엣지 케이스를 가집니다. 배포 후 모니터링하세요. |
| "이건 feature flag가 필요 없어" | 모든 기능이 kill switch의 이점을 받습니다. "단순한" 변경도 깨질 수 있습니다. |
| "모니터링은 오버헤드야" | 모니터링이 없으면 대시보드 대신 사용자 불만으로 문제를 발견합니다. |
| "모니터링은 나중에 추가할게" | 런칭 전에 추가하세요. 볼 수 없는 것은 디버깅할 수 없습니다. |
| "roll back은 실패를 인정하는 거야" | roll back은 책임 있는 엔지니어링입니다. 깨진 기능을 ship하는 것이 실패입니다. |

## Red Flags

- rollback 계획 없이 배포
- production에 모니터링이나 에러 리포팅 없음
- big-bang 릴리스 (한 번에 전부, staging 없음)
- 만료나 소유자 없는 feature flag
- 첫 시간 동안 배포를 모니터링하는 사람 없음
- 코드가 아니라 기억으로 한 production 환경 구성
- "금요일 오후네, ship하자"

## Verification

배포 전:

- [ ] 런칭 전 체크리스트 완료 (모든 섹션 green)
- [ ] feature flag 구성됨 (해당 시)
- [ ] rollback 계획 문서화됨
- [ ] 모니터링 대시보드 설정됨
- [ ] 팀에 배포 통지됨

배포 후:

- [ ] health check가 200 반환
- [ ] 에러율 정상
- [ ] 지연 정상
- [ ] critical 사용자 흐름 동작
- [ ] 로그가 흐름
- [ ] rollback 테스트되거나 준비 확인됨
