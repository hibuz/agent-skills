# simplify-ignore hook

`/code-simplify`를 위한 블록 단위 보호. 절대 단순화되어서는 안 되는 코드를 표시하세요 — 모델은 그것을 보지 못합니다.

## 설정

1. 보호하려는 블록에 주석을 답니다:

```js
/* simplify-ignore-start: perf-critical */
// manually unrolled XOR — 3x faster than a loop
result[0] = buf[0] ^ key[0];
result[1] = buf[1] ^ key[1];
result[2] = buf[2] ^ key[2];
result[3] = buf[3] ^ key[3];
/* simplify-ignore-end */
```

2. `.claude/settings.json`에 hook을 추가합니다:

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

3. `/code-simplify`를 실행합니다 — 보호된 블록은 `/* BLOCK_de115a1d: perf-critical */` placeholder가 됩니다. 모델은 보호된 구현을 보지 않고 주변 코드를 추론합니다.

> **참고:** hook은 임시 백업을 `.claude/.simplify-ignore-cache/`에 저장합니다. 이 경로가 `.gitignore`에 있는지 확인하세요.

## 어떻게 동작하는가

스크립트 하나, hook 이벤트 세 개:

| 이벤트 | 동작 |
|---|---|
| `PreToolUse Read` | 파일을 백업하고, 블록을 `BLOCK_<hash>` placeholder로 in-place 교체 |
| `PostToolUse Edit\|Write` | placeholder를 실제 코드로 다시 확장하고, 모델의 변경을 저장하고, 재-필터링 |
| `Stop` | 세션이 끝나면 모든 파일을 백업에서 복원 |

각 블록은 content-hash(`shasum`/`sha1sum`을 통한 8 hex 문자)되어, 모델이 placeholder를 복제하거나 재정렬하더라도 왕복이 모호하지 않습니다. 캐시는 프로젝트 스코프로, 세션 간 간섭을 방지합니다.

## 주석 문법

```js
/* simplify-ignore-start */           // 기본 — 블록을 숨김
/* simplify-ignore-start: reason */   // reason 포함 — placeholder에 나타남
/* simplify-ignore-end */
```

어떤 주석 스타일도 동작합니다 (`//`, `/*`, `#`, `<!--`). 파일당 여러 블록과 한 줄 블록 지원. placeholder는 원래 주석 문법을 보존합니다 (예: Python은 `# BLOCK_xxx`, HTML은 `<!-- BLOCK_xxx -->`).

## 크래시 복구

Claude Code가 Stop hook을 트리거하지 않고 크래시하면, 디스크의 파일에 여전히 `BLOCK_<hash>` placeholder가 남아 있을 수 있습니다. 수동으로 복원하려면:

```bash
echo '{}' | bash hooks/simplify-ignore.sh
```

백업은 프로젝트 디렉터리 내 `.claude/.simplify-ignore-cache/`에 저장됩니다.

## 알려진 제약

- **한 줄 블록은 줄 전체를 숨깁니다.** `simplify-ignore-start`와 `simplify-ignore-end`가 다른 코드와 같은 줄에 나타나면, 주석 단 부분만이 아니라 줄 전체가 모델에서 숨겨집니다. 주석에는 전용 줄을 사용하세요.
- **주석 접미사 감지는 `*/`와 `-->`만 커버합니다.** 비표준 주석 종료자(ERB `%>`, Blade `--}}`)를 가진 템플릿 엔진은 불균형 placeholder를 생성할 수 있습니다. 대신 `#`나 `//` 스타일 주석을 사용하세요.
- **Fallback 확장은 점진적이며 정확하지 않습니다.** 모델이 placeholder의 포맷팅을 바꾸면(예: reason 텍스트 변경), hook은 점진적으로 더 단순한 매치를 시도합니다: 전체 placeholder → prefix+hash+suffix → hash-only. hash-only fallback은 외형적 잔해(예: 떠도는 `:`나 reason 텍스트)를 남길 수 있습니다. 이때 stderr에 경고가 출력됩니다.
- **파일 이름 변경은 placeholder를 남깁니다.** 모델이 shell 커맨드로 파일을 이름 변경하거나 이동하면, 새 파일은 `BLOCK_<hash>` placeholder를 유지합니다. 원래 코드는 세션이 멈출 때 `<old-filename>.recovered`로 저장됩니다. 복구된 코드를 새 파일로 수동 복원해야 합니다.

## 요구사항

- `jq`, `shasum` 또는 `sha1sum` (자동 감지), Bash 3.2+
