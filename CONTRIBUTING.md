# Agent Skills에 기여

기여해 주신 interest에 감사드립니다! 이 프로젝트는 AI 코딩 agents를 위한 production-grade 엔지니어링 skills 모음입니다.

## 새로운 Skill 추가

1. kebab-case 이름으로 `skills/` 아래에 디렉터리를 만듭니다.
2. [docs/skill-anatomy.md](docs/skill-anatomy.md)의 format 다음에 `SKILL.md`를 추가합니다.
3. `name` 및 `description` 필드에 YAML frontmatter를 포함합니다.
4. `description`가 "Use when"으로 시작하고 트리거 조건을 설명하는지 확인하세요.

### Skill 품질 바

Skills는 다음과 같아야 합니다.

- **구체적** — 모호한 조언이 아닌 실행 가능한 단계
- **검증 가능** — 증거가 있는 명확한 종료 기준 requirements
- **전투 테스트** — 이론적 이상이 아닌 실제 엔지니어링 workflows를 기반으로 합니다.
- **최소** — agent를 올바르게 guide하는 데 필요한 콘텐츠만

### 구조

모든 skill에는 다음 섹션이 포함되어야 합니다.

- **개요** — skill의 기능과 중요한 이유
- **사용 시기** — 트리거 조건
- **프로세스** — Step-by-step workflow
- **일반적인 합리화** — 반박과 함께 단계를 건너뛰기 위해 agents를 사용하는 변명
- **위험 신호** — skill가 잘못 적용되고 있음을 나타내는 경고 신호
- **확인** — skill가 올바르게 적용되었는지 확인하는 방법

### 하지 말아야 할 일

- skills 간에 콘텐츠를 중복하지 마세요. 대신 다른 skills를 참조하세요.
- 실행 가능한 프로세스 대신 모호한 조언인 skills를 추가하지 마세요.
- 내용이 100줄을 초과하지 않는 한 지원 파일을 만들지 마세요.
- 참조 자료를 skill 디렉터리 안에 넣지 말고 대신 `references/`를 사용하세요.

## 기존 Skills 수정

- 변경 사항을 집중적으로 최소화하고 최소화하세요.
- 기존의 구조와 톤을 유지
- 편집 후에도 YAML frontmatter가 유효한지 테스트합니다.

## Reporting 문제

다음을 찾으면 문제를 열어보세요.

- 부정확하거나 오래된 guidance를 제공하는 skill
- 공통 엔지니어링 workflow에 대한 적용 범위가 누락되었습니다.
- skills 간의 불일치

## 라이센스

기여함으로써 귀하는 귀하의 기여가 MIT 라이선스에 따라 라이선스가 부여된다는 데 동의합니다.
