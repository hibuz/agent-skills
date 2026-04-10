# OpenCode 설정

이 guide에서는 Claude Code 환경(자동 skill 선택, lifecycle-driven workflows 및 엄격한 프로세스 적용)을 밀접하게 미러링하는 방식으로 OpenCode와 함께 Agent Skills를 사용하는 방법을 설명합니다.

## 개요

OpenCode는 사용자 정의 `/commands`를 지원하지만 기본 plugin 시스템이나 Claude Code와 같은 자동 skill 라우팅이 없습니다.

대신 다음을 통해 패리티를 달성합니다.

- 강력한 시스템 프롬프트(`AGENTS.md`)
- built-in `skill` 도구
- `/skills` 디렉터리에서 일관된 skill 검색

그러면 skills가 선택되고 자동으로 실행되는 **agent-driven workflow**가 생성됩니다.

OpenCode에서 `/spec`, `/plan` 및 기타 명령을 다시 생성할 수 있지만 이 통합은 의도적으로 대신 agent-driven 접근 방식을 사용합니다.

- Skills는 의도에 따라 자동으로 선택됩니다.
- Workflows는 `AGENTS.md`를 통해 시행됩니다.
- 수동 명령 호출은 required가 아닙니다.

이는 skills가 수동이 아닌 자동으로 트리거되는 Claude Code가 실제로 작동하는 방식과 더 밀접하게 일치합니다.

---

## 설치

1. repository를 복제합니다.

```bash
git clone https://github.com/addyosmani/agent-skills.git
```

2. OpenCode에서 프로젝트를 엽니다.

3. 작업 공간에 다음 파일이 있는지 확인하십시오.

- `AGENTS.md`(루트)
- `skills/` 디렉토리

추가 설치는 required가 아닙니다.

---

## 작동 방식

### 1. Skill 검색

모든 skills 거주 지역:

```
skills/<skill-name>/SKILL.md
```

OpenCode agents는 (`AGENTS.md`를 통해) 다음을 수행하도록 지시됩니다.

- skill가 적용되는 경우 감지
- `skill` 도구 호출
- skill를 정확하게 따르세요.

### 2. 자동 Skill 호출

agent는 모든 요청을 평가하고 이를 적절한 skill에 매핑합니다.

예:

- "build 기능" → `incremental-implementation` + `test-driven-development`
- "시스템 설계" → `spec-driven-development`
- "버그 수정" → `debugging-and-error-recovery`
- "이 코드 검토" → `code-review-and-quality`

사용자는 skills를 명시적으로 요청할 필요가 **없습니다**.

### 3. 수명주기 매핑(암시적 명령)

개발 라이프사이클은 암시적으로 인코딩됩니다.

- DEFINE → `spec-driven-development`
- PLAN → `planning-and-task-breakdown`
- BUILD → `incremental-implementation` + `test-driven-development`
- VERIFY → `debugging-and-error-recovery`
- REVIEW → `code-review-and-quality`
- SHIP → `shipping-and-launch`

이는 `/spec`, `/plan` 등과 같은 slash commands를 대체합니다.

---

## 사용 예

### 예시 1: 기능 개발

사용자:
```
Add authentication to this app
```

Agent 동작:
- 기능 작업 감지
- `spec-driven-development`를 호출합니다.
- 코드를 작성하기 전에 스펙을 생성합니다.
- 기획 및 구현 skills로 이동

---

### 예시 2: 버그 수정

사용자:
```
This endpoint is returning 500 errors
```

Agent 동작:
- `debugging-and-error-recovery`를 호출합니다.
- 재현 → 현지화 → 수정 → guards 추가

---

### 예시 3: 코드 검토

사용자:
```
Review this PR
```

Agent 동작:
- `code-review-and-quality`를 호출합니다.
- 구조화된 검토 적용(정확성, 디자인, 가독성 등)

---

## Agent 기대사항(중요)

OpenCode가 올바르게 작동하려면 agent가 다음 규칙을 따라야 합니다.

- 행동하기 전에 항상 skill가 적용되는지 확인하세요.
- skill가 적용되는 경우 MUST가 사용됩니다.
- required workflows(스펙, 계획, 테스트 등)을 절대 건너뛰지 마세요.
- 구현으로 바로 넘어가지 마세요.

이러한 규칙은 `AGENTS.md`를 통해 시행됩니다.

---

## 제한사항

- 네이티브 slash commands 없음(대신 인텐트 매핑을 통해 처리됨)
- plugin 시스템 없음(프롬프트 + 구조를 통해 처리됨)
- Skill 호출은 모델 준수에 따라 다릅니다.

그럼에도 불구하고 workflow는 실제로 Claude Code와 거의 일치합니다.

---

## 추천 Workflow

자연어를 사용하세요.

- "기능 디자인"
- "이 변화를 계획하세요"
- "이것을 구현해 보세요"
- "이 버그를 수정하세요"
- "이것을 검토해 보세요"

agent는 올바른 skills를 자동으로 선택하고 실행합니다.

---

## 요약

OpenCode 통합은 다음을 결합하여 작동합니다.

- 구조화된 skills(이 repo)
- 강력한 agent 규칙(`AGENTS.md`)
- 추론을 통한 자동 skill 호출

이로 인해 requiring plugins 또는 수동 명령 없이 **완전히 agent-driven, production-grade 엔지니어링 workflow**가 발생합니다.
