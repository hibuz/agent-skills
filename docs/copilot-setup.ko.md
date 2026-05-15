# GitHub Copilot에서 agent-skills 사용하기

## 설정

### Copilot Instructions

Copilot은 repository에 `.github/skills`, `.claude/skills`, 또는 `.agents/skills` 디렉터리를 사용해 agent skill을 만드는 것을 지원합니다.

```bash
mkdir -p .github

# 필수 skill에 대한 파일 생성
cat /path/to/agent-skills/skills/test-driven-development/SKILL.md > .github/skills/test-driven-development/SKILL.md
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md > .github/skills/code-review-and-quality/SKILL.md
```

자세한 내용은 [Creating agent skills for GitHub Copilot](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-skills)을 참고하세요.

### Agent Persona (agents.md)

Copilot은 전문 agent persona를 지원합니다. agent-skills의 agent를 사용하세요:

```bash
# agent 정의 복사
cp /path/to/agent-skills/agents/code-reviewer.md .github/agents/code-reviewer.md
cp /path/to/agent-skills/agents/test-engineer.md .github/agents/test-engineer.md
cp /path/to/agent-skills/agents/security-auditor.md .github/agents/security-auditor.md
```

Copilot Chat에서 agent를 invoke합니다:
- `@code-reviewer Review this PR`
- `@test-engineer Analyze test coverage for this module`
- `@security-auditor Check this endpoint for vulnerabilities`

### Custom Instructions (사용자 레벨)

모든 repository에서 사용하고 싶은 skill의 경우:

1. VS Code 열기 → Settings → GitHub Copilot → Custom Instructions
2. 가장 자주 쓰는 skill 요약을 추가

## 권장 구성

### .github/copilot-instructions.md

GitHub Copilot은 `.github/copilot-instructions.md`를 통한 프로젝트 레벨 지침을 지원합니다.

```markdown
# Project Coding Standards

## Testing
- Write tests before code (TDD)
- For bugs: write a failing test first, then fix (Prove-It pattern)
- Test hierarchy: unit > integration > e2e (use the lowest level that captures the behavior)
- Run `npm test` after every change

## Code Quality
- Review across five axes: correctness, readability, architecture, security, performance
- Every PR must pass: lint, type check, tests, build
- No secrets in code or version control

## Implementation
- Build in small, verifiable increments
- Each increment: implement → test → verify → commit
- Never mix formatting changes with behavior changes

## Boundaries
- Always: Run tests before commits, validate user input
- Ask first: Database schema changes, new dependencies
- Never: Commit secrets, remove failing tests, skip verification
```

### 전문 Agent

Copilot Chat에서 타겟화된 리뷰 워크플로에 agent를 사용하세요.

## 사용 팁

1. **지침을 간결하게 유지** — Copilot 지침은 집중되어 있을 때 가장 잘 동작합니다. 전체 skill 파일을 포함하기보다 핵심 규칙을 요약하세요.
2. **리뷰에 agent 사용** — code-reviewer, test-engineer, security-auditor agent는 Copilot의 agent 모델에 맞게 설계되었습니다.
3. **chat에서 참조** — 특정 단계 작업 시 관련 skill 내용을 Copilot Chat에 붙여넣어 context로 제공하세요.
4. **PR 리뷰와 결합** — code-reviewer agent persona를 사용해 Copilot이 PR을 리뷰하도록 설정하세요.
