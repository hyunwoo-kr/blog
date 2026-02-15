---
title: "n8n + Claude Code 연동 가이드 (WSL2 홈랩)"
date: 2025-02-01
categories:
- devops
tags:
- n8n
- claude-code
- docker
- ssh
- automation
keywords:
- n8n claude code
- workflow automation
- ssh node
- wsl2 homelab
author: "hyunwoo"
draft: false
---

WSL2에서 n8n 워크플로우로 Claude Code CLI를 호출하는 방법입니다. 호스트 네트워크 모드와 SSH 노드를 활용하여 Docker 컨테이너 내부에서 호스트의 CLI 도구를 호출합니다.

<!--more-->

---

## 1. 이 가이드가 필요한 이유

n8n은 강력한 워크플로우 자동화 도구지만, Docker 컨테이너 내부에서 호스트의 CLI 도구를 호출하는 건 까다롭습니다.

이 가이드에서는:
- n8n을 **호스트 네트워크 모드**로 실행
- **SSH 노드**를 통해 로컬 Claude CLI 호출
- Telegram 봇과 연동하여 AI 질의응답 시스템 구축

---

## 2. 아키텍처

```
+─────────────────────────────────────────────────────────────────────+
|                              인터넷                                   |
+─────────────────────────────────────────────────────────────────────+
                                    |
                                    v
+─────────────────────────────────────────────────────────────────────+
|  Caddy (:80/:443) - HTTPS 종단, 자동 인증서                          |
+─────────────────────────────────────────────────────────────────────+
                                    |
                                    v
+─────────────────────────────────────────────────────────────────────+
|  Traefik - Docker 라벨 기반 동적 라우팅                               |
+─────────────────────────────────────────────────────────────────────+
                                    |
          +─────────────────────────+─────────────────────────+
          v                         v                         v
+──────────────────+    +──────────────────+    +──────────────────+
|  GitLab CE       |    |  n8n (호스트모드)  |    |  기타 서비스      |
+──────────────────+    +────────┬─────────+    +──────────────────+
                                 |
                                 v SSH
                        +──────────────────+
                        |  Claude Code CLI |
                        +──────────────────+
```

---

## 3. n8n Docker 구성

### 3.1 docker-compose.yml

```yaml
services:
  n8n:
    image: n8nio/n8n:2.4.8
    container_name: n8n
    restart: unless-stopped
    network_mode: host  # 핵심: 호스트 네트워크
    user: "1000:1000"   # 호스트 사용자 UID:GID
    environment:
      - N8N_HOST=n8n.your-domain.com
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - WEBHOOK_URL=https://n8n.your-domain.com/
      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
      - TZ=Asia/Seoul
    volumes:
      - /srv/n8n:/home/node/.n8n
      - /srv/n8n/ssh:/home/node/.ssh:ro  # SSH 키 마운트
```

> **주의**: 호스트 네트워크 모드는 컨테이너가 호스트의 모든 포트에 접근 가능합니다. n8n 사용자 인증을 반드시 활성화하세요.

### 3.2 Traefik 파일 프로바이더 설정

호스트 네트워크 모드에서는 Docker 라벨 라우팅이 불가능합니다. **파일 프로바이더**를 사용합니다:

```yaml
# traefik/dynamic/n8n.yml
http:
  routers:
    n8n:
      rule: "Host(`n8n.your-domain.com`)"
      entrypoints:
        - web
      service: n8n

  services:
    n8n:
      loadBalancer:
        servers:
          - url: "http://172.17.0.1:5678"  # docker0 게이트웨이
```

---

## 4. SSH 설정 (핵심)

n8n 2.x에서는 Execute Command 노드가 보안상 비활성화되어 있습니다. **SSH 노드**로 localhost에 연결합니다.

### 4.1 SSH 키 생성

```bash
# n8n 전용 SSH 키 생성
mkdir -p /srv/n8n/ssh
ssh-keygen -t ed25519 -f /srv/n8n/ssh/id_ed25519 -N ""
chown -R 1000:1000 /srv/n8n/ssh

# 공개키를 authorized_keys에 추가
cat /srv/n8n/ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
```

### 4.2 n8n Credentials 설정

n8n UI에서:

1. **Settings** -> **Credentials** -> **Add**
2. **SSH** 선택
3. 설정:
   - Name: `localhost-ssh`
   - Host: `127.0.0.1`
   - Port: `22`
   - Username: `<your-username>`
   - Authentication: `Private Key`
   - Private Key: `/srv/n8n/ssh/id_ed25519` 내용 붙여넣기

---

## 5. 워크플로우 예시

### Telegram -> Claude 질의응답

```
+─────────+      +─────────+      +──────────+
| Webhook |─────>|   SSH   |─────>| Telegram |
| Trigger |      | Claude  |      |  Send    |
+─────────+      +─────────+      +──────────+
```

**SSH 노드 명령어**:

```bash
=cd /home/user/project && /home/user/.local/bin/claude -p "{{ $json.body.question }}" --allowedTools WebSearch 2>&1 | head -c 4000
```

> **핵심 포인트**:
> 1. 명령어 앞에 `=` 접두사 필수 (n8n 표현식)
> 2. claude 전체 경로 사용 (SSH는 PATH가 다름)
> 3. `$json.body.question` - Webhook 데이터 경로 확인

---

## 6. 자주 하는 실수 & 해결

| 문제 | 원인 | 해결 |
|------|------|------|
| `claude: command not found` | SSH 세션에서 PATH 미설정 | 전체 경로 사용: `/home/user/.local/bin/claude` |
| 표현식 `{{ }}` 그대로 출력 | `=` 접두사 누락 | 명령어 앞에 `=` 추가 |
| `access to env vars denied` | n8n 2.x 환경변수 접근 제한 | Credentials 또는 Variables 사용 |
| `Unrecognized node type: executeCommand` | n8n 2.x에서 노드 비활성화 | SSH 노드로 대체 |

---

## 7. 보안 체크리스트

- [ ] n8n 사용자 인증 활성화
- [ ] SSH 키 권한 600 설정
- [ ] Webhook에 시크릿 토큰 설정
- [ ] n8n 포트 외부 직접 접근 차단
- [ ] 암호화 키 안전하게 보관

---

## 8. 참고 자료

- [n8n SSH 노드 문서](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.ssh/)
- [n8n 표현식 가이드](https://docs.n8n.io/code/expressions/)
- [Claude Code CLI 문서](https://docs.anthropic.com/claude-code/)
