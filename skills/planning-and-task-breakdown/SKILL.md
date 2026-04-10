---
name: planning-and-task-breakdown
description: 작업을 순서가 지정된 작업으로 나눕니다. 사양이 있거나 requirements가 명확하고 작업을 구현 가능한 작업으로 나누어야 할 때 사용하세요. 작업이 너무 커서 시작하기 어려울 때, 범위를 추정해야 할 때 또는 병렬 작업이 가능할 때 사용하세요.
---

# 계획 및 업무 분류

## 개요

명시적인 승인 기준을 사용하여 작업을 작고 검증 가능한 작업으로 분해합니다. 좋은 작업 분석은 작업을 안정적으로 완료하는 agent와 엉킨 혼란을 일으키는 agent의 차이입니다. 모든 작업은 단일 집중 세션에서 구현, 테스트 및 검증할 수 있을 만큼 작아야 합니다.

## 사용 시기

- 사양이 있고 이를 구현 가능한 단위로 나누어야 합니다.
- 작업이 시작하기에 너무 크거나 모호한 느낌
- 작업은 여러 agents 또는 세션에 걸쳐 병렬화되어야 합니다.
- 사람에게 범위를 전달해야 합니다.
- 구현 순서가 명확하지 않음

**사용하지 말아야 할 때:** 단일 파일이 명확한 범위로 변경되거나 사양에 이미 well-defined 작업이 포함된 경우.

## 기획 과정

### 1단계: 계획 모드 시작

코드를 작성하기 전에 read-only 모드에서 작동하십시오.

- 사양 및 관련 코드베이스 섹션을 읽어보세요.
- 기존 패턴 및 관례 식별
- 구성 요소 간의 종속성을 매핑합니다.
- 위험과 알려지지 않은 사항을 기록하세요.

**계획 중에 NOT 코드를 작성하세요.** 출력은 구현이 아닌 계획 문서입니다.

### 2단계: 종속성 그래프 식별

무엇이 무엇에 따라 달라지는지 매핑합니다.

```
Database schema
    │
    ├── API models/types
    │       │
    │       ├── API endpoints
    │       │       │
    │       │       └── Frontend API client
    │       │               │
    │       │               └── UI components
    │       │
    │       └── Validation logic
    │
    └── Seed data / migrations
```

구현 순서는 종속성 그래프 bottom-up: build 기반을 먼저 따릅니다.

### 3단계: 세로로 자르기

building 대신 모든 데이터베이스, 모든 API, 그런 다음 모든 UI - build 한 번에 하나의 완전한 기능 경로:

**나쁨(수평 분할):**
```
Task 1: Build entire database schema
Task 2: Build all API endpoints
Task 3: Build all UI components
Task 4: Connect everything
```

**좋음(세로 자르기):**
```
Task 1: User can create an account (schema + API + UI for registration)
Task 2: User can log in (auth schema + API + UI for login)
Task 3: User can create a task (task schema + API + UI for creation)
Task 4: User can view task list (query + API + UI for list view)
```

각 수직 슬라이스는 작동하고 테스트 가능한 기능을 제공합니다.

### 4단계: 작업 작성

각 작업은 다음 구조를 따릅니다.

```markdown
## Task [N]: [Short descriptive title]

**Description:** One paragraph explaining what this task accomplishes.

**Acceptance criteria:**
- [ ] [Specific, testable condition]
- [ ] [Specific, testable condition]

**Verification:**
- [ ] Tests pass: `npm test -- --grep "feature-name"`
- [ ] Build succeeds: `npm run build`
- [ ] Manual check: [description of what to verify]

**Dependencies:** [Task numbers this depends on, or "None"]

**Files likely touched:**
- `src/path/to/file.ts`
- `tests/path/to/test.ts`

**Estimated scope:** [Small: 1-2 files | Medium: 3-5 files | Large: 5+ files]
```

### 5단계: 주문 및 체크포인트

다음과 같이 작업을 정렬합니다.

1. 종속성이 충족됩니다(build 기반 먼저).
2. 각 작업은 시스템을 작동 상태로 둡니다.
3. 2~3개의 작업마다 검증 체크포인트가 발생합니다.
4. 고위험 작업은 조기에 수행됩니다(Fail Fast).

명시적 체크포인트를 추가합니다.

```markdown
## Checkpoint: After Tasks 1-3
- [ ] All tests pass
- [ ] Application builds without errors
- [ ] Core user flow works end-to-end
- [ ] Review with human before proceeding
```

## 작업 크기 조정 Guidelines

| 사이즈 | 파일 | 범위 | 예 |
|------|-------|-------|---------|
| **XS** | 1 | 단일 기능 또는 구성 변경 | 유효성 검사 규칙 추가 |
| **쩝** | 1-2 | 하나의 구성요소 또는 엔드포인트 | 새 API 엔드포인트 추가 |
| **엠** | 3-5 | 하나의 기능 조각 | 사용자 등록 흐름 |
| **엘** | 5-8 | 다중 구성 요소 기능 | 필터링 및 페이지 매김으로 검색 |
| **XL** | 8세 이상 | **너무 큽니다. 더 자세히 분류하세요** | — |

작업이 L 이상인 경우 더 작은 작업으로 나누어야 합니다. agent 성능은 S 및 M 작업에 가장 적합합니다.

**작업을 더 세분화해야 하는 경우:**
- 하나 이상의 집중 세션이 필요합니다(agent 작업에 대략 2시간 이상 소요).
- 합격 기준을 3개 이하로 기술할 수 없습니다.
- 두 개 이상의 독립적인 하위 시스템(예: 인증 및 청구)에 닿습니다.
- 작업 제목에 "and"라고 적힌 자신을 발견하게 됩니다(두 개의 작업이라는 표시).

## 계획 문서 템플릿

```markdown
# Implementation Plan: [Feature/Project Name]

## Overview
[One paragraph summary of what we're building]

## Architecture Decisions
- [Key decision 1 and rationale]
- [Key decision 2 and rationale]

## Task List

### Phase 1: Foundation
- [ ] Task 1: ...
- [ ] Task 2: ...

### Checkpoint: Foundation
- [ ] Tests pass, builds clean

### Phase 2: Core Features
- [ ] Task 3: ...
- [ ] Task 4: ...

### Checkpoint: Core Features
- [ ] End-to-end flow works

### Phase 3: Polish
- [ ] Task 5: ...
- [ ] Task 6: ...

### Checkpoint: Complete
- [ ] All acceptance criteria met
- [ ] Ready for review

## Risks and Mitigations
| Risk | Impact | Mitigation |
|------|--------|------------|
| [Risk] | [High/Med/Low] | [Strategy] |

## Open Questions
- [Question needing human input]
```

## 병렬화 기회

여러 agents 또는 세션을 사용할 수 있는 경우:

- **안전한 병렬화:** 독립적인 기능 슬라이스, already-implemented 기능 테스트, 문서
- **순차적이어야 함:** 데이터베이스 마이그레이션, 공유 상태 변경, 종속성 체인
- **조정 필요:** API 계약을 공유하는 기능(계약을 먼저 정의한 후 병렬화)

## 일반적인 합리화

| 합리화 | 현실 |
|---|---|
| "내가 가서 알아낼게" | 그렇게 엉킨 난장판과 재작업으로 끝나고 만다. 10분만 계획하면 시간이 절약됩니다. |
| "과제는 분명하다" | 어쨌든 적어보세요. 명시적 작업은 숨겨진 종속성과 잊어버린 예외 사례를 드러냅니다. |
| "계획은 오버헤드입니다" | 기획이 임무입니다. 계획 없는 구현은 단지 타이핑일 뿐입니다. |
| "내 머릿속엔 다 담을 수 있어" | 컨텍스트 창은 유한합니다. 서면 계획은 세션 경계와 압축을 유지합니다. |

## 위험 신호

- 서면 작업 목록 없이 구현 시작
- 승인 기준 없이 "기능 구현"이라고 말하는 작업
- 계획에 검증 단계가 없습니다.
- 모든 작업은 XL 크기입니다.
- 작업 사이에 체크포인트가 없습니다.
- 종속성 순서는 고려되지 않습니다.

## 확인

구현을 시작하기 전에 다음을 확인하세요.

- [ ] 모든 작업에는 승인 기준이 있습니다.
- [ ] 모든 작업에는 확인 단계가 있습니다.
- [ ] 작업 종속성이 식별되고 올바르게 정렬되었습니다.
- [ ] 작업이 ~5개 이상의 파일을 건드리지 않습니다.
- [ ] 주요 단계 사이에 체크포인트가 존재합니다.
- [ ] 사람이 계획을 검토하고 승인했습니다.
