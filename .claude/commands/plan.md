---
description: acceptance criteria와 dependency 순서를 기준으로 작업을 작고 검증 가능한 단위로 나눕니다
---

agent-skills:planning-and-task-breakdown skill을 호출합니다.

기존 spec(`SPEC.md` 또는 equivalent)과 관련 코드베이스 섹션을 읽은 뒤 다음을 수행합니다.

1. plan mode로 들어갑니다. 읽기 전용이며 코드 변경은 하지 않습니다.
2. 컴포넌트 간 dependency graph를 식별합니다.
3. 작업을 수직으로 분할합니다. 수평 레이어가 아니라 작업마다 하나의 완전한 경로가 되도록 합니다.
4. acceptance criteria와 verification step을 포함해 작업을 작성합니다.
5. phase 사이에 checkpoint를 추가합니다.
6. human review를 위해 계획을 제시합니다.

계획은 `tasks/plan.md`에 저장하고, 작업 목록은 `tasks/todo.md`에 저장합니다.
