# simplify-ignore hook

`/code-simplify`를 위한 Block 레벨 보호 도구입니다. 절대 단순화해서는 안 되는 코드를 표시하세요 — 모델이 해당 코드를 보지 못하게 됩니다.

## Setup

1. 보호하고 싶은 Block에 주석을 답니다:

```js
/* simplify-ignore-start: perf-critical */
// 수동으로 전개된 XOR — 루프보다 3배 빠름
result[0] = buf[0] ^ key[0];
result[1] = buf[1] ^ key[1];
result[2] = buf[2] ^ key[2];
result[3] = buf[3] ^ key[3];
/* simplify-ignore-end */
```

2. `.claude/settings.json`에 Hook을 추가합니다:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Read",
        "hooks": [{ "type": "command", "command": "bash ${CLAUDE_PROJECT_DIR}/hooks/simplify-ignore.sh" }]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{ "type": "command", "command": "bash ${CLAUDE_PROJECT_DIR}/hooks/simplify-ignore.sh" }]
      }
    ],
    "Stop": [
      {
        "hooks": [{ "type": "command", "command": "bash ${CLAUDE_PROJECT_DIR}/hooks/simplify-ignore.sh" }]
      }
    ]
  }
}
```

3. `/code-simplify`를 실행합니다 — 보호된 Block은 `/* BLOCK_de115a1d: perf-critical */` Placeholder로 바뀝니다. 모델은 보호된 구현 내용을 보지 않은 채 주변 코드를 기반으로 추론합니다.

> **Note:** 이 Hook은 임시 백업을 `.claude/.simplify-ignore-cache/`에 저장합니다. 이 경로가 `.gitignore`에 포함되어 있는지 확인하세요.

## How it works

하나의 Script가 세 가지 Hook Event를 처리합니다:

| Event | Action |
|---|---|
| `PreToolUse Read` | 파일을 백업하고, Block을 즉석에서 `BLOCK_<hash>` Placeholder로 교체함 |
| `PostToolUse Edit\|Write` | Placeholder를 실제 코드로 다시 확장하고, 모델의 변경 사항을 저장한 후 다시 필터링함 |
| `Stop` | 세션이 종료될 때 백업에서 모든 파일을 복원함 |

각 Block은 콘텐츠 해시(shasum/sha1sum을 통한 8자리 16진수) 처리되므로 모델이 Placeholder를 복제하거나 순서를 바꾸더라도 왕복 과정이 명확합니다. Cache는 세션 간 간섭을 방지하기 위해 프로젝트 범위(Project-scoped)로 관리됩니다.

## Annotation syntax

```js
/* simplify-ignore-start */           // 기본 — Block을 숨김
/* simplify-ignore-start: reason */   // 이유 포함 — Placeholder에 나타남
/* simplify-ignore-end */
```

어떤 주석 스타일(`//`, `/*`, `#`, `<!--`)도 작동합니다. 파일당 여러 Block 및 단일 행 Block을 지원합니다. Placeholder는 원래의 주석 구문을 유지합니다 (예: Python의 경우 `# BLOCK_xxx`, HTML의 경우 `<!-- BLOCK_xxx -->`).

## Crash recovery

Claude Code가 Stop Hook을 트리거하지 못하고 중단된 경우, 디스크의 파일에 여전히 `BLOCK_<hash>` Placeholder가 남아 있을 수 있습니다. 수동으로 복원하려면 다음을 실행하세요:

```bash
echo '{}' | bash hooks/simplify-ignore.sh
```

백업은 프로젝트 디렉토리 내의 `.claude/.simplify-ignore-cache/`에 저장됩니다.

## Known limitations

- **단일 행 Block은 행 전체를 숨깁니다.** `simplify-ignore-start`와 `simplify-ignore-end`가 다른 코드와 같은 행에 나타나면, 주석 처리된 부분뿐만 아니라 행 전체가 모델에게 숨겨집니다. 주석에는 전용 행을 사용하세요.
- **주석 접미사 감지는 `*/`와 `-->`만 지원합니다.** 비표준 주석 종료자(ERB `%>`, Blade `--}}`)를 사용하는 템플릿 엔진은 Placeholder가 불균형하게 생성될 수 있습니다. 대신 `#` 또는 `//` 스타일의 주석을 사용하세요.
- **대체 확장(Fallback expansion)은 정확하지 않고 점진적입니다.** 모델이 Placeholder의 형식을 변경하는 경우(예: 이유 텍스트 변경), Hook은 점진적으로 더 단순한 매칭을 시도합니다: 전체 Placeholder → 접두사+해시+접미사 → 해시 전용. 해시 전용 대체 방식은 화장품 데브리(예: 길을 잃은 `:` 또는 이유 텍스트)를 남길 수 있습니다. 이 경우 stderr에 경고가 출력됩니다.
- **파일 이름을 변경하면 Placeholder가 남습니다.** 모델이 Shell 명령을 통해 파일 이름을 변경하거나 이동하면, 새 파일에 `BLOCK_<hash>` Placeholder가 유지됩니다. 세션이 중단될 때 원본 코드는 `<old-filename>.recovered`로 저장됩니다. 복구된 코드를 새 파일에 수동으로 복원해야 합니다.

## Requirements

- `jq`, `shasum` 또는 `sha1sum` (자동 감지), Bash 3.2 이상
