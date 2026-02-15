---
title: "Telegram MCP 알림 설정 가이드 - Claude Code 작업 알림을 텔레그램으로 받기"
date: 2026-02-09
categories:
- ai
tags:
- mcp
- telegram
- claude-code
- notification
- bot
keywords:
- Telegram MCP
- Claude Code 알림
- 텔레그램 봇
- BotFather
author: "hyunwoo"
draft: false
---

Claude Code로 빌드, 테스트, 코드 생성 같은 오래 걸리는 작업을 맡기면 완료될 때까지 화면 앞에서 기다려야 합니다. Telegram MCP 알림을 설정하면 작업 완료, 사용자 입력 대기, 빌드 결과를 텔레그램으로 받을 수 있습니다.

<!--more-->

> **TL;DR (3줄 요약)**
> 1. Telegram BotFather로 봇을 만들고 Token을 발급받는다
> 2. `claude-telegram-alerts` MCP 서버를 설치하고 `.mcp.json`에 설정한다
> 3. Claude Code가 작업 완료/입력 필요 시 Telegram 알림이 온다

---

## 이 가이드가 필요한 이유

Claude Code로 **빌드, 테스트, 코드 생성** 같은 오래 걸리는 작업을 맡기면, 완료될 때까지 화면 앞에서 기다려야 합니다.

Telegram MCP 알림을 설정하면:

- 작업이 **완료**되었을 때 알림
- Claude가 **사용자 입력/승인**을 기다릴 때 알림
- **빌드 성공/실패** 결과 알림

모바일에서도 받을 수 있어서, 다른 일을 하다가 알림이 오면 돌아와서 확인하면 됩니다.

---

## 알림이 오는 상황

| 이벤트 | 설명 | 예시 |
|--------|------|------|
| 사용자 입력 대기 | Claude가 승인이나 입력을 기다릴 때 | "파일을 삭제할까요?" 같은 확인 요청 |
| 장시간 작업 완료 | 오래 걸리는 작업이 끝났을 때 | 빌드, 테스트, 대규모 리팩토링 완료 |
| 빌드 성공/실패 | 빌드 결과 알림 | `mvn clean install` 성공/실패 |
| 커스텀 알림 | 사용자가 직접 요청한 알림 | "완료되면 텔레그램으로 알려줘" |

---

## 설정 절차

### Step 1: Telegram Bot 생성

> **Bot이란?** Telegram에서 자동으로 메시지를 보내는 프로그램입니다. BotFather라는 공식 봇을 통해 쉽게 만들 수 있습니다.

1. Telegram 앱에서 **@BotFather** 검색 후 대화 시작
2. `/newbot` 명령어 입력
3. **봇 이름** 설정 (예: `Claude 알림봇`)
4. **사용자명** 설정 (예: `my_claude_alert_bot` -- 반드시 `_bot`으로 끝나야 함)
5. 발급되는 **Bot Token** 복사

```
예시 토큰 형식: 123456789:ABCdefGHIjklMNOpqrsTUVwxyz
```

6. **중요!** 생성된 봇에게 아무 메시지나 **한 번 전송** (이걸 안 하면 봇이 메시지를 보낼 수 없음)

---

### Step 2: 서버 설치

```bash
# WSL2 환경에서 실행
git clone https://github.com/anthony-potts/claude-telegram-alerts.git
cd claude-telegram-alerts
npm install
npm run build
```

빌드 완료 후 `build/index.js` 파일이 생성됩니다.

> **Node.js 필요!** `npm` 명령어가 안 되면 먼저 Node.js를 설치하세요: `sudo apt install nodejs npm`

---

### Step 3: MCP 설정 추가

설정 파일에 Telegram MCP 서버를 등록합니다. 두 가지 방법 중 선택하세요.

**방법 A: 프로젝트 단위 설정 (`.mcp.json`)**

해당 프로젝트에서만 알림을 받으려면 프로젝트 루트의 `.mcp.json`에 추가합니다.

```json
{
  "mcpServers": {
    "telegram-alerts": {
      "command": "node",
      "args": ["/설치경로/claude-telegram-alerts/build/index.js"],
      "env": {
        "TELEGRAM_BOT_TOKEN": "봇토큰",
        "TELEGRAM_CHAT_ID": "채팅ID"
      }
    }
  }
}
```

**방법 B: 전역 설정 (`~/.claude/settings.json`)**

모든 프로젝트에서 알림을 받으려면 전역 설정에 추가합니다.

```json
{
  "mcpServers": {
    "telegram-alerts": {
      "command": "node",
      "args": ["/설치경로/claude-telegram-alerts/build/index.js"],
      "env": {
        "TELEGRAM_BOT_TOKEN": "봇토큰",
        "TELEGRAM_CHAT_ID": "채팅ID"
      }
    }
  }
}
```

---

### Step 4: Chat ID 확인

Bot Token만 먼저 설정한 상태에서 Claude Code를 재시작합니다.

Claude Code에게 다음과 같이 요청:

> get_chat_id 도구를 사용해서 내 Telegram Chat ID를 찾아줘

반환된 Chat ID를 설정 파일의 `TELEGRAM_CHAT_ID`에 입력 후 다시 재시작합니다.

> **수동 확인 방법**: `get_chat_id` 도구가 안 되면 터미널에서 직접 확인할 수 있습니다:
> ```bash
> curl -s "https://api.telegram.org/bot<BOT_TOKEN>/getUpdates" | python3 -m json.tool
> ```
> 응답의 `message.chat.id` 값이 Chat ID입니다.

---

### Step 5: 동작 확인

Claude Code 재시작 후 테스트:

```
"Telegram으로 테스트 알림 보내줘"
```

Telegram에 메시지가 도착하면 설정 완료!

---

## 환경 변수 정리

| 변수명 | 필수 | 설명 |
|--------|------|------|
| `TELEGRAM_BOT_TOKEN` | O | BotFather에서 발급받은 봇 인증 토큰 |
| `TELEGRAM_CHAT_ID` | O | 알림을 받을 Telegram 채팅 ID |

---

## 자주 하는 실수 & 해결법

> **실수 1: 봇에게 먼저 메시지를 안 보냄**
> Telegram 봇은 사용자가 먼저 대화를 시작해야 메시지를 보낼 수 있습니다. 봇을 만든 후 반드시 한 번 메시지를 보내세요.

> **실수 2: MCP 설정 변경 후 재시작 안 함**
> `.mcp.json`이나 `settings.json`을 수정한 후에는 반드시 Claude Code를 **완전히 재시작**해야 합니다. 설정 변경이 자동 반영되지 않습니다.

> **실수 3: Node.js 경로 문제 (WSL2)**
> WSL2 환경에서 `node` 명령어가 안 되는 경우 `which node`로 경로를 확인하세요. 경로가 출력되지 않으면 Node.js 설치가 필요합니다.

> **실수 4: Bot Token 재발급이 필요한 경우**
> 토큰이 노출되었거나 동작하지 않으면 BotFather에서 `/token` -> 봇 선택으로 재발급할 수 있습니다.

---

## 기타 Telegram MCP 서버 (참고)

알림이 아닌 다른 용도(채팅, 모바일 제어 등)가 필요하면 아래 프로젝트를 참고하세요.

| 프로젝트 | 용도 | 링크 |
|----------|------|------|
| telegram-mcp (chigwell) | 채팅 읽기/보내기, 그룹 관리 | [GitHub](https://github.com/chigwell/telegram-mcp) |
| tsgram-mcp | 모바일에서 Claude Code 제어 | [GitHub](https://github.com/areweai/tsgram-mcp) |
| mcp-telegram (sparfenyuk) | MTProto 네이티브 프로토콜 | [GitHub](https://github.com/sparfenyuk/mcp-telegram) |
| telegram-mcp-server (l1v0n1) | Claude용 간단한 서버 | [GitHub](https://github.com/l1v0n1/telegram-mcp-server) |

---

> **핵심 요약**: BotFather로 봇 생성 -> Token 복사 -> 봇에게 메시지 전송 -> `claude-telegram-alerts` 설치 -> `.mcp.json`에 설정 -> Chat ID 확인 -> 재시작 -> 끝!
