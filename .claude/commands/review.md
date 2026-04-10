---
description: five-axis 코드 review를 수행합니다 — correctness, readability, architecture, security, performance
---

agent-skills:code-review-and-quality skill을 호출합니다.

다섯 축 전체에서 현재 변경 사항(staged 변경 또는 최근 commit)을 검토합니다.

1. **Correctness** — spec과 일치하는가? edge case가 처리되었는가? 테스트는 충분한가?
2. **Readability** — 이름이 명확한가? 로직이 직관적인가? 구조가 잘 정리되어 있는가?
3. **Architecture** — 기존 패턴을 따르는가? 경계가 깔끔한가? 추상화 수준이 적절한가?
4. **Security** — 입력 검증이 되어 있는가? secret은 안전한가? auth가 확인되었는가? (`security-and-hardening` skill 사용)
5. **Performance** — N+1 query는 없는가? unbounded operation은 없는가? (`performance-optimization` skill 사용)

결과는 Critical, Important, Suggestion으로 분류합니다.
구체적인 `file:line` 참조와 수정 권장 사항을 포함한 구조화된 review를 출력합니다.
