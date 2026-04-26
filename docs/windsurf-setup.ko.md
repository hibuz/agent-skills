# Using agent-skills with Windsurf

## Setup

### Project Rules

Windsurf는 프로젝트별 Agent 지침을 위해 `.windsurfrules`를 사용합니다:

```bash
# 가장 중요한 기술들로 결합된 규칙 파일을 생성합니다
cat /path/to/agent-skills/skills/test-driven-development/SKILL.md > .windsurfrules
echo "\n---\n" >> .windsurfrules
cat /path/to/agent-skills/skills/incremental-implementation/SKILL.md >> .windsurfrules
echo "\n---\n" >> .windsurfrules
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md >> .windsurfrules
```

### Global Rules

모든 프로젝트에 적용하고 싶은 기술은 Windsurf의 Global Rules에 추가하세요:

1. Windsurf 열기 → Settings → AI → Global Rules
2. 가장 많이 사용하는 기술의 내용을 붙여넣으세요

## Recommended Configuration

Context 제한 내에 머물기 위해 `.windsurfrules`는 2~3개의 필수 기술에 집중하세요:

```
# .windsurfrules
# 이 프로젝트를 위한 필수 agent-skills

[test-driven-development SKILL.md 내용 붙여넣기]

---

[incremental-implementation SKILL.md 내용 붙여넣기]

---

[code-review-and-quality SKILL.md 내용 붙여넣기]
```

## Usage Tips

1. **선택적으로 사용하세요** — Windsurf의 Context는 제한되어 있습니다. 가장 큰 품질 격차를 해결해주는 기술을 선택하세요.
2. **대화 중에 참조하세요** — 특정 단계에서 작업할 때 추가 기술 내용을 채팅에 붙여넣으세요 (예: 인증 기능을 구축할 때 `security-and-hardening` 내용 붙여넣기).
3. **Reference를 Checklist로 사용하세요** — `references/security-checklist.md` 내용을 붙여넣고 Windsurf에게 각 항목을 확인해달라고 요청하세요.
