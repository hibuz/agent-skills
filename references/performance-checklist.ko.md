# Performance Checklist

Web Application Performance를 위한 빠른 참조 Checklist입니다. `performance-optimization` 기술과 함께 사용하세요.

## Table of Contents

- [Core Web Vitals Targets](#core-web-vitals-targets)
- [Frontend Checklist](#frontend-checklist)
- [Backend Checklist](#backend-checklist)
- [Measurement Commands](#measurement-commands)
- [Common Anti-Patterns](#common-anti-patterns)

## Core Web Vitals Targets

| Metric | Good | Needs Work | Poor |
|--------|------|------------|------|
| LCP (Largest Contentful Paint) | ≤ 2.5s | ≤ 4.0s | > 4.0s |
| INP (Interaction to Next Paint) | ≤ 200ms | ≤ 500ms | > 500ms |
| CLS (Cumulative Layout Shift) | ≤ 0.1 | ≤ 0.25 | > 0.25 |

## TTFB Diagnosis

TTFB가 느릴 때(> 800ms), DevTools Network Waterfall에서 각 구성 요소를 확인하세요:

- [ ] **DNS resolution** 느림 → 알려진 Origin에 대해 `<link rel="dns-prefetch">` 또는 `<link rel="preconnect">` 추가
- [ ] **TCP/TLS handshake** 느림 → HTTP/2 활성화, Edge Deployment 고려, Keep-alive 확인
- [ ] **Server processing** 느림 → Backend 프로파일링, 느린 Query 확인, Caching 추가

## Frontend Checklist

### Images
- [ ] 이미지가 현대적인 포맷(WebP, AVIF)을 사용함
- [ ] 이미지가 Responsive 사이즈를 사용함 (`srcset` 및 `sizes`)
- [ ] 이미지와 `<source>` 요소에 명시적인 `width`와 `height`가 있음 (Art Direction 시 CLS 방지)
- [ ] Below-the-fold(스크롤해야 보이는 영역) 이미지는 `loading="lazy"` 및 `decoding="async"`를 사용함
- [ ] Hero/LCP 이미지는 `fetchpriority="high"`를 사용하고 Lazy Loading을 사용하지 않음

### JavaScript
- [ ] Bundle 크기가 Gzipped 상태에서 200KB 미만임 (초기 로드 기준)
- [ ] Route 및 무거운 기능을 위해 동적 `import()`를 사용한 Code Splitting 적용
- [ ] Tree Shaking 활성화됨 (Dependency가 ESM을 제공하고 `sideEffects: false`로 표시되었는지 확인)
- [ ] `<head>` 내에 Blocking JavaScript가 없음 (`defer` 또는 `async` 사용)
- [ ] 무거운 계산은 Web Worker로 오프로드함 (해당하는 경우)
- [ ] 동일한 Prop으로 재렌더링되는 비용이 큰 Component에 `React.memo()` 적용
- [ ] 프로파일링 결과 이득이 있는 곳에만 `useMemo()` / `useCallback()` 사용

### CSS
- [ ] Critical CSS가 Inline화되거나 Preload됨
- [ ] 중요하지 않은 Style에 대해 Render-blocking CSS가 없음
- [ ] Production에서 CSS-in-JS Runtime 비용이 없음 (Extraction 사용)
- [ ] Font Display 전략 설정됨 (`font-display: swap` 또는 `optional`)
- [ ] Custom Font 전에 System Font Stack 고려

### Network
- [ ] Static Asset이 긴 `max-age`와 Content Hashing으로 캐싱됨
- [ ] API Response가 적절한 곳에서 캐싱됨 (`Cache-Control`)
- [ ] HTTP/2 또는 HTTP/3 활성화됨
- [ ] 알려진 Origin에 대해 리소스가 Preconnect됨 (`<link rel="preconnect">`)
- [ ] 불필요한 Redirect 없음

### Rendering
- [ ] Layout Thrashing(강제 동기 레이아웃) 없음
- [ ] 애니메이션이 `transform`과 `opacity`를 사용함 (GPU 가속)
- [ ] 긴 리스트는 Virtualization(예: `react-window`)을 사용함
- [ ] 불필요한 전체 페이지 재렌더링 없음

## Backend Checklist

### Database
- [ ] N+1 Query 패턴 없음 (Eager Loading / Join 사용)
- [ ] Query에 적절한 Index가 있음
- [ ] 목록 Endpoint가 Paginate됨 (절대 `SELECT * FROM table`을 하지 않음)
- [ ] Connection Pooling 구성됨
- [ ] Slow Query Logging 활성화됨

### API
- [ ] Response 시간 < 200ms (p95 기준)
- [ ] Request Handler 내에 동기적인 무거운 계산 없음
- [ ] 개별 호출의 루프 대신 Bulk Operation 사용
- [ ] Response 압축 (gzip/brotli)
- [ ] 적절한 Caching (In-memory, Redis, CDN)

### Infrastructure
- [ ] Static Asset을 위한 CDN 사용
- [ ] 사용자와 가까운 곳에 서버 위치 (또는 Edge Deployment)
- [ ] Horizontal Scaling 구성됨 (필요한 경우)
- [ ] Load Balancer를 위한 Health Check Endpoint

## Measurement Commands

```bash
# Lighthouse CLI
npx lighthouse https://localhost:3000 --output json --output-path ./report.json

# Bundle analysis
npx webpack-bundle-analyzer stats.json
# 또는 Vite의 경우:
npx vite-bundle-visualizer

# Bundle 크기 확인
npx bundlesize

# 코드 내 Web Vitals 측정
import { onLCP, onINP, onCLS } from 'web-vitals';
onLCP(console.log);
onINP(console.log);
onCLS(console.log);
```

## Common Anti-Patterns

| Anti-Pattern | Impact | Fix |
|---|---|---|
| N+1 queries | 선형적인 DB 부하 증가 | Join, Include 또는 Batch Loading 사용 |
| Unbounded queries | 메모리 고갈, Timeout | 항상 Pagination 적용, LIMIT 추가 |
| Missing indexes | 데이터 증가에 따라 Read 속도 저하 | Filter/Sort되는 컬럼에 Index 추가 |
| Layout thrashing | Jank(끊김), 프레임 드랍 | DOM Read를 모아서 하고, 그 다음 Write를 모아서 수행 |
| Unoptimized images | LCP 저하, 대역폭 낭비 | WebP 사용, Responsive 사이즈, Lazy Load 적용 |
| Large bundles | Time to Interactive 지연 | Code Split, Tree Shake, Dependency Audit 수행 |
| Blocking main thread | INP 저하, UI 반응 없음 | Web Worker 사용, 작업 지연(Defer) 처리 |
| Memory leaks | 메모리 사용량 증가, 결국 Crash | Listener, Interval, Ref 정리 |
