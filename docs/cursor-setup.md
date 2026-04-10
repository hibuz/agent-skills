# Cursor와 함께 agent-skills 사용

## 설정

### 옵션 1: 규칙 디렉터리(권장)

Cursor는 project-specific 규칙에 대해 `.cursor/rules/` 디렉터리를 지원합니다.

```bash
# Create the rules directory
mkdir -p .cursor/rules

# Copy skills you want as rules
cp /path/to/agent-skills/skills/test-driven-development/SKILL.md .cursor/rules/test-driven-development.md
cp /path/to/agent-skills/skills/code-review-and-quality/SKILL.md .cursor/rules/code-review-and-quality.md
cp /path/to/agent-skills/skills/incremental-implementation/SKILL.md .cursor/rules/incremental-implementation.md
```

이 디렉토리의 규칙은 Cursor의 컨텍스트에 자동으로 로드됩니다.

### 옵션 2:.cursorrules 파일

필수 skills가 인라인된 프로젝트 루트에 `.cursorrules` 파일을 만듭니다.

```bash
# Generate a combined rules file
cat /path/to/agent-skills/skills/test-driven-development/SKILL.md > .cursorrules
echo "\n---\n" >> .cursorrules
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md >> .cursorrules
```

### 옵션 3: 메모장

Cursor의 메모장 기능을 사용하면 재사용 가능한 컨텍스트를 저장할 수 있습니다. 자주 사용하는 각 skill에 대해 메모장을 만듭니다.

1. Cursor 열기 → 설정 → 메모장
2. "swe: Test-Driven Development"라는 이름의 새 메모장을 만듭니다.
3. `skills/test-driven-development/SKILL.md`의 내용을 붙여넣습니다.
4. `@notepad swe: Test-Driven Development`와의 채팅에서 참고하세요.

## 권장 구성

### 필수 Skills(항상 로드)

`.cursor/rules/`에 다음을 추가합니다.

1. `test-driven-development.md` — TDD workflow 및 증명 패턴
2. `code-review-and-quality.md` — 5축 검토
3. `incremental-implementation.md` - 검증 가능한 작은 조각의 Build

### 단계별 Skills(메모장으로 로드)

상황에 맞게 사용하는 skills에 대한 메모장을 만듭니다.

- "swe: 사양 개발" → `spec-driven-development/SKILL.md`
- "swe: 프런트엔드 UI" → `frontend-ui-engineering/SKILL.md`
- "swe: 보안" → `security-and-hardening/SKILL.md`
- "swe: Performance" → `performance-optimization/SKILL.md`

관련 작업을 할 때 `@notepad`로 참고하세요.

## 사용 팁

1. **모든 skills를 한 번에 로드하지 마세요** — Cursor에는 컨텍스트 제한이 있습니다. 2-3 skills를 규칙으로 로드하고 나머지는 메모장으로 유지합니다.
2. **skills를 명시적으로 참조** — Cursor에 "이 변경 사항에 대해서는 test-driven-development 규칙을 따르십시오"라고 지시하여 로드된 규칙을 읽도록 합니다.
3. **검토를 위해 agents 사용** — `agents/code-reviewer.md` 콘텐츠를 복사하고 Cursor에게 "이 코드 검토 프레임워크를 사용하여 이 차이점을 검토"하라고 지시합니다.
4. **요청 시 참조 로드** — performance 작업 시 `@notepad performance-checklist`를 참조하거나 체크리스트 내용을 붙여넣습니다.
