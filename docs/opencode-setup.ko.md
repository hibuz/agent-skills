# OpenCode 설정

이 가이드는 Claude Code 경험(자동 skill 선택, 라이프사이클 주도 워크플로, 엄격한 프로세스 강제)을 가깝게 재현하는 방식으로 OpenCode에서 Agent Skills를 사용하는 방법을 설명합니다.

## 개요

OpenCode는 커스텀 `/commands`를 지원하지만, Claude Code 같은 네이티브 플러그인 시스템이나 자동 skill 라우팅은 없습니다.

대신 우리는 다음을 통해 동등성을 달성합니다:

- 강력한 시스템 프롬프트 (`AGENTS.md`)
- 내장 `skill` 도구
- `/skills` 디렉터리로부터의 일관된 skill 검색

이는 skill이 자동으로 선택되고 실행되는 **agent 주도 워크플로(agent-driven workflow)**를 만듭니다.

OpenCode에서 `/spec`, `/plan` 및 기타 커맨드를 재현하는 것이 가능하지만, 이 연동은 의도적으로 agent 주도 접근을 대신 사용합니다:

- skill은 intent에 따라 자동으로 선택됨
- 워크플로는 `AGENTS.md`를 통해 강제됨
- 수동 커맨드 호출이 필요 없음

이는 skill이 수동이 아니라 자동으로 트리거되는, Claude Code의 실제 동작 방식과 더 가깝게 일치합니다.

---

## 설치

1. repository를 clone합니다:

```bash
git clone https://github.com/addyosmani/agent-skills.git
```

2. OpenCode에서 프로젝트를 엽니다.

3. workspace에 다음 파일이 있는지 확인합니다:

- `AGENTS.md` (루트)
- `skills/` 디렉터리

추가 설치는 필요 없습니다.

---

## 어떻게 동작하는가

### 1. Skill 검색

모든 skill은 다음에 위치합니다:

```
skills/<skill-name>/SKILL.md
```

OpenCode agent는 (`AGENTS.md`를 통해) 다음을 하도록 지시받습니다:

- skill이 적용되는 시점 감지
- `skill` 도구 invoke
- skill을 정확히 따르기

### 2. 자동 Skill 호출

agent는 모든 요청을 평가하고 적절한 skill에 매핑합니다.

예시:

- "build a feature" → `incremental-implementation` + `test-driven-development`
- "design a system" → `spec-driven-development`
- "fix a bug" → `debugging-and-error-recovery`
- "review this code" → `code-review-and-quality`

사용자는 skill을 명시적으로 요청할 필요가 **없습니다**.

### 3. 라이프사이클 매핑 (암묵적 커맨드)

개발 라이프사이클은 암묵적으로 인코딩됩니다:

- DEFINE → `spec-driven-development`
- PLAN → `planning-and-task-breakdown`
- BUILD → `incremental-implementation` + `test-driven-development`
- VERIFY → `debugging-and-error-recovery`
- REVIEW → `code-review-and-quality`
- SHIP → `shipping-and-launch`

이는 `/spec`, `/plan` 등의 슬래시 커맨드를 대체합니다.

---

## 사용 예시

### 예시 1: 기능 개발

사용자:
```
Add authentication to this app
```

agent 동작:
- 기능 작업 감지
- `spec-driven-development` invoke
- 코드 작성 전 spec 생성
- 계획 및 구현 skill로 이동

---

### 예시 2: 버그 수정

사용자:
```
This endpoint is returning 500 errors
```

agent 동작:
- `debugging-and-error-recovery` invoke
- 재현 → 위치 파악 → 수정 → 가드 추가

---

### 예시 3: 코드 리뷰

사용자:
```
Review this PR
```

agent 동작:
- `code-review-and-quality` invoke
- 구조화된 리뷰 적용 (정확성, 설계, 가독성 등)

---

## Agent 기대사항 (중요)

OpenCode가 올바르게 동작하려면 agent는 다음 규칙을 따라야 합니다:

- 행동하기 전에 항상 skill이 적용되는지 확인
- skill이 적용되면 반드시 사용해야 함
- 필요한 워크플로(spec, plan, test 등)를 절대 건너뛰지 않음
- 구현으로 바로 점프하지 않음

이 규칙들은 `AGENTS.md`를 통해 강제됩니다.

---

## 제약

- 네이티브 슬래시 커맨드 없음 (대신 intent 매핑으로 처리)
- 플러그인 시스템 없음 (프롬프트 + 구조로 처리)
- skill 호출은 모델 준수에 의존

이러한 제약에도 불구하고, 워크플로는 실제로 Claude Code와 가깝게 일치합니다.

---

## 권장 워크플로

그냥 자연어를 사용하세요:

- "Design a feature"
- "Plan this change"
- "Implement this"
- "Fix this bug"
- "Review this"

agent가 자동으로 올바른 skill을 선택하고 실행합니다.

---

## 요약

OpenCode 연동은 다음을 결합해 동작합니다:

- 구조화된 skill (이 repo)
- 강력한 agent 규칙 (`AGENTS.md`)
- 추론을 통한 자동 skill 호출

이는 플러그인이나 수동 커맨드 없이도 **완전히 agent 주도적인 production 수준 엔지니어링 워크플로**를 만들어 냅니다.
