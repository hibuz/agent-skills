# Gemini CLI와 함께 agent-skills 사용하기

## 설정

### 옵션 1: Skills로 설치(권장)

Gemini CLI에는 auto-discovers `SKILL.md`가 `.gemini/skills/` 또는 `.agents/skills/` 디렉터리에 파일을 저장하는 기본 skills 시스템이 있습니다. 각 skill는 작업과 일치할 때 요청 시 활성화됩니다.

**repo에서 설치:**

```bash
gemini skills install https://github.com/addyosmani/agent-skills.git --path skills
```

**또는 로컬 클론에서 설치:**

```bash
git clone https://github.com/addyosmani/agent-skills.git
gemini skills install /path/to/agent-skills/skills/
```

**특정 작업공간에만 설치:**

```bash
gemini skills install /path/to/agent-skills/skills/ --scope workspace
```

작업 공간 범위에 설치된 Skills는 `.gemini/skills/`(또는 `.agents/skills/`)로 이동합니다. 사용자 수준 skills는 `~/.gemini/skills/`로 이동합니다.

설치가 완료되면 다음을 통해 확인하세요.

```
/skills list
```

Gemini CLI는 skill 이름과 설명을 프롬프트에 자동으로 삽입합니다. 일치하는 작업을 인식하면 전체 지침을 로드하기 전에 skill를 활성화할 수 있는 권한을 요청합니다.

### 옵션 2: GEMINI.md(영구 컨텍스트)

skills의 경우 항상 영구 프로젝트 컨텍스트(on-demand 활성화가 아닌)로 로드하려면 프로젝트의 `GEMINI.md`에 추가하세요.

```bash
# Create GEMINI.md with core skills as persistent context
cat /path/to/agent-skills/skills/incremental-implementation/SKILL.md > GEMINI.md
echo -e "\n---\n" >> GEMINI.md
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md >> GEMINI.md
```

별도의 파일에서 가져와서 모듈화할 수도 있습니다.

```markdown
# Project Instructions

@skills/test-driven-development/SKILL.md
@skills/incremental-implementation/SKILL.md
```

로드된 컨텍스트를 확인하려면 `/memory show`를 사용하고 변경 후 새로 고치려면 `/memory reload`를 사용하세요.

> **Skills 대 GEMINI.md:** Skills는 관련성이 있는 경우에만 활성화되어 컨텍스트 창을 깨끗하게 유지하는 on-demand 전문 기술입니다. GEMINI.md는 모든 프롬프트에 대해 로드된 영구 컨텍스트를 제공합니다. phase-specific workflows에는 skills를 사용하고 always-on 프로젝트 규칙에는 GEMINI.md를 사용합니다.

## 권장 구성

### 상시 가동(GEMINI.md)

모든 세션에 대해 영구 컨텍스트로 다음을 추가합니다.

- `incremental-implementation` — 검증 가능한 작은 조각의 Build
- `code-review-and-quality` — 5축 검토

### 온디맨드(Skills)

관련된 경우에만 활성화되도록 skills로 설치하십시오.

- `test-driven-development` — 로직을 구현하거나 버그를 수정할 때 활성화됩니다.
- `spec-driven-development` — 새 프로젝트나 기능을 시작할 때 활성화됩니다.
- `frontend-ui-engineering` — build가 UI일 때 활성화됩니다.
- `security-and-hardening` — 보안 검토 중에 활성화됩니다.
- `performance-optimization` — performance 작업 중에 활성화됩니다.

## 고급 구성

### MCP 통합

이 팩에 포함된 많은 skills는 [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) 도구를 활용하여 환경과 상호 작용합니다. 예를 들면:

- `browser-testing-with-devtools`는 `chrome-devtools` MCP 확장을 사용합니다.
- `performance-optimization`는 performance-related MCP 도구의 이점을 누릴 수 있습니다.

이를 활성화하려면 Gemini CLI 구성(`~/.gemini/config.json`)에 관련 MCP 확장이 설치되어 있는지 확인하세요.

### 세션 후크

Gemini CLI는 세션 수명주기 후크를 지원합니다. 이를 사용하여 세션 시작 시 자동으로 컨텍스트를 삽입하거나 유효성 검사 스크립트를 실행할 수 있습니다.

다른 도구에서 `agent-skills` 경험을 복제하려면 사용 가능한 skills를 상기시키거나 meta-skill를 로드하는 `SessionStart` 후크를 구성할 수 있습니다.

### 명시적 컨텍스트 로딩

프롬프트에서 `@` 기호를 사용하여 skill를 참조하여 현재 세션에 명시적으로 로드할 수 있습니다.

```markdown
Use the @skills/test-driven-development/SKILL.md skill to implement this fix.
```

이는 auto-discovery를 기다리지 않고 특정 workflow를 따르려는 경우에 유용합니다.

## 사용 팁

1. **GEMINI.md보다 skills를 선호합니다** — Skills는 요청 시 활성화하고 컨텍스트 창에 집중합니다. 항상 로드하려면 GEMINI.md에 skills만 넣으세요.
2. **Skill 설명이 중요합니다** — 각 SKILL.md에는 frontmatter에 agents를 활성화할 시기를 알려주는 `description` 필드가 있습니다. 이 repo의 설명은 skill가 수행하는 *무엇*과 트리거되어야 하는 *언제*를 모두 명확하게 명시하여 지원되는 모든 도구(Claude Code, Gemini CLI 등) 전반에 걸쳐 auto-discovery에 최적화되어 있습니다.
3. **검토를 위해 agents 사용** — 구조화된 코드 검토를 요청할 때 `agents/code-reviewer.md` 콘텐츠를 복사합니다.
4. **참조와 결합** — 테스트 또는 performance와 같은 특정 품질 영역에서 작업할 때 `references/`의 참조 체크리스트입니다.
