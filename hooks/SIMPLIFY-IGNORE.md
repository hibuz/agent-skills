# simplify-ignore 후크

`/code-simplify`에 대한 블록 수준 보호. 단순화해서는 안 되는 코드를 표시하십시오. 모델에서는 이를 볼 수 없습니다.

## 설정

1. 보호하려는 블록에 주석을 답니다.

```js
/* simplify-ignore-start: perf-critical */
// manually unrolled XOR — 3x faster than a loop
result[0] = buf[0] ^ key[0];
result[1] = buf[1] ^ key[1];
result[2] = buf[2] ^ key[2];
result[3] = buf[3] ^ key[3];
/* simplify-ignore-end */
```

2. `.claude/settings.json`에 후크를 추가합니다.

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

3. `/code-simplify`를 실행합니다. 보호된 블록은 `/* BLOCK_de115a1d: perf-critical */` 자리 표시자가 됩니다. 모델은 보호된 구현을 보지 않고 주변 코드에 대해 추론합니다.

> **참고:** 후크는 `.claude/.simplify-ignore-cache/`에 임시 백업을 저장합니다. 이 경로가 `.gitignore`에 있는지 확인하세요.

## 작동 방식

하나의 스크립트, 세 개의 후크 이벤트:

| 이벤트 | 액션 |
|---|---|
| `PreToolUse Read` | 파일을 백업하고 블록을 `BLOCK_<hash>` 자리 표시자로 교체합니다. in-place |
| `PostToolUse Edit\|Write` | 자리 표시자를 다시 실제 코드로 확장하고 모델의 변경 사항을 저장합니다. re-filters |
| `Stop` | Rest는 세션이 종료되면 백업에서 모든 파일을 가져옵니다. |

각 블록은 content-hashed(`shasum`/`sha1sum`를 통한 8개의 16진수 문자)이므로 모델이 자리 표시자를 복제하거나 재정렬하더라도 round-trip는 명확합니다. 캐시는 cross-session 간섭을 방지하기 위한 project-scoped입니다.

## 주석 구문

```js
/* simplify-ignore-start */           // basic — hides the block
/* simplify-ignore-start: reason */   // with reason — appears in placeholder
/* simplify-ignore-end */
```

모든 주석 스타일이 작동합니다(`//`, `/*`, `#`, `<!--`). 파일당 다중 블록 및 single-line 블록이 지원됩니다. 자리 표시자는 원래 주석 구문을 유지합니다(예: Python의 경우 `# BLOCK_xxx`, HTML의 경우 `<!-- BLOCK_xxx -->`).

## 충돌 복구

중지 후크를 트리거하지 않고 Claude Code가 충돌하는 경우 디스크의 파일에는 여전히 `BLOCK_<hash>` 자리 표시자가 있을 수 있습니다. restore를 수동으로 수행하려면:

```bash
echo '{}' | bash hooks/simplify-ignore.sh
```

백업은 프로젝트 디렉터리 내의 `.claude/.simplify-ignore-cache/`에 저장됩니다.

## 알려진 제한사항

- **한 줄 블록은 전체 줄을 숨깁니다.** `simplify-ignore-start` 및 `simplify-ignore-end`가 다른 코드와 같은 줄에 나타나면 주석이 달린 부분뿐만 아니라 전체 줄이 모델에서 숨겨집니다. 주석에는 전용선을 사용하세요.
- **댓글 접미사 감지에는 `*/` 및 `-->`만 포함됩니다.** non-standard 댓글 클로저(ERB `%>`, Blade `--}}`)가 포함된 템플릿 엔진은 불균형 자리 표시자를 생성할 수 있습니다. 대신 `#` 또는 `//` 스타일 주석을 사용하세요.
- **대체 확장은 정확하지 않고 점진적입니다.** 모델이 자리 표시자의 formatting을 변경하는 경우(예: 이유 텍스트 변경) 후크는 점진적으로 더 간단한 일치를 시도합니다. 전체 자리 표시자 → 접두사+해시+접미사 → hash-only. hash-only 대체는 외관상 잔해(예: 길 잃은 `:` 또는 이유 텍스트)를 남길 수 있습니다. 이런 일이 발생하면 stderr에 경고가 인쇄됩니다.
- **파일 이름을 바꾸면 자리 표시자가 남습니다.** 모델이 셸 명령을 통해 파일 이름을 바꾸거나 파일을 이동하는 경우 새 파일은 `BLOCK_<hash>` 자리 표시자를 유지합니다. 세션이 중지되면 원본 코드는 `<old-filename>.recovered`로 저장됩니다. 복구된 코드를 새 파일에 수동으로 restore해야 합니다.

## 요청 uirements

- `jq`, `shasum` 또는 `sha1sum` (auto-detected), 배쉬 3.2+
