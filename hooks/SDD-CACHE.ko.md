# sdd-cache hook

[`source-driven-development`](../skills/source-driven-development/SKILL.ko.md)를 위한 세션 간 인용(citation) 캐시. skill의 "현재 문서에 대해 검증" 보장을 약화시키지 않으면서 중복 `WebFetch` 호출을 건너뜁니다.

## 왜

`source-driven-development`는 프레임워크별 결정마다 공식 문서를 fetch합니다. 같은 프로젝트를 여러 세션에 걸쳐 작업하면 같은 페이지를 계속해서 fetch하게 됩니다. 내용을 로컬 메모리로 캐싱하면 skill과 모순됩니다 — 문서는 변하고, 오래된 캐시는 그것을 숨깁니다.

이 hook은 fetch한 내용을 디스크에 캐싱하지만, HTTP `If-None-Match` / `If-Modified-Since`를 통해 **재사용할 때마다 origin 서버와 재검증**합니다. 내용은 서버가 `304 Not Modified`로 응답할 때만 캐시에서 제공되며, 이는 메모리 읽기가 아니라 새로운 검증입니다.

## 설정

1. `.claude/settings.json`(또는 개인용은 `.claude/settings.local.json`)에 hook을 추가합니다:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "WebFetch",
        "hooks": [
          {
            "type": "command",
            "command": "bash ${CLAUDE_PROJECT_DIR}/hooks/sdd-cache-pre.sh",
            "timeout": 10
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "WebFetch",
        "hooks": [
          {
            "type": "command",
            "command": "bash ${CLAUDE_PROJECT_DIR}/hooks/sdd-cache-post.sh",
            "async": true,
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

   `${CLAUDE_PROJECT_DIR}`는 Claude Code를 실행한 디렉터리로 해석됩니다. 위 스니펫은 hook이 동일한 프로젝트 안에 있을 때 동작합니다. `agent-skills`를 다른 곳에 설치했다면(예: `~/agent-skills` 아래 공유 플러그인으로), `${CLAUDE_PROJECT_DIR}/hooks/...`를 각 스크립트의 절대 경로로 교체하세요.

2. `.claude/sdd-cache/`가 `.gitignore`에 있는지 확인하세요 (이 repo에는 이미 포함됨).

3. 평소처럼 `/source-driven-development`(또는 skill)를 사용하세요. skill이나 agent 워크플로에 변경 없음 — 캐시는 투명합니다.

## 멘탈 모델

URL을 키로 하는 HTTP 리소스 캐시. freshness는 `ETag` / `Last-Modified`를 통해 origin에 위임됨; TTL 없음, 키에 프롬프트 없음.

저장된 본문은 raw HTML이 아닙니다 — `WebFetch`는 호출자의 프롬프트를 사용해 각 응답을 모델로 후처리하므로, 캐싱하는 것은 한 agent의 페이지 읽기입니다. 키는 URL만으로 유지되어 세션 간 읽기가 재사용되며; 원래 프롬프트는 메타데이터로 보관되고 hit 메시지에 표시되어 다음 agent가 이전 읽기가 맞는지 판단할 수 있습니다.

## 어떻게 동작하는가

URL당 하나의 캐시 엔트리, `.claude/sdd-cache/<sha>.json`에 JSON으로 저장:

| 이벤트 | 동작 |
|---|---|
| `PreToolUse WebFetch` | 엔트리가 존재하면 `If-None-Match` / `If-Modified-Since`로 `HEAD` 요청을 보냄. `304`이면 fetch를 차단하고 캐시된 내용을 stderr로 agent에 반환하며, 원래 프롬프트를 메타데이터로 표시. 그렇지 않으면 fetch 허용. |
| `PostToolUse WebFetch` | 응답을 캡처하고, 현재 `ETag` / `Last-Modified`를 기록하기 위해 `HEAD` 요청을 발행하고, `{url, prompt, etag, last_modified, content, fetched_at}`를 저장. |

**Freshness 규칙:**

- origin이 `304 Not Modified`를 확인할 때만 엔트리가 제공됩니다.
- `ETag`나 `Last-Modified` 헤더가 없는 엔트리는 절대 캐싱되지 않습니다 — validator가 없으면 hook이 나중에 freshness를 검증할 수 없고, 캐싱은 메모리를 신뢰하는 것이 됩니다.
- 캐시 키는 `sha256(url)`입니다. 같은 URL을 다른 프롬프트로 물어도 같은 엔트리에 hit하며; 캐시된 본문은 첫 fetch에 사용된 프롬프트를 반영하고, 그 프롬프트가 hit과 함께 표시되어 agent가 재사용할지 수동으로 다시 fetch할지 결정할 수 있습니다.

**agent가 보는 것:**

- 캐시 hit: `WebFetch`가 exit code 2로 차단됩니다. Claude Code는 hook의 stderr payload를 tool error로 agent에 전달합니다 — 이는 실패가 아니라 캐시 hit에 대한 의도된 신호입니다. payload는 `[sdd-cache] Cache hit for <url>`로 접두되고 캐시된 본문을 `----- BEGIN CACHED CONTENT -----` / `----- END CACHED CONTENT -----` 마커로 감싸 agent가 `WebFetch`가 방금 반환한 것처럼 사용할 수 있게 합니다.
- 캐시 miss 또는 stale: `WebFetch`가 정상 실행됨; 결과는 다음을 위해 저장됨.

skill 자체는 변경되지 않습니다. 계속 `DETECT → FETCH → IMPLEMENT → CITE`를 따릅니다. hook은 `FETCH`가 실행될 때 내부에서 일어나는 일만 바꿉니다.

## 로컬 테스트

### 1. 스크립트를 직접 smoke test

```bash
# PostToolUse payload 시뮬레이션: 페이지 캐싱
echo '{
  "tool_input": {
    "url": "https://react.dev/reference/react/useActionState",
    "prompt": "extract the signature"
  },
  "tool_response": "useActionState(action, initialState) returns [state, formAction, isPending]"
}' | bash hooks/sdd-cache-post.sh

# 저장된 엔트리 검사
ls .claude/sdd-cache/
cat .claude/sdd-cache/*.json | jq .

# 같은 URL + 프롬프트에 대한 다음 PreToolUse 시뮬레이션
echo '{
  "tool_input": {
    "url": "https://react.dev/reference/react/useActionState",
    "prompt": "extract the signature"
  }
}' | bash hooks/sdd-cache-pre.sh
echo "exit=$?"
```

예상:

- 첫 번째 커맨드는 `.claude/sdd-cache/` 아래에 파일 하나를 생성합니다 (서버가 `ETag`나 `Last-Modified`를 반환한 경우에만).
- 두 번째 커맨드는 origin이 `304`로 응답하면 캐시된 내용을 stderr에 담아 `2`로 종료하고, 그렇지 않으면 조용히 `0`으로 종료합니다.

### 2. 실제 세션에서 end-to-end

1. 위에 보인 대로 `.claude/settings.local.json`에 hook을 등록합니다.
2. 이 repo에서 Claude Code 세션을 시작합니다.
3. agent에게 문서 페이지를 fetch하도록 요청합니다 (예: "fetch `https://react.dev/reference/react/useActionState` and summarize").
4. `.claude/sdd-cache/` 아래에 파일이 나타나는지 확인합니다.
5. agent에게 같은 프롬프트로 같은 페이지를 다시 fetch하도록 요청합니다.
6. 두 번째 `WebFetch`가 차단되고 캐시된 내용이 반환되는지 확인합니다 (세션 transcript에 `[sdd-cache]` 접두가 붙은 tool error로 표시됨).

### 3. Freshness 검증

문서가 변할 때 캐시가 무효화됨을 확인하려면 `ETag` 불일치를 강제하세요. 특정 엔트리 하나를 고르세요 — 캐시에 파일이 하나보다 많으면 `*.json`은 안전하지 않습니다:

```bash
# 손상시킬 엔트리 선택 (실제 파일명으로 교체)
ENTRY=.claude/sdd-cache/e49c9f378670cfbb1d7d871b6dee16d9.json

# origin이 인식하지 못할 값으로 ETag 패치
jq '.etag = "W/\"stale-etag-forced\""' "$ENTRY" > "$ENTRY.tmp" && mv "$ENTRY.tmp" "$ENTRY"

# 다음 PreToolUse는 miss해야 함 (서버가 304가 아니라 200 반환)
echo '{"tool_input":{"url":"...", "prompt":"..."}}' | bash hooks/sdd-cache-pre.sh
echo "exit=$?"   # 0 예상 (fetch 통과 허용)
```

### 4. 디버깅

두 hook 모두 debug 모드가 켜져 있으면 타임스탬프가 찍힌 이벤트를 `.claude/sdd-cache/.debug.log`에 기록합니다. 다음 중 하나로 활성화하세요:

```bash
# 옵션 A: env var (세션별)
SDD_CACHE_DEBUG=1 claude

# 옵션 B: sentinel 파일 (지속적)
mkdir -p .claude/sdd-cache && touch .claude/sdd-cache/.debug
# …비활성화: rm .claude/sdd-cache/.debug
```

로그는 URL, 감지된 `tool_response` 형태, HEAD 상태, 각 호출이 hit 또는 miss한 이유를 캡처합니다. 캐시 miss가 예상치 못하게 보일 때 유용합니다 (보통: origin이 validator 방출을 멈춤).

## 알려진 제약

- **본문은 프롬프트 형태입니다.** hit은 이전 agent의 페이지 읽기를 반환하며, 원래 프롬프트가 표시되어 현재 agent가 적용 가능한지 결정할 수 있습니다. 적용되지 않으면, `.claude/sdd-cache/` 아래 파일을 삭제해 재-fetch를 강제하세요.
- **모든 캐시 쓰기는 추가 HEAD 비용이 듭니다.** Claude Code는 `WebFetch`가 이미 받은 응답 헤더를 노출하지 않으므로, post hook이 `ETag` / `Last-Modified`를 캡처하기 위해 origin을 다시 쿼리합니다. miss당 추가 왕복 한 번 — core 변경 없이 순수 hook으로 유지하는 대가입니다.
- **`ETag`나 `Last-Modified`가 없는 서버는 절대 캐싱되지 않습니다.** 대부분의 공식 문서 사이트(react.dev, docs.djangoproject.com, developer.mozilla.org)는 validator를 방출합니다. 그렇지 않은 사이트는 항상 재-fetch됩니다.
- **오작동하는 서버는 잘못된 `304`를 제공할 수 있습니다.** 그것은 방어해야 할 캐시 불변식이 아니라 진단할 서버 버그입니다; TTL로 덮지 않습니다. stale한 것을 발견하면 엔트리를 삭제하세요.
- **캐시는 로컬이고 프로젝트별입니다.** 팀 전체 공유 캐시는 없습니다. 추가하려면 서명된 content-addressable 스토리지 레이어가 필요한데, 이는 범위 밖입니다.

## 요구사항

- `jq`
- `curl`
- `shasum` 또는 `sha256sum` (자동 감지)
- Bash 3.2+
