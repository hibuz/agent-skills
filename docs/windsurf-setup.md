# Windsurf와 함께 agent-skills 사용

## 설정

### 프로젝트 규칙

Windsurf는 project-specific agent 명령어에 대해 `.windsurfrules`를 사용합니다.

```bash
# Create a combined rules file from your most important skills
cat /path/to/agent-skills/skills/test-driven-development/SKILL.md > .windsurfrules
echo "\n---\n" >> .windsurfrules
cat /path/to/agent-skills/skills/incremental-implementation/SKILL.md >> .windsurfrules
echo "\n---\n" >> .windsurfrules
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md >> .windsurfrules
```

### 전역 규칙

모든 프로젝트에서 원하는 skills의 경우 Windsurf의 전역 규칙에 추가하세요.

1. Windsurf 열기 → 설정 → AI → 전역 규칙
2. most-used skills의 내용을 붙여넣으세요.

## 권장 구성

`.windsurfrules`를 2~3개의 필수 skills에 집중하여 컨텍스트 제한 내에서 유지하세요.

```
# .windsurfrules
# Essential agent-skills for this project

[Paste test-driven-development SKILL.md]

---

[Paste incremental-implementation SKILL.md]

---

[Paste code-review-and-quality SKILL.md]
```

## 사용 팁

1. **선택적이어야 합니다** — Windsurf의 컨텍스트는 제한되어 있습니다. 가장 큰 품질 격차를 해결하는 skills를 선택하십시오.
2. **대화에서 참조** — 특정 단계에서 작업할 때 추가 skill 콘텐츠를 채팅에 붙여넣습니다(예: building 인증 시 `security-and-hardening` 붙여넣기).
3. **참조를 체크리스트로 사용** — `references/security-checklist.md`를 붙여넣고 Windsurf에게 요청하여 각 항목을 확인하세요.
