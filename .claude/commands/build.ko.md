---
description: 다음 Task를 점진적으로 구현합니다 — 구현(Build), 테스트, 검증, 커밋.
---

agent-skills:incremental-implementation 기술을 agent-skills:test-driven-development와 함께 호출하세요.

플랜에서 다음 대기 중인 Task를 선택하세요. 각 Task에 대해:

1. Task의 인수 조건(Acceptance Criteria)을 읽습니다.
2. 관련 Context(기존 코드, 패턴, Type)를 로드합니다.
3. 기대되는 동작에 대해 실패하는 Test를 작성합니다 (RED).
4. Test를 통과하기 위한 최소한의 코드를 구현합니다 (GREEN).
5. Regression 확인을 위해 전체 Test Suite를 실행합니다.
6. 컴파일 확인을 위해 Build를 실행합니다.
7. 설명적인 메시지와 함께 커밋합니다.
8. Task를 완료로 표시하고 다음 Task로 이동합니다.

어느 단계에서든 실패하면 agent-skills:debugging-and-error-recovery 기술을 따르세요.
