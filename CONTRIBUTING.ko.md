# Contributing to Agent Skills

기여에 관심을 가져주셔서 감사합니다! 이 프로젝트는 AI 코딩 Agent를 위한 프로덕션 등급의 엔지니어링 기술(Skill) 모음입니다.

## Adding a New Skill

1. `skills/` 하위에 kebab-case 이름으로 디렉토리를 생성합니다.
2. [docs/skill-anatomy.ko.md](docs/skill-anatomy.ko.md)의 형식을 따라 `SKILL.md`를 추가합니다.
3. `name`과 `description` 필드가 포함된 YAML Frontmatter를 포함합니다.
4. `description`은 "Use when"으로 시작하며 트리거 조건을 설명해야 합니다.

### Skill Quality Bar

기술은 다음과 같아야 합니다:

- **Specific** — 모호한 조언이 아닌 실행 가능한 단계
- **Verifiable** — 증거 요구 사항이 포함된 명확한 종료 기준(Exit Criteria)
- **Battle-tested** — 이론적인 이상향이 아닌 실제 엔지니어링 Workflow에 기반함
- **Minimal** — Agent를 올바르게 가이드하는 데 필요한 최소한의 내용

### Structure

모든 기술은 다음 섹션들을 포함해야 합니다:

- **Overview** — 이 기술이 무엇을 하며 왜 중요한지
- **When to Use** — 트리거 조건
- **Process** — 단계별 Workflow
- **Common Rationalizations** — Agent가 단계를 건너뛰기 위해 사용하는 변명과 그에 대한 반박
- **Red Flags** — 기술이 잘못 적용되고 있음을 나타내는 경고 징후
- **Verification** — 기술이 올바르게 적용되었음을 확인하는 방법

### What Not to Do

- 기술 간에 내용을 중복시키지 마세요 — 대신 다른 기술을 참조하세요.
- 실행 가능한 프로세스가 아닌 모호한 조언인 기술을 추가하지 마세요.
- 내용이 100행을 초과하지 않는 한 보조 파일을 만들지 마세요.
- 참조 자료를 기술 디렉토리 내부에 두지 마세요 — 대신 `references/`를 사용하세요.

## Modifying Existing Skills

- 변경 사항을 집중적이고 최소한으로 유지하세요.
- 기존의 구조와 어조를 유지하세요.
- 수정 후에도 YAML Frontmatter가 유효한지 확인하세요.

## Reporting Issues

다음과 같은 경우 이슈를 열어주세요:

- 잘못되었거나 오래된 가이드를 제공하는 기술을 발견했을 때
- 일반적인 엔지니어링 Workflow에 대한 커버리지가 누락되었을 때
- 기술 간에 불일치가 있을 때

## License

기여함으로써 귀하는 귀하의 기여가 MIT 라이선스에 따라 라이선스가 부여되는 것에 동의하게 됩니다.
