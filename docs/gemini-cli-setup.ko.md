# Gemini CLI에서 agent-skills 사용하기

## 설정

### 옵션 1: Skill로 설치 (권장)

Gemini CLI에는 `.gemini/skills/` 또는 `.agents/skills/` 디렉터리의 `SKILL.md` 파일을 자동 검색하는 네이티브 skill 시스템이 있습니다. 각 skill은 작업에 매칭될 때 on-demand로 활성화됩니다.

**repo에서 설치:**

```bash
gemini skills install https://github.com/addyosmani/agent-skills.git --path skills
```

**또는 로컬 clone에서 설치:**

```bash
git clone https://github.com/addyosmani/agent-skills.git
gemini skills install /path/to/agent-skills/skills/
```

**특정 workspace에만 설치:**

```bash
gemini skills install /path/to/agent-skills/skills/ --scope workspace
```

workspace 스코프로 설치된 skill은 `.gemini/skills/`(또는 `.agents/skills/`)로 들어갑니다. 사용자 레벨 skill은 `~/.gemini/skills/`로 들어갑니다.

설치 후 다음으로 확인하세요:

```
/skills list
```

Gemini CLI는 skill 이름과 description을 프롬프트에 자동으로 주입합니다. 매칭되는 작업을 인식하면 전체 지침을 로드하기 전에 skill을 활성화할 권한을 요청합니다.

### 옵션 2: GEMINI.md (지속적 Context)

(on-demand 활성화가 아니라) 지속적인 프로젝트 context로 항상 로드하고 싶은 skill은 프로젝트의 `GEMINI.md`에 추가하세요:

```bash
# 핵심 skill을 지속적 context로 하는 GEMINI.md 생성
cat /path/to/agent-skills/skills/incremental-implementation/SKILL.md > GEMINI.md
echo -e "\n---\n" >> GEMINI.md
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md >> GEMINI.md
```

별도 파일에서 import해 모듈화할 수도 있습니다:

```markdown
# Project Instructions

@skills/test-driven-development/SKILL.md
@skills/incremental-implementation/SKILL.md
```

로드된 context를 확인하려면 `/memory show`를, 변경 후 새로고침하려면 `/memory reload`를 사용하세요.

> **Skills vs GEMINI.md:** skill은 관련 있을 때만 활성화되는 on-demand 전문성으로, context window를 깨끗하게 유지합니다. GEMINI.md는 모든 프롬프트에 로드되는 지속적 context를 제공합니다. 단계별 워크플로에는 skill을, 항상 켜져 있는 프로젝트 규약에는 GEMINI.md를 사용하세요.

## 권장 구성

### 항상 켜짐 (GEMINI.md)

모든 세션의 지속적 context로 다음을 추가하세요:

- `incremental-implementation` — 작고 검증 가능한 슬라이스로 빌드
- `code-review-and-quality` — 5축 리뷰

### On-Demand (Skill)

관련 있을 때만 활성화되도록 다음을 skill로 설치하세요:

- `test-driven-development` — 로직 구현이나 버그 수정 시 활성화
- `spec-driven-development` — 새 프로젝트나 기능 시작 시 활성화
- `frontend-ui-engineering` — UI 빌드 시 활성화
- `security-and-hardening` — 보안 리뷰 시 활성화
- `performance-optimization` — 성능 작업 시 활성화

## 고급 구성

### MCP 연동

이 팩의 많은 skill은 환경과 상호작용하기 위해 [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) 도구를 활용합니다. 예를 들어:

- `browser-testing-with-devtools`는 `chrome-devtools` MCP 확장을 사용합니다.
- `performance-optimization`은 성능 관련 MCP 도구의 도움을 받을 수 있습니다.

이를 활성화하려면, Gemini CLI 설정(`~/.gemini/config.json`)에 관련 MCP 확장이 설치되어 있는지 확인하세요.

### Session Hook

Gemini CLI는 세션 라이프사이클 hook을 지원합니다. 이를 사용해 세션 시작 시 자동으로 context를 주입하거나 검증 스크립트를 실행할 수 있습니다.

다른 도구의 `agent-skills` 경험을 재현하려면, 사용 가능한 skill을 상기시키거나 메타 skill을 로드하는 `SessionStart` hook을 구성할 수 있습니다.

### 명시적 Context 로딩

프롬프트에서 `@` 기호로 참조해 현재 세션에 어떤 skill이든 명시적으로 로드할 수 있습니다:

```markdown
Use the @skills/test-driven-development/SKILL.md skill to implement this fix.
```

자동 검색을 기다리지 않고 특정 워크플로가 따라지도록 보장하고 싶을 때 유용합니다.

## 슬래시 커맨드

이 repo는 개발 라이프사이클에 매핑되는 7개의 슬래시 커맨드를 `.gemini/commands/` 아래에 제공합니다. 프로젝트 루트에서 실행하면 Gemini CLI가 자동으로 검색합니다.

| 커맨드 | 무엇을 하는가 |
|---------|--------------|
| `/spec` | 코드 작성 전에 구조화된 spec 작성 |
| `/planning` | 작업을 작고 검증 가능한 작업으로 분해 |
| `/build` | 다음 작업을 점진적으로 구현 |
| `/test` | TDD 워크플로 실행 — red, green, refactor |
| `/review` | 5축 코드 리뷰 |
| `/code-simplify` | 동작 변경 없이 복잡도 감소 |
| `/ship` | 병렬 persona fan-out을 통한 런칭 전 체크리스트 |

각 커맨드는 해당 skill을 자동으로 invoke합니다 — 수동 skill 로딩이 필요 없습니다.

> **참고:** `/plan` 대신 `/planning`을 사용하세요 — `/plan`은 Gemini CLI 내부 커맨드 이름과 충돌합니다.

## 사용 팁

1. **GEMINI.md보다 skill 선호** — skill은 on-demand로 활성화되어 context window를 집중되게 유지합니다. 항상 로드하고 싶을 때만 skill을 GEMINI.md에 넣으세요.
2. **skill description이 중요** — 각 SKILL.md는 frontmatter에 `description` 필드가 있어 agent에게 언제 활성화할지 알려줍니다. 이 repo의 description은 skill이 *무엇*을 하고 *언제* 트리거되어야 하는지를 명확히 기술해 모든 지원 도구(Claude Code, Gemini CLI 등)에서 자동 검색에 최적화되어 있습니다.
3. **리뷰에 agent 사용** — 구조화된 코드 리뷰를 요청할 때 `agents/code-reviewer.md` 내용을 복사하세요.
4. **참조와 결합** — 테스트나 성능 같은 특정 품질 영역 작업 시 `references/`의 체크리스트를 참조하세요.
