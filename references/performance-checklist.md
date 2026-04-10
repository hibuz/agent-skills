# Performance 체크리스트

웹 애플리케이션 performance에 대한 Quick 참조 체크리스트입니다. `performance-optimization` skill와 함께 사용하세요.

## 목차

- [Core Web Vitals Targets](#core-web-vitals-targets)
- [Frontend Checklist](#frontend-checklist)
- [Backend Checklist](#backend-checklist)
- [Measurement Commands](#measurement-commands)
- [Common Anti-Patterns](#common-anti-patterns)

## Core Web Vitals 대상

| 미터법 | 좋음 | 작업 필요 | 가난한 |
|--------|------|------------|------|
| LCP(콘텐츠가 포함된 최대 페인트) | 2.5초 이하 | 4.0초 이하 | > 4.0초 |
| INP(다음 페인트와의 상호 작용) | 200ms 이하 | ≤ 500ms | > 500ms |
| CLS(누적 레이아웃 이동) | ≤ 0.1 | ≤ 0.25 | > 0.25 |

## TTFB 진단

TTFB가 느린 경우(> 800ms) DevTools 네트워크 폭포의 각 구성 요소를 확인하세요.

- [ ] **DNS 해상도** 느림 → 알려진 출처에 대해 `<link rel="dns-prefetch">` 또는 `<link rel="preconnect">` 추가
- [ ] **TCP/TLS 핸드셰이크** 느림 → HTTP/2 활성화, 에지 배포 고려, keep-alive 확인
- [ ] **서버 처리** 느림 → 프로필 백엔드, 느린 쿼리 확인, 캐싱 추가

## 프런트엔드 체크리스트

### 이미지
- [ ] 이미지는 최신 formats(WebP, AVIF)를 사용합니다.
- [ ] 이미지 크기가 반응적으로 조정됩니다(`srcset` 및 `sizes`).
- [ ] 이미지 및 `<source>` 요소에는 명시적인 `width` 및 `height`가 있습니다(아트 디렉션에서 CLS 방지).
- [ ] Below-the-fold images use `loading="lazy"` and `decoding="async"`
- [ ] Hero/LCP 이미지는 `fetchpriority="high"`를 사용하며 지연 로딩이 없습니다.

### JavaScript
- [ ] gzip으로 압축된 200KB 미만의 번들 크기(초기 로드)
- [ ] 경로 및 무거운 기능에 대해 동적 `import()`를 사용한 코드 분할
- [ ] 트리 흔들기 활성화됨(종속성 배송 ESM 확인 및 `sideEffects: false` 표시)
- [ ] `<head>`에서 JavaScript를 차단하지 않습니다(`defer` 또는 `async` 사용).
- [ ] 웹 작업자에게 무거운 계산이 오프로드됨(해당되는 경우)
- [ ] 동일한 소품을 사용하는 re-render와 값비싼 구성 요소의 `React.memo()`
- [ ] `useMemo()` / `useCallback()` 프로파일링이 이점을 보여주는 경우에만 해당

### CSS
- [ ] 중요 CSS 인라인 또는 사전 로드됨
- [ ] non-critical 스타일의 경우 render-blocking CSS 없음
- [ ] 생산 시 CSS-in-JS 런타임 비용 없음(추출 사용)
- [ ] 글꼴 표시 전략 세트(`font-display: swap` 또는 `optional`)
- [ ] 사용자 정의 글꼴 이전에 시스템 글꼴 스택이 고려됨

### 네트워크
- [ ] 긴 `max-age` + 콘텐츠 해싱으로 캐시된 정적 자산
- [ ] 적절한 경우 캐시된 API 응답(`Cache-Control`)
- [ ] HTTP/2 또는 HTTP/3 활성화됨
- [ ] 알려진 원본에 대해 미리 연결된 리소스(`<link rel="preconnect">`)
- [ ] 불필요한 리디렉션 없음

### 렌더링
- [ ] 레이아웃 스래싱 없음(강제 동기 레이아웃)
- [ ] 애니메이션은 `transform` 및 `opacity`(GPU 가속)를 사용합니다.
- [ ] 긴 목록은 가상화를 사용합니다(예: `react-window`).
- [ ] 불필요한 full-page re-renders 없음

## 백엔드 체크리스트

### 데이터베이스
- [ ] N+1 쿼리 패턴 없음(열심히 로드/조인 사용)
- [ ] 쿼리에 적절한 인덱스가 있음
- [ ] 페이지가 매겨진 목록 엔드포인트(`SELECT * FROM table`가 아님)
- [ ] 연결 풀링 구성됨
- [ ] 느린 쿼리 로깅 활성화

### API
- [ ] 응답 시간 < 200ms(p95)
- [ ] 요청 핸들러에 동기식 무거운 계산이 없습니다.
- [ ] 개별 호출의 루프 대신 대량 작업
- [ ] 응답 압축(gzip/brotli)
- [ ] 적절한 캐싱(in-memory, Redis, CDN)

### 인프라
- [ ] 정적 자산의 경우 CDN
- [ ] 사용자와 가까운 곳에 위치한 서버(또는 에지 배포)
- [ ] 수평 확장 구성됨(필요한 경우)
- [ ] 로드 밸런서에 대한 상태 확인 엔드포인트

## 측정 명령

```bash
# Lighthouse CLI
npx lighthouse https://localhost:3000 --output json --output-path ./report.json

# Bundle analysis
npx webpack-bundle-analyzer stats.json
# or for Vite:
npx vite-bundle-visualizer

# Check bundle size
npx bundlesize

# Web Vitals in code
import { onLCP, onINP, onCLS } from 'web-vitals';
onLCP(console.log);
onINP(console.log);
onCLS(console.log);
```

## 공통 Anti-Patterns

| Anti-Pattern | 영향 | 수정 |
|---|---|---|
| N+1 쿼리 | 선형 DB 부하 증가 | 조인, 포함 또는 일괄 로드 사용 |
| 무제한 쿼리 | 메모리 고갈, 시간 초과 | 항상 페이지를 매기고 LIMIT |
| 누락된 인덱스 | 데이터가 증가함에 따라 읽기 속도가 느려짐 | 필터링된 /sorted 열에 대한 인덱스 추가 |
| 레이아웃 스래싱 | Jank, 프레임 삭제 | 일괄 DOM 읽기 후 일괄 쓰기 |
| 최적화되지 않은 이미지 | 느린 LCP, 대역폭 낭비 | WebP, 반응형 크기, 지연 로드 사용 |
| 대형 번들 | 상호작용 속도가 느려짐 | 코드 분할, 트리 쉐이크, 감사 부서 |
| 메인 스레드 차단 | 불쌍한 INP, 응답하지 않는 UI | 웹 작업자 사용, 작업 연기 |
| 메모리 누수 | 메모리 증가, 결국 충돌 | 청취자, 간격, 참조 정리 |
