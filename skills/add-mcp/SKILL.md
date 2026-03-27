# Add MCP Skill

MCP 서버를 추가하고 설정을 git에 커밋하는 스킬.
"MCP 추가", "add-mcp", "/add-mcp" 요청에 사용.

## Step 1: 서버 정보 수집

AskUserQuestion으로 아래 항목을 받는다:
- 서버 이름 (예: slack, playwright, github)
- 패키지 또는 URL (예: @modelcontextprotocol/server-slack)
- 환경변수 (토큰, API 키 — 절대 채팅에 붙여넣지 말고 별도로 전달)

## Step 2: 추가 실행

설정 파일 위치: `~/.claude.json` (settings.json이 아님)

```bash
# 환경변수 없는 경우
claude mcp add {name} -- npx -y {package}

# 환경변수 있는 경우
claude mcp add {name} -e KEY=VALUE -- npx -y {package}
```

## Step 3: 설정 확인

```bash
claude mcp list
```

## Step 4: claude-code 저장소에 반영

MCP 설정은 `~/.claude.json`에 있으므로, 관련 문서를 업데이트한다:

```bash
cd /c/Users/chanh/claude-code
# MEMORY.md의 MCP 서버 목록 업데이트
git add .
git commit -m "chore: add {name} MCP server"
git push
```

## Step 5: 결과 리포트

- 추가된 서버명
- 사용 가능한 주요 툴 목록
- 재시작 필요 여부 안내 (Claude Code 재시작 필요)

## 보안 주의

- 토큰/시크릿은 채팅에 직접 붙여넣지 않는다
- 노출된 토큰은 즉시 revoke 후 재발급
