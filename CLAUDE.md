# agent-skills

이것은 agent-skills 프로젝트입니다. AI 코딩 agents를 위한 production-grade 엔지니어링 skills 모음입니다.

## 프로젝트 구조

```
skills/       → Core skills (SKILL.md per directory)
agents/       → Reusable agent personas (code-reviewer, test-engineer, security-auditor)
hooks/        → Session lifecycle hooks
.claude/commands/ → Slash commands (/spec, /plan, /build, /test, /review, /code-simplify, /ship)
references/   → Supplementary checklists (testing, performance, security, accessibility)
docs/         → Setup guides for different tools
```

## Skills 단계별

**정의:** spec-driven-development
**계획:** planning-and-task-breakdown
**Build:** incremental-implementation, test-driven-development, context-engineering, source-driven-development, frontend-ui-engineering, api-and-interface-design
**확인:** browser-testing-with-devtools, debugging-and-error-recovery
**검토:** code-review-and-quality, code-simplification, security-and-hardening, performance-optimization
**선박:** git-workflow-and-versioning, ci-cd-and-automation, deprecation-and-migration, documentation-and-adrs, shipping-and-launch

## 규칙

- 모든 skill는 `skills/<name>/SKILL.md`에 있습니다.
- `name` 및 `description` 필드가 있는 YAML frontmatter
- 설명은 skill가 수행하는 작업(3인칭)으로 시작하고 이어서 트리거 조건("사용할 경우...")이 이어집니다.
- 모든 skill에는 개요, 사용 시기, 프로세스, 공통 합리화, 위험 신호, 검증이 포함됩니다.
- 참조는 skill 디렉토리 내부가 아닌 `references/`에 있습니다.
- 내용이 100줄을 초과하는 경우에만 생성되는 지원 파일

## 명령

- `npm test` — 해당 사항 없음(문서화 프로젝트입니다)
- 유효성 검사: 모든 SKILL.md 파일에 이름과 설명이 포함된 유효한 YAML frontmatter가 있는지 확인하세요.

## 경계

- 항상: 새로운 skills를 보려면 skill-anatomy.md format를 따르세요.
- 안 함: 실행 가능한 프로세스 대신 모호한 조언인 skills를 추가합니다.
- 안 함: skills 사이에 콘텐츠가 중복됩니다. 대신 다른 skills를 참조하세요.
