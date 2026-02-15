---
title: "Tailscale 기반 원격 개발환경 구축 가이드"
date: 2026-01-19
categories:
- devops
tags:
- tailscale
- vpn
- remote-development
- ssh
- wireguard
keywords:
- Tailscale
- 원격 개발
- VPN
- SSH
- WireGuard
author: "hyunwoo"
draft: false
---

집의 Mac Studio를 원격지에서 안전하게 접속하여 개발하기 위한 환경 구성 가이드입니다. Tailscale(WireGuard 기반 메시 VPN)을 활용하여 어디서든 개발 장비에 접근할 수 있는 환경을 구축합니다.

<!--more-->

## 개요

**구성 환경**

- 서버: Mac Studio (Cursor IDE, IntelliJ, Claude Code Max, Docker 컨테이너)
- 클라이언트: Windows 11 노트북
- VPN: Tailscale (WireGuard 기반 메시 VPN)

**구성도**

```
[원격지: 노트북 Win11] ──Tailscale VPN──> [집: Mac Studio]
                                              ├── Cursor IDE
                                              ├── IntelliJ
                                              ├── Claude Code Max
                                              └── Docker Containers
```

---

## 1단계: Tailscale 설치 및 설정

### Mac Studio (서버) 설정

**1. Tailscale 설치**

```bash
# Homebrew로 설치
brew install --cask tailscale

# 또는 App Store에서 Tailscale 검색하여 설치
```

**2. 로그인 및 활성화**

- Tailscale 앱 실행 -> Google/GitHub/Microsoft 계정으로 로그인
- 메뉴바에서 Tailscale 아이콘 -> "Connect" 클릭
- 할당된 IP 확인 (예: `100.x.x.x`)

**3. SSH 서버 활성화**

```bash
# macOS 시스템 설정에서 SSH 활성화
# 시스템 설정 -> 일반 -> 공유 -> 원격 로그인 켜기

# 또는 터미널에서
sudo systemsetup -setremotelogin on
```

**4. 머신 이름 설정 (선택)**

- Tailscale 관리 콘솔(https://login.tailscale.com/admin/machines)에서 머신 이름 변경
- 예: `mac-studio` -> `mac-studio.tail1234.ts.net`으로 접속 가능

### Windows 11 (클라이언트) 설정

**1. Tailscale 설치**

- https://tailscale.com/download/windows 에서 다운로드
- 설치 후 동일한 계정으로 로그인

**2. 연결 확인**

```powershell
# PowerShell에서 Mac Studio ping 테스트
ping 100.x.x.x  # Mac Studio의 Tailscale IP

# 또는 머신 이름으로
ping mac-studio
```

---

## 2단계: SSH 접속 설정

### Windows SSH 키 생성 (최초 1회)

```powershell
# PowerShell에서 실행
ssh-keygen -t ed25519 -C "원격개발용"

# 기본 경로: C:\Users\사용자명\.ssh\id_ed25519
```

### Mac Studio에 공개키 등록

```powershell
# Windows에서 실행 - Mac Studio에 공개키 복사
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh 사용자명@mac-studio "cat >> ~/.ssh/authorized_keys"
```

### SSH Config 설정 (Windows)

`C:\Users\사용자명\.ssh\config` 파일 생성 또는 편집:

```
Host mac-studio
    HostName mac-studio  # 또는 Tailscale IP
    User 사용자명
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

설정 후 간단히 접속:

```powershell
ssh mac-studio
```

---

## 3단계: 터미널 기반 개발 (Claude Code)

### 접속 및 사용

```powershell
# SSH 접속
ssh mac-studio

# 프로젝트 디렉토리로 이동
cd ~/projects/my-project

# Claude Code 실행
claude
```

### Windows Terminal 프로필 추가 (권장)

Windows Terminal 설정 -> 새 프로필 추가:

```json
{
    "name": "Mac Studio (Claude Code)",
    "commandline": "ssh mac-studio",
    "icon": "🖥️",
    "startingDirectory": "~"
}
```

---

## 4단계: IDE 원격 연결

### VS Code / Cursor Remote SSH

**1. Remote - SSH 확장 설치**

- Cursor/VS Code -> 확장 -> "Remote - SSH" 검색 및 설치

**2. 원격 호스트 연결**

- `Ctrl + Shift + P` -> "Remote-SSH: Connect to Host"
- `mac-studio` 선택 (또는 SSH config에 등록된 이름)
- 첫 연결 시 VS Code Server가 Mac Studio에 자동 설치됨

**3. 프로젝트 열기**

- 연결 후 "Open Folder" -> Mac Studio의 프로젝트 경로 선택

### IntelliJ Gateway

**1. JetBrains Gateway 설치**

- https://www.jetbrains.com/remote-development/gateway/ 에서 다운로드

**2. SSH 연결 설정**

- Gateway 실행 -> "SSH" 선택
- Host: `mac-studio` (또는 Tailscale IP)
- Username: Mac Studio 사용자명
- 인증: Key pair 선택

**3. IDE 선택 및 프로젝트 열기**

- Mac Studio에 설치할 IDE 버전 선택 (IntelliJ IDEA)
- 프로젝트 경로 지정
- "Connect" -> 원격 IDE 백엔드 자동 설치 및 연결

---

## 5단계: 컨테이너 서비스 접근

### 웹 서비스 접근

Mac Studio에서 실행 중인 컨테이너 서비스에 직접 접근:

```
# 예시 - Mac Studio의 Tailscale IP가 100.64.0.1인 경우

개발 서버: http://100.64.0.1:3000
DB 관리도구: http://100.64.0.1:8080
API 서버: http://100.64.0.1:8000
```

### hosts 파일 설정 (선택)

`C:\Windows\System32\drivers\etc\hosts` 편집 (관리자 권한 필요):

```
100.64.0.1  mac-studio.local
```

이후 `http://mac-studio.local:3000` 형태로 접근 가능

---

## 6단계: Tailscale 보안 강화

### MFA (다중 인증) 활성화

1. https://login.tailscale.com/admin/settings/keys 접속
2. "Require multi-factor authentication" 활성화

### ACL (접근 제어) 설정

https://login.tailscale.com/admin/acls 에서 설정:

```json
{
  "acls": [
    {
      "action": "accept",
      "src": ["*"],
      "dst": ["*:*"]
    }
  ],
  "tagOwners": {},
  "autoApprovers": {}
}
```

### 키 만료 설정

- 관리 콘솔에서 각 기기의 키 만료 기간 설정 가능
- 기본값: 180일 (필요시 조정)

---

## 트러블슈팅

### 연결이 안 될 때

```powershell
# Tailscale 상태 확인
tailscale status

# 네트워크 재연결
tailscale down
tailscale up
```

### SSH 연결 끊김이 잦을 때

SSH config에 keep-alive 설정 추가:

```
Host mac-studio
    ServerAliveInterval 30
    ServerAliveCountMax 5
    TCPKeepAlive yes
```

### 속도가 느릴 때

```powershell
# DERP 릴레이 사용 여부 확인
tailscale netcheck

# Direct 연결이 아니면 방화벽/공유기 설정 확인
# UDP 41641 포트 개방 권장
```

---

## 일일 워크플로우

**출근 후**

1. Windows 노트북에서 Tailscale 연결 확인 (자동 연결됨)
2. Windows Terminal에서 `ssh mac-studio` 또는
3. Cursor에서 Remote SSH로 프로젝트 열기
4. Claude Code로 개발 시작

**퇴근 후 집에서**

- Mac Studio에서 직접 작업 (Tailscale 불필요)

---

## 참고 자료

- Tailscale 공식 문서: https://tailscale.com/kb
- VS Code Remote Development: https://code.visualstudio.com/docs/remote/ssh
- JetBrains Gateway: https://www.jetbrains.com/help/idea/remote-development-overview.html
