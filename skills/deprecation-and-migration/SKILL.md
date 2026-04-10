---
name: deprecation-and-migration
description: 지원 중단 및 마이그레이션을 관리합니다. 기존 시스템, APIs 또는 기능을 제거할 때 사용합니다. 한 구현에서 다른 구현으로 사용자를 마이그레이션할 때 사용합니다. 기존 코드를 유지할지, 종료할지 결정할 때 사용합니다.
---

# 지원 중단 및 마이그레이션

## 개요

코드는 자산이 아니라 부채입니다. 모든 코드 줄에는 수정해야 할 버그, 업데이트해야 할 종속성, 적용해야 할 보안 패치, 온보딩해야 할 새로운 엔지니어 등 지속적인 유지 관리 비용이 있습니다. 지원 중단은 더 이상 유지되지 않는 코드를 제거하는 규율이며, 마이그레이션은 사용자를 이전 버전에서 새 코드로 안전하게 이동하는 프로세스입니다.

대부분의 엔지니어링 조직은 building 작업에 능숙합니다. 그것들을 제거하는 데 능숙한 사람은 거의 없습니다. 이 skill는 이러한 격차를 해소합니다.

## 사용 시기

- 기존 시스템, API 또는 라이브러리를 새 시스템으로 교체
- 더 이상 필요하지 않은 기능을 종료합니다.
- 중복 구현 통합
- 아무도 소유하지 않지만 모두가 의존하는 데드 코드 제거
- 새로운 시스템의 수명주기 계획(지원 중단 계획은 설계 시점에 시작됨)
- 레거시 시스템을 유지할 것인지 마이그레이션에 투자할 것인지 결정

## 핵심 원칙

### 코드는 책임이다

모든 코드 줄에는 지속적인 비용이 발생합니다. 테스트, 문서화, 보안 패치, 종속성 업데이트 및 근처에서 작업하는 모든 사람을 위한 정신적 오버헤드가 필요합니다. 코드의 가치는 코드 자체가 아니라 코드가 제공하는 기능입니다. 더 적은 코드, 더 적은 복잡성 또는 더 나은 추상화로 동일한 기능을 제공할 수 있다면 이전 코드는 사라져야 합니다.

### Hyrum's Law로 인해 제거가 어려워짐

사용자가 충분하면 버그, quirks 타이밍, 문서화되지 않은 부작용을 포함하여 관찰 가능한 모든 동작이 이에 따라 달라집니다. 이것이 바로 발표뿐만 아니라 지원 중단이 활성 마이그레이션을 요구하는 이유입니다. 사용자는 교체가 복제되지 않는 동작에 의존하는 경우 "단순히 전환"할 수 없습니다.

### 지원 중단 계획은 디자인 타임부터 시작됩니다.

build가 새로운 것을 발견할 때 "3년 안에 이것을 어떻게 제거할 것인가?"라고 질문하십시오. 깔끔한 인터페이스, feature flags 및 최소한의 표면적을 사용하여 설계된 시스템은 구현 세부 정보가 어디에서나 유출되는 시스템보다 더 이상 사용되지 않습니다.

## 지원 중단 결정

더 이상 사용되지 않는 항목을 사용하기 전에 다음 질문에 답하세요.

```
1. Does this system still provide unique value?
   → If yes, maintain it. If no, proceed.

2. How many users/consumers depend on it?
   → Quantify the migration scope.

3. Does a replacement exist?
   → If no, build the replacement first. Don't deprecate without an alternative.

4. What's the migration cost for each consumer?
   → If trivially automated, do it. If manual and high-effort, weigh against maintenance cost.

5. What's the ongoing maintenance cost of NOT deprecating?
   → Security risk, engineer time, opportunity cost of complexity.
```

## 필수 및 권고 지원 중단

| 유형 | 사용 시기 | 메커니즘 |
|------|-------------|-----------|
| **권고** | 마이그레이션은 선택 사항이며 기존 시스템은 안정적입니다 | 경고, 문서화, 넛지. 사용자는 자신의 타임라인에 따라 마이그레이션합니다. |
| **필수** | 오래된 시스템에는 보안 문제가 있거나, 진행이 차단되거나, 유지 관리 비용이 지속 불가능합니다 | 마감 기한이 빡빡합니다. 이전 시스템은 X 날짜까지 제거됩니다. 마이그레이션 도구를 제공합니다. |

**기본값은 권고입니다.** 유지 관리 비용이나 위험으로 인해 강제 마이그레이션을 정당화할 경우에만 필수를 사용하십시오. 마이그레이션 도구, 문서 및 지원을 제공하는 필수 지원 중단 requires — 마감일만 발표할 수는 없습니다.

## 마이그레이션 프로세스

### 1단계: Build 교체

효과적인 대안 없이는 더 이상 사용하지 마십시오. 교체품은 다음을 충족해야 합니다.

- 기존 시스템의 중요한 사용 사례를 모두 다룹니다.
- 문서화 및 마이그레이션 guides 보유
- 프로덕션에서 입증되어야 함(단지 "이론적으로 더 나은" 것이 아님)

### 2단계: 발표 및 문서화

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

### 3단계: 증분 마이그레이션

소비자를 한 번에 모두 마이그레이션하지 않고 한 번에 하나씩 마이그레이션합니다. 각 소비자에 대해:

```
1. Identify all touchpoints with the deprecated system
2. Update to use the replacement
3. Verify behavior matches (tests, integration checks)
4. Remove references to the old system
5. Confirm no regressions
```

** 이탈 규칙:** 더 이상 사용되지 않는 인프라를 소유한 경우 사용자를 마이그레이션하거나 마이그레이션이 필요하지 않은 backward-compatible 업데이트를 제공할 책임이 있습니다. 지원 중단을 알리거나 사용자가 이를 알아내도록 두지 마세요.

### 4단계: 기존 시스템 제거

모든 소비자가 마이그레이션된 후에만:

```
1. Verify zero active usage (metrics, logs, dependency analysis)
2. Remove the code
3. Remove associated tests, documentation, and configuration
4. Remove the deprecation notices
5. Celebrate — removing code is an achievement
```

## 마이그레이션 패턴

### 교살자 패턴

기존 시스템과 새 시스템을 병렬로 실행하세요. 기존 항목에서 새 항목으로 점진적으로 트래픽을 라우팅합니다. 기존 시스템이 트래픽의 0%를 처리하면 이를 제거합니다.

```
Phase 1: New system handles 0%, old handles 100%
Phase 2: New system handles 10% (canary)
Phase 3: New system handles 50%
Phase 4: New system handles 100%, old system idle
Phase 5: Remove old system
```

### 어댑터 패턴

이전 인터페이스의 호출을 새 구현으로 변환하는 어댑터를 만듭니다. 백엔드를 마이그레이션하는 동안 소비자는 이전 인터페이스를 계속 사용합니다.

```typescript
// Adapter: old interface, new implementation
class LegacyTaskService implements OldTaskAPI {
  constructor(private newService: NewTaskService) {}

  // Old method signature, delegates to new implementation
  getTask(id: number): OldTask {
    const task = this.newService.findById(String(id));
    return this.toOldFormat(task);
  }
}
```

### Feature Flag 마이그레이션

소비자를 이전 시스템에서 새 시스템으로 한 번에 하나씩 전환하려면 feature flags를 사용하십시오.

```typescript
function getTaskService(userId: string): TaskService {
  if (featureFlags.isEnabled('new-task-service', { userId })) {
    return new NewTaskService();
  }
  return new LegacyTaskService();
}
```

## 좀비 코드

좀비 코드는 누구도 소유하지 않지만 모두가 의존하는 코드입니다. 적극적으로 유지관리되지 않고, 명확한 소유자가 없으며, 보안 취약점과 호환성 문제가 누적됩니다. 징후:

- 6개월 이상 커밋이 없지만 활성 소비자가 존재함
- 지정된 관리자나 팀이 없습니다.
- 아무도 고치지 않는 실패한 테스트
- 아무도 업데이트하지 않는 알려진 취약점이 있는 종속성
- 더 이상 존재하지 않는 시스템을 참조하는 문서

**답변:** 소유자를 할당하고 적절하게 유지관리하거나 구체적인 마이그레이션 계획을 통해 사용을 중단하세요. 좀비 코드는 림보에 머물 수 없습니다. 투자를 받거나 제거됩니다.

## 일반적인 합리화

| 합리화 | 현실 |
|---|---|
| "여전히 작동하는데 왜 제거하나요?" | 아무도 유지 관리하지 않는 작업 코드는 보안 부채와 복잡성을 축적합니다. 유지관리 비용은 소리 없이 증가합니다. |
| "나중에 필요할 수도 있습니다." | 나중에 필요하다면 rebuilt일 수 있습니다. 사용하지 않는 코드를 "만약의 경우"에 보관하는 것은 rebuilding보다 비용이 더 많이 듭니다. |
| "마이그레이션 비용이 너무 많이 듭니다." | 마이그레이션 비용과 2~3년간의 지속적인 유지 관리 비용을 비교해 보세요. 마이그레이션은 일반적으로 long-term보다 저렴합니다. |
| "새 시스템이 완성되면 더 이상 사용하지 않을 예정입니다." | 지원 중단 계획은 디자인 타임부터 시작됩니다. 새로운 시스템이 완성될 때쯤에는 새로운 우선순위를 갖게 될 것입니다. 지금 계획하세요. |
| "사용자가 스스로 마이그레이션합니다" | 그렇지 않습니다. 도구, 문서 및 인센티브를 제공하거나 직접 마이그레이션을 수행하십시오(변동 규칙). |
| "두 시스템을 무기한 유지 관리할 수 있습니다." | 동일한 작업을 수행하는 두 시스템은 유지 관리, 테스트, 문서화 및 온보딩 비용이 두 배로 늘어납니다. |

## 위험 신호

- 교체가 불가능한 더 이상 사용되지 않는 시스템
- 마이그레이션 도구나 문서가 없는 지원 중단 공지
- 수년 동안 아무런 진전도 없이 권장되었던 "소프트" 지원 중단
- 소유자도 없고 활성 소비자도 없는 좀비 코드
- 더 이상 사용되지 않는 시스템에 새로운 기능 추가(대신 대체 시스템에 투자)
- 현재 사용량을 측정하지 않고 지원 중단
- 활성 소비자가 0인지 확인하지 않고 코드 제거

## 확인

지원 중단을 완료한 후:

- [ ] 교체는 production-proven이며 모든 중요한 사용 사례를 포괄합니다.
- [ ] 마이그레이션 guide가 구체적인 단계와 예시와 함께 존재합니다.
- [ ] 모든 활성 소비자가 마이그레이션되었습니다(metric/logs로 확인).
- [ ] 이전 코드, 테스트, 문서 및 구성이 완전히 제거되었습니다.
- [ ] 더 이상 사용되지 않는 시스템에 대한 참조가 코드베이스에 남아 있지 않습니다.
- [ ] Deprecation notices are removed (they served their purpose)
