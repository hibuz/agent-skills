# AGENTS.md

이 파일은 이 repository의 코드로 작업할 때 agents(Claude Code, Cursor, Copilot, Antigravity 등)를 코딩하는 AI에 대한 guidance를 제공합니다.

## Repository 개요

수석 소프트웨어 엔지니어를 위한 Claude.ai 및 Claude Code용 skills 컬렉션입니다. Skills는 Claude와 코딩 agents 기능을 확장하는 패키지 지침 및 스크립트입니다.

## OpenCode 통합

OpenCode는 `skill` 도구와 이 repository의 `/skills` 디렉터리로 구동되는 **skill-driven 실행 모델**을 사용합니다.

### 핵심 규칙

- 작업이 skill와 일치하면 MUST를 호출합니다.
- Skills는 `skills/<skill-name>/SKILL.md`에 위치합니다.
- skill가 적용되는 경우 직접 구현하지 마십시오.
- 항상 skill 지침을 정확하게 따르십시오(부분적으로 적용하지 마십시오).

### 의도 → Skill 매핑

agent는 사용자 의도를 skills에 자동으로 매핑해야 합니다.

- 특징/새로운 기능 → `spec-driven-development`, 그 다음 `incremental-implementation`, `test-driven-development`
- 기획/구분 → `planning-and-task-breakdown`
- 버그/실패/예상치 못한 동작 → `debugging-and-error-recovery`
- 코드 리뷰 → `code-review-and-quality`
- 리팩토링/단순화 → `code-simplification`
- API 또는 인터페이스 디자인 → `api-and-interface-design`
- UI 작업 → `frontend-ui-engineering`

### 수명 주기 매핑(암시적 명령)

OpenCode는 `/spec` 또는 `/plan`와 같은 slash commands를 지원하지 않습니다.

대신 agent는 내부적으로 다음 수명 주기를 따라야 합니다.

- DEFINE → `spec-driven-development`
- PLAN → `planning-and-task-breakdown`
- BUILD → `incremental-implementation` + `test-driven-development`
- VERIFY → `debugging-and-error-recovery`
- REVIEW → `code-review-and-quality`
- SHIP → `shipping-and-launch`

### 실행 모델

모든 요청에 대해:

1. skill가 적용되는지 확인합니다(1%의 확률이라도).
2. `skill` 도구를 사용하여 적절한 skill를 호출합니다.
3. skill workflow를 엄격히 따르십시오.
4. 필수 단계(사양, 계획 등)가 완료된 후에만 구현을 진행하세요.

### 반합리화

다음 생각은 잘못된 것이므로 무시해야 합니다.

- "skill에 비해 너무 작습니다."
- "이것을 quickly로 구현할 수 있습니다."
- "먼저 맥락을 수집하겠다"

올바른 행동:

- 항상 skills를 먼저 확인하고 사용하세요.

이를 통해 OpenCode는 전체 workflow 적용을 통해 Claude Code와 유사하게 작동합니다.

## 새 Skill 만들기

### 디렉토리 구조

```
skills/
  {skill-name}/           # kebab-case directory name
    SKILL.md              # Required: skill definition
    scripts/              # Required: executable scripts
      {script-name}.sh    # Bash scripts (preferred)
  {skill-name}.zip        # Required: packaged for distribution
```

### 명명 규칙

- **Skill 디렉터리**: `kebab-case`(예: `web-quality`)
- **SKILL.md**: 항상 대문자, 항상 정확한 파일 이름
- **스크립트**: `kebab-case.sh`(예: `deploy.sh`, `fetch-logs.sh`)
- **Zip 파일**: 디렉터리 이름과 정확히 일치해야 합니다: `{skill-name}.zip`

### SKILL.md Format

```markdown
---
name: {skill-name}
description: {One sentence describing when to use this skill. Include trigger phrases like "Deploy my app", "Check logs", etc.}
---

# {Skill Title}

{Brief description of what the skill does.}

## How It Works

{Numbered list explaining the skill's workflow}

## Usage

```bash
bash /mnt/skills/user/{skill-name}/scripts/{script}.sh [args]
```

**Arguments:**
- `arg1` - Description (defaults to X)

**Examples:**
{Show 2-3 common usage patterns}

## Output

{Show example output users will see}

## Present Results to User

{Template for how Claude should format results when presenting to users}

## Troubleshooting

{Common issues and solutions, especially network/permissions errors}
```

### 상황 효율성을 위한 모범 사례

Skills가 로드됩니다. on-demand — 시작 시 skill 이름과 설명만 로드됩니다. 전체 `SKILL.md`는 agent가 skill가 관련이 있다고 결정할 때만 컨텍스트에 로드됩니다. 컨텍스트 사용을 최소화하려면 다음을 수행하십시오.

- **SKILL.md를 500줄 미만으로 유지** — 자세한 참조 자료를 별도의 파일에 넣습니다.
- **구체적인 설명 작성** — agent가 skill를 활성화할 시기를 정확히 알 수 있도록 도와줍니다.
- **점진적 공개 사용** — 필요할 때만 읽을 수 있는 지원 파일을 참조하세요.
- **인라인 코드보다 스크립트를 선호** — 스크립트 실행은 컨텍스트를 소비하지 않습니다(출력만 소비함).
- **파일 참조는 한 수준 깊이로 작동** — SKILL.md에서 지원 파일로 직접 연결

### 스크립트 요청 uirements

- `#!/bin/bash` 셰뱅을 사용하세요.
- fail-fast 동작에는 `set -e`를 사용하세요.
- stderr에 상태 메시지 쓰기: `echo "Message" >&2`
- machine-readable 출력(JSON)을 stdout에 씁니다.
- 임시 파일에 대한 정리 트랩 포함
- 스크립트 경로를 `/mnt/skills/user/{skill-name}/scripts/{script}.sh`로 참조하세요.

### Zip 패키지 만들기

skill를 생성하거나 업데이트한 후:

```bash
cd skills
zip -r {skill-name}.zip {skill-name}/
```

### 최종 사용자 설치

사용자를 위해 다음 두 가지 설치 방법을 문서화하십시오.

**Claude Code:**
```bash
cp -r skills/{skill-name} ~/.claude/skills/
```

**claude.ai:**
프로젝트 지식에 skill를 추가하거나 대화에 SKILL.md 내용을 붙여넣으세요.

skill가 네트워크 액세스를 요구하는 경우 사용자에게 `claude.ai/settings/capabilities`에 required domains를 추가하도록 지시합니다.
