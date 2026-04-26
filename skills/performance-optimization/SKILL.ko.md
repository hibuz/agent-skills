---
name: performance-optimization
description: 애플리케이션 성능을 최적화합니다. 성능 요구 사항이 있거나, 성능 저하가 의심될 때, 또는 Core Web Vitals나 로딩 시간 개선이 필요할 때 사용합니다. 프로파일링을 통해 수정이 필요한 병목 지점이 발견되었을 때 사용합니다.
---

# 성능 최적화 (Performance Optimization)

## 개요 (Overview)

최적화하기 전에 먼저 측정하세요. 측정 없는 성능 작업은 추측일 뿐입니다. 그리고 추측은 중요한 부분을 개선하지 못한 채 복잡성만 더하는 조기 최적화(premature optimization)로 이어집니다. 먼저 프로파일링(profile)하고, 실제 병목 지점(bottleneck)을 식별하고, 그것을 수정한 다음 다시 측정하세요. 측정이 가치 있다고 증명한 것만 최적화하세요.

## 사용 시점

- 스펙에 성능 요구 사항(로딩 시간 예산, 응답 시간 SLA 등)이 있을 때
- 사용자나 모니터링 시스템에서 속도 저하를 보고할 때
- Core Web Vitals 점수가 임계값 미만일 때
- 변경 사항이 성능 저하를 유발했을 것으로 의심될 때
- 대량의 데이터나 높은 트래픽을 처리하는 기능을 구축할 때

**사용하지 않는 경우:** 문제의 증거가 없는데 최적화하지 마세요. 조기 최적화는 성능 이득보다 더 큰 유지보수 비용을 초래하는 복잡성을 추가합니다.

## Core Web Vitals 목표

| 지표 | 좋음 (Good) | 개선 필요 | 나쁨 (Poor) |
|--------|------|-------------------|------|
| **LCP** (Largest Contentful Paint) | ≤ 2.5초 | ≤ 4.0초 | > 4.0초 |
| **INP** (Interaction to Next Paint) | ≤ 200ms | ≤ 500ms | > 500ms |
| **CLS** (Cumulative Layout Shift) | ≤ 0.1 | ≤ 0.25 | > 0.25 |

## 최적화 워크플로우

```
1. 측정 (MEASURE)  → 실제 데이터로 기준점(baseline) 수립
2. 식별 (IDENTIFY) → 추측이 아닌 실제 병목 지점 발견
3. 수정 (FIX)      → 특정 병목 지점 해결
4. 검증 (VERIFY)   → 다시 측정하여 개선 사항 확인
5. 방어 (GUARD)    → 성능 저하 방지를 위한 모니터링 또는 테스트 추가
```

### 1단계: 측정 (Measure)

보완적인 두 가지 접근 방식을 모두 사용하세요:

- **합성 측정 (Synthetic - Lighthouse, DevTools Performance 탭):** 통제된 환경, 재현 가능함. CI에서의 회귀 테스트 감지 및 특정 이슈 격리에 가장 적합합니다.
- **실제 사용자 측정 (RUM - web-vitals 라이브러리, CrUX):** 실제 환경에서의 실제 사용자 데이터. 수정 사항이 실제로 사용자 경험을 개선했는지 검증하는 데 필수적입니다.

**프론트엔드:**
```bash
# 합성 측정: Chrome DevTools(또는 CI)의 Lighthouse
# Chrome DevTools → Performance 탭 → Record
# Chrome DevTools MCP → Performance trace

# RUM: 코드 내 web-vitals 라이브러리 사용
import { onLCP, onINP, onCLS } from 'web-vitals';

onLCP(console.log);
onINP(console.log);
onCLS(console.log);
```

**백엔드:**
```bash
# 응답 시간 로깅
# 애플리케이션 성능 모니터링 (APM) 도구 사용
# 실행 시간 포함 데이터베이스 쿼리 로그

# 간단한 시간 측정
console.time('db-query');
const result = await db.query(...);
console.timeEnd('db-query');
```

### 어디서부터 측정할 것인가

증상에 따라 무엇을 먼저 측정할지 결정하세요:

```
무엇이 느린가요?
├── 첫 페이지 로드
│   ├── 거대한 번들? --> 번들 크기 측정, 코드 분할(code splitting) 확인
│   ├── 느린 서버 응답? --> DevTools Network waterfall에서 TTFB 측정
│   │   ├── DNS 지연? --> 알려진 오리진에 dns-prefetch / preconnect 추가
│   │   ├── TCP/TLS 지연? --> HTTP/2 활성화, 에지(edge) 배포, keep-alive 확인
│   │   └── 서버 대기(waiting) 지연? --> 백엔드 프로파일링, 쿼리 및 캐싱 확인
│   └── 렌더링 차단 리소스? --> CSS/JS 차단 여부를 Network waterfall에서 확인
├── 상호작용이 굼뜨게 느껴짐
│   ├── 클릭 시 UI 얼어붙음? --> 메인 스레드 프로파일링, 긴 작업(long tasks, >50ms) 확인
│   ├── 폼 입력 지연? --> 리렌더링 확인, 제어 컴포넌트(controlled component) 오버헤드 확인
│   └── 애니메이션 끊김(jank)? --> 레이아웃 스래싱(layout thrashing), 강제 리플로우(forced reflows) 확인
├── 페이지 이동 후
│   ├── 데이터 로딩 중? --> API 응답 시간 측정, 폭포수(waterfalls) 확인
│   └── 클라이언트 렌더링 중? --> 컴포넌트 렌더링 시간 프로파일링, N+1 페칭 확인
└── 백엔드 / API
    ├── 단일 엔드포인트가 느림? --> 데이터베이스 쿼리 프로파일링, 인덱스 확인
    ├── 모든 엔드포인트가 느림? --> 커넥션 풀, 메모리, CPU 확인
    └── 간헐적 속도 저하? --> 락 경합(lock contention), GC 일시 중단, 외부 의존성 확인
```

### 2단계: 병목 지점 식별 (Identify)

카테고리별 일반적인 병목 지점:

**프론트엔드:**

| 증상 | 가능성 높은 원인 | 조사 방법 |
|---------|-------------|---------------|
| 느린 LCP | 거대한 이미지, 렌더링 차단 리소스, 느린 서버 | Network waterfall 확인, 이미지 크기 확인 |
| 높은 CLS | 크기 미지정 이미지, 늦게 로드되는 컨텐츠, 폰트 변화 | Layout shift attribution 확인 |
| 낮은 INP | 메인 스레드의 무거운 JS, 거대한 DOM 업데이트 | Performance trace에서 긴 작업(long tasks) 확인 |
| 느린 초기 로드 | 거대한 번들, 너무 많은 네트워크 요청 | 번들 크기 확인, 코드 분할 확인 |

**백엔드:**

| 증상 | 가능성 높은 원인 | 조사 방법 |
|---------|-------------|---------------|
| 느린 API 응답 | N+1 쿼리, 인덱스 누락, 최적화되지 않은 쿼리 | 데이터베이스 쿼리 로그 확인 |
| 메모리 증가 | 참조 누수, 무분별한 캐시, 거대한 페이로드 | 힙 스냅샷(Heap snapshot) 분석 |
| CPU 급증 | 동기식의 무거운 계산, 정규식 백트래킹 | CPU 프로파일링 |
| 높은 지연 시간 | 캐싱 누락, 중복 계산, 네트워크 홉(hops) | 스택 전체의 요청 추적(trace) |

### 3단계: 일반적인 안티 패턴 수정

#### N+1 쿼리 (백엔드)

```typescript
// 나쁨: N+1 — 작업마다 소유자 정보를 위해 별도의 쿼리 실행
const tasks = await db.tasks.findMany();
for (const task of tasks) {
  task.owner = await db.users.findUnique({ where: { id: task.ownerId } });
}

// 좋음: join/include를 사용한 단일 쿼리
const tasks = await db.tasks.findMany({
  include: { owner: true },
});
```

#### 무분별한 데이터 페칭

```typescript
// 나쁨: 모든 레코드를 한꺼번에 가져옴
const allTasks = await db.tasks.findMany();

// 좋음: 제한된 크기로 페이지네이션 적용
const tasks = await db.tasks.findMany({
  take: 20,
  skip: (page - 1) * 20,
  orderBy: { createdAt: 'desc' },
});
```

#### 이미지 최적화 누락 (프론트엔드)

```html
<!-- 나쁨: 크기 미지정, 형식 최적화 없음 -->
<img src="/hero.jpg" />

<!-- 좋음: Hero / LCP 이미지 — 아트 디렉션 + 해상도 전환, 높은 우선순위 -->
<!--
  두 가지 기술 결합:
  - 아트 디렉션 (media): 브레이크포인트별 다른 크롭/구성
  - 해상도 전환 (srcset + sizes): 화면 밀도별 적절한 파일 크기
-->
<picture>
  <!-- 모바일: 세로 크롭 (8:10) -->
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
  <!-- 데스크탑: 가로 크롭 (2:1) -->
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
    alt="Hero 이미지 설명"
  />
</picture>

<!-- 좋음: 화면 하단 이미지 — 지연 로딩 + 비동기 디코딩 -->
<img
  src="/content.webp"
  width="800"
  height="400"
  loading="lazy"
  decoding="async"
  alt="컨텐츠 이미지 설명"
/>
```

#### 불필요한 리렌더링 (React)

```tsx
// 나쁨: 렌더링마다 새로운 객체를 생성하여 자식 컴포넌트의 리렌더링 유발
function TaskList() {
  return <TaskFilters options={{ sortBy: 'date', order: 'desc' }} />;
}

// 좋음: 안정적인 참조 유지
const DEFAULT_OPTIONS = { sortBy: 'date', order: 'desc' } as const;
function TaskList() {
  return <TaskFilters options={DEFAULT_OPTIONS} />;
}

// 비용이 많이 드는 컴포넌트에는 React.memo 사용
const TaskItem = React.memo(function TaskItem({ task }: Props) {
  return <div>{/* 무거운 렌더링 로직 */}</div>;
});

// 비용이 많이 드는 계산에는 useMemo 사용
function TaskStats({ tasks }: Props) {
  const stats = useMemo(() => calculateStats(tasks), [tasks]);
  return <div>{stats.completed} / {stats.total}</div>;
}
```

#### 거대한 번들 크기

```typescript
// 최신 번들러(Vite, webpack 5+ 등)는 패키지가 ESM을 지원하고 package.json에 `sideEffects: false`가 설정되어 있다면
// 트리 쉐이킹(tree-shaking)을 통해 명명된 임포트(named imports)를 자동으로 처리합니다.
// 임포트 스타일을 바꾸기 전에 먼저 프로파일링하세요 — 실제 이득은 코드 분할과 지연 로딩에서 옵니다.

// 좋음: 무겁고 드물게 사용되는 기능에 동적 임포트 사용
const ChartLibrary = lazy(() => import('./ChartLibrary'));

// 좋음: Suspense로 감싼 라우트 레벨의 코드 분할
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
// 자주 읽히고 드물게 변경되는 데이터 캐싱
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

// 정적 자산을 위한 HTTP 캐싱 헤더
app.use('/static', express.static('public', {
  maxAge: '1y',           // 1년 동안 캐싱
  immutable: true,        // 재검증하지 않음 (파일명에 컨텐츠 해시 사용 권장)
}));

// API 응답을 위한 Cache-Control
res.set('Cache-Control', 'public, max-age=300'); // 5분
```

## 성능 예산 (Performance Budget)

예산을 설정하고 강제하세요:

```
JavaScript 번들: < 200KB (gzipped, 초기 로드)
CSS: < 50KB (gzipped)
이미지: 이미지당 < 200KB (화면 상단 기준)
폰트: 전체 < 100KB
API 응답 시간: < 200ms (p95)
상호작용 시작 시간 (Time to Interactive): < 3.5초 (4G 기준)
Lighthouse 성능 점수: ≥ 90
```

**CI에서 강제하기:**
```bash
# 번들 크기 체크
npx bundlesize --config bundlesize.config.json

# Lighthouse CI
npx lhci autorun
```

## 더 보기

상세한 성능 체크리스트, 최적화 명령어 및 안티 패턴 참조는 `references/performance-checklist.ko.md`를 참조하세요.


## 일반적인 합리화

| 합리화 | 실제 |
|---|---|
| "나중에 최적화할게요" | 성능 부채는 누적됩니다. 뻔한 안티 패턴은 지금 수정하고, 미세 최적화는 나중으로 미루세요. |
| "내 컴퓨터에서는 빨라요" | 당신의 컴퓨터는 사용자의 컴퓨터가 아닙니다. 대표적인 하드웨어와 네트워크 환경에서 프로파일링하세요. |
| "이 최적화는 뻔해요" | 측정하지 않았다면 모르는 것입니다. 먼저 프로파일링하세요. |
| "사용자는 100ms 차이를 못 느껴요" | 연구에 따르면 100ms 지연이 전환율에 영향을 미칩니다. 사용자는 생각보다 더 예민합니다. |
| "프레임워크가 성능을 처리해 줘요" | 프레임워크가 일부 문제를 방지해주지만, N+1 쿼리나 거대한 번들까지 해결해주지는 못합니다. |

## 레드 플래그 (Red Flags)

- 정당화할 프로파일링 데이터 없는 최적화
- 데이터 페칭에서의 N+1 쿼리 패턴
- 페이지네이션 없는 목록 엔드포인트
- 크기 미지정, 지연 로딩 누락, 또는 반응형 크기가 아닌 이미지
- 리뷰 없이 늘어나는 번들 크기
- 프로덕션 환경의 성능 모니터링 부재
- 모든 곳에 남발된 `React.memo` 및 `useMemo` (남용은 안 쓰는 것만큼 나쁠 수 있습니다)

## 검증 (Verification)

성능 관련 변경 후 다음을 확인하세요:

- [ ] 변경 전후의 측정 수치가 존재함 (구체적인 숫자)
- [ ] 특정 병목 지점이 식별되고 해결됨
- [ ] Core Web Vitals가 "좋음" 임계값 내에 있음
- [ ] 번들 크기가 크게 증가하지 않음
- [ ] 새로운 데이터 페칭 코드에 N+1 쿼리가 없음
- [ ] CI의 성능 예산 테스트를 통과함 (설정된 경우)
- [ ] 기존 테스트가 여전히 통과함 (최적화가 동작을 깨뜨리지 않았음)
