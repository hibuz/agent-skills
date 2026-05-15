---
description: TDD 워크플로 실행 — 실패하는 테스트 작성, 구현, 검증. 버그에는 Prove-It 패턴 사용.
---

agent-skills:test-driven-development skill을 invoke하세요.

새 기능의 경우:
1. 예상 동작을 기술하는 테스트 작성 (FAIL해야 함)
2. 통과시키는 코드 구현
3. 테스트를 green으로 유지하면서 refactor

버그 수정의 경우 (Prove-It 패턴):
1. 버그를 재현하는 테스트 작성 (반드시 FAIL해야 함)
2. 테스트가 실패하는지 확인
3. 수정 구현
4. 테스트가 통과하는지 확인
5. 회귀를 위해 전체 테스트 스위트 실행

browser 관련 이슈의 경우, Chrome DevTools MCP로 검증하기 위해 agent-skills:browser-testing-with-devtools도 invoke하세요.
