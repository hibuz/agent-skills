---
description: 정확성, 가독성, 아키텍처, 보안, 성능의 5개 축으로 코드 리뷰를 수행합니다.
---

agent-skills:code-review-and-quality 기술을 호출하세요.

현재 변경 사항(Staged 또는 최근 Commit)을 다음 5개 축으로 리뷰하세요:

1. **Correctness (정확성)** — Spec과 일치합니까? Edge Case가 처리되었습니까? 테스트가 충분합니까?
2. **Readability (가독성)** — 이름이 명확합니까? 로직이 단순합니까? 잘 조직되어 있습니까?
3. **Architecture (아키텍처)** — 기존 패턴을 따릅니까? 경계가 명확합니까? 추상화 레벨이 적절합니까?
4. **Security (보안)** — 입력이 검증되었습니까? Secret이 안전합니까? 인증이 확인되었습니까? (security-and-hardening 기술 사용)
5. **Performance (성능)** — N+1 Query가 없습니까? 제한 없는 작업이 없습니까? (performance-optimization 기술 사용)

발견 사항을 Critical, Important 또는 Suggestion으로 분류하세요.
구체적인 파일:라인 참조 및 수정 권장 사항을 포함한 구조화된 리뷰를 출력하세요.
