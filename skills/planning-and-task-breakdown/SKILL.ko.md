---
name: planning-and-task-breakdown
description: 작업을 순서가 있는 task로 분해. spec이나 명확한 요구사항이 있고 작업을 구현 가능한 task로 분해해야 할 때 사용. task가 시작하기엔 너무 커 보일 때, 범위를 추정해야 할 때, 또는 병렬 작업이 가능할 때 사용.
---

# Planning and Task Breakdown

## Overview

명시적 수용 기준을 가진 작고 검증 가능한 task로 작업을 분해하세요. 좋은 task 분해는 작업을 안정적으로 완료하는 agent와 엉킨 난장판을 만드는 agent의 차이입니다. 모든 task는 단일 집중 세션에서 구현하고, 테스트하고, 검증할 만큼 작아야 합니다.

## When to Use

- spec이 있고 구현 가능한 단위로 분해해야 함
- task가 시작하기엔 너무 크거나 막연하게 느껴짐
- 작업을 여러 agent나 세션에 걸쳐 병렬화해야 함
- 사람에게 범위를 전달해야 함
- 구현 순서가 명백하지 않음

**사용하지 말아야 할 때:** 범위가 명백한 단일 파일 변경, 또는 spec이 이미 잘 정의된 task를 포함할 때.

## 계획 프로세스

### Step 1: Plan 모드 진입

코드를 작성하기 전에, 읽기 전용 모드로 작동:

- spec과 관련 코드베이스 섹션 읽기
- 기존 패턴과 규약 식별
- 컴포넌트 간 의존성 매핑
- 위험과 미지수 메모

**계획 중 코드를 작성하지 마세요.** 출력은 구현이 아니라 plan 문서입니다.

### Step 2: 의존성 그래프 식별

무엇이 무엇에 의존하는지 매핑:

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

구현 순서는 의존성 그래프를 bottom-up으로 따릅니다: 기초를 먼저 빌드.

### Step 3: 수직으로 슬라이스

모든 데이터베이스, 그 다음 모든 API, 그 다음 모든 UI를 빌드하는 대신 — 한 번에 하나의 완전한 기능 경로를 빌드:

**나쁨 (수평 슬라이싱):**
```
Task 1: 전체 데이터베이스 스키마 빌드
Task 2: 모든 API endpoint 빌드
Task 3: 모든 UI 컴포넌트 빌드
Task 4: 전부 연결
```

**좋음 (수직 슬라이싱):**
```
Task 1: 사용자가 계정을 생성할 수 있음 (등록용 schema + API + UI)
Task 2: 사용자가 로그인할 수 있음 (auth schema + API + 로그인용 UI)
Task 3: 사용자가 task를 생성할 수 있음 (task schema + API + 생성용 UI)
Task 4: 사용자가 task 목록을 볼 수 있음 (query + API + 목록 뷰용 UI)
```

각 수직 슬라이스가 동작하고 테스트 가능한 기능을 전달합니다.

### Step 4: Task 작성

각 task는 이 구조를 따릅니다:

```markdown
## Task [N]: [짧은 서술적 제목]

**Description:** 이 task가 무엇을 달성하는지 설명하는 한 단락.

**Acceptance criteria:**
- [ ] [구체적, 테스트 가능한 조건]
- [ ] [구체적, 테스트 가능한 조건]

**Verification:**
- [ ] 테스트 통과: `npm test -- --grep "feature-name"`
- [ ] 빌드 성공: `npm run build`
- [ ] 수동 확인: [검증할 것에 대한 설명]

**Dependencies:** [이것이 의존하는 task 번호, 또는 "None"]

**Files likely touched:**
- `src/path/to/file.ts`
- `tests/path/to/test.ts`

**Estimated scope:** [Small: 1-2 파일 | Medium: 3-5 파일 | Large: 5+ 파일]
```

### Step 5: 순서와 체크포인트

다음이 되도록 task를 배열:

1. 의존성이 충족됨 (기초를 먼저 빌드)
2. 각 task가 시스템을 동작 상태로 남김
3. 2-3개 task마다 검증 체크포인트 발생
4. 고위험 task가 일찍 (fail fast)

명시적 체크포인트 추가:

```markdown
## Checkpoint: Tasks 1-3 후
- [ ] 모든 테스트 통과
- [ ] 애플리케이션이 에러 없이 빌드
- [ ] 핵심 사용자 흐름이 end-to-end로 동작
- [ ] 진행 전 사람과 리뷰
```

## Task 크기 조정 가이드라인

| 크기 | 파일 | 범위 | 예시 |
|------|-------|-------|---------|
| **XS** | 1 | 단일 함수나 config 변경 | 검증 규칙 추가 |
| **S** | 1-2 | 하나의 컴포넌트나 endpoint | 새 API endpoint 추가 |
| **M** | 3-5 | 하나의 기능 슬라이스 | 사용자 등록 흐름 |
| **L** | 5-8 | 다중 컴포넌트 기능 | 필터링과 pagination이 있는 검색 |
| **XL** | 8+ | **너무 큼 — 더 분해하라** | — |

task가 L 이상이면, 더 작은 task로 분해되어야 합니다. agent는 S와 M task에서 가장 잘 수행합니다.

**task를 더 분해할 때:**
- 단일 집중 세션 이상이 걸림 (대략 agent 작업 2시간+)
- 수용 기준을 3개 이하 글머리 기호로 설명 못 함
- 두 개 이상의 독립 서브시스템을 건드림 (예: auth와 billing)
- task 제목에 "and"를 쓰고 있는 자신을 발견 (두 task라는 신호)

## Plan 문서 템플릿

```markdown
# Implementation Plan: [Feature/Project Name]

## Overview
[무엇을 빌드하는지 한 단락 요약]

## Architecture Decisions
- [핵심 결정 1과 근거]
- [핵심 결정 2와 근거]

## Task List

### Phase 1: Foundation
- [ ] Task 1: ...
- [ ] Task 2: ...

### Checkpoint: Foundation
- [ ] 테스트 통과, 빌드 깨끗

### Phase 2: Core Features
- [ ] Task 3: ...
- [ ] Task 4: ...

### Checkpoint: Core Features
- [ ] end-to-end 흐름 동작

### Phase 3: Polish
- [ ] Task 5: ...
- [ ] Task 6: ...

### Checkpoint: Complete
- [ ] 모든 수용 기준 충족
- [ ] 리뷰 준비됨

## Risks and Mitigations
| Risk | Impact | Mitigation |
|------|--------|------------|
| [Risk] | [High/Med/Low] | [Strategy] |

## Open Questions
- [사람 입력이 필요한 질문]
```

## 병렬화 기회

여러 agent나 세션이 가능할 때:

- **병렬화해도 안전:** 독립 기능 슬라이스, 이미 구현된 기능의 테스트, 문서
- **순차여야 함:** 데이터베이스 migration, 공유 상태 변경, 의존성 체인
- **조율 필요:** API contract를 공유하는 기능 (contract를 먼저 정의한 뒤 병렬화)

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "하면서 알아낼게" | 그게 엉킨 난장판과 재작업으로 끝나는 방식. 10분의 계획이 몇 시간을 절약. |
| "task가 명백해" | 그래도 적으세요. 명시적 task는 숨은 의존성과 잊힌 엣지 케이스를 표면화. |
| "계획은 오버헤드야" | 계획이 task입니다. plan 없는 구현은 그냥 타이핑. |
| "다 머릿속에 담을 수 있어" | context window는 유한. 작성된 plan은 세션 경계와 compaction을 살아남음. |

## Red Flags

- 작성된 task 목록 없이 구현 시작
- 수용 기준 없이 "기능 구현"이라고 하는 task
- plan에 검증 단계 없음
- 모든 task가 XL 크기
- task 사이 체크포인트 없음
- 의존성 순서가 고려되지 않음

## Verification

구현 시작 전, 확인:

- [ ] 모든 task에 수용 기준 있음
- [ ] 모든 task에 검증 단계 있음
- [ ] task 의존성이 식별되고 올바르게 순서됨
- [ ] 어떤 task도 ~5개 파일 이상 건드리지 않음
- [ ] 주요 페이즈 사이 체크포인트 존재
- [ ] 사람이 plan을 리뷰하고 승인함
