---
title: "SuperClaude v4 설치 가이드 (PowerShell 7)"
date: 2026-01-28
categories:
- claude-code
tags:
- superclaude
- claude-code
- powershell
- wsl2
- installation
keywords:
- superclaude 설치
- claude code 확장
- pipx
- uv
author: "hyunwoo"
draft: false
---

SuperClaude는 Claude Code를 확장하는 메타-프로그래밍 구성 프레임워크입니다. 30개 슬래시 명령어와 14개 전문 에이전트를 제공하며, Windows(PowerShell 7)와 WSL2/Linux 환경 모두에서 설치할 수 있습니다.

<!--more-->

> **작성일:** 2026-01-28 | **난이도:** 초급~중급 | **대상:** Claude Code 사용자

---

## TL;DR (핵심 요약)

> 1. **SuperClaude**는 Claude Code를 확장하는 프레임워크로, 30개 슬래시 명령어와 14개 전문 에이전트를 제공합니다.
> 2. **설치 방법:**
>    - Windows: `pipx install superclaude` → `superclaude install` → `superclaude doctor`
>    - WSL2/Linux: `uv tool install superclaude` → `superclaude install` → `superclaude doctor`
> 3. **필수 요구사항:** Python 3.8+, pipx 또는 uv, Claude Code

---

## 이 가이드가 필요한 이유

Claude Code는 기본적으로 대화, 파일 편집, 코드 실행 기능을 제공합니다. 하지만 복잡한 개발 작업을 할 때는 더 많은 기능이 필요합니다.

**SuperClaude가 해결하는 문제:**

- `/sc:research` - 주제 리서치 자동화
- `/sc:implement` - 기능 구현 가이드
- `/sc:brainstorm` - 아이디어 브레인스토밍
- `/sc:review` - 코드 리뷰 지원

---

## 1. SuperClaude 개요

### SuperClaude란?

SuperClaude는 Claude Code를 확장하는 **메타-프로그래밍 구성 프레임워크**입니다.

> **메타-프로그래밍이란?**
> 프로그램이 다른 프로그램을 작성하거나 조작하는 기법입니다. SuperClaude는 Claude Code의 동작을 확장하고 새로운 명령어를 추가합니다.

### 핵심 기능

| 기능 | 설명 |
|------|------|
| 30개 슬래시 명령어 | /sc:research, /sc:implement, /sc:brainstorm 등 |
| 14개 전문 에이전트 | backend-architect, security-engineer 등 |
| Deep Research 시스템 | 자율적 웹 리서치 기능 |
| 토큰 효율성 70% 향상 | 더 긴 대화, 더 큰 컨텍스트 지원 |
| MCP 서버 통합 | Tavily, Context7, Sequential 등 |

### v3 vs v4 주요 차이점

| 항목 | v3 | v4 |
|------|-----|-----|
| 명령어 위치 | 루트 디렉토리 | sc/ 서브디렉토리 |
| 문서 폴더 | docs/ | Guides/ |
| 전문 에이전트 | 없음 | 14개 |
| Deep Research | 없음 | v4.2에서 추가 |
| 토큰 효율성 | 기준 | 70% 감소 |

---

## 2. 사전 요구 사항

### 필수 구성 요소

| 구성 요소 | 최소 버전 | 확인 명령어 |
|-----------|----------|------------|
| PowerShell | 7.0+ | $PSVersionTable.PSVersion |
| Python | 3.8+ | python --version |
| pipx | 최신 | pipx --version |
| Claude Code | 최신 | claude --version |
| Git | 2.0+ | git --version |

### 버전 확인 스크립트

```powershell
# 모든 요구 사항 한 번에 확인
Write-Host "=== 사전 요구 사항 확인 ===" -ForegroundColor Cyan

# PowerShell 버전
$psVersion = $PSVersionTable.PSVersion
Write-Host "PowerShell: $psVersion" -ForegroundColor $(if ($psVersion.Major -ge 7) { "Green" } else { "Red" })

# Python 버전
try {
    $pythonVersion = python --version 2>&1
    Write-Host "Python: $pythonVersion" -ForegroundColor Green
} catch {
    Write-Host "Python: 설치 필요" -ForegroundColor Red
}

# pipx 버전
try {
    $pipxVersion = pipx --version 2>&1
    Write-Host "pipx: $pipxVersion" -ForegroundColor Green
} catch {
    Write-Host "pipx: 설치 필요" -ForegroundColor Red
}

# Claude Code 버전
try {
    $claudeVersion = claude --version 2>&1
    Write-Host "Claude Code: $claudeVersion" -ForegroundColor Green
} catch {
    Write-Host "Claude Code: 설치 필요" -ForegroundColor Red
}
```

---

## 2-1. WSL2(Ubuntu) 환경 준비

> **WSL2 사용자를 위한 가이드**
> Windows에서 WSL2(Ubuntu)를 사용하는 경우, PowerShell 대신 아래 방법을 따르세요.

### 필수 구성 요소 (Ubuntu)

| 구성 요소 | 최소 버전 | 확인 명령어 |
|-----------|----------|------------|
| Ubuntu | 22.04+ | lsb_release -a |
| Python | 3.8+ | python3 --version |
| uv | 최신 | uv --version |
| Claude Code | 최신 | claude --version |

### 버전 확인 스크립트 (Bash)

```bash
#!/bin/bash
echo "=== 사전 요구 사항 확인 ==="

# Ubuntu 버전
echo -n "Ubuntu: "
lsb_release -d | cut -f2

# Python 버전
echo -n "Python: "
python3 --version 2>/dev/null || echo "설치 필요"

# uv 버전
echo -n "uv: "
uv --version 2>/dev/null || echo "설치 필요"

# Claude Code 버전
echo -n "Claude Code: "
claude --version 2>/dev/null || echo "설치 필요"
```

### uv 설치 (sudo 불필요)

```bash
# uv 설치
curl -LsSf https://astral.sh/uv/install.sh | sh

# PATH 적용 (자동으로 ~/.bashrc에 추가됨)
source ~/.bashrc

# 설치 확인
uv --version
```

### Claude Code 설치 (Ubuntu)

```bash
# npm으로 설치
npm install -g @anthropic-ai/claude-code

# 또는 공식 설치 스크립트
curl -fsSL https://claude.ai/install.sh | sh

# 설치 확인
claude --version
```

---

## 3. PowerShell 7 환경 준비

### 3.1 PowerShell 7 설치

```powershell
# winget으로 설치 (권장)
winget install Microsoft.PowerShell

# 설치 후 새 터미널에서 확인
pwsh
$PSVersionTable.PSVersion
```

### 3.2 Python 설치

```powershell
# winget으로 Python 설치
winget install Python.Python.3.12

# 설치 확인
python --version

# pip 업그레이드
python -m pip install --upgrade pip
```

### 3.3 pipx 설치

> **pipx란?**
> Python CLI 도구를 격리된 환경에서 설치/실행하는 도구입니다. 패키지 충돌 없이 안전하게 설치할 수 있습니다.

```powershell
# pipx 설치
python -m pip install --user pipx

# PATH에 pipx 추가
python -m pipx ensurepath

# 터미널 재시작 후 확인
pipx --version
```

### 3.4 Claude Code 설치

```powershell
# npm으로 설치
npm install -g @anthropic-ai/claude-code

# 또는 공식 설치 스크립트
irm https://claude.ai/install.ps1 | iex

# 설치 확인
claude --version
```

### 3.5 실행 정책 설정

```powershell
# 현재 사용자에 대해 스크립트 실행 허용
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# 확인
Get-ExecutionPolicy -List
```

---

## 4. SuperClaude 설치

### 방법 1: pipx 설치 (권장)

```powershell
# SuperClaude 설치
pipx install superclaude

# 슬래시 명령어 설치 (30개)
superclaude install

# 설치 상태 확인
superclaude doctor
```

**설치 과정 출력 예시:**

```
Installing superclaude...
  installed package superclaude 4.2.0, installed using Python 3.12.0
  These apps are now globally available
    - superclaude

Installing slash commands...
  ✓ /sc:research
  ✓ /sc:implement
  ✓ /sc:brainstorm
  ... (30개 명령어)

Installation complete! Restart Claude Code to activate.
```

### 방법 2: Git Clone 설치

```powershell
# 저장소 클론
git clone https://github.com/anthropics/superclaude.git

# 디렉토리 이동
cd superclaude

# 설치
pip install --user -e .
```

### 방법 3: uv 설치 (WSL2/Linux 권장)

> **uv란?**
> Astral에서 만든 초고속 Python 패키지 관리자입니다. pip/pipx보다 10-100배 빠르며, sudo 없이 설치 가능합니다.

```bash
# uv 설치 (sudo 불필요)
curl -LsSf https://astral.sh/uv/install.sh | sh

# PATH 적용
source ~/.bashrc

# SuperClaude 설치
uv tool install superclaude

# 슬래시 명령어 설치
superclaude install

# 설치 확인
superclaude doctor
```

**설치 과정 출력 예시:**

```
Installed 10 packages in 18ms
 + superclaude==4.2.0
Installed 1 executable: superclaude

Installing SuperClaude commands...
Installed 31 commands
```

### 설치 방법 비교

| 방법 | 장점 | 단점 | 권장 대상 |
|------|------|------|----------|
| pipx | 격리 환경, 충돌 없음 | pipx 설치 필요 | Windows/PowerShell |
| uv | 초고속, sudo 불필요, 격리 환경 | uv 설치 필요 | WSL2/Linux (권장) |
| Git Clone | 소스 수정 가능 | 수동 업데이트 | 개발자/기여자 |
| pip | 간단함 | 패키지 충돌 가능 | 단순 테스트 |

---

## 5. MCP 서버 구성 (선택)

> **MCP(Model Context Protocol)란?**
> Claude Code와 외부 서비스를 연결하는 프로토콜입니다. 웹 검색, 문서 조회 등 추가 기능을 사용할 수 있습니다.

### 사용 가능한 MCP 서버

| 서버 | 기능 | 용도 |
|------|------|------|
| Tavily | 웹 검색 | 실시간 정보 검색 |
| Context7 | 문서 검색 | 라이브러리/API 문서 조회 |
| Sequential | 추론 체인 | 단계별 논리적 추론 |
| Serena | 메모리 | 대화 컨텍스트 유지 |

### MCP 서버 설치

```powershell
# 대화형 설치 (권장)
superclaude mcp

# 특정 서버만 설치
superclaude mcp --servers tavily context7

# 모든 서버 설치
superclaude mcp --all
```

---

## 6. 설치 확인 및 테스트

### 6.1 설치 상태 진단

```powershell
# SuperClaude 진단 도구
superclaude doctor
```

**정상 출력 예시:**

```
SuperClaude Doctor v4.2.0
═════════════════════════

✓ Python version: 3.12.0
✓ Claude Code: installed
✓ SuperClaude: 4.2.0
✓ Slash commands: 30 installed
✓ MCP servers: 3 configured

All checks passed!
```

### 6.2 주요 슬래시 명령어

| 명령어 | 기능 |
|--------|------|
| /sc:research | 주제 리서치 및 분석 |
| /sc:implement | 기능/코드 구현 |
| /sc:brainstorm | 아이디어 브레인스토밍 |
| /sc:review | 코드 리뷰 |
| /sc:debug | 디버깅 지원 |
| /sc:refactor | 코드 리팩토링 |
| /sc:test | 테스트 작성 |
| /sc:docs | 문서 생성 |

### 6.3 기능 테스트

Claude Code를 재시작한 후 테스트:

```
# Claude Code 실행
claude

# 슬래시 명령어 테스트
> /sc:research PowerShell 7 새로운 기능

# 정상 동작 시 리서치 결과 출력
```

---

## 7. 문제 해결

### 7.1 pipx 명령어를 찾을 수 없음

> **오류:** 'pipx'은(는) 내부 또는 외부 명령... 이 아닙니다

```powershell
# 해결: PATH 재설정
python -m pipx ensurepath

# 터미널 재시작 또는 PATH 새로고침
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
```

### 7.2 superclaude 명령어를 찾을 수 없음

```powershell
# 해결 1: pipx 경로 확인
pipx list

# 해결 2: 재설치
pipx uninstall superclaude
pipx install superclaude
```

### 7.3 Claude Code에서 /sc: 명령어 미인식

```powershell
# 해결 1: 명령어 재설치
superclaude install --force

# 해결 2: Claude Code 완전 재시작
# Windows: 작업 관리자에서 Claude 프로세스 종료 후 재실행
```

### 7.4 인코딩 오류 (한글)

```powershell
# 해결: PowerShell 7에서 UTF-8 강제 설정
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
$OutputEncoding = [System.Text.Encoding]::UTF8

# 영구 설정: 프로필에 추가
Add-Content $PROFILE '[Console]::OutputEncoding = [System.Text.Encoding]::UTF8'
```

### 7.5 v3에서 v4로 업그레이드 충돌

```powershell
# 1. 기존 v3 완전 제거
pipx uninstall superclaude

# 2. 잔여 파일 삭제
$claudeDir = "$env:APPDATA\Claude"
Remove-Item "$claudeDir\commands" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item "$claudeDir\*.md" -Force -ErrorAction SilentlyContinue

# 3. v4 새로 설치
pipx install superclaude
superclaude install
```

---

## 8. 자주 묻는 질문

### 7.6 WSL2/Ubuntu에서 uv 명령어를 찾을 수 없음

> **오류:** uv: command not found

```bash
# 해결: PATH 적용
source ~/.bashrc

# 또는 직접 PATH 추가
export PATH="$HOME/.local/bin:$PATH"

# 영구 설정 확인
grep '.local/bin' ~/.bashrc || echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
```

### 7.7 WSL2에서 uv tool install 실패

```bash
# 해결 1: uv 재설치
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.bashrc

# 해결 2: 캐시 삭제 후 재설치
rm -rf ~/.cache/uv
uv tool install superclaude
```

### 7.8 WSL2에서 superclaude 명령어를 찾을 수 없음

```bash
# 해결: uv tool 경로 확인
uv tool list

# 재설치
uv tool uninstall superclaude
uv tool install superclaude
```

**Q1: SuperClaude와 Claude Code의 차이점은?**

**Claude Code**는 Anthropic의 공식 CLI 도구입니다. **SuperClaude**는 Claude Code를 확장하는 서드파티 프레임워크로, 추가 명령어와 에이전트를 제공합니다.

**Q2: Windows PowerShell 5.1에서도 사용 가능한가요?**

기술적으로 가능하지만 **PowerShell 7을 권장**합니다. UTF-8 인코딩 기본 지원, pipx 안정성, 최신 Python 도구와의 호환성이 더 좋습니다.

**Q3: 업데이트는 어떻게 하나요?**

```powershell
# pipx로 설치한 경우
pipx upgrade superclaude

# 슬래시 명령어도 업데이트
superclaude install --upgrade
```

**Q4: 제거는 어떻게 하나요?**

```powershell
pipx uninstall superclaude
```

**Q5: 오프라인에서 사용 가능한가요?**

SuperClaude 자체는 오프라인에서 동작하지만, MCP 서버(Tavily, Context7 등)는 **인터넷 연결이 필요**합니다.

---

## 요약: 빠른 시작

> **1분 안에 설치 완료하기**

**Step 1: 사전 요구 사항 설치**

```powershell
winget install Microsoft.PowerShell
winget install Python.Python.3.12
python -m pip install --user pipx
python -m pipx ensurepath
```

**Step 2: SuperClaude 설치**

```powershell
pwsh
pipx install superclaude
superclaude install
superclaude doctor
```

**Step 3: (선택) MCP 서버 설치**

```powershell
superclaude mcp
```

**Step 4: Claude Code 재시작 후 사용**

```
claude
> /sc:research [주제]
> /sc:implement [기능]
```

---

## 참고 자료

### WSL2(Ubuntu) + uv 빠른 시작

> **30초 안에 설치 완료하기 (WSL2)**

**Step 1: uv 설치 (sudo 불필요)**

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.bashrc
```

**Step 2: SuperClaude 설치**

```bash
uv tool install superclaude
superclaude install
superclaude doctor
```

**Step 3: (선택) MCP 서버 설치**

```bash
superclaude mcp
```

**Step 4: Claude Code 재시작 후 사용**

```bash
claude
> /sc:research [주제]
> /sc:implement [기능]
```

- [SuperClaude GitHub](https://github.com/anthropics/superclaude)
- [SuperClaude PyPI](https://pypi.org/project/superclaude/)
- [Claude Code 공식 문서](https://docs.anthropic.com/claude-code)

---

> *이 문서는 SuperClaude v4.2.0 기준으로 작성되었습니다.*
