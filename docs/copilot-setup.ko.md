# Using agent-skills with GitHub Copilot

## Setup

### Copilot Instructions

Copilot은 리포지토리 내의 `.github/skills`, `.claude/skills` 또는 `.agents/skills` 디렉토리를 사용하여 Agent Skill 생성을 지원합니다.

```bash
mkdir -p .github

# 필수 기술들을 위한 파일 생성
cat /path/to/agent-skills/skills/test-driven-development/SKILL.md > .github/skills/test-driven-development/SKILL.md
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md > .github/skills/code-review-and-quality/SKILL.md
```

더 자세한 내용은 [Creating agent skills for GitHub Copilot](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-skills)을 참조하세요.

### Agent Personas (agents.md)

Copilot은 특화된 Agent 페르소나를 지원합니다. agent-skills의 Agent들을 사용하세요:

```bash
# Agent 정의 복사
cp /path/to/agent-skills/agents/code-reviewer.md .github/agents/code-reviewer.md
cp /path/to/agent-skills/agents/test-engineer.md .github/agents/test-engineer.md
cp /path/to/agent-skills/agents/security-auditor.md .github/agents/security-auditor.md
```

Copilot Chat에서 Agent를 호출하세요:
- `@code-reviewer 이 PR을 리뷰해줘`
- `@test-engineer 이 모듈의 테스트 커버리지를 분석해줘`
- `@security-auditor 이 Endpoint의 취약점을 점검해줘`

### Custom Instructions (사용자 수준)

모든 리포지토리에 적용하고 싶은 기술의 경우:

1. VS Code 열기 → Settings → GitHub Copilot → Custom Instructions
2. 가장 많이 사용하는 기술들의 요약을 추가하세요

## Recommended Configuration

### .github/copilot-instructions.md

GitHub Copilot은 `.github/copilot-instructions.md`를 통해 프로젝트 수준의 지침을 지원합니다.

```markdown
# Project Coding Standards

## Testing
- 코드 작성 전 테스트 작성 (TDD)
- 버그 수정 시: 실패하는 테스트를 먼저 작성한 후 수정 (Prove-It 패턴)
- 테스트 계층: unit > integration > e2e (동작을 포착하는 가장 낮은 수준 사용)
- 모든 변경 후 `npm test` 실행

## Code Quality
- 5개 축으로 리뷰: 정확성(Correctness), 가독성(Readability), 아키텍처(Architecture), 보안(Security), 성능(Performance)
- 모든 PR은 다음을 통과해야 함: lint, type check, tests, build
- 코드나 버전 관리 시스템에 Secret(비밀 정보) 포함 금지

## Implementation
- 작고 검증 가능한 단위로 구축
- 각 단위별로: 구현 → 테스트 → 검증 → 커밋
- 포맷팅 변경과 동작 변경을 절대 섞지 말 것

## Boundaries
- 항상: 커밋 전 테스트 실행, 사용자 입력 검증
- 먼저 질문할 것: Database Schema 변경, 새로운 Dependency 추가
- 절대 금지: Secret 커밋, 실패하는 테스트 제거, 검증 단계 건너뛰기
```

### Specialized Agents

Copilot Chat에서 타겟팅된 리뷰 Workflow를 위해 Agent를 사용하세요.

## Usage Tips

1. **지침을 간결하게 유지하세요** — Copilot 지침은 집중되어 있을 때 가장 잘 작동합니다. 전체 기술 파일보다는 핵심 규칙 위주로 요약하세요.
2. **리뷰를 위해 Agent를 사용하세요** — code-reviewer, test-engineer, security-auditor Agent들은 Copilot의 Agent 모델에 맞게 설계되었습니다.
3. **채팅에서 참조하세요** — 특정 단계에서 작업할 때 관련 기술 내용을 Copilot Chat에 붙여넣어 Context를 제공하세요.
4. **PR 리뷰와 결합하세요** — code-reviewer Agent 페르소나를 사용하여 PR을 리뷰하도록 Copilot을 설정하세요.
