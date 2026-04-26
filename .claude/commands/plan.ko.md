---
description: 작업을 인수 조건(Acceptance Criteria)과 의존성 순서가 포함된 작고 검증 가능한 Task로 나눕니다.
---

agent-skills:planning-and-task-breakdown 기술을 호출하세요.

기존 Spec(SPEC.md 또는 이와 동등한 문서)과 관련 코드베이스 섹션을 읽으세요. 그 다음:

1. Plan 모드로 진입합니다 — 읽기 전용으로, 코드 변경은 하지 않습니다.
2. Component 간의 의존성 그래프(Dependency Graph)를 식별합니다.
3. 작업을 수직적으로 분할합니다 (수평적 레이어가 아닌, Task당 하나의 완전한 경로).
4. 인수 조건 및 검증 단계가 포함된 Task를 작성합니다.
5. 단계(Phase) 사이에 Checkpoint를 추가합니다.
6. 인간의 검토를 위해 플랜을 제시합니다.

플랜을 tasks/plan.md에, Task 목록을 tasks/todo.md에 저장하세요.
