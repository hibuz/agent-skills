# Performance Checklist

웹 애플리케이션 성능을 위한 빠른 참조 체크리스트. `performance-optimization` skill과 함께 사용하세요.

## 목차

- [Core Web Vitals 목표](#core-web-vitals-targets)
- [TTFB 진단](#ttfb-diagnosis)
- [프론트엔드 체크리스트](#frontend-checklist)
- [백엔드 체크리스트](#backend-checklist)
- [측정 커맨드](#measurement-commands)
- [흔한 안티패턴](#common-anti-patterns)

## Core Web Vitals 목표

| 메트릭 | Good | Needs Work | Poor |
|--------|------|------------|------|
| LCP (Largest Contentful Paint) | ≤ 2.5s | ≤ 4.0s | > 4.0s |
| INP (Interaction to Next Paint) | ≤ 200ms | ≤ 500ms | > 500ms |
| CLS (Cumulative Layout Shift) | ≤ 0.1 | ≤ 0.25 | > 0.25 |

## TTFB 진단

TTFB가 느릴 때(> 800ms), DevTools Network waterfall에서 각 컴포넌트를 확인하세요:

- [ ] **DNS resolution** 느림 → 알려진 origin에 `<link rel="dns-prefetch">` 또는 `<link rel="preconnect">` 추가
- [ ] **TCP/TLS handshake** 느림 → HTTP/2 활성화, edge 배포 고려, keep-alive 확인
- [ ] **서버 처리** 느림 → 백엔드 profiling, 느린 쿼리 확인, 캐싱 추가

## 프론트엔드 체크리스트

### 이미지
- [ ] 이미지가 최신 포맷 사용 (WebP, AVIF)
- [ ] 이미지가 반응형으로 sizing됨 (`srcset`과 `sizes`)
- [ ] 이미지와 `<source>` 요소에 명시적 `width`와 `height` (art direction에서 CLS 방지)
- [ ] below-the-fold 이미지가 `loading="lazy"`와 `decoding="async"` 사용
- [ ] Hero/LCP 이미지가 `fetchpriority="high"` 사용, lazy loading 없음

### JavaScript
- [ ] 번들 크기 200KB gzipped 미만 (초기 로드)
- [ ] route와 무거운 기능에 동적 `import()`로 code splitting
- [ ] Tree shaking 활성화 (의존성이 ESM을 제공하고 `sideEffects: false`를 표시하는지 확인)
- [ ] `<head>`에 blocking JavaScript 없음 (`defer` 또는 `async` 사용)
- [ ] 무거운 계산을 Web Worker로 offload (해당되는 경우)
- [ ] 같은 props로 re-render되는 비싼 컴포넌트에 `React.memo()`
- [ ] profiling이 이점을 보이는 곳에만 `useMemo()` / `useCallback()`
- [ ] 긴 task(> 50ms)를 쪼개 main thread를 사용 가능하게 유지 — INP의 주요 lever
- [ ] 장시간 실행 루프 안에서 `yieldToMain` 패턴 사용해 chunk 사이에 input 이벤트가 실행될 수 있게 함
- [ ] 가능한 곳에 최신 스케줄링 API 사용: `scheduler.yield()`(선호), 우선순위가 있는 `scheduler.postTask()`, 필요할 때만 yield하는 `isInputPending()`
- [ ] 연기 가능하고 긴급하지 않은 작업에 `requestIdleCallback` (analytics flush, prefetch, warmup)
- [ ] 비핵심 작업을 이벤트 핸들러 밖으로 연기 (예: analytics, logging) 해 상호작용에 대한 응답이 지연되지 않게 함
- [ ] 서드파티 스크립트를 `async` / `defer`로 로드하고, 크기를 감사하고, 무거운 경우 facade로 front (chat 위젯, embed)

### CSS
- [ ] Critical CSS inline 또는 preload
- [ ] 비핵심 스타일에 render-blocking CSS 없음
- [ ] production에 CSS-in-JS 런타임 비용 없음 (extraction 사용)

### 폰트
- [ ] 2–3개 폰트 패밀리, 각 2–3개 weight로 제한 (추가 weight마다 또 다른 요청)
- [ ] WOFF2 포맷만 (가장 작고 보편 지원 — WOFF/TTF/EOT 건너뛰기)
- [ ] 가능하면 self-host (서드파티 폰트 CDN은 DNS + TCP + TLS 왕복 추가)
- [ ] LCP-critical 폰트 preload: `<link rel="preload" as="font" type="font/woff2" crossorigin>`
- [ ] `font-display: swap` (또는 비핵심은 `optional`) 로 render를 막는 FOIT 회피
- [ ] `unicode-range`로 subset해 각 페이지가 필요한 glyph만 제공
- [ ] 여러 weight/style이 필요하면 variable font 고려 (한 파일이 여럿을 대체)
- [ ] fallback 폰트 메트릭을 `size-adjust`, `ascent-override`, `descent-override`로 조정해 폰트 swap 시 CLS 감소
- [ ] 커스텀 폰트 전에 system font stack 고려

### 네트워크
- [ ] 정적 asset을 긴 `max-age` + content hashing으로 캐싱
- [ ] 적절한 곳에 API 응답 캐싱 (`Cache-Control`)
- [ ] HTTP/2 또는 HTTP/3 활성화
- [ ] 알려진 origin에 리소스 preconnect (`<link rel="preconnect">`)
- [ ] 중요한 비이미지 리소스(예: 핵심 `<link rel="preload">`, above-the-fold `<script>`)에 `fetchpriority` 사용 — `<img>`에만이 아님
- [ ] 불필요한 redirect 없음

### 렌더링
- [ ] layout thrashing 없음 (강제 동기 layout)
- [ ] 애니메이션이 `transform`과 `opacity` 사용 (GPU 가속)
- [ ] 긴 목록에 virtualization 사용 (예: `react-window`)
- [ ] 불필요한 전체 페이지 re-render 없음
- [ ] 화면 밖 섹션에 `content-visibility: auto`와 `contain-intrinsic-size` 사용해 비가시 영역의 layout/paint 건너뛰기
- [ ] HTML 응답에 `unload` 이벤트 핸들러 없음, `Cache-Control: no-store` 없음 — back/forward 캐시(bfcache) 자격 보존

## 백엔드 체크리스트

### 데이터베이스
- [ ] N+1 쿼리 패턴 없음 (eager loading / join 사용)
- [ ] 쿼리에 적절한 인덱스
- [ ] 목록 endpoint pagination (절대 `SELECT * FROM table` 안 함)
- [ ] connection pooling 구성됨
- [ ] 느린 쿼리 logging 활성화

### API
- [ ] 응답 시간 < 200ms (p95)
- [ ] 요청 핸들러에 동기 무거운 계산 없음
- [ ] 개별 호출의 루프 대신 bulk 작업
- [ ] 응답 압축 (gzip/brotli)
- [ ] 적절한 캐싱 (in-memory, Redis, CDN)

### 인프라
- [ ] 정적 asset용 CDN
- [ ] 사용자 가까이 위치한 서버 (또는 edge 배포)
- [ ] horizontal scaling 구성됨 (필요한 경우)
- [ ] load balancer용 health check endpoint

## 측정 커맨드

### INP field data와 DevTools 워크플로

1. **Field data 먼저** — 최적화 전에 [CrUX Vis](https://developer.chrome.com/docs/crux/vis)나 RUM 도구에서 실제 사용자 INP 확인
2. **느린 상호작용 식별** — DevTools → Performance 패널 열기 → 상호작용하며 record; 클릭/키 입력이 트리거한 긴 task 찾기
3. **중급 Android에서 테스트** — INP 이슈는 종종 느린 하드웨어에서만 표면화됨; 실제 기기나 DevTools CPU throttling(4×–6× 감속) 사용

```bash
# Lighthouse CLI
npx lighthouse https://localhost:3000 --output json --output-path ./report.json

# 번들 분석
npx webpack-bundle-analyzer stats.json
# 또는 Vite의 경우:
npx vite-bundle-visualizer

# 번들 크기 확인
npx bundlesize

# 코드 내 Web Vitals
import { onLCP, onINP, onCLS } from 'web-vitals';
onLCP(console.log);
onINP(console.log);
onCLS(console.log);

# 상호작용 수준 상세를 포함한 INP (attribution build)
import { onINP } from 'web-vitals/attribution';
onINP(({ value, attribution }) => {
  const { interactionTarget, inputDelay, processingDuration, presentationDelay } = attribution;
  console.log({ value, interactionTarget, inputDelay, processingDuration, presentationDelay });
});
```

## 흔한 안티패턴

| 안티패턴 | 영향 | 수정 |
|---|---|---|
| N+1 쿼리 | 선형 DB 부하 증가 | join, include, 또는 batch loading 사용 |
| 무한 쿼리 | 메모리 고갈, timeout | 항상 paginate, LIMIT 추가 |
| 인덱스 누락 | 데이터 증가에 따른 느린 읽기 | 필터/정렬 컬럼에 인덱스 추가 |
| Layout thrashing | jank, 프레임 드롭 | DOM 읽기를 batch한 뒤 쓰기를 batch |
| 최적화 안 된 이미지 | 느린 LCP, 낭비된 대역폭 | WebP, 반응형 크기, lazy load 사용 |
| 큰 번들 | 느린 Time to Interactive | code split, tree shake, 의존성 감사 |
| main thread 차단 | 부실한 INP, 반응 없는 UI | 긴 task를 `scheduler.yield()` / `yieldToMain`으로 chunk, Web Worker로 offload |
| 메모리 누수 | 증가하는 메모리, 결국 크래시 | listener, interval, ref 정리 |
