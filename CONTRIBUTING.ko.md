# Agent Skills에 기여하기

기여에 관심 가져 주셔서 감사합니다! 이 프로젝트는 AI 코딩 agent를 위한 production 수준 엔지니어링 skill 모음입니다.

## 새 Skill 추가하기

1. `skills/` 아래에 kebab-case 이름으로 디렉터리를 만듭니다
2. [docs/skill-anatomy.ko.md](docs/skill-anatomy.ko.md)의 형식을 따르는 `SKILL.md`를 추가합니다
3. `name`과 `description` 필드를 가진 YAML frontmatter를 포함합니다
4. `description`이 skill이 무엇을 하는지(3인칭)로 시작하고, 이어서 하나 이상의 `Use when` 트리거 조건을 포함하도록 합니다

### Skill 품질 기준

skill은 다음과 같아야 합니다:

- **구체적(Specific)** — 막연한 조언이 아니라 실행 가능한 단계
- **검증 가능(Verifiable)** — 증거 요구사항을 갖춘 명확한 종료 기준
- **실전 검증됨(Battle-tested)** — 이론적 이상이 아니라 실제 엔지니어링 워크플로 기반
- **최소한(Minimal)** — agent를 올바르게 안내하는 데 필요한 내용만

### 구조

모든 새 skill에는 다음이 있어야 합니다:

- skill 디렉터리 안의 `SKILL.md`
- 유효한 `name`과 `description`을 가진 YAML frontmatter

새 skill은 일반적으로 표준 anatomy를 따라야 합니다:

- **Overview** — 이 skill이 무엇을 하고 왜 중요한지
- **When to Use** — 트리거 조건
- **Process** — 단계별 워크플로
- **Common Rationalizations** — agent가 단계를 건너뛰는 데 쓰는 핑계와 그에 대한 반박
- **Red Flags** — skill이 잘못 적용되고 있다는 경고 신호
- **Verification** — skill이 올바르게 적용되었는지 확인하는 방법

위 frontmatter 필드는 필수입니다. 섹션 anatomy는 권장 패턴입니다: `How It Works`, `Workflow`, `Core Process` 같은 동등한 제목도 동일한 의도를 유지하고 skill을 따라가기 쉽게 한다면 괜찮습니다.

### 하지 말아야 할 것

- skill 간 내용을 중복하지 마세요 — 대신 다른 skill을 참조하세요
- 실행 가능한 프로세스가 아니라 막연한 조언인 skill을 추가하지 마세요
- 내용이 100줄을 초과하지 않으면 보조 파일을 만들지 마세요
- 다른 skill과 맞추려고 빈 `scripts/` 디렉터리를 만들지 마세요 — skill에 실행 가능한 헬퍼가 포함될 때만 `scripts/`를 추가하세요
- 참조 자료를 skill 디렉터리 안에 두지 마세요 — 대신 `references/`를 사용하세요

## 기존 Skill 수정하기

- 변경은 집중적이고 최소한으로 유지하세요
- 기존 구조와 톤을 보존하세요
- 편집 후 YAML frontmatter가 여전히 유효한지 테스트하세요

## Hook 테스트하기

session-start hook(`hooks/session-start.sh`)은 모든 새 Claude Code 세션에 `using-agent-skills` 메타 skill을 주입합니다. `hooks/session-start-test.sh`의 회귀 테스트는 hook의 JSON payload를 검증합니다 — `jq`가 있을 때와 없을 때 모두에 대해.

다음을 건드리는 PR을 열기 전에 이 테스트를 실행하세요:

- `hooks/session-start.sh`
- `skills/using-agent-skills/SKILL.md` (hook이 임베드하는 메타 skill 내용)

```bash
bash hooks/session-start-test.sh
```

예상 출력: `session-start JSON payload OK`. assertion이 하나라도 실패하면 스크립트는 0이 아닌 코드로 종료됩니다.

### no-jq fallback 재현하기

hook은 `jq`가 `PATH`에 없을 때 `INFO` 우선순위 payload로 우아하게 degrade됩니다. 그 분기를 로컬에서 실행해 보려면, 테스트 호출 시 `PATH`에서 `jq`의 디렉터리를 제거하세요:

```bash
JQ_DIR=$(dirname "$(command -v jq)")
PATH=$(echo "$PATH" | tr ':' '\n' | grep -v "^${JQ_DIR}$" | tr '\n' ':' | sed 's/:$//') \
  bash hooks/session-start-test.sh
```

이 방법은 `jq`가 자체 디렉터리에 있을 때(예: Homebrew의 `/opt/homebrew/bin`, 수동 설치의 `/usr/local/bin`) 깔끔하게 동작합니다. `jq`가 테스트가 의존하는 다른 도구(예: `/usr/bin`의 `mktemp`)와 시스템 bin을 공유한다면, 별도의 패키지 관리자로 `jq`를 설치해 자체 bin 디렉터리를 갖게 한 뒤 다시 실행하는 것이 더 간단합니다.

stripped `PATH`에서 hook의 `command -v jq` 검사가 실패하면 `INFO` 우선순위 fallback이 실행되고, 테스트는 일반 payload 대신 `jq is required` 안내 메시지를 assert합니다.

## 이슈 보고하기

다음을 발견하면 이슈를 열어 주세요:

- 부정확하거나 오래된 가이드를 주는 skill
- 흔한 엔지니어링 워크플로에 대한 커버리지 누락
- skill 간 불일치

## 라이선스

기여함으로써, 귀하의 기여가 MIT License로 라이선스됨에 동의하는 것입니다.
