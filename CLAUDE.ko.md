# agent-skills

agent-skills 프로젝트입니다 — AI 코딩 Agent를 위한 프로덕션 등급의 엔지니어링 기술(Skill) 모음입니다.

## Project Structure

```
skills/       → 핵심 기술 (디렉토리당 SKILL.md)
agents/       → 재사용 가능한 Agent 페르소나 (code-reviewer, test-engineer, security-auditor)
hooks/        → 세션 Lifecycle Hook
.claude/commands/ → 슬래시 명령어 (/spec, /plan, /build, /test, /review, /code-simplify, /ship)
references/   → 보조 Checklist (테스트, 성능, 보안, 접근성)
docs/         → 다양한 도구에 대한 설정 가이드
```

## Skills by Phase

**Define:** spec-driven-development
**Plan:** planning-and-task-breakdown
**Build:** incremental-implementation, test-driven-development, context-engineering, source-driven-development, frontend-ui-engineering, api-and-interface-design
**Verify:** browser-testing-with-devtools, debugging-and-error-recovery
**Review:** code-review-and-quality, code-simplification, security-and-hardening, performance-optimization
**Ship:** git-workflow-and-versioning, ci-cd-and-automation, deprecation-and-migration, documentation-and-adrs, shipping-and-launch

## Conventions

- 모든 기술은 `skills/<이름>/SKILL.md`에 위치합니다.
- `name`과 `description` 필드가 포함된 YAML Frontmatter를 가집니다.
- 설명은 기술이 하는 일(3인칭)로 시작하며, 트리거 조건("Use when...")이 뒤따릅니다.
- 모든 기술은 Overview, When to Use, Process, Common Rationalizations, Red Flags, Verification 섹션을 가집니다.
- Reference는 기술 디렉토리 내부가 아닌 `references/`에 위치합니다.
- 보조 파일은 내용이 100행을 초과하는 경우에만 생성합니다.

## Commands

- `npm test` — 해당 없음 (이것은 문서 프로젝트입니다)
- Validate: 모든 SKILL.md 파일이 name과 description이 포함된 유효한 YAML Frontmatter를 가지고 있는지 확인합니다.

## Boundaries

- 항상: 새로운 기술을 만들 때 [skill-anatomy.ko.md](docs/skill-anatomy.ko.md) 형식을 따르세요.
- 절대 금지: 실행 가능한 프로세스가 아닌 모호한 조언인 기술을 추가하지 마세요.
- 절대 금지: 기술 간에 내용을 중복시키지 마세요 — 대신 다른 기술을 참조하세요.
