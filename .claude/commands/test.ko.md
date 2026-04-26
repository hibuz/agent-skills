---
description: TDD Workflow를 실행합니다 — 실패하는 Test 작성, 구현, 검증. 버그의 경우 Prove-It 패턴을 사용합니다.
---

agent-skills:test-driven-development 기술을 호출하세요.

새로운 기능의 경우:
1. 기대되는 동작을 설명하는 Test를 작성함 (반드시 실패(FAIL)해야 함)
2. 이를 통과시키기 위한 코드를 구현함
3. Test가 Green 상태를 유지하는 동안 Refactor 수행

버그 수정의 경우 (Prove-It 패턴):
1. 버그를 재현하는 Test를 작성함 (반드시 실패(FAIL)해야 함)
2. Test가 실패함을 확인함
3. 수정을 구현함
4. Test가 통과함을 확인함
5. Regression 확인을 위해 전체 Test Suite를 실행함

브라우저 관련 이슈의 경우, Chrome DevTools MCP로 검증하기 위해 agent-skills:browser-testing-with-devtools도 호출하세요.
