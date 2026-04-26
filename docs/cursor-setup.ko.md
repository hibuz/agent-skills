# Using agent-skills with Cursor

## Setup

### Option 1: Rules Directory (권장)

Cursor는 프로젝트별 규칙을 위해 `.cursor/rules/` 디렉토리를 지원합니다:

```bash
# 규칙 디렉토리 생성
mkdir -p .cursor/rules

# 규칙으로 사용할 기술들을 복사
cp /path/to/agent-skills/skills/test-driven-development/SKILL.md .cursor/rules/test-driven-development.md
cp /path/to/agent-skills/skills/code-review-and-quality/SKILL.md .cursor/rules/code-review-and-quality.md
cp /path/to/agent-skills/skills/incremental-implementation/SKILL.md .cursor/rules/incremental-implementation.md
```

이 디렉토리의 규칙들은 Cursor의 Context에 자동으로 로드됩니다.

### Option 2: .cursorrules File

필수 기술들을 포함하여 프로젝트 루트에 `.cursorrules` 파일을 생성하세요:

```bash
# 결합된 규칙 파일 생성
cat /path/to/agent-skills/skills/test-driven-development/SKILL.md > .cursorrules
echo "\n---\n" >> .cursorrules
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md >> .cursorrules
```

### Option 3: Notepads

Cursor의 Notepads 기능을 사용하면 재사용 가능한 Context를 저장할 수 있습니다. 자주 사용하는 각 기술에 대해 Notepad를 만드세요:

1. Cursor 열기 → Settings → Notepads
2. "swe: Test-Driven Development"라는 이름으로 새 Notepad 생성
3. `skills/test-driven-development/SKILL.md`의 내용 붙여넣기
4. 채팅에서 `@notepad swe: Test-Driven Development`로 참조

## Recommended Configuration

### Essential Skills (항상 로드)

`.cursor/rules/`에 다음을 추가하세요:

1. `test-driven-development.md` — TDD Workflow 및 Prove-It 패턴
2. `code-review-and-quality.md` — 5개 축 리뷰
3. `incremental-implementation.md` — 작고 검증 가능한 단위로 구축

### Phase-Specific Skills (Notepad로 로드)

상황에 따라 사용하는 기술들을 위해 Notepad를 만드세요:

- "swe: Spec Development" → `spec-driven-development/SKILL.md`
- "swe: Frontend UI" → `frontend-ui-engineering/SKILL.md`
- "swe: Security" → `security-and-hardening/SKILL.md`
- "swe: Performance" → `performance-optimization/SKILL.md`

관련 작업을 할 때 `@notepad`를 사용하여 참조하세요.

## Usage Tips

1. **모든 기술을 한꺼번에 로드하지 마세요** — Cursor에는 Context 제한이 있습니다. 2~3개의 기술을 Rule로 로드하고 나머지는 Notepad로 유지하세요.
2. **기술을 명시적으로 참조하세요** — Cursor가 로드된 규칙을 읽도록 "이 변경 사항에 대해 test-driven-development 규칙을 따라줘"라고 말하세요.
3. **리뷰를 위해 Agent를 사용하세요** — `agents/code-reviewer.md` 내용을 복사하고 Cursor에게 "이 코드 리뷰 프레임워크를 사용하여 이 Diff를 리뷰해줘"라고 말하세요.
4. **필요할 때 Reference를 로드하세요** — 성능 작업을 할 때 `@notepad performance-checklist`를 참조하거나 Checklist 내용을 붙여넣으세요.
