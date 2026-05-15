# Cursor에서 agent-skills 사용하기

## 설정

### 옵션 1: Rules 디렉터리 (권장)

Cursor는 프로젝트별 rules를 위한 `.cursor/rules/` 디렉터리를 지원합니다:

```bash
# rules 디렉터리 생성
mkdir -p .cursor/rules

# rules로 쓸 skill 복사
cp /path/to/agent-skills/skills/test-driven-development/SKILL.md .cursor/rules/test-driven-development.md
cp /path/to/agent-skills/skills/code-review-and-quality/SKILL.md .cursor/rules/code-review-and-quality.md
cp /path/to/agent-skills/skills/incremental-implementation/SKILL.md .cursor/rules/incremental-implementation.md
```

이 디렉터리의 rules는 Cursor의 context에 자동으로 로드됩니다.

### 옵션 2: .cursorrules 파일

프로젝트 루트에 필수 skill을 인라인으로 넣은 `.cursorrules` 파일을 만듭니다:

```bash
# 결합된 rules 파일 생성
cat /path/to/agent-skills/skills/test-driven-development/SKILL.md > .cursorrules
echo "\n---\n" >> .cursorrules
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md >> .cursorrules
```

## 권장 구성

### 필수 Skill (항상 로드)

다음을 `.cursor/rules/`에 추가하세요:

1. `test-driven-development.md` — TDD 워크플로와 Prove-It 패턴
2. `code-review-and-quality.md` — 5축 리뷰
3. `incremental-implementation.md` — 작고 검증 가능한 슬라이스로 빌드

### 단계별 Skill (필요 시 로드)

단계별 작업을 위해 필요에 따라 추가 rule 파일을 만드세요:

- `spec-development.md` -> `spec-driven-development/SKILL.md`
- `frontend-ui.md` -> `frontend-ui-engineering/SKILL.md`
- `security.md` -> `security-and-hardening/SKILL.md`
- `performance.md` -> `performance-optimization/SKILL.md`

관련 작업을 할 때 이것들을 `.cursor/rules/`에 추가하고, context 한도를 관리하기 위해 끝나면 제거하세요.

## 사용 팁

1. **모든 skill을 한 번에 로드하지 마세요** - Cursor에는 context 한도가 있습니다. 필수 skill 2-3개를 rules로 로드하고 단계별 skill은 필요에 따라 추가하세요.
2. **skill을 명시적으로 참조하세요** - Cursor에게 "이 변경에는 test-driven-development rules를 따르라"고 말해 로드된 rules를 읽도록 하세요.
3. **리뷰에 agent 사용** - `agents/code-reviewer.md` 내용을 복사하고 Cursor에게 "이 코드 리뷰 프레임워크로 이 diff를 리뷰하라"고 말하세요.
4. **참조는 필요 시 로드** - 성능 작업 시 `performance.md`를 `.cursor/rules/`에 추가하거나 체크리스트 내용을 직접 붙여넣으세요.
