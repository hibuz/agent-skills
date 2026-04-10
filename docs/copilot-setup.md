# GitHub Copilot과 함께 agent-skills 사용

## 설정

### 부조종사 지침

Copilot은 repository에서 `.github/skills`, `.claude/skills` 또는 `.agents/skills` 디렉터리를 사용하여 agent skills 생성을 지원합니다.

```bash
mkdir -p .github

# Create files for essential skills
cat /path/to/agent-skills/skills/test-driven-development/SKILL.md > .github/skills/test-driven-development/SKILL.md
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md > .github/skills/code-review-and-quality/SKILL.md
```

자세한 내용은 [Creating agent skills for GitHub Copilot](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-skills)를 참조하세요.

### Agent 페르소나(agents.md)

Copilot은 특수한 agent 페르소나를 지원합니다. agent-skills agents를 사용하세요.

```bash
# Copy agent definitions
cp /path/to/agent-skills/agents/code-reviewer.md .github/agents/code-reviewer.md
cp /path/to/agent-skills/agents/test-engineer.md .github/agents/test-engineer.md
cp /path/to/agent-skills/agents/security-auditor.md .github/agents/security-auditor.md
```

Copilot Chat에서 agents를 호출합니다.
- `@code-reviewer Review this PR`
- `@test-engineer Analyze test coverage for this module`
- `@security-auditor Check this endpoint for vulnerabilities`

### 맞춤 지침(사용자 수준)

skills의 경우 모든 repositories에서 원하는:

1. VS 코드 열기 → 설정 → GitHub Copilot → 사용자 정의 지침
2. most-used skill 요약을 추가하세요.

## 권장 구성

### .github/copilot-instructions.md

GitHub Copilot은 `.github/copilot-instructions.md`를 통해 project-level 지침을 지원합니다.

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

### 전문 Agents

Copilot Chat에서 대상 검토 workflows를 위해 agents를 사용하세요.

## 사용 팁

1. **지침을 간결하게 유지하세요** — Copilot 지침은 집중할 때 가장 잘 작동합니다. 전체 skill 파일을 포함하는 대신 주요 규칙을 요약합니다.
2. **검토를 위해 agents 사용** — code-reviewer, test-engineer 및 security-auditor agents는 Copilot의 agent 모델용으로 설계되었습니다.
3. **채팅 참조** — 특정 단계에서 작업할 때 관련 skill 콘텐츠를 Copilot Chat에 붙여넣어 맥락을 파악하세요.
4. **PR 리뷰와 결합** — code-reviewer agent 페르소나를 사용하여 PRs를 검토하도록 Copilot을 설정합니다.
