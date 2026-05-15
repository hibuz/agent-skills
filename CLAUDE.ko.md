# agent-skills

agent-skills 프로젝트입니다 — AI 코딩 agent를 위한 production 수준 엔지니어링 skill 모음입니다.

## 프로젝트 구조

```
skills/       → 핵심 skill (디렉터리마다 SKILL.md)
agents/       → 재사용 가능한 agent persona (code-reviewer, test-engineer, security-auditor)
hooks/        → 세션 라이프사이클 hook
.claude/commands/ → 슬래시 커맨드 (/spec, /plan, /build, /test, /review, /code-simplify, /ship)
references/   → 보조 체크리스트 (testing, performance, security, accessibility)
docs/         → 도구별 설정 가이드
```

## 단계별 Skill

**Define:** interview-me, idea-refine, spec-driven-development
**Plan:** planning-and-task-breakdown
**Build:** incremental-implementation, test-driven-development, context-engineering, source-driven-development, doubt-driven-development, frontend-ui-engineering, api-and-interface-design
**Verify:** browser-testing-with-devtools, debugging-and-error-recovery
**Review:** code-review-and-quality, code-simplification, security-and-hardening, performance-optimization
**Ship:** git-workflow-and-versioning, ci-cd-and-automation, deprecation-and-migration, documentation-and-adrs, shipping-and-launch

## 규약

- 모든 skill은 `skills/<name>/SKILL.md`에 위치합니다
- `name`과 `description` 필드를 가진 YAML frontmatter
- description은 skill이 무엇을 하는지(3인칭)로 시작하고, 이어서 트리거 조건("Use when...")이 옵니다
- 모든 skill에는 다음이 있습니다: Overview, When to Use, Process, Common Rationalizations, Red Flags, Verification
- 참조 자료는 skill 디렉터리 내부가 아니라 `references/`에 둡니다
- 보조 파일은 내용이 100줄을 초과할 때만 생성합니다

## 커맨드

- `npm test` — 해당 없음 (이것은 문서 프로젝트입니다)
- 검증: 모든 SKILL.md 파일이 name과 description을 가진 유효한 YAML frontmatter를 갖는지 확인

## 경계

- 항상: 새 skill은 skill-anatomy.md 형식을 따를 것
- 절대 금지: 실행 가능한 프로세스가 아니라 막연한 조언인 skill을 추가하지 말 것
- 절대 금지: skill 간 내용을 중복하지 말 것 — 대신 다른 skill을 참조할 것
