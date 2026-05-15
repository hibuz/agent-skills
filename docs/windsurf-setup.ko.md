# Windsurf에서 agent-skills 사용하기

## 설정

### Project Rules

Windsurf는 프로젝트별 agent 지침에 `.windsurfrules`를 사용합니다:

```bash
# 가장 중요한 skill들로 결합된 rules 파일 생성
cat /path/to/agent-skills/skills/test-driven-development/SKILL.md > .windsurfrules
echo "\n---\n" >> .windsurfrules
cat /path/to/agent-skills/skills/incremental-implementation/SKILL.md >> .windsurfrules
echo "\n---\n" >> .windsurfrules
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md >> .windsurfrules
```

### Global Rules

모든 프로젝트에서 사용하고 싶은 skill은 Windsurf의 global rules에 추가하세요:

1. Windsurf 열기 → Settings → AI → Global Rules
2. 가장 자주 쓰는 skill 내용을 붙여넣기

## 권장 구성

context 한도 내에 머물도록 `.windsurfrules`를 필수 skill 2-3개에 집중하세요:

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

1. **선별하세요** — Windsurf의 context는 제한적입니다. 가장 큰 품질 공백을 다루는 skill을 선택하세요.
2. **대화에서 참조** — 특정 단계 작업 시 추가 skill 내용을 chat에 붙여넣으세요 (예: 인증 빌드 시 `security-and-hardening` 붙여넣기).
3. **참조를 체크리스트로 사용** — `references/security-checklist.md`를 붙여넣고 Windsurf에게 각 항목을 검증하라고 요청하세요.
