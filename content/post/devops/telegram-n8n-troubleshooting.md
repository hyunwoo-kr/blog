---
title: "Telegram -> n8n -> Claude Code 연동 트러블슈팅"
date: 2025-02-01
categories:
- devops
tags:
- telegram
- n8n
- claude-code
- troubleshooting
- automation
keywords:
- telegram bot n8n
- claude code integration
- n8n troubleshooting
- workflow automation
author: "hyunwoo"
draft: false
---

Telegram에서 `/ask` 명령어로 질문하면 n8n을 통해 Claude Code가 답변하는 시스템을 구축하는 과정에서 발생한 n8n 2.x의 여러 제약사항을 해결한 실전 가이드입니다.

<!--more-->

---

## 1. 최종 시스템 구성

```
+──────────+    +──────────────+    +─────────────────+    +────────────+
| Telegram |───>| telegram-bot |───>|   n8n Webhook   |───>|    SSH     |
|   User   |    |   (Node.js)  |    |  /claude-ask    |    | localhost  |
+──────────+    +──────────────+    +─────────────────+    +──────┬─────+
     ^                                                            |
     |                                                            v
     |          +──────────────+    +─────────────────+    +────────────+
     +──────────|   Telegram   |<───|  n8n Telegram   |<───|   Claude   |
                |     API      |    |     Node        |    |    CLI     |
                +──────────────+    +─────────────────+    +────────────+
```

---

## 2. 초기 설계 vs 실제 구현

> n8n 2.x의 보안 제약으로 인해 초기 설계와 다르게 구현되었습니다.

| 초기 설계 | 실제 구현 | 변경 이유 |
|----------|----------|----------|
| Execute Command 노드 | SSH 노드 | n8n 2.x에서 노드 비활성화 |
| HTTP Request + `$env` | Telegram 노드 + Credentials | 환경변수 접근 제한 |

---

## 3. 구현 과정에서 발생한 문제와 해결

### 3.1 Execute Command 노드 미지원

**오류**: `Unrecognized node type: n8n-nodes-base.executeCommand`

**원인**: n8n 2.x에서 보안상 기본 비활성화

**해결**: SSH 노드로 localhost 연결

```bash
# SSH 키 생성 및 설정
mkdir -p /srv/n8n/ssh
ssh-keygen -t ed25519 -f /srv/n8n/ssh/id_ed25519 -N ""
cat /srv/n8n/ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
```

---

### 3.2 환경변수 접근 거부

**오류**: `access to env vars denied`

**원인**: n8n 2.x에서 `$env` 접근 제한

**해결**: Telegram 노드 + Credentials 사용

```javascript
# 기존 (HTTP Request)
https://api.telegram.org/bot{{ $env.TELEGRAM_BOT_TOKEN }}/sendMessage

# 변경 (Telegram 노드)
Telegram 노드에서 Credentials로 Bot Token 관리
```

---

### 3.3 Claude 명령어 경로 문제

**오류**: `claude: command not found`

**원인**: SSH 비대화형 세션에서 PATH 미설정

**해결**: 전체 경로 사용

```bash
# 수정 전
claude -p "질문"

# 수정 후
/home/user/.local/bin/claude -p "질문"
```

> **팁**: `which claude` 명령어로 전체 경로를 확인할 수 있습니다.

---

### 3.4 n8n 표현식 미해석

**오류**: `{{ $json.body.question }}` 문자 그대로 출력

**원인**: n8n 표현식에 `=` 접두사 누락

**해결**: 명령어 앞에 `=` 추가

```bash
# 수정 전
cd /project && claude -p "{{ $json.body.question }}"

# 수정 후 (맨 앞에 = 추가)
=cd /project && /home/user/.local/bin/claude -p "{{ $json.body.question }}"
```

---

### 3.5 Webhook 데이터 경로 오류

**오류**: `chatId is empty`

**원인**: 잘못된 JSON 경로 참조

**해결**: Webhook 데이터 구조 확인 후 올바른 경로 사용

```json
// Webhook이 받는 데이터 구조
{
  "headers": { ... },
  "body": {
    "chatId": "123456789",
    "question": "질문 내용"
  }
}
```

```javascript
# 수정 전
$json.chatId

# 수정 후
$json.body.chatId
```

---

### 3.6 N8N_WEBHOOK_BASE_URL 미설정

**오류**: `undefined/claude-ask`

**원인**: telegram-bot config에 URL 미설정

**해결**: 환경변수 추가

```bash
# ~/.config/telegram-monitor/config.env
N8N_WEBHOOK_BASE_URL=https://n8n.your-domain.com/webhook
```

---

## 4. 최종 n8n 워크플로우 구성

```
+─────────+      +─────────+      +──────────+
| Webhook |─────>|   SSH   |─────>| Telegram |
| Trigger |      | Claude  |      |  Send    |
+─────────+      +─────────+      +──────────+
```

### 노드별 설정

**Webhook 노드**:
- Path: `claude-ask`
- Method: POST
- Response Mode: Immediately

**SSH 노드**:
- Credentials: `localhost-ssh`
- Command:

```bash
=cd /home/user/project && /home/user/.local/bin/claude -p "{{ $json.body.question }} (1000자 이내로 요약)" --allowedTools WebSearch 2>&1 | head -c 4000
```

**Telegram 노드**:
- Credentials: `telegram-bot`
- Chat ID: `={{ $('Webhook').item.json.body.chatId }}`
- Text: `={{ $('SSH').item.json.stdout }}`

---

## 5. 필수 Credentials

| Credential Name | Type | 용도 |
|----------------|------|------|
| `localhost-ssh` | SSH Private Key | 호스트 SSH 연결 |
| `telegram-bot` | Telegram API | 메시지 전송 |

---

## 6. 디버깅 팁

### n8n 실행 로그 확인

n8n UI -> 워크플로우 -> **Executions** 탭에서 각 노드의 입출력 확인

### 직접 테스트

```bash
# Webhook 직접 호출
curl -s -X POST "https://n8n.your-domain.com/webhook/claude-ask" \
  -H "Content-Type: application/json" \
  -d '{"chatId": "123456789", "question": "안녕하세요"}'

# SSH 명령어 직접 테스트
ssh -i /srv/n8n/ssh/id_ed25519 user@127.0.0.1 \
  "/home/user/.local/bin/claude -p 'hi' 2>&1"
```

---

## 7. 향후 확장 아이디어

### 뉴스 브리핑 자동화

```
Schedule Trigger (매일 9시)
    |
SSH (claude -p "오늘 주요 뉴스 브리핑")
    |
Telegram (결과 전송)
```

### GitLab MR 자동 리뷰

```
GitLab Trigger (MR 생성)
    |
HTTP Request (diff 가져오기)
    |
SSH (claude -p "코드 리뷰: ...")
    |
GitLab (코멘트 작성)
```

---

## 8. 참고 자료

- [n8n SSH 노드 문서](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.ssh/)
- [n8n Telegram 노드 문서](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.telegram/)
- [n8n 표현식 가이드](https://docs.n8n.io/code/expressions/)
- [Claude Code CLI 문서](https://docs.anthropic.com/claude-code/)
