---
description: 다음 작업을 점진적으로 구현합니다 — build, test, verify, commit
---

agent-skills:incremental-implementation skill을 agent-skills:test-driven-development와 함께 호출합니다.

계획에서 다음 pending 작업을 선택합니다. 각 작업마다 다음을 수행합니다.

1. 작업의 acceptance criteria를 읽습니다.
2. 관련 context(기존 코드, 패턴, 타입)를 로드합니다.
3. 기대 동작에 대한 failing test를 작성합니다(RED).
4. 테스트를 통과시키기 위한 최소 코드를 구현합니다(GREEN).
5. 전체 test suite를 실행해 regression이 없는지 확인합니다.
6. build를 실행해 컴파일이 통과하는지 확인합니다.
7. 설명적인 메시지로 commit합니다.
8. 작업을 완료로 표시하고 다음 작업으로 이동합니다.

어느 단계에서든 실패하면 agent-skills:debugging-and-error-recovery skill을 따릅니다.
