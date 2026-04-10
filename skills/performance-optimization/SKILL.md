---
name: performance-optimization
description: 애플리케이션 performance를 최적화합니다. performance requirements가 존재하는 경우, performance 회귀가 의심되는 경우 또는 Core Web Vitals 또는 로드 시간 개선이 필요한 경우에 사용하세요. 프로파일링을 통해 수정이 필요한 병목 현상이 드러날 때 사용합니다.
---

# Performance 최적화

## 개요

최적화하기 전에 측정하세요. 측정 없는 Performance 작업은 추측입니다. 추측은 중요한 사항을 개선하지 않고 복잡성을 추가하는 조기 최적화로 이어집니다. 먼저 프로필을 작성하고 실제 병목 현상을 식별하고 수정한 후 다시 측정하세요. 중요한 측정값만 최적화하세요.

## 사용 시기

- Performance requirements가 사양에 존재합니다(로드 시간 예산, 응답 시간 SLAs).
- 사용자 또는 모니터링 report 느린 동작
- Core Web Vitals 점수가 임계값보다 낮습니다.
- 변경으로 인해 회귀가 발생한 것으로 의심됩니다.
- 대규모 데이터 세트 또는 높은 트래픽을 처리하는 Building 기능

**사용하지 말아야 할 때:** 문제의 증거가 있기 전에는 최적화하지 마십시오. 조기 최적화는 얻는 performance보다 비용이 더 많이 드는 복잡성을 추가합니다.

## Core Web Vitals 대상

| 미터법 | 좋음 | 개선 필요 | 가난한 |
|--------|------|-------------------|------|
| **LCP**(콘텐츠가 포함된 최대 페인트) | 2.5초 이하 | 4.0초 이하 | > 4.0초 |
| **INP**(다음 페인트와의 상호 작용) | 200ms 이하 | ≤ 500ms | > 500ms |
| **CLS**(누적 레이아웃 이동) | ≤ 0.1 | ≤ 0.25 | > 0.25 |

## 최적화 Workflow

```
1. MEASURE  → Establish baseline with real data
2. IDENTIFY → Find the actual bottleneck (not assumed)
3. FIX      → Address the specific bottleneck
4. VERIFY   → Measure again, confirm improvement
5. GUARD    → Add monitoring or tests to prevent regression
```

### 1단계: 측정

두 가지 보완적인 접근 방식 — 둘 다 사용:

- **합성(Lighthouse, DevTools Performance 탭):** 제어된 조건, 재현 가능. CI 회귀 감지 및 특정 문제 격리에 가장 적합합니다.
- **RUM (web-vitals 라이브러리, CrUX):** 실제 조건의 실제 사용자 데이터입니다. Required를 사용하여 수정 사항이 실제로 사용자 경험을 개선했는지 검증합니다.

**프런트엔드:**
```bash
# Synthetic: Lighthouse in Chrome DevTools (or CI)
# Chrome DevTools → Performance tab → Record
# Chrome DevTools MCP → Performance trace

# RUM: Web Vitals library in code
import { onLCP, onINP, onCLS } from 'web-vitals';

onLCP(console.log);
onINP(console.log);
onCLS(console.log);
```

**백엔드:**
```bash
# Response time logging
# Application Performance Monitoring (APM)
# Database query logging with timing

# Simple timing
console.time('db-query');
const result = await db.query(...);
console.timeEnd('db-query');
```

### 측정 시작 위치

증상을 사용하여 무엇을 먼저 측정할지 결정합니다.

```
What is slow?
├── First page load
│   ├── Large bundle? --> Measure bundle size, check code splitting
│   ├── Slow server response? --> Measure TTFB in DevTools Network waterfall
│   │   ├── DNS long? --> Add dns-prefetch / preconnect for known origins
│   │   ├── TCP/TLS long? --> Enable HTTP/2, check edge deployment, keep-alive
│   │   └── Waiting (server) long? --> Profile backend, check queries and caching
│   └── Render-blocking resources? --> Check network waterfall for CSS/JS blocking
├── Interaction feels sluggish
│   ├── UI freezes on click? --> Profile main thread, look for long tasks (>50ms)
│   ├── Form input lag? --> Check re-renders, controlled component overhead
│   └── Animation jank? --> Check layout thrashing, forced reflows
├── Page after navigation
│   ├── Data loading? --> Measure API response times, check for waterfalls
│   └── Client rendering? --> Profile component render time, check for N+1 fetches
└── Backend / API
    ├── Single endpoint slow? --> Profile database queries, check indexes
    ├── All endpoints slow? --> Check connection pool, memory, CPU
    └── Intermittent slowness? --> Check for lock contention, GC pauses, external deps
```

### 2단계: 병목 현상 식별

범주별 일반적인 병목 현상:

**프런트엔드:**

| 증상 | 가능한 원인 | 조사 |
|---------|-------------|---------------|
| 느린 LCP | 큰 이미지, render-blocking 리소스, 느린 서버 | 네트워크 워터폴, 이미지 크기 확인 |
| 높음 CLS | 치수가 없는 이미지, late-loading 콘텐츠, 글꼴 이동 | 레이아웃 변경 속성 확인 |
| 불쌍한 INP | 메인 스레드의 무거운 JavaScript, 대규모 DOM 업데이트 | Performance 추적에서 긴 작업 확인 |
| 느린 초기 로드 | 대규모 번들, 많은 네트워크 요청 | 번들 크기, 코드 분할 확인 |

**백엔드:**

| 증상 | 가능한 원인 | 조사 |
|---------|-------------|---------------|
| 느린 API 응답 | N+1 쿼리, 누락된 인덱스, 최적화되지 않은 쿼리 | 데이터베이스 쿼리 로그 확인 |
| 메모리 성장 | 유출된 참조, 무제한 캐시, 대규모 페이로드 | 힙 스냅샷 분석 |
| CPU 스파이크 | 동기식 무거운 계산, 정규식 역추적 | CPU 프로파일링 |
| 높은 대기 시간 | 캐싱 누락, 중복 계산, 네트워크 홉 | 스택을 통해 요청 추적 |

### 3단계: 공통 Anti-Patterns 수정

#### N+1 쿼리(백엔드)

```typescript
// BAD: N+1 — one query per task for the owner
const tasks = await db.tasks.findMany();
for (const task of tasks) {
  task.owner = await db.users.findUnique({ where: { id: task.ownerId } });
}

// GOOD: Single query with join/include
const tasks = await db.tasks.findMany({
  include: { owner: true },
});
```

#### 무제한 데이터 가져오기

```typescript
// BAD: Fetching all records
const allTasks = await db.tasks.findMany();

// GOOD: Paginated with limits
const tasks = await db.tasks.findMany({
  take: 20,
  skip: (page - 1) * 20,
  orderBy: { createdAt: 'desc' },
});
```

#### 이미지 최적화 누락(프런트엔드)

```html
<!-- BAD: No dimensions, no format optimization -->
<img src="/hero.jpg" />

<!-- GOOD: Hero / LCP image — art direction + resolution switching, high priority -->
<!--
  Two techniques combined:
  - Art direction (media): different crop/composition per breakpoint
  - Resolution switching (srcset + sizes): right file size per screen density
-->
<picture>
  <!-- Mobile: portrait crop (8:10) -->
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
  <!-- Desktop: landscape crop (2:1) -->
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

<!-- GOOD: Below-the-fold image — lazy loaded + async decoding -->
<img
  src="/content.webp"
  width="800"
  height="400"
  loading="lazy"
  decoding="async"
  alt="Content image description"
/>
```

#### 불필요한 재렌더링(React)

```tsx
// BAD: Creates new object on every render, causing children to re-render
function TaskList() {
  return <TaskFilters options={{ sortBy: 'date', order: 'desc' }} />;
}

// GOOD: Stable reference
const DEFAULT_OPTIONS = { sortBy: 'date', order: 'desc' } as const;
function TaskList() {
  return <TaskFilters options={DEFAULT_OPTIONS} />;
}

// Use React.memo for expensive components
const TaskItem = React.memo(function TaskItem({ task }: Props) {
  return <div>{/* expensive render */}</div>;
});

// Use useMemo for expensive computations
function TaskStats({ tasks }: Props) {
  const stats = useMemo(() => calculateStats(tasks), [tasks]);
  return <div>{stats.completed} / {stats.total}</div>;
}
```

#### 대형 번들 크기

```typescript
// Modern bundlers (Vite, webpack 5+) handle named imports with tree-shaking automatically,
// provided the dependency ships ESM and is marked `sideEffects: false` in package.json.
// Profile before changing import styles — the real gains come from splitting and lazy loading.

// GOOD: Dynamic import for heavy, rarely-used features
const ChartLibrary = lazy(() => import('./ChartLibrary'));

// GOOD: Route-level code splitting wrapped in Suspense
const SettingsPage = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <SettingsPage />
    </Suspense>
  );
}
```

#### 캐싱 누락(백엔드)

```typescript
// Cache frequently-read, rarely-changed data
const CACHE_TTL = 5 * 60 * 1000; // 5 minutes
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

// HTTP caching headers for static assets
app.use('/static', express.static('public', {
  maxAge: '1y',           // Cache for 1 year
  immutable: true,        // Never revalidate (use content hashing in filenames)
}));

// Cache-Control for API responses
res.set('Cache-Control', 'public, max-age=300'); // 5 minutes
```

## Performance 예산

예산을 설정하고 시행합니다.

```
JavaScript bundle: < 200KB gzipped (initial load)
CSS: < 50KB gzipped
Images: < 200KB per image (above the fold)
Fonts: < 100KB total
API response time: < 200ms (p95)
Time to Interactive: < 3.5s on 4G
Lighthouse Performance score: ≥ 90
```

**CI에서 시행:**
```bash
# Bundle size check
npx bundlesize --config bundlesize.config.json

# Lighthouse CI
npx lhci autorun
```

## 참고 항목

자세한 performance 체크리스트, 최적화 명령 및 anti-pattern 참조는 `references/performance-checklist.md`를 참조하세요.


## 일반적인 합리화

| 합리화 | 현실 |
|---|---|
| "나중에 최적화하겠습니다" | Performance 부채 화합물. 이제 명백한 anti-patterns를 수정하고 micro-optimizations를 연기하세요. |
| "내 컴퓨터에서는 빠릅니다." | 당신의 기계는 사용자의 것이 아닙니다. 대표적인 하드웨어 및 네트워크에 대한 프로필입니다. |
| "이러한 최적화는 명백합니다" | 측정하지 않으면 알 수 없습니다. 먼저 프로필을 작성하세요. |
| "사용자는 100ms 동안 눈치 채지 못할 것입니다" | 연구에 따르면 100ms 지연이 전환율에 영향을 미치는 것으로 나타났습니다. 사용자는 생각보다 더 많은 것을 알아차립니다. |
| "프레임워크는 performance를 처리합니다." | 프레임워크는 일부 문제를 방지하지만 N+1 쿼리 또는 크기가 큰 번들을 수정할 수는 없습니다. |

## 위험 신호

- 정당화하기 위해 데이터를 프로파일링하지 않고 최적화
- 데이터 가져오기의 N+1 쿼리 패턴
- 페이지 매김 없이 엔드포인트 나열
- 크기, 지연 로딩 또는 반응형 크기가 없는 이미지
- 검토 없이 번들 크기가 증가함
- 프로덕션에서 performance 모니터링이 없습니다.
- 어디에서나 `React.memo` 및 `useMemo`(과도한 사용은 과소사용만큼 나쁨)

## 확인

performance-related 변경 후:

- [ ] 측정 전과 후가 존재함(구체적인 숫자)
- [ ] 특정 병목 현상이 식별되고 해결됩니다.
- [ ] Core Web Vitals는 "양호" 임계값 내에 있습니다.
- [ ] 번들 크기가 크게 증가하지 않았습니다.
- [ ] 새로운 데이터 가져오기 코드에는 N+1 쿼리가 없습니다.
- [ ] Performance 예산이 CI에 전달됩니다(구성된 경우).
- [ ] 기존 테스트는 여전히 통과합니다(최적화가 동작을 중단하지 않았습니다).
