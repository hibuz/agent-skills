# Using agent-skills with Gemini CLI

## Setup

### Option 1: Install as Skills (권장)

Gemini CLI에는 `.gemini/skills/` 또는 `.agents/skills/` 디렉토리에서 `SKILL.md` 파일을 자동으로 발견하는 네이티브 기술 시스템이 있습니다. 각 기술은 작업과 일치할 때 필요에 따라 활성화됩니다.

**리포지토리에서 설치:**

```bash
gemini skills install https://github.com/addyosmani/agent-skills.git --path skills
```

**또는 로컬 클론에서 설치:**

```bash
git clone https://github.com/addyosmani/agent-skills.git
gemini skills install /path/to/agent-skills/skills/
```

**특정 워크스페이스에만 설치:**

```bash
gemini skills install /path/to/agent-skills/skills/ --scope workspace
```

워크스페이스 범위로 설치된 기술은 `.gemini/skills/` (또는 `.agents/skills/`)로 들어갑니다. 사용자 수준의 기술은 `~/.gemini/skills/`로 들어갑니다.

설치 후 다음 명령으로 확인하세요:

```
/skills list
```

Gemini CLI는 기술 이름과 설명을 Prompt에 자동으로 주입합니다. 일치하는 작업을 인식하면 전체 지침을 로드하기 전에 기술 활성화 권한을 요청합니다.

### Option 2: GEMINI.md (Persistent Context)

필요할 때 활성화하는 대신 항상 로드된 영구적인 프로젝트 Context로 사용하고 싶은 기술은 프로젝트의 `GEMINI.md`에 추가하세요:

```bash
# 핵심 기술들을 영구 Context로 포함하여 GEMINI.md 생성
cat /path/to/agent-skills/skills/incremental-implementation/SKILL.md > GEMINI.md
echo -e "\n---\n" >> GEMINI.md
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md >> GEMINI.md
```

별도의 파일에서 가져오도록 모듈화할 수도 있습니다:

```markdown
# Project Instructions

@skills/test-driven-development/SKILL.md
@skills/incremental-implementation/SKILL.md
```

`/memory show`를 사용하여 로드된 Context를 확인하고, 변경 후에는 `/memory reload`를 사용하여 새로고침하세요.

> **Skills vs GEMINI.md:** Skills는 관련이 있을 때만 활성화되는 온디맨드 전문 지식으로, Context 윈도우를 깨끗하게 유지해 줍니다. GEMINI.md는 모든 Prompt에 대해 로드되는 영구 Context를 제공합니다. 단계별 Workflow에는 Skills를 사용하고, 항상 켜져 있어야 하는 프로젝트 규칙에는 GEMINI.md를 사용하세요.

## Recommended Configuration

### Always-On (GEMINI.md)

매 세션마다 영구 Context로 다음을 추가하세요:

- `incremental-implementation` — 작고 검증 가능한 단위로 구축
- `code-review-and-quality` — 5개 축 리뷰

### On-Demand (Skills)

관련이 있을 때만 활성화되도록 다음을 기술로 설치하세요:

- `test-driven-development` — 로직을 구현하거나 버그를 수정할 때 활성화됨
- `spec-driven-development` — 새로운 프로젝트나 기능을 시작할 때 활성화됨
- `frontend-ui-engineering` — UI를 구축할 때 활성화됨
- `security-and-hardening` — 보안 리뷰 중에 활성화됨
- `performance-optimization` — 성능 작업 중에 활성화됨

## Advanced Configuration

### MCP Integration

이 팩의 많은 기술은 환경과 상호작용하기 위해 [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) 도구를 활용합니다. 예시:

- `browser-testing-with-devtools`는 `chrome-devtools` MCP 확장을 사용합니다.
- `performance-optimization`은 성능 관련 MCP 도구의 이점을 얻을 수 있습니다.

이를 사용하려면 Gemini CLI 설정(`~/.gemini/config.json`)에 관련 MCP 확장이 설치되어 있는지 확인하세요.

### Session Hooks

Gemini CLI는 세션 Lifecycle Hook을 지원합니다. 이를 사용하여 세션 시작 시 자동으로 Context를 주입하거나 검증 스크립트를 실행할 수 있습니다.

다른 도구의 `agent-skills` 경험을 재현하기 위해, 사용 가능한 기술을 상기시키거나 Meta-skill을 로드하는 `SessionStart` Hook을 구성할 수 있습니다.

### Explicit Context Loading

Prompt에서 `@` 기호를 사용하여 현재 세션에 모든 기술을 명시적으로 로드할 수 있습니다:

```markdown
이 수정을 구현하기 위해 @skills/test-driven-development/SKILL.md 기술을 사용해줘.
```

이는 자동 발견을 기다리지 않고 특정 Workflow를 따르도록 하고 싶을 때 유용합니다.

## Usage Tips

1. **GEMINI.md보다 Skills를 선호하세요** — Skills는 필요할 때만 활성화되어 Context 윈도우를 집중된 상태로 유지합니다. 항상 로드되기를 원하는 경우에만 GEMINI.md에 기술을 넣으세요.
2. **기술 설명(Description)이 중요합니다** — 각 SKILL.md의 Frontmatter에는 Agent에게 언제 활성화할지 알려주는 `description` 필드가 있습니다. 이 리포지토리의 설명은 기술이 하는 일과 트리거되어야 하는 시점을 모두 명확히 명시함으로써 모든 지원 도구(Claude Code, Gemini CLI 등)에서 자동 발견이 되도록 최적화되어 있습니다.
3. **리뷰를 위해 Agent를 사용하세요** — 구조화된 코드 리뷰를 요청할 때 `agents/code-reviewer.md` 내용을 복사하세요.
4. **Reference와 결합하세요** — 테스트나 성능과 같은 특정 품질 영역을 작업할 때 `references/`의 Checklist를 참조하세요.
