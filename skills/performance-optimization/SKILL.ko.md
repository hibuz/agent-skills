---
name: performance-optimization
description: 애플리케이션 성능을 최적화. 성능 요구사항이 있을 때, 성능 회귀가 의심될 때, 또는 Core Web Vitals나 로드 시간 개선이 필요할 때 사용. profiling이 수정해야 할 병목을 드러낼 때 사용.
---

# Performance Optimization

## Overview

최적화 전에 측정하세요. 측정 없는 성능 작업은 추측입니다 — 그리고 추측은 중요한 것을 개선하지 않으면서 복잡도를 더하는 성급한 최적화로 이어집니다. 먼저 profile하고, 실제 병목을 식별하고, 고치고, 다시 측정하세요. 측정이 중요하다고 증명한 것만 최적화하세요.

## When to Use

- spec에 성능 요구사항이 존재 (로드 시간 예산, 응답 시간 SLA)
- 사용자나 모니터링이 느린 동작을 보고
- Core Web Vitals 점수가 임계값 이하
- 변경이 회귀를 도입했다고 의심
- 큰 데이터셋이나 높은 트래픽을 다루는 기능 빌드

**사용하지 말아야 할 때:** 문제의 증거가 있기 전에 최적화하지 마세요. 성급한 최적화는 얻는 성능보다 비싼 복잡도를 더합니다.

## Core Web Vitals 목표

| 메트릭 | Good | Needs Improvement | Poor |
|--------|------|-------------------|------|
| **LCP** (Largest Contentful Paint) | ≤ 2.5s | ≤ 4.0s | > 4.0s |
| **INP** (Interaction to Next Paint) | ≤ 200ms | ≤ 500ms | > 500ms |
| **CLS** (Cumulative Layout Shift) | ≤ 0.1 | ≤ 0.25 | > 0.25 |

## 최적화 워크플로

```
1. MEASURE  → 실제 데이터로 baseline 수립
2. IDENTIFY → 실제 병목 찾기 (가정이 아니라)
3. FIX      → 특정 병목 해결
4. VERIFY   → 다시 측정, 개선 확인
5. GUARD    → 회귀 방지를 위해 모니터링이나 테스트 추가
```

### Step 1: 측정

두 가지 상호 보완적 접근 — 둘 다 사용:

- **Synthetic (Lighthouse, DevTools Performance 탭):** 통제된 조건, 재현 가능. CI 회귀 탐지와 특정 이슈 격리에 최적.
- **RUM (web-vitals 라이브러리, CrUX):** 실제 조건의 실제 사용자 데이터. 수정이 실제로 사용자 경험을 개선했는지 검증하는 데 필요.

**프론트엔드:**
```bash
# Synthetic: Chrome DevTools의 Lighthouse (또는 CI)
# Chrome DevTools → Performance 탭 → Record
# Chrome DevTools MCP → Performance trace

# RUM: 코드 내 Web Vitals 라이브러리
import { onLCP, onINP, onCLS } from 'web-vitals';

onLCP(console.log);
onINP(console.log);
onCLS(console.log);
```

**백엔드:**
```bash
# 응답 시간 logging
# Application Performance Monitoring (APM)
# timing을 가진 데이터베이스 쿼리 logging

# 간단한 timing
console.time('db-query');
const result = await db.query(...);
console.timeEnd('db-query');
```

### 어디서 측정을 시작하나

증상을 사용해 무엇을 먼저 측정할지 결정:

```
무엇이 느린가?
├── 첫 페이지 로드
│   ├── 큰 번들? --> 번들 크기 측정, code splitting 확인
│   ├── 느린 서버 응답? --> DevTools Network waterfall에서 TTFB 측정
│   │   ├── DNS 김? --> 알려진 origin에 dns-prefetch / preconnect 추가
│   │   ├── TCP/TLS 김? --> HTTP/2 활성화, edge 배포 확인, keep-alive
│   │   └── Waiting (server) 김? --> 백엔드 profile, 쿼리와 캐싱 확인
│   └── render-blocking 리소스? --> network waterfall에서 CSS/JS 차단 확인
├── 상호작용이 굼뜬 느낌
│   ├── 클릭 시 UI freeze? --> main thread profile, 긴 task(>50ms) 찾기
│   ├── form 입력 lag? --> re-render, controlled 컴포넌트 오버헤드 확인
│   └── 애니메이션 jank? --> layout thrashing, 강제 reflow 확인
├── 내비게이션 후 페이지
│   ├── 데이터 로딩? --> API 응답 시간 측정, waterfall 확인
│   └── 클라이언트 렌더링? --> 컴포넌트 render 시간 profile, N+1 fetch 확인
└── 백엔드 / API
    ├── 단일 endpoint 느림? --> 데이터베이스 쿼리 profile, 인덱스 확인
    ├── 모든 endpoint 느림? --> connection pool, 메모리, CPU 확인
    └── 간헐적 느림? --> lock contention, GC pause, 외부 의존성 확인
```

### Step 2: 병목 식별

카테고리별 흔한 병목:

**프론트엔드:**

| 증상 | 가능한 원인 | 조사 |
|---------|-------------|---------------|
| 느린 LCP | 큰 이미지, render-blocking 리소스, 느린 서버 | network waterfall, 이미지 크기 확인 |
| 높은 CLS | 치수 없는 이미지, 늦게 로딩되는 콘텐츠, 폰트 shift | layout shift attribution 확인 |
| 부실한 INP | main thread의 무거운 JavaScript, 큰 DOM 업데이트 | Performance trace에서 긴 task 확인 |
| 느린 초기 로드 | 큰 번들, 많은 network 요청 | 번들 크기, code splitting 확인 |

**백엔드:**

| 증상 | 가능한 원인 | 조사 |
|---------|-------------|---------------|
| 느린 API 응답 | N+1 쿼리, 인덱스 누락, 최적화 안 된 쿼리 | 데이터베이스 쿼리 로그 확인 |
| 메모리 증가 | 누출된 참조, 무한 캐시, 큰 payload | heap snapshot 분석 |
| CPU 스파이크 | 동기 무거운 계산, regex backtracking | CPU profiling |
| 높은 지연 | 캐싱 누락, 중복 계산, network hop | 스택을 통한 요청 trace |

### Step 3: 흔한 안티패턴 수정

#### N+1 쿼리 (백엔드)

```typescript
// 나쁨: N+1 — owner를 위해 task당 쿼리 하나
const tasks = await db.tasks.findMany();
for (const task of tasks) {
  task.owner = await db.users.findUnique({ where: { id: task.ownerId } });
}

// 좋음: join/include로 단일 쿼리
const tasks = await db.tasks.findMany({
  include: { owner: true },
});
```

#### 무한 데이터 Fetch

```typescript
// 나쁨: 모든 레코드 fetch
const allTasks = await db.tasks.findMany();

// 좋음: 한도를 가진 paginated
const tasks = await db.tasks.findMany({
  take: 20,
  skip: (page - 1) * 20,
  orderBy: { createdAt: 'desc' },
});
```

#### 이미지 최적화 누락 (프론트엔드)

```html
<!-- 나쁨: 치수 없음, 포맷 최적화 없음 -->
<img src="/hero.jpg" />

<!-- 좋음: Hero / LCP 이미지 — art direction + 해상도 전환, 높은 우선순위 -->
<!--
  결합된 두 기법:
  - Art direction (media): breakpoint별 다른 crop/구성
  - 해상도 전환 (srcset + sizes): 화면 밀도별 올바른 파일 크기
-->
<picture>
  <!-- 모바일: portrait crop (8:10) -->
  <source
    media="(max-width: 767px)"
    srcset="/hero-mobile-400.avif 400w, /hero-mobile-800.avif 800w"
    sizes="100vw"
    width="800"
    height="1000"
    type="image/avif"
  />
  <source
    media="(max-width: 767px)"
    srcset="/hero-mobile-400.webp 400w, /hero-mobile-800.webp 800w"
    sizes="100vw"
    width="800"
    height="1000"
    type="image/webp"
  />
  <!-- 데스크톱: landscape crop (2:1) -->
  <source
    srcset="/hero-800.avif 800w, /hero-1200.avif 1200w, /hero-1600.avif 1600w"
    sizes="(max-width: 1200px) 100vw, 1200px"
    width="1200"
    height="600"
    type="image/avif"
  />
  <source
    srcset="/hero-800.webp 800w, /hero-1200.webp 1200w, /hero-1600.webp 1600w"
    sizes="(max-width: 1200px) 100vw, 1200px"
    width="1200"
    height="600"
    type="image/webp"
  />
  <img
    src="/hero-desktop.jpg"
    width="1200"
    height="600"
    fetchpriority="high"
    alt="Hero image description"
  />
</picture>

<!-- 좋음: Below-the-fold 이미지 — lazy load + async decoding -->
<img
  src="/content.webp"
  width="800"
  height="400"
  loading="lazy"
  decoding="async"
  alt="Content image description"
/>
```

#### 불필요한 Re-render (React)

```tsx
// 나쁨: 매 render마다 새 객체 생성, 자식이 re-render되게 함
function TaskList() {
  return <TaskFilters options={{ sortBy: 'date', order: 'desc' }} />;
}

// 좋음: 안정적 참조
const DEFAULT_OPTIONS = { sortBy: 'date', order: 'desc' } as const;
function TaskList() {
  return <TaskFilters options={DEFAULT_OPTIONS} />;
}

// 비싼 컴포넌트에 React.memo 사용
const TaskItem = React.memo(function TaskItem({ task }: Props) {
  return <div>{/* expensive render */}</div>;
});

// 비싼 계산에 useMemo 사용
function TaskStats({ tasks }: Props) {
  const stats = useMemo(() => calculateStats(tasks), [tasks]);
  return <div>{stats.completed} / {stats.total}</div>;
}
```

#### 큰 번들 크기

```typescript
// 현대 번들러(Vite, webpack 5+)는 의존성이 ESM을 제공하고 package.json에
// `sideEffects: false`로 표시되면 named import의 tree-shaking을 자동 처리.
// import 스타일을 바꾸기 전에 profile하세요 — 진짜 이득은 splitting과 lazy loading에서.

// 좋음: 무겁고 드물게 쓰는 기능에 동적 import
const ChartLibrary = lazy(() => import('./ChartLibrary'));

// 좋음: Suspense로 감싼 route 수준 code splitting
const SettingsPage = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <SettingsPage />
    </Suspense>
  );
}
```

#### 캐싱 누락 (백엔드)

```typescript
// 자주 읽고 드물게 바뀌는 데이터 캐싱
const CACHE_TTL = 5 * 60 * 1000; // 5분
let cachedConfig: AppConfig | null = null;
let cacheExpiry = 0;

async function getAppConfig(): Promise<AppConfig> {
  if (cachedConfig && Date.now() < cacheExpiry) {
    return cachedConfig;
  }
  cachedConfig = await db.config.findFirst();
  cacheExpiry = Date.now() + CACHE_TTL;
  return cachedConfig;
}

// 정적 asset용 HTTP 캐싱 헤더
app.use('/static', express.static('public', {
  maxAge: '1y',           // 1년간 캐시
  immutable: true,        // 절대 재검증 안 함 (파일명에 content hashing 사용)
}));

// API 응답용 Cache-Control
res.set('Cache-Control', 'public, max-age=300'); // 5분
```

## Performance Budget

예산을 설정하고 강제하세요:

```
JavaScript 번들: < 200KB gzipped (초기 로드)
CSS: < 50KB gzipped
이미지: 이미지당 < 200KB (above the fold)
폰트: 총 < 100KB
API 응답 시간: < 200ms (p95)
Time to Interactive: 4G에서 < 3.5s
Lighthouse Performance 점수: ≥ 90
```

**CI에서 강제:**
```bash
# 번들 크기 확인
npx bundlesize --config bundlesize.config.json

# Lighthouse CI
npx lhci autorun
```

## See Also

상세한 성능 체크리스트, 최적화 커맨드, 안티패턴 참조는 `references/performance-checklist.md`를 참고하세요.


## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "나중에 최적화할게" | 성능 부채는 복리로 불어납니다. 명백한 안티패턴을 지금 고치고, 마이크로 최적화는 미루세요. |
| "내 머신에선 빨라" | 당신의 머신은 사용자의 것이 아닙니다. 대표적 하드웨어와 network에서 profile하세요. |
| "이 최적화는 명백해" | 측정 안 했으면, 모르는 겁니다. 먼저 profile하세요. |
| "사용자는 100ms를 못 느껴" | 연구는 100ms 지연이 전환율에 영향을 줌을 보여줍니다. 사용자는 생각보다 더 느낍니다. |
| "프레임워크가 성능을 처리해" | 프레임워크는 일부 이슈를 막지만 N+1 쿼리나 과대 번들을 못 고칩니다. |

## Red Flags

- 정당화할 profiling 데이터 없는 최적화
- 데이터 fetch의 N+1 쿼리 패턴
- pagination 없는 목록 endpoint
- 치수, lazy loading, 또는 반응형 크기 없는 이미지
- 리뷰 없이 증가하는 번들 크기
- production에 성능 모니터링 없음
- 곳곳에 `React.memo`와 `useMemo` (남용은 미사용만큼 나쁨)

## Verification

성능 관련 변경 후:

- [ ] before와 after 측정이 존재 (구체적 숫자)
- [ ] 특정 병목이 식별되고 해결됨
- [ ] Core Web Vitals가 "Good" 임계값 내
- [ ] 번들 크기가 크게 증가하지 않음
- [ ] 새 데이터 fetch 코드에 N+1 쿼리 없음
- [ ] CI에서 performance budget 통과 (구성된 경우)
- [ ] 기존 테스트가 여전히 통과 (최적화가 동작을 깨지 않음)
