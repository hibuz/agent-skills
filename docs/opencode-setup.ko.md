# OpenCode Setup

이 가이드는 Claude Code 경험(자동 기술 선택, Lifecycle 기반 Workflow 및 엄격한 프로세스 적용)을 최대한 유사하게 OpenCode에서 Agent Skills를 사용하는 방법을 설명합니다.

## Overview

OpenCode는 커스텀 `/commands`를 지원하지만, Claude Code와 같은 네이티브 플러그인 시스템이나 자동 기술 Routing 기능은 없습니다.

대신 다음을 통해 기능적 동등성을 달성합니다:

- 강력한 System Prompt (`AGENTS.md`)
- 내장된 `skill` 도구
- `/skills` 디렉토리를 통한 일관된 기술 발견

이를 통해 기술이 자동으로 선택되고 실행되는 **Agent-driven Workflow**를 구축합니다.

OpenCode에서 `/spec`, `/plan` 및 기타 명령어를 재현하는 것이 가능하지만, 이 통합은 의도적으로 Agent 중심의 접근 방식을 사용합니다:

- 의도(Intent)에 따라 기술이 자동으로 선택됨
- `AGENTS.md`를 통해 Workflow 강제 적용
- 수동 명령어 호출이 필요 없음

이는 기술이 수동이 아닌 자동으로 트리거되는 실제 Claude Code의 동작 방식과 더 밀접하게 일치합니다.

---

## Installation

1. 리포지토리 클론:

```bash
git clone https://github.com/addyosmani/agent-skills.git
```

2. OpenCode에서 프로젝트를 엽니다.

3. 워크스페이스에 다음 파일들이 있는지 확인하세요:

- `AGENTS.md` (루트)
- `skills/` 디렉토리

추가 설치는 필요하지 않습니다.

---

## How It Works

### 1. Skill Discovery

모든 기술은 다음 위치에 있습니다:

```
skills/<skill-name>/SKILL.md
```

OpenCode Agent는 (`AGENTS.md`를 통해) 다음을 수행하도록 지시받습니다:

- 기술이 적용되는 시점 감지
- `skill` 도구 호출
- 기술을 정확히 따름

### 2. Automatic Skill Invocation

Agent는 모든 요청을 평가하고 적절한 기술에 매핑합니다.

예시:

- "기능 구현" → `incremental-implementation` + `test-driven-development`
- "시스템 설계" → `spec-driven-development`
- "버그 수정" → `debugging-and-error-recovery`
- "코드 리뷰" → `code-review-and-quality`

사용자는 기술을 명시적으로 요청할 필요가 **없습니다**.

### 3. Lifecycle Mapping (암시적 명령어)

개발 Lifecycle은 암시적으로 인코딩되어 있습니다:

- DEFINE → `spec-driven-development`
- PLAN → `planning-and-task-breakdown`
- BUILD → `incremental-implementation` + `test-driven-development`
- VERIFY → `debugging-and-error-recovery`
- REVIEW → `code-review-and-quality`
- SHIP → `shipping-and-launch`

이는 `/spec`, `/plan` 등과 같은 슬래시 명령어를 대체합니다.

---

## Usage Examples

### Example 1: Feature Development

사용자:
```
이 앱에 인증 기능을 추가해줘
```

Agent 행동:
- 기능 개발 작업임을 감지함
- `spec-driven-development` 호출
- 코드를 작성하기 전에 Spec을 생성함
- Planning 및 Implementation 기술로 이동함

---

### Example 2: Bug Fix

사용자:
```
이 Endpoint가 500 Error를 반환하고 있어
```

Agent 행동:
- `debugging-and-error-recovery` 호출
- 재현 → 위치 파악 → 수정 → 가드(Guard) 추가

---

### Example 3: Code Review

사용자:
```
이 PR을 리뷰해줘
```

Agent 행동:
- `code-review-and-quality` 호출
- 구조화된 리뷰 적용 (정확성, 설계, 가독성 등)

---

## Agent Expectations (중요)

OpenCode가 올바르게 작동하려면 Agent가 다음 규칙을 따라야 합니다:

- 행동하기 전에 항상 기술이 적용되는지 확인할 것
- 기술이 적용된다면 반드시 사용해야 함
- 필수 Workflow(Spec, Plan, Test 등)를 절대 건너뛰지 말 것
- 구현 단계로 바로 넘어가지 말 것

이 규칙들은 `AGENTS.md`를 통해 강제됩니다.

---

## Limitations

- 네이티브 슬래시 명령어가 없음 (대신 의도 매핑을 통해 처리됨)
- 플러그인 시스템이 없음 (Prompt + 구조를 통해 처리됨)
- 기술 호출은 모델의 준수 여부에 달려 있음

이러한 제한에도 불구하고, 실제 Workflow는 Claude Code와 매우 유사하게 작동합니다.

---

## Recommended Workflow

자연어를 사용하세요:

- "기능 설계해줘"
- "이 변경 사항을 계획해줘"
- "이걸 구현해줘"
- "이 버그를 고쳐줘"
- "이걸 리뷰해줘"

Agent가 자동으로 올바른 기술을 선택하고 실행할 것입니다.

---

## Summary

OpenCode 통합은 다음을 결합하여 작동합니다:

- 구조화된 기술 (이 리포지토리)
- 강력한 Agent 규칙 (`AGENTS.md`)
- 추론을 통한 자동 기술 호출

그 결과 플러그인이나 수동 명령어 없이도 **완전히 Agent 중심적인 프로덕션 등급의 엔지니어링 Workflow**를 얻을 수 있습니다.
