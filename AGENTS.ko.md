# AGENTS.md

이 파일은 이 repository의 코드를 다룰 때 AI 코딩 agent(Claude Code, Cursor, Copilot, Antigravity 등)에게 가이드를 제공합니다.

## Repository 개요

senior 소프트웨어 엔지니어를 위한 Claude.ai 및 Claude Code용 skill 모음입니다. skill은 Claude와 코딩 agent의 역량을 확장하는, 패키징된 지침과 스크립트입니다.

## OpenCode 연동

OpenCode는 `skill` 도구와 이 repository의 `/skills` 디렉터리를 기반으로 하는 **skill 주도 실행 모델(skill-driven execution model)**을 사용합니다.

### 핵심 규칙

- 작업이 어떤 skill에 해당한다면 반드시 그 skill을 invoke해야 합니다
- skill은 `skills/<skill-name>/SKILL.md`에 위치합니다
- 적용 가능한 skill이 있다면 직접 구현하지 마세요
- 항상 skill 지침을 정확히 따르세요 (부분적으로만 적용하지 마세요)

### Intent → Skill 매핑

agent는 사용자 의도를 자동으로 skill에 매핑해야 합니다:

- 기능 / 새 기능 추가 → `spec-driven-development`, 이어서 `incremental-implementation`, `test-driven-development`
- 계획 / 분해 → `planning-and-task-breakdown`
- 버그 / 실패 / 예기치 않은 동작 → `debugging-and-error-recovery`
- 코드 리뷰 → `code-review-and-quality`
- 리팩터링 / 단순화 → `code-simplification`
- API 또는 인터페이스 설계 → `api-and-interface-design`
- UI 작업 → `frontend-ui-engineering`

### 라이프사이클 매핑 (암묵적 커맨드)

OpenCode는 `/spec`이나 `/plan` 같은 슬래시 커맨드를 지원하지 않습니다.

대신 agent는 내부적으로 다음 라이프사이클을 따라야 합니다:

- DEFINE → `spec-driven-development`
- PLAN → `planning-and-task-breakdown`
- BUILD → `incremental-implementation` + `test-driven-development`
- VERIFY → `debugging-and-error-recovery`
- REVIEW → `code-review-and-quality`
- SHIP → `shipping-and-launch`

### 실행 모델

모든 요청에 대해:

1. 적용 가능한 skill이 있는지 판단합니다 (가능성이 1%라도 있다면)
2. `skill` 도구를 사용해 적절한 skill을 invoke합니다
3. skill 워크플로를 엄격히 따릅니다
4. 필요한 단계(spec, plan 등)가 완료된 후에만 구현으로 진행합니다

### 합리화 방지(Anti-Rationalization)

다음과 같은 생각은 잘못된 것이며 무시해야 합니다:

- "이건 skill을 쓰기엔 너무 작아"
- "그냥 빠르게 구현하면 돼"
- "먼저 context를 모아야지"

올바른 동작:

- 항상 먼저 skill을 확인하고 사용하세요

이를 통해 OpenCode가 전체 워크플로 강제를 갖춘 Claude Code와 유사하게 동작하도록 보장합니다.

## 오케스트레이션: Persona, Skill, Command

이 repo에는 세 가지 조합 가능한 레이어가 있습니다. 각각 역할이 다르므로 혼동해서는 안 됩니다:

- **Skill** (`skills/<name>/SKILL.md`) — 단계와 종료 기준을 가진 워크플로. *어떻게(how)*에 해당. intent가 매칭되면 거쳐야 하는 필수 단계.
- **Persona** (`agents/<role>.md`) — 관점과 출력 형식을 가진 역할. *누가(who)*에 해당.
- **슬래시 커맨드** (`.claude/commands/*.md`) — 사용자 대면 진입점. *언제(when)*에 해당. 오케스트레이션 레이어.

조합 규칙: **사용자(또는 슬래시 커맨드)가 오케스트레이터입니다. persona는 다른 persona를 invoke하지 않습니다.** persona는 skill을 invoke할 수 있습니다.

이 repo가 인정하는 유일한 다중 persona 오케스트레이션 패턴은 **병렬 fan-out 후 merge 단계(parallel fan-out with a merge step)**입니다 — `/ship`이 `code-reviewer`, `security-auditor`, `test-engineer`를 동시에 실행하고 그 보고서를 종합하는 데 사용합니다. 어떤 persona를 호출할지 결정하는 "router" persona를 만들지 마세요. 그것은 슬래시 커맨드와 intent 매핑의 역할입니다.

결정 매트릭스는 [agents/README.ko.md](agents/README.ko.md)를, 전체 패턴 카탈로그는 [references/orchestration-patterns.ko.md](references/orchestration-patterns.ko.md)를 참고하세요.

**Claude Code 상호운용:** `agents/`의 persona들은 Claude Code subagent로 동작하며(이 플러그인의 `agents/` 디렉터리에서 자동 검색됨), Agent Teams의 팀원으로도 동작합니다(스폰 시 이름으로 참조). 두 가지 플랫폼 제약이 우리 규칙과 부합합니다: subagent는 다른 subagent를 스폰할 수 없고, 팀은 중첩될 수 없습니다. 플러그인 agent는 `hooks`, `mcpServers`, `permissionMode` frontmatter 필드를 조용히 무시합니다.

## 새 Skill 만들기

### 디렉터리 구조

```
skills/
  {skill-name}/           # kebab-case 디렉터리 이름
    SKILL.md              # 필수: skill 정의
    scripts/              # 필수: 실행 가능한 스크립트
      {script-name}.sh    # Bash 스크립트 (권장)
  {skill-name}.zip        # 필수: 배포용 패키지
```

### 명명 규칙

- **Skill 디렉터리**: `kebab-case` (예: `web-quality`)
- **SKILL.md**: 항상 대문자, 항상 정확히 이 파일명
- **스크립트**: `kebab-case.sh` (예: `deploy.sh`, `fetch-logs.sh`)
- **Zip 파일**: 디렉터리 이름과 정확히 일치해야 함: `{skill-name}.zip`

### SKILL.md 형식

```markdown
---
name: {skill-name}
description: {skill이 무엇을 하는지 한 문장으로 설명하고, 이어서 하나 이상의 "Use when" 트리거 조건을 기술. "Deploy my app"이나 "Check logs" 같은 트리거 문구를 도움이 될 때 포함.}
---

# {Skill Title}

{skill이 무엇을 하고 왜 중요한지에 대한 간략한 개요.}

## How It Works

{skill 워크플로를 설명하는 번호 매긴 목록}

`Workflow`, `Core Process`, `When to Use` 같은 동등한 제목도 동일한 구조를 명확히 전달한다면 괜찮습니다.

## Usage (선택)

이 섹션은 skill이 `scripts/` 아래에 실행 가능한 헬퍼를 제공할 때만 포함하세요. Markdown만으로 된 skill은 섹션과 디렉터리를 모두 생략할 수 있습니다.

```bash
bash /mnt/skills/user/{skill-name}/scripts/{script}.sh [args]
```

**Arguments:**
- `arg1` - 설명 (기본값 X)

**Examples:**
{흔한 사용 패턴 2-3개 제시}

## Output

{사용자가 보게 될 예시 출력 제시}

## Present Results to User

{사용자에게 결과를 제시할 때 Claude가 따라야 할 형식 템플릿}

## Troubleshooting

{흔한 문제와 해결책, 특히 네트워크/권한 오류}
```

### Context 효율을 위한 모범 사례

skill은 on-demand로 로드됩니다 — 시작 시에는 skill 이름과 description만 로드됩니다. 전체 `SKILL.md`는 agent가 해당 skill이 관련 있다고 판단할 때만 context에 로드됩니다. context 사용을 최소화하려면:

- **SKILL.md를 500줄 이하로 유지** — 상세 참조 자료는 별도 파일로 분리하세요
- **구체적인 description 작성** — agent가 언제 skill을 활성화할지 정확히 알 수 있게 합니다
- **점진적 공개(progressive disclosure) 사용** — 필요할 때만 읽히는 보조 파일을 참조하세요
- **인라인 코드보다 스크립트 선호** — 스크립트 실행은 context를 소비하지 않습니다 (출력만 소비)
- **파일 참조는 한 단계 깊이만 동작** — SKILL.md에서 보조 파일로 직접 링크하세요

### 스크립트 요구사항

- `#!/bin/bash` shebang 사용
- fail-fast 동작을 위해 `set -e` 사용
- 상태 메시지는 stderr로: `echo "Message" >&2`
- 기계가 읽을 수 있는 출력(JSON)은 stdout으로
- 임시 파일에 대한 cleanup trap 포함
- 스크립트 경로는 `/mnt/skills/user/{skill-name}/scripts/{script}.sh`로 참조

### Zip 패키지 만들기

skill을 생성하거나 업데이트한 후:

```bash
cd skills
zip -r {skill-name}.zip {skill-name}/
```

### 최종 사용자 설치

사용자를 위해 다음 두 가지 설치 방법을 문서화하세요:

**Claude Code:**
```bash
cp -r skills/{skill-name} ~/.claude/skills/
```

**claude.ai:**
project knowledge에 skill을 추가하거나 SKILL.md 내용을 대화에 붙여넣으세요.

skill이 네트워크 접근을 필요로 한다면, 사용자에게 `claude.ai/settings/capabilities`에서 필요한 도메인을 추가하도록 안내하세요.
