---
description: TDD workflow를 실행합니다 — failing test를 작성하고, 구현하고, verify합니다. 버그에는 Prove-It 패턴을 사용합니다
---

agent-skills:test-driven-development skill을 호출합니다.

새 기능의 경우:
1. 기대 동작을 설명하는 테스트를 작성합니다(반드시 FAIL이어야 함).
2. 테스트를 통과시키도록 코드를 구현합니다.
3. 테스트가 green인 상태를 유지하면서 리팩터링합니다.

버그 수정(Prove-It 패턴)의 경우:
1. 버그를 재현하는 테스트를 작성합니다(반드시 FAIL이어야 함).
2. 테스트가 실제로 실패하는지 확인합니다.
3. 수정 사항을 구현합니다.
4. 테스트가 통과하는지 확인합니다.
5. regression 확인을 위해 전체 test suite를 실행합니다.

browser 관련 문제라면 agent-skills:browser-testing-with-devtools도 호출해 Chrome DevTools MCP로 검증합니다.
