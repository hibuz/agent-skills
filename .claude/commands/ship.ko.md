---
description: 출시 전 Checklist를 실행하고 Production Deployment를 준비합니다.
---

agent-skills:shipping-and-launch 기술을 호출하세요.

전체 출시 전 Checklist를 실행하세요:

1. **Code Quality** — Test 통과, Build 성공, Lint 통과, TODO 없음, console.log 없음
2. **Security** — npm audit 통과, 코드 내 Secret 없음, 인증 구현됨, Header 구성됨
3. **Performance** — Core Web Vitals 양호, N+1 Query 없음, 이미지 최적화됨, Bundle 크기 적절함
4. **Accessibility** — 키보드 탐색 작동, Screen Reader 호환, 대비 적절함
5. **Infrastructure** — 환경 변수 설정됨, Migration 준비됨, Monitoring 구성됨
6. **Documentation** — README 최신화됨, ADR 작성됨, Changelog 업데이트됨

실패한 항목을 보고하고 배포 전에 해결하도록 돕습니다.
진행하기 전에 Rollback Plan을 정의하세요.
