# Skill 해부학

이 문서에서는 agent-skills skill 파일의 구조와 format에 대해 설명합니다. 새로운 skills에 기여하거나 기존 uide를 이해할 때 이를 guide로 사용하세요.

## 파일 위치

모든 skill는 `skills/` 아래의 자체 디렉토리에 있습니다.

```
skills/
  skill-name/
    SKILL.md          # Required: The skill definition
    supporting-file.md # Optional: Reference material loaded on demand
```

## SKILL.md Format

### Frontmatter (Required)

```yaml
---
name: skill-name-with-hyphens
description: Brief statement of what the skill does. Use when [specific trigger conditions].
---
```

**규칙:**
- `name`: 소문자, hyphen-separated. 디렉터리 이름과 일치해야 합니다.
- `description`: skill가 수행하는 작업(3인칭)으로 시작하고 이어서 트리거 조건이 따릅니다. *무엇*과 *언제*를 모두 포함하세요. 최대 1024자입니다.

**이것이 중요한 이유:** Agents 설명을 읽고 skills를 발견하세요. 설명은 시스템 프롬프트에 삽입되므로 skill가 제공하는 내용과 활성화 시기를 모두 agent에 알려야 합니다. workflow를 요약하지 마십시오. 설명에 프로세스 단계가 포함된 경우 agent는 전체 skill를 읽는 대신 요약을 따를 수 있습니다.

### 표준 섹션

```markdown
# Skill Title

## Overview
One-two sentences explaining what this skill does and why it matters.

## When to Use
- Bullet list of triggering conditions (symptoms, task types)
- When NOT to use (exclusions)

## [Core Process / The Workflow / Steps]
The main workflow, broken into numbered steps or phases.
Include code examples where they help.
Use flowcharts (ASCII) where decision points exist.

## [Specific Techniques / Patterns]
Detailed guidance for specific scenarios.
Code examples, templates, configuration.

## Common Rationalizations
| Rationalization | Reality |
|---|---|
| Excuse agents use to skip steps | Why the excuse is wrong |

## Red Flags
- Behavioral patterns indicating the skill is being violated
- Things to watch for during review

## Verification
After completing the skill's process, confirm:
- [ ] Checklist of exit criteria
- [ ] Evidence requirements
```

## 섹션 목적

### 개요
skill의 "엘리베이터 피치"입니다. 답변해야 합니다: 이 skill는 무엇을 하며, agent가 이를 따라야 하는 이유는 무엇입니까?

### 사용 시기
agents와 사람이 이 skill가 현재 작업에 적용되는지 결정하는 데 도움이 됩니다. 긍정적인 트리거("X일 때 사용")와 부정적인 제외("Y의 경우 NOT")를 모두 포함합니다.

### 핵심 프로세스
skill의 핵심입니다. 이는 step-by-step workflow이며 agent는 다음과 같습니다. 막연한 조언이 아니라 구체적이고 실행 가능해야 합니다.

**좋음:** "`npm test`를 실행하고 모든 테스트가 통과하는지 확인하세요."
**나쁨:** "테스트가 제대로 작동하는지 확인하세요."

### 일반적인 합리화
well-crafted skills의 가장 독특한 특징입니다. 이는 agents가 반박과 함께 중요한 단계를 건너뛰기 위해 사용하는 변명입니다. 이는 agent가 프로세스를 따르지 않는 길을 합리화하는 것을 방지합니다.

agent가 "나중에 테스트를 추가하겠습니다" 또는 "이것은 사양을 건너뛸 만큼 간단합니다"라고 말할 때마다 생각해 보십시오. 여기에는 사실적인 counter-argument가 포함됩니다.

### 위험 신호
skill가 위반되고 있음을 나타내는 관찰 가능한 징후입니다. 코드 검토 및 self-monitoring 중에 유용합니다.

### 확인
종료 기준. agent가 skill의 프로세스가 완료되었는지 확인하는 데 사용하는 체크리스트입니다. 모든 체크박스는 증거(테스트 출력, build 결과, 스크린샷 등)로 검증 가능해야 합니다.

## 지원 파일

다음과 같은 경우에만 지원 파일을 만듭니다.
- 참조 자료가 100줄을 초과합니다(주 SKILL.md에 초점을 맞추세요).
- 코드 도구나 스크립트가 필요합니다.
- 체크리스트는 별도의 파일을 정당화할 만큼 충분히 깁니다.

50줄 미만인 경우 패턴과 원칙을 인라인으로 유지하세요.

## 작성 원칙

1. **지식을 통한 프로세스.** Skills는 참조 문서가 아닌 workflows입니다. 사실이 아닌 단계.
2. **일반보다는 특정.** "`npm test` 실행"은 "테스트 확인"을 능가합니다.
3. **가정에 대한 증거.** 모든 확인 확인란에는 증거가 필요합니다.
4. **Anti-rationalization.** Every skip-worthy step needs a counter-argument in the rationalizations table.
5. **점진적 공개.** 메인 SKILL.md가 진입점입니다. 지원 파일은 필요할 때만 로드됩니다.
6. **토큰을 고려합니다.** 모든 섹션은 해당 항목의 포함을 정당화해야 합니다. 이를 제거해도 agent 동작이 변경되지 않으면 제거하십시오.

## 명명 규칙

- Skill 디렉토리: `lowercase-hyphen-separated`
- Skill 파일: `SKILL.md`(항상 대문자)
- 지원 파일: `lowercase-hyphen-separated.md`
- 참조: skill 디렉터리 내부가 아닌 프로젝트 루트의 `references/`에 저장됩니다.

## 교차 Skill 참조

다른 skills를 이름으로 참조:

```markdown
Follow the `test-driven-development` skill for writing tests.
If the build breaks, use the `debugging-and-error-recovery` skill.
```

skills 간에 콘텐츠를 복제하지 마세요. 대신 참조와 링크를 사용하세요.
