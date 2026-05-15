---
description: 5축 코드 리뷰 수행 — 정확성, 가독성, 아키텍처, 보안, 성능
---

agent-skills:code-review-and-quality skill을 invoke하세요.

현재 변경(staged 또는 최근 commit)을 다섯 축 모두에서 리뷰하세요:

1. **정확성(Correctness)** — spec과 일치하는가? 엣지 케이스 처리? 테스트 충분?
2. **가독성(Readability)** — 명확한 이름? 단순한 로직? 잘 조직됨?
3. **아키텍처(Architecture)** — 기존 패턴을 따르는가? 깨끗한 경계? 올바른 추상화 수준?
4. **보안(Security)** — 입력 검증? secrets 안전? 인증 검사? (security-and-hardening skill 사용)
5. **성능(Performance)** — N+1 쿼리 없음? 무한 작업 없음? (performance-optimization skill 사용)

발견을 Critical, Important, 또는 Suggestion으로 분류하세요.
구체적인 file:line 참조와 수정 권장을 갖춘 구조화된 리뷰를 출력하세요.
