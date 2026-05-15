# Skill Anatomy

이 문서는 agent-skills의 skill 파일 구조와 형식을 설명합니다. 새 skill을 기여하거나 기존 skill을 이해할 때 가이드로 사용하세요.

## 파일 위치

모든 skill은 `skills/` 아래 자체 디렉터리에 위치합니다:

```
skills/
  skill-name/
    SKILL.md           # 필수: skill 정의
    scripts/           # 선택: skill 워크플로가 사용하는 실행 가능한 헬퍼
    supporting-file.md # 선택: on-demand로 로드되는 참조 자료
```

`SKILL.md`가 유일한 필수 파일입니다. skill이 실제로 실행 가능한 헬퍼를 제공할 때만 `scripts/`를 추가하고, Markdown만으로 된 skill은 디렉터리를 완전히 생략하세요.

## SKILL.md 형식

### Frontmatter (필수)

```yaml
---
name: skill-name-with-hyphens
description: Guides agents through [task/workflow]. Use when [specific trigger conditions].
---
```

**규칙:**
- `name`: 소문자, 하이픈 구분. 디렉터리 이름과 일치해야 함.
- `description`: skill이 무엇을 하는지 3인칭으로 시작하고, 이어서 하나 이상의 명확한 "Use when" 트리거 조건을 포함. *무엇*과 *언제* 모두 포함. 최대 1024자.

**왜 중요한가:** agent는 description을 읽어 skill을 검색합니다. description은 시스템 프롬프트에 주입되므로, skill이 무엇을 제공하는지와 언제 활성화할지를 모두 agent에게 알려야 합니다. 워크플로를 요약하지 마세요 — description에 프로세스 단계가 들어 있으면, agent가 전체 skill을 읽는 대신 요약을 따를 수 있습니다.

### 표준 섹션 (권장 패턴)

위 frontmatter 계약은 필수입니다. 아래 섹션 레이아웃은 권장 패턴이지 엄격한 템플릿이 아닙니다: 동일한 목적을 명확히 수행한다면 동등한 제목도 허용됩니다.

```markdown
# Skill Title

## Overview
이 skill이 무엇을 하고 왜 중요한지 한두 문장으로 설명.

## When to Use
- 트리거 조건의 글머리 기호 목록 (증상, 작업 유형)
- 사용하지 말아야 할 때 (제외 조건)

## [Core Process / The Workflow / Steps]
번호 매긴 단계나 페이즈로 나눈 주요 워크플로.
도움이 되는 곳에 코드 예시 포함.
결정 지점이 있는 곳에 (ASCII) 플로차트 사용.

## [Specific Techniques / Patterns]
특정 시나리오에 대한 상세 가이드.
코드 예시, 템플릿, 설정.

## Common Rationalizations
| Rationalization | Reality |
|---|---|
| agent가 단계를 건너뛰는 데 쓰는 핑계 | 그 핑계가 왜 틀렸는지 |

## Red Flags
- skill이 위반되고 있음을 나타내는 행동 패턴
- 리뷰 중 주의할 것

## Verification
skill의 프로세스를 완료한 후 확인:
- [ ] 종료 기준 체크리스트
- [ ] 증거 요구사항
```

## 섹션의 목적

### Overview
skill의 "엘리베이터 피치". 다음에 답해야 함: 이 skill이 무엇을 하고, agent가 왜 이를 따라야 하는가?

### When to Use
agent와 사람이 이 skill이 현재 작업에 적용되는지 결정하도록 돕습니다. 긍정 트리거("Use when X")와 부정 제외("NOT for Y")를 모두 포함하세요.

### Core Process
skill의 핵심. agent가 따르는 단계별 워크플로입니다. 구체적이고 실행 가능해야 함 — 막연한 조언이 아니라.

**좋음:** "`npm test`를 실행하고 모든 테스트가 통과하는지 확인"
**나쁨:** "테스트가 동작하는지 확인"

### Common Rationalizations
잘 만들어진 skill의 가장 독특한 특징. agent가 중요한 단계를 건너뛰는 데 쓰는 핑계와 그에 대한 반박입니다. agent가 프로세스를 따르지 않으려고 합리화하는 것을 방지합니다.

agent가 "테스트는 나중에 추가할게"나 "이건 spec을 건너뛸 만큼 단순해"라고 말했던 모든 경우를 떠올려 보세요 — 그것들이 사실에 근거한 반론과 함께 여기에 들어갑니다.

### Red Flags
skill이 위반되고 있다는 관찰 가능한 신호. 코드 리뷰와 자기 모니터링 중에 유용합니다.

### Verification
종료 기준. agent가 skill의 프로세스가 완료되었는지 확인하는 데 쓰는 체크리스트입니다. 모든 체크박스는 증거(테스트 출력, 빌드 결과, 스크린샷 등)로 검증 가능해야 합니다.

## 보조 파일

다음의 경우에만 보조 파일을 만드세요:
- 참조 자료가 100줄을 초과 (메인 SKILL.md를 집중되게 유지)
- 코드 도구나 스크립트가 필요
- 체크리스트가 별도 파일을 정당화할 만큼 김

50줄 미만이면 패턴과 원칙을 인라인으로 유지하세요.

skill이 실행 가능한 헬퍼를 필요로 하지 않는다면, 다른 skill을 흉내 내려고 빈 `scripts/` 디렉터리를 만들지 마세요. 빈 디렉터리는 skill 동작을 바꾸지 않으면서 노이즈만 더합니다.

## 작성 원칙

1. **지식보다 프로세스.** skill은 참조 문서가 아니라 워크플로입니다. 사실이 아니라 단계.
2. **일반보다 구체.** "`npm test` 실행"이 "테스트를 검증"보다 낫습니다.
3. **가정보다 증거.** 모든 검증 체크박스는 증명을 요구합니다.
4. **합리화 방지.** 건너뛸 만한 모든 단계에는 합리화 표에 반론이 필요합니다.
5. **점진적 공개.** 메인 SKILL.md가 진입점입니다. 보조 파일은 필요할 때만 로드됩니다.
6. **토큰 의식.** 모든 섹션은 포함될 이유를 정당화해야 합니다. 제거해도 agent 동작이 바뀌지 않는다면, 제거하세요.

## 명명 규칙

- skill 디렉터리: `lowercase-hyphen-separated`
- skill 파일: `SKILL.md` (항상 대문자)
- 보조 파일: `lowercase-hyphen-separated.md`
- 참조: skill 디렉터리 내부가 아니라 프로젝트 루트의 `references/`에 저장

## Skill 간 참조

다른 skill을 이름으로 참조하세요:

```markdown
Follow the `test-driven-development` skill for writing tests.
If the build breaks, use the `debugging-and-error-recovery` skill.
```

skill 간 내용을 중복하지 마세요 — 대신 참조하고 링크하세요.

## 필수 vs 권장

필수:

- `skills/<skill-name>/SKILL.md` 파일
- `name`과 `description`을 가진 유효한 YAML frontmatter
- skill이 무엇을 하고 언제 사용하는지를 모두 포함하는 description

권장:

- 위에 보인 표준 섹션 흐름
- skill에 더 자연스럽게 읽힐 때 `How It Works`, `Core Process`, `Workflow` 같은 동등한 제목
- 메인 `SKILL.md`를 집중되게 유지할 때만 보조 파일
