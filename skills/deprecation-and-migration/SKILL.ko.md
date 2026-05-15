---
name: deprecation-and-migration
description: deprecation과 migration을 관리. 오래된 시스템, API, 또는 기능을 제거할 때 사용. 사용자를 한 구현에서 다른 구현으로 migration할 때 사용. 기존 코드를 유지할지 종료할지 결정할 때 사용.
---

# Deprecation and Migration

## Overview

코드는 자산이 아니라 부채입니다. 모든 코드 줄은 지속적 유지보수 비용을 가집니다 — 고칠 버그, 업데이트할 의존성, 적용할 보안 패치, 온보딩할 새 엔지니어. deprecation은 더 이상 제 몫을 못 하는 코드를 제거하는 규율이고, migration은 사용자를 안전하게 옛것에서 새것으로 옮기는 프로세스입니다.

대부분의 엔지니어링 조직은 만드는 데 능합니다. 제거에 능한 곳은 거의 없습니다. 이 skill은 그 간극을 다룹니다.

## When to Use

- 오래된 시스템, API, 또는 라이브러리를 새것으로 교체
- 더 이상 필요 없는 기능 종료
- 중복 구현 통합
- 아무도 소유하지 않지만 모두가 의존하는 죽은 코드 제거
- 새 시스템의 라이프사이클 계획 (deprecation 계획은 설계 시점에 시작)
- 레거시 시스템을 유지할지 migration에 투자할지 결정

## Core Principles

### 코드는 부채

모든 코드 줄은 지속적 비용을 가집니다: 테스트, 문서, 보안 패치, 의존성 업데이트, 그리고 근처에서 작업하는 누구에게나 정신적 부담이 필요합니다. 코드의 가치는 그것이 제공하는 기능이지, 코드 자체가 아닙니다. 같은 기능을 더 적은 코드, 더 적은 복잡도, 또는 더 나은 추상화로 제공할 수 있다면 — 옛 코드는 사라져야 합니다.

### Hyrum's Law가 제거를 어렵게 만든다

사용자가 충분하면, 모든 관찰 가능한 동작이 의존됩니다 — 버그, 타이밍 quirk, 문서화되지 않은 부수효과를 포함. 이것이 deprecation이 단순한 공지가 아니라 적극적 migration을 요구하는 이유입니다. 사용자는 교체물이 복제하지 않는 동작에 의존할 때 "그냥 전환"할 수 없습니다.

### Deprecation 계획은 설계 시점에 시작

새것을 만들 때, 물어보세요: "이걸 3년 후에 어떻게 제거할까?" 깨끗한 인터페이스, feature flag, 최소 표면적으로 설계된 시스템은 구현 세부사항을 곳곳에 누출하는 시스템보다 deprecate하기 쉽습니다.

## Deprecation 결정

무언가를 deprecate하기 전에, 이 질문들에 답하세요:

```
1. 이 시스템이 여전히 고유한 가치를 제공하는가?
   → 예라면, 유지. 아니라면, 진행.

2. 얼마나 많은 사용자/소비자가 의존하는가?
   → migration 범위를 정량화.

3. 교체물이 존재하는가?
   → 아니라면, 교체물을 먼저 만들기. 대안 없이 deprecate하지 말 것.

4. 각 소비자의 migration 비용은?
   → 사소하게 자동화되면, 하기. 수동이고 고노력이면, 유지보수 비용과 저울질.

5. deprecate하지 않는 것의 지속적 유지보수 비용은?
   → 보안 위험, 엔지니어 시간, 복잡도의 기회비용.
```

## 강제 vs 권고 Deprecation

| 유형 | 사용 시점 | 메커니즘 |
|------|-------------|-----------|
| **권고(Advisory)** | migration이 선택적, 옛 시스템이 안정적 | 경고, 문서, nudge. 사용자가 자신의 일정으로 migration. |
| **강제(Compulsory)** | 옛 시스템에 보안 이슈, 진전을 막음, 또는 유지보수 비용이 지속 불가 | 하드 데드라인. 옛 시스템이 날짜 X까지 제거됨. migration 도구 제공. |

**기본은 권고.** 유지보수 비용이나 위험이 migration 강제를 정당화할 때만 강제를 사용하세요. 강제 deprecation은 migration 도구, 문서, 지원 제공을 요구합니다 — 데드라인만 공지할 수 없습니다.

## Migration 프로세스

### Step 1: 교체물 만들기

동작하는 대안 없이 deprecate하지 마세요. 교체물은:

- 옛 시스템의 모든 중요한 사용 사례를 커버해야 함
- 문서와 migration 가이드가 있어야 함
- production에서 검증되어야 함 ("이론적으로 더 낫다"가 아니라)

### Step 2: 공지와 문서화

```markdown
## Deprecation Notice: OldService

**Status:** Deprecated as of 2025-03-01
**Replacement:** NewService (see migration guide below)
**Removal date:** Advisory — no hard deadline yet
**Reason:** OldService requires manual scaling and lacks observability.
            NewService handles both automatically.

### Migration Guide
1. Replace `import { client } from 'old-service'` with `import { client } from 'new-service'`
2. Update configuration (see examples below)
3. Run the migration verification script: `npx migrate-check`
```

### Step 3: 점진적 Migration

소비자를 한 번에 모두가 아니라 하나씩 migration하세요. 각 소비자에 대해:

```
1. deprecate된 시스템과의 모든 접점 식별
2. 교체물을 사용하도록 업데이트
3. 동작이 일치하는지 검증 (테스트, integration 검사)
4. 옛 시스템에 대한 참조 제거
5. 회귀 없음 확인
```

**Churn 규칙:** deprecate되는 인프라를 소유한다면, 사용자를 migration할 책임이 당신에게 있습니다 — 또는 migration이 필요 없는 하위 호환 업데이트 제공. deprecation을 공지하고 사용자가 알아서 하도록 두지 마세요.

### Step 4: 옛 시스템 제거

모든 소비자가 migration한 후에만:

```
1. 활성 사용 0 확인 (메트릭, 로그, 의존성 분석)
2. 코드 제거
3. 관련 테스트, 문서, 설정 제거
4. deprecation 공지 제거
5. 축하 — 코드 제거는 성취입니다
```

## Migration 패턴

### Strangler 패턴

옛 시스템과 새 시스템을 병렬로 실행. 트래픽을 옛것에서 새것으로 점진적으로 라우팅. 옛 시스템이 트래픽의 0%를 처리하면 제거.

```
Phase 1: 새 시스템 0% 처리, 옛것 100%
Phase 2: 새 시스템 10% 처리 (canary)
Phase 3: 새 시스템 50% 처리
Phase 4: 새 시스템 100% 처리, 옛 시스템 유휴
Phase 5: 옛 시스템 제거
```

### Adapter 패턴

옛 인터페이스의 호출을 새 구현으로 번역하는 adapter를 만드세요. 소비자는 옛 인터페이스를 계속 쓰고, 당신은 백엔드를 migration합니다.

```typescript
// Adapter: 옛 인터페이스, 새 구현
class LegacyTaskService implements OldTaskAPI {
  constructor(private newService: NewTaskService) {}

  // 옛 method 시그니처, 새 구현에 위임
  getTask(id: number): OldTask {
    const task = this.newService.findById(String(id));
    return this.toOldFormat(task);
  }
}
```

### Feature Flag Migration

feature flag를 사용해 소비자를 옛 시스템에서 새 시스템으로 하나씩 전환:

```typescript
function getTaskService(userId: string): TaskService {
  if (featureFlags.isEnabled('new-task-service', { userId })) {
    return new NewTaskService();
  }
  return new LegacyTaskService();
}
```

## Zombie Code

좀비 코드는 아무도 소유하지 않지만 모두가 의존하는 코드입니다. 적극적으로 유지보수되지 않고, 명확한 소유자가 없고, 보안 취약점과 호환성 이슈를 누적합니다. 징후:

- 6개월 이상 commit 없지만 활성 소비자 존재
- 할당된 maintainer나 팀 없음
- 아무도 고치지 않는 실패하는 테스트
- 아무도 업데이트하지 않는 알려진 취약점 있는 의존성
- 더 이상 존재하지 않는 시스템을 참조하는 문서

**대응:** 소유자를 할당하고 제대로 유지보수하거나, 구체적 migration 계획과 함께 deprecate하세요. 좀비 코드는 limbo에 머물 수 없습니다 — 투자받거나 제거되거나입니다.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "여전히 동작하는데 왜 제거해?" | 아무도 유지보수하지 않는 동작 코드는 보안 부채와 복잡도를 누적합니다. 유지보수 비용은 조용히 자랍니다. |
| "누군가 나중에 필요할 수도" | 나중에 필요하면 다시 만들 수 있습니다. "혹시 몰라" 사용 안 된 코드를 유지하는 것이 다시 만드는 것보다 비쌉니다. |
| "migration이 너무 비싸" | migration 비용을 2-3년에 걸친 지속적 유지보수 비용과 비교하세요. migration이 보통 장기적으로 더 쌉니다. |
| "새 시스템 끝낸 후 deprecate할게" | deprecation 계획은 설계 시점에 시작합니다. 새 시스템이 끝날 때쯤이면 새 우선순위가 생깁니다. 지금 계획하세요. |
| "사용자가 알아서 migration할 거야" | 안 합니다. 도구, 문서, 인센티브를 제공하거나 — migration을 직접 하세요 (Churn 규칙). |
| "두 시스템을 무기한 유지할 수 있어" | 같은 일을 하는 두 시스템은 두 배의 유지보수, 테스트, 문서, 온보딩 비용입니다. |

## Red Flags

- 교체물이 없는 deprecate된 시스템
- migration 도구나 문서 없는 deprecation 공지
- 진전 없이 수년간 권고였던 "soft" deprecation
- 소유자 없고 활성 소비자 있는 좀비 코드
- deprecate된 시스템에 새 기능 추가 (대신 교체물에 투자)
- 현재 사용을 측정하지 않은 deprecation
- 활성 소비자 0 확인 없이 코드 제거

## Verification

deprecation 완료 후:

- [ ] 교체물이 production 검증되었고 모든 중요한 사용 사례를 커버
- [ ] 구체적 단계와 예시를 갖춘 migration 가이드 존재
- [ ] 모든 활성 소비자가 migration됨 (메트릭/로그로 검증)
- [ ] 옛 코드, 테스트, 문서, 설정이 완전히 제거됨
- [ ] 코드베이스에 deprecate된 시스템에 대한 참조가 남지 않음
- [ ] deprecation 공지가 제거됨 (목적을 다함)
