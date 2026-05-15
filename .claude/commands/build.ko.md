---
description: 다음 작업을 점진적으로 구현 — build, test, verify, commit
---

agent-skills:incremental-implementation skill을 agent-skills:test-driven-development와 함께 invoke하세요.

plan에서 다음 pending 작업을 고르세요. 각 작업에 대해:

1. 작업의 수용 기준을 읽기
2. 관련 context 로드 (기존 코드, 패턴, 타입)
3. 예상 동작에 대한 실패하는 테스트 작성 (RED)
4. 테스트를 통과시키는 최소 코드 구현 (GREEN)
5. 회귀를 확인하기 위해 전체 테스트 스위트 실행
6. 컴파일 검증을 위해 build 실행
7. 서술적인 메시지로 commit
8. 작업을 완료로 표시하고 다음으로 이동

어떤 단계든 실패하면, agent-skills:debugging-and-error-recovery skill을 따르세요.
