# Skill Anatomy

이 문서는 agent-skills의 기술(Skill) 파일 구조와 형식을 설명합니다. 새로운 기술을 기여하거나 기존 기술을 이해할 때 이 가이드를 참조하세요.

## File Location

모든 기술은 `skills/` 하위의 전용 디렉토리에 위치합니다:

```
skills/
  skill-name/
    SKILL.md          # 필수: 기술 정의
    supporting-file.md # 선택: 필요할 때 로드되는 참조 자료
```

## SKILL.md Format

### Frontmatter (필수)

```yaml
---
name: skill-name-with-hyphens
description: 기술이 무엇을 하는지 간략하게 설명합니다. [특정 트리거 조건]일 때 사용하세요.
---
```

**규칙:**
- `name`: 소문자, 하이픈으로 구분합니다. 디렉토리 이름과 일치해야 합니다.
- `description`: 기술이 하는 일(3인칭)로 시작하고, 트리거 조건이 뒤따릅니다. *무엇*을 하는지와 *언제* 사용하는지를 모두 포함하세요. 최대 1024자입니다.

**이것이 중요한 이유:** Agent는 설명을 읽고 기술을 발견합니다. 이 설명은 System Prompt에 삽입되므로, Agent에게 기술이 무엇을 제공하고 언제 활성화해야 하는지 알려주어야 합니다. 워크플로우를 요약하지 마세요 — 설명에 프로세스 단계가 포함되어 있으면 Agent가 전체 기술을 읽는 대신 요약을 따를 수 있습니다.

### Standard Sections

```markdown
# Skill Title

## Overview
이 기술이 무엇을 하며 왜 중요한지 설명하는 1~2개의 문장입니다.

## When to Use
- 트리거 조건(증상, 작업 유형)의 불렛 리스트
- 사용하지 말아야 할 때 (Exclusion)

## [Core Process / The Workflow / Steps]
번구 매겨진 단계나 페이즈로 나뉜 메인 워크플로우입니다.
도움이 되는 경우 코드 예시를 포함하세요.
결정 포인트가 있는 경우 ASCII Flowchart를 사용하세요.

## [Specific Techniques / Patterns]
특정 시나리오에 대한 상세 가이드입니다.
코드 예시, 템플릿, 설정 등을 포함합니다.

## Common Rationalizations
| Rationalization | Reality |
|---|---|
| Agent가 단계를 건너뛰기 위해 사용하는 변명 | 그 변명이 틀린 이유 |

## Red Flags
- 기술이 위반되고 있음을 나타내는 행동 패턴
- 리뷰 중에 주의 깊게 확인해야 할 사항

## Verification
기술의 프로세스를 완료한 후 다음을 확인하세요:
- [ ] 종료 기준(Exit Criteria) Checklist
- [ ] 증거 요구 사항
```

## Section Purposes

### Overview
기술에 대한 "엘리베이터 피치"입니다. 다음 질문에 답해야 합니다: 이 기술은 무엇을 하며, Agent가 왜 이것을 따라야 하는가?

### When to Use
Agent와 인간이 이 기술이 현재 작업에 적용되는지 결정하는 데 도움을 줍니다. 긍정적 트리거("X일 때 사용")와 부정적 제외 사항("Y에는 사용 금지")을 모두 포함하세요.

### Core Process
기술의 핵심입니다. Agent가 따르는 단계별 워크플로우입니다. 모호한 조언이 아닌 구체적이고 실행 가능해야 합니다.

**좋은 예:** "`npm test`를 실행하고 모든 테스트가 통과하는지 확인하세요"
**나쁜 예:** "테스트가 작동하는지 확인하세요"

### Common Rationalizations
잘 만들어진 기술의 가장 독특한 특징입니다. Agent가 중요한 단계를 건너뛰기 위해 사용하는 변명들과 그에 대한 반박을 짝지어 놓은 것입니다. Agent가 프로세스를 따르지 않기 위해 스스로 합리화하는 것을 방지합니다.

Agent가 "나중에 테스트를 추가할게요"나 "이건 간단하니까 Spec은 건너뛸게요"라고 말했던 모든 순간을 떠올려보세요 — 그런 내용들이 사실적인 반론과 함께 여기에 들어갑니다.

### Red Flags
기술이 위반되고 있다는 관찰 가능한 징후입니다. 코드 리뷰와 자가 모니터링 시 유용합니다.

### Verification
종료 기준입니다. Agent가 기술의 프로세스가 완료되었는지 확인하기 위해 사용하는 Checklist입니다. 모든 체크박스는 증거(테스트 출력, 빌드 결과, 스크린샷 등)를 통해 검증 가능해야 합니다.

## Supporting Files

다음의 경우에만 보조 파일을 생성하세요:
- 참조 자료가 100행을 초과하는 경우 (메인 SKILL.md의 집중도 유지)
- 코드 도구 또는 스크립트가 필요한 경우
- Checklist가 별도의 파일로 만들 만큼 긴 경우

패턴과 원칙이 50행 미만인 경우 본문에 포함하세요.

## Writing Principles

1. **Knowledge보다 Process 중심.** 기술은 참조 문서가 아니라 워크플로우입니다. 사실이 아니라 단계여야 합니다.
2. **General보다 Specific 중심.** "테스트를 확인하세요"보다 "`npm test`를 실행하세요"가 낫습니다.
3. **Assumption보다 Evidence 중심.** 모든 검증 체크박스는 증거를 요구합니다.
4. **Anti-rationalization.** 건너뛸 가치가 있어 보이는 모든 단계는 Rationalization 테이블에 반론이 필요합니다.
5. **Progressive Disclosure.** 메인 SKILL.md가 진입점입니다. 보조 파일은 필요할 때만 로드됩니다.
6. **Token-conscious.** 모든 섹션은 포함되어야 하는 정당성이 있어야 합니다. 제거해도 Agent의 행동이 변하지 않는다면 제거하세요.

## Naming Conventions

- 기술 디렉토리: `소문자-하이픈-구분`
- 기술 파일: `SKILL.md` (항상 대문자)
- 보조 파일: `소문자-하이픈-구분.md`
- Reference: 기술 디렉토리 내부가 아닌 프로젝트 루트의 `references/`에 저장

## Cross-Skill References

다른 기술을 이름으로 참조하세요:

```markdown
테스트를 작성할 때는 `test-driven-development` 기술을 따르세요.
빌드가 깨지면 `debugging-and-error-recovery` 기술을 사용하세요.
```

기술 간에 내용을 중복시키지 마세요 — 대신 참조하고 링크하세요.
