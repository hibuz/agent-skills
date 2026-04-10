---
description: pre-launch checklist를 실행하고 production deployment를 준비합니다
---

agent-skills:shipping-and-launch skill을 호출합니다.

전체 pre-launch checklist를 실행합니다.

1. **Code Quality** — 테스트 통과, build clean, lint clean, TODO 없음, `console.log` 없음
2. **Security** — `npm audit` clean, 코드에 secret 없음, auth 구성 완료, header 설정 완료
3. **Performance** — Core Web Vitals 양호, N+1 query 없음, 이미지 최적화, bundle size 점검
4. **Accessibility** — keyboard navigation 동작, screen reader 호환, 대비 적절
5. **Infrastructure** — env var 설정 완료, migration 준비 완료, monitoring 구성 완료
6. **Documentation** — README 최신 상태, ADR 작성, changelog 업데이트

실패한 체크는 report하고, deployment 전에 해결을 돕습니다.
진행하기 전에 rollback 계획을 정의합니다.
