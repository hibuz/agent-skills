# 에이전트 가이드 (AGENTS.md)

이 파일은 이 저장소의 코드로 작업하는 AI 코딩 에이전트(Claude Code, Cursor, Copilot, Antigravity 등)에게 지침을 제공합니다.

## 저장소 개요

시니어 소프트웨어 엔지니어를 위한 Claude.ai 및 Claude Code용 스킬 모음입니다. 스킬은 Claude 및 코딩 에이전트의 기능을 확장하는 패키지화된 지침과 스크립트입니다.

## OpenCode 통합

OpenCode는 `skill` 도구와 이 저장소의 `/skills` 디렉토리를 기반으로 하는 **스킬 주도 실행 모델(skill-driven execution model)**을 사용합니다.

### 핵심 규칙

- 작업이 스킬과 일치하면 반드시 해당 스킬을 호출해야 합니다.
- 스킬은 `skills/<skill-name>/SKILL.ko.md`에 위치합니다.
- 스킬이 적용 가능한 경우 직접 구현하지 마세요.
- 항상 스킬 지침을 정확하게 따르세요 (부분적으로 적용하지 마세요).

### 의도 → 스킬 매핑 (Intent → Skill Mapping)

에이전트는 사용자의 의도를 스킬에 자동으로 매핑해야 합니다:

- 기능 / 새로운 기능 → `spec-driven-development`, 이후 `incremental-implementation`, `test-driven-development`
- 계획 / 세부 분해 → `planning-and-task-breakdown`
- 버그 / 실패 / 예상치 못한 동작 → `debugging-and-error-recovery`
- 코드 리뷰 → `code-review-and-quality`
- 리팩토링 / 단순화 → `code-simplification`
- API 또는 인터페이스 설계 → `api-and-interface-design`
- UI 작업 → `frontend-ui-engineering`

### 생명주기 매핑 (Implicit Commands)

OpenCode는 `/spec` 이나 `/plan` 과 같은 슬래시 명령어를 지원하지 않습니다.

대신 에이전트는 내부적으로 다음 생명주기를 따라야 합니다:

- 정의 (DEFINE) → `spec-driven-development`
- 계획 (PLAN) → `planning-and-task-breakdown`
- 구축 (BUILD) → `incremental-implementation` + `test-driven-development`
- 검증 (VERIFY) → `debugging-and-error-recovery`
- 리뷰 (REVIEW) → `code-review-and-quality`
- 배포 (SHIP) → `shipping-and-launch`

### 실행 모델

모든 요청에 대해:

1. 적용 가능한 스킬이 있는지 확인합니다 (1%의 확률이라도 있는 경우).
2. `skill` 도구를 사용하여 적절한 스킬을 호출합니다.
3. 스킬 워크플로우를 엄격히 따릅니다.
4. 필요한 단계(스펙, 계획 등)가 완료된 후에만 구현을 진행합니다.

### 안티-합리화 (Anti-rationalization)

다음과 같은 생각은 잘못된 것이며 무시해야 합니다:

- "이것은 스킬을 쓰기에는 너무 작다"
- "그냥 빨리 구현할 수 있다"
- "먼저 컨텍스트를 수집하겠다"

올바른 동작:

- 항상 스킬을 먼저 확인하고 사용합니다.

이를 통해 OpenCode가 전체 워크플로우를 강제하는 Claude Code와 유사하게 동작하도록 보장합니다.

## 새로운 스킬 만들기

### 디렉토리 구조

```
skills/
  {skill-name}/           # 케밥 케이스(kebab-case) 디렉토리 이름
    SKILL.ko.md           # 필수: 스킬 정의
    scripts/              # 필수: 실행 가능한 스크립트
      {script-name}.sh    # Bash 스크립트 (권장)
  {skill-name}.zip        # 필수: 배포를 위해 패키지화됨
```

### 명명 규칙

- **스킬 디렉토리**: `케밥 케이스` (예: `web-quality`)
- **SKILL.ko.md**: 항상 대문자(SKILL)와 `.ko.md` 확장자, 항상 이 정확한 파일 이름
- **스크립트**: `kebab-case.sh` (예: `deploy.sh`, `fetch-logs.sh`)
- **Zip 파일**: 디렉토리 이름과 정확히 일치해야 함: `{skill-name}.zip`

### SKILL.ko.md 형식

```markdown
---
name: {skill-name}
description: {이 스킬을 언제 사용할지 설명하는 한 문장. "내 앱 배포", "로그 확인" 등과 같은 트리거 구문을 포함하세요.}
---

# {스킬 제목}

{스킬이 수행하는 작업에 대한 간략한 설명.}

## 작동 방식

{스킬의 워크플로우를 설명하는 번호가 매겨진 목록}

## 사용법

```bash
bash /mnt/skills/user/{skill-name}/scripts/{script}.sh [args]
```

**인수(Arguments):**
- `arg1` - 설명 (기본값: X)

**예시:**
{2-3가지 일반적인 사용 패턴 제시}

## 출력 (Output)

{사용자가 보게 될 예시 출력}

## 사용자에게 결과 표시

{Claude가 사용자에게 결과를 표시할 때의 포맷 템플릿}

## 트러블슈팅 (Troubleshooting)

{일반적인 문제 및 해결 방법, 특히 네트워크/권한 오류}
```

### 컨텍스트 효율성을 위한 베스트 프랙티스

스킬은 필요할 때 로드됩니다. 시작 시에는 스킬 이름과 설명만 로드됩니다. 에이전트가 스킬이 관련이 있다고 판단할 때만 전체 `SKILL.ko.md`가 컨텍스트에 로드됩니다. 컨텍스트 사용량을 최소화하려면:

- **SKILL.ko.md를 500줄 이하로 유지하세요** — 세부 참조 자료는 별도의 파일에 넣으세요.
- **구체적인 설명을 작성하세요** — 에이전트가 스킬을 활성화할 시점을 정확히 알 수 있게 도와줍니다.
- **점진적 공개를 사용하세요** — 필요할 때만 읽히는 지원 파일을 참조하세요.
- **인라인 코드보다 스크립트를 선호하세요** — 스크립트 실행은 컨텍스트를 소비하지 않습니다 (출력만 소비함).
- **파일 참조는 한 단계 깊이까지 작동합니다** — SKILL.ko.md에서 지원 파일로 직접 링크하세요.

### 스크립트 요구 사항

- `#!/bin/bash` 셰뱅(shebang) 사용
- 빠른 실패 동작을 위해 `set -e` 사용
- stderr에 상태 메시지 작성: `echo "Message" >&2`
- stdout에 머신 러닝이 읽을 수 있는 출력(JSON) 작성
- 임시 파일을 위한 정리 트랩(cleanup trap) 포함
- 스크립트 경로를 `/mnt/skills/user/{skill-name}/scripts/{script}.sh`로 참조

### Zip 패키지 생성

스킬을 생성하거나 업데이트한 후:

```bash
cd skills
zip -r {skill-name}.zip {skill-name}/
```

### 엔드 유저 설치

사용자를 위해 다음 두 가지 설치 방법을 문서화하세요:

**Claude Code:**
```bash
cp -r skills/{skill-name} ~/.claude/skills/
```

**claude.ai:**
프로젝트 지식에 스킬을 추가하거나 대화에 SKILL.ko.md 내용을 붙여넣으세요.

스킬에 네트워크 액세스가 필요한 경우, 사용자가 `claude.ai/settings/capabilities`에서 필요한 도메인을 추가하도록 안내하세요.
