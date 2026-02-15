---
title: "Claude Code 다중 계정 설정 가이드"
date: 2026-01-28
categories:
- ai
tags:
- claude-code
- multi-account
- powershell
- CLAUDE_CONFIG_DIR
keywords:
- claude code 다중 계정
- 계정 전환
- CLAUDE_CONFIG_DIR
author: "hyunwoo"
draft: false
---

Claude Code는 공식적으로 다중 계정을 지원하지 않습니다. 하지만 `CLAUDE_CONFIG_DIR` 환경변수를 활용하면 재부팅 없이 업무용/개인용 계정을 간편하게 전환할 수 있습니다. 이 가이드에서는 PowerShell 프로필에 함수를 등록하여 다중 계정을 운영하는 방법을 설명합니다.

<!--more-->

> **TL;DR (3줄 요약)**
> - `CLAUDE_CONFIG_DIR` 환경변수로 계정별 별도 데이터 디렉토리를 지정하면 다중 계정 운영 가능
> - PowerShell 프로필에 계정별 함수(`claude-work`, `claude-personal`)를 등록하여 간편하게 전환
> - 심볼릭 링크로 슬래시 명령어 공유, 전역 CLAUDE.md로 프로필별 규칙 관리

---

## 이 가이드가 필요한 이유

Claude Code는 공식적으로 다중 계정을 지원하지 않습니다. 하지만 업무용/개인용 계정을 분리하거나, 같은 프로젝트에서 두 계정을 동시에 사용해야 하는 경우가 있습니다.

이 가이드에서는 `CLAUDE_CONFIG_DIR` 환경변수를 활용하여 **재부팅 없이** 계정을 전환하는 방법을 설명합니다.

> **`CLAUDE_CONFIG_DIR`이란?**
> Claude Code가 설정 및 데이터 파일을 저장하는 디렉토리 경로를 변경하는 환경변수입니다. 미설정 시 기본 경로는 `~/.claude/`입니다. 자세한 내용은 [Claude Code Settings 공식 문서](https://code.claude.com/docs/en/settings)를 참조하세요.

### 이 가이드로 가능한 것

- 재부팅/로그아웃 없이 계정 전환
- 프로젝트별 다른 계정 사용
- 같은 프로젝트에서 2개 계정 동시 실행

---

## 사전 요구사항

- Windows 10/11
- PowerShell 5.1 이상 (또는 PowerShell 7)
- Claude Code CLI 설치 완료
- 2개 이상의 Claude 계정 (Pro, Max, Teams 등)

---

## 설정 방법

### 1단계: PowerShell 프로필 확인

> **PowerShell 프로필이란?** PowerShell이 시작될 때 자동으로 실행되는 스크립트 파일입니다. 여기에 함수를 등록하면 매번 수동으로 설정할 필요가 없습니다.

```powershell
# 프로필 경로 확인
$PROFILE

# 프로필 파일 존재 여부 확인
Test-Path $PROFILE
```

프로필이 없으면 생성합니다:

```powershell
New-Item -Path $PROFILE -ItemType File -Force
```

### 2단계: 프로필에 함수 추가

프로필 파일을 열어 함수를 추가합니다:

```powershell
# 프로필 편집 (메모장)
notepad $PROFILE

# 또는 VS Code
code $PROFILE
```

아래 내용을 프로필에 추가합니다:

```powershell
# ============================================
# Claude Code 다중 계정 설정
# ============================================

# 업무용 계정
function claude-work {
    $env:CLAUDE_CONFIG_DIR = "C:\Users\$env:USERNAME\.claude-work"
    claude $args
}

# 개인용 계정
function claude-personal {
    $env:CLAUDE_CONFIG_DIR = "C:\Users\$env:USERNAME\.claude-personal"
    claude $args
}

# 계정 확인용 함수
function claude-whoami {
    Write-Host "=== Claude Code 계정 정보 ===" -ForegroundColor Cyan

    $accounts = @(
        @{ Name = "work"; Path = "C:\Users\$env:USERNAME\.claude-work" },
        @{ Name = "personal"; Path = "C:\Users\$env:USERNAME\.claude-personal" }
    )

    foreach ($account in $accounts) {
        $credPath = Join-Path $account.Path ".credentials.json"
        if (Test-Path $credPath) {
            Write-Host "[$($account.Name)] " -NoNewline -ForegroundColor Green
            Write-Host "설정됨 - $($account.Path)"
        } else {
            Write-Host "[$($account.Name)] " -NoNewline -ForegroundColor Yellow
            Write-Host "미설정 - 로그인 필요"
        }
    }
}

# 별칭 설정 (선택사항)
Set-Alias cw claude-work
# 주의: 'cp'는 PowerShell 기본 별칭 Copy-Item과 충돌합니다.
# 충돌을 피하려면 'cpe' 등 다른 별칭을 사용하세요.
Set-Alias cpe claude-personal
```

> **주의:** `Set-Alias cp claude-personal`을 사용하면 PowerShell 기본 명령어 `Copy-Item`의 별칭 `cp`와 충돌합니다. `cpe` 등 다른 별칭을 사용하세요.

### 3단계: 프로필 적용

```powershell
# 현재 세션에 프로필 적용
. $PROFILE
```

### 4단계: 각 계정 초기 로그인 (최초 1회)

**업무용 계정 로그인:**

```powershell
claude-work
# Claude Code가 실행되면 로그인 진행
# 로그인 완료 후 /exit로 종료
```

**개인용 계정 로그인:**

```powershell
claude-personal
# 다른 계정으로 로그인 진행
# 로그인 완료 후 /exit로 종료
```

---

## 사용 방법

### 기본 사용

```powershell
# 업무용 계정으로 Claude Code 실행
claude-work

# 개인용 계정으로 Claude Code 실행
claude-personal

# 별칭 사용 (설정한 경우)
cw     # claude-work
cpe    # claude-personal
```

### 특정 디렉토리에서 실행

```powershell
# 업무 프로젝트
cd C:\projects\work-project
claude-work

# 개인 프로젝트
cd C:\projects\personal-project
claude-personal
```

### 인자 전달

```powershell
# --help 등 인자 전달
claude-work --help
claude-personal --version

# 프롬프트 직접 전달
claude-work "이 프로젝트 구조를 설명해줘"
```

### 계정 상태 확인

```powershell
claude-whoami
```

출력 예시:

```
=== Claude Code 계정 정보 ===
[work] 설정됨 - C:\Users\username\.claude-work
[personal] 설정됨 - C:\Users\username\.claude-personal
```

---

## 동시 실행 (같은 프로젝트)

두 개의 터미널 창에서 각각 다른 계정으로 동시 실행이 가능합니다.

**터미널 1:**

```powershell
cd C:\projects\my-project
claude-work
```

**터미널 2:**

```powershell
cd C:\projects\my-project
claude-personal
```

> **주의:** 같은 파일을 동시에 수정하면 충돌이 발생할 수 있습니다. 작업 영역을 분리하거나, 한쪽은 읽기 전용으로 사용하세요.

---

## 고급 설정

### 계정 3개 이상 추가

프로필에 함수를 추가로 정의합니다:

```powershell
# 팀 프로젝트용
function claude-team {
    $env:CLAUDE_CONFIG_DIR = "C:\Users\$env:USERNAME\.claude-team"
    claude $args
}

# 테스트용
function claude-test {
    $env:CLAUDE_CONFIG_DIR = "C:\Users\$env:USERNAME\.claude-test"
    claude $args
}
```

### 프로젝트별 자동 계정 선택

특정 디렉토리에서 자동으로 계정을 선택하려면:

```powershell
function claude-auto {
    $currentPath = Get-Location

    if ($currentPath -like "*\work\*" -or $currentPath -like "*\company\*") {
        Write-Host "[자동 선택] 업무용 계정 사용" -ForegroundColor Cyan
        $env:CLAUDE_CONFIG_DIR = "C:\Users\$env:USERNAME\.claude-work"
    } else {
        Write-Host "[자동 선택] 개인용 계정 사용" -ForegroundColor Cyan
        $env:CLAUDE_CONFIG_DIR = "C:\Users\$env:USERNAME\.claude-personal"
    }

    claude $args
}

Set-Alias ca claude-auto
```

### 슬래시 명령어(commands) 공유

기본 `claude` 명령으로 먼저 사용하다가 다중 계정을 구성하면, 커스텀 슬래시 명령어나 플러그인이 이미 `~/.claude/commands/`에 축적되어 있습니다. `CLAUDE_CONFIG_DIR`을 변경한 프로필에서는 이 경로를 인식하지 못하므로 기존 슬래시 명령어를 사용할 수 없습니다.

> **해결 방법:** 각 프로필의 config 디렉토리에 심볼릭 링크를 생성하여 기본 `~/.claude/commands/`를 공유합니다.

```powershell
# 관리자 PowerShell에서 실행
New-Item -ItemType SymbolicLink `
    -Path "$env:USERPROFILE\.claude-work\commands" `
    -Target "$env:USERPROFILE\.claude\commands"

New-Item -ItemType SymbolicLink `
    -Path "$env:USERPROFILE\.claude-personal\commands" `
    -Target "$env:USERPROFILE\.claude\commands"
```

이렇게 하면 기본 계정(`~/.claude`)에서 명령어를 관리하고, 다른 프로필에서도 동일한 슬래시 명령어를 사용할 수 있습니다.

확인:

```powershell
# 심볼릭 링크 확인
Get-Item "$env:USERPROFILE\.claude-work\commands" | Select-Object FullName, LinkTarget
```

### 계정 재로그인

특정 계정을 다시 로그인해야 할 경우:

```powershell
# 업무용 계정 로그아웃 후 재로그인
$env:CLAUDE_CONFIG_DIR = "C:\Users\$env:USERNAME\.claude-work"
claude logout
claude login
```

### 프로필별 전역 CLAUDE.md 설정

각 프로필의 config 디렉토리에 CLAUDE.md 파일을 두면, 해당 프로필로 실행하는 **모든 프로젝트**에 전역 규칙이 적용됩니다.

```
C:\Users\<username>\.claude-work\CLAUDE.md      # claude-work 전용 전역 규칙
C:\Users\<username>\.claude-personal\CLAUDE.md   # claude-personal 전용 전역 규칙
```

> **활용 예시: PowerShell 한글 출력 깨짐 해결**
>
> Windows 환경에서 Claude Code가 Bash 도구를 통해 PowerShell 명령을 실행하면 한글이 깨지는 고질적인 문제가 있습니다. 이는 Windows 콘솔 기본 인코딩이 UTF-8이 아닌 cp949(EUC-KR)이기 때문입니다.

프로필별 전역 CLAUDE.md에 아래와 같은 규칙을 명시하면, Claude Code가 항상 UTF-8 인코딩을 설정한 뒤 명령을 실행하도록 강제할 수 있습니다:

**CLAUDE.md에 포함할 PowerShell 한글 깨짐 방지 규칙 예시**

1. **Bash에서 PowerShell 실행 시** 반드시 아래 형식을 사용:

```
powershell.exe -NoProfile -Command "[Console]::OutputEncoding = [System.Text.Encoding]::UTF8; $OutputEncoding = [System.Text.Encoding]::UTF8; <실제 명령>"
```

2. **.ps1 스크립트 생성/수정 시:**
   - 파일 인코딩은 UTF-8 BOM으로 저장
   - 스크립트 상단에 인코딩 설정 포함:

```powershell
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
$OutputEncoding = [System.Text.Encoding]::UTF8
```

3. **파일 출력 시** `-Encoding UTF8` 파라미터 명시:

```powershell
$result | Out-File -FilePath "output.txt" -Encoding UTF8
```

---

## 문제 해결

**프로필이 로드되지 않음**

```powershell
# 실행 정책 확인
Get-ExecutionPolicy

# 필요시 정책 변경
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

**로그인 정보가 유지되지 않음**

데이터 디렉토리 권한을 확인합니다:

```powershell
# 디렉토리 존재 및 권한 확인
Get-Acl "C:\Users\$env:USERNAME\.claude-work"
```

**계정 전환이 안 됨**

환경변수가 제대로 설정되었는지 확인합니다:

```powershell
# 현재 설정된 데이터 디렉토리 확인
$env:CLAUDE_CONFIG_DIR
```

**기존 기본 계정과의 관계**

기본 Claude Code 데이터는 `~/.claude` (Windows: `%USERPROFILE%\.claude`)에 저장됩니다. 이 가이드의 설정은 별도 디렉토리를 사용하므로 기존 설정에 영향을 주지 않습니다.

```powershell
# 환경변수 제거 후 기본 claude 실행
Remove-Item Env:CLAUDE_CONFIG_DIR -ErrorAction SilentlyContinue
claude
```

---

## 디렉토리 구조

설정 완료 후 생성되는 디렉토리:

```
C:\Users\<username>\
├── .claude\                  # 기본 Claude Code 데이터 (기존)
│   ├── .credentials.json
│   ├── settings.json
│   ├── commands\             # 커스텀 슬래시 명령어 (원본)
│   └── ...
├── .claude-work\             # 업무용 계정 데이터
│   ├── .credentials.json
│   ├── settings.json
│   ├── commands\             # → .claude\commands 심볼릭 링크 (선택)
│   ├── CLAUDE.md             # 프로필 전역 규칙 (선택)
│   └── ...
└── .claude-personal\         # 개인용 계정 데이터
    ├── .credentials.json
    ├── settings.json
    ├── commands\             # → .claude\commands 심볼릭 링크 (선택)
    ├── CLAUDE.md             # 프로필 전역 규칙 (선택)
    └── ...
```

---

## 참고

> - 이 방법은 공식 기능이 아닌 워크어라운드입니다
> - Claude Code 업데이트 시 동작이 변경될 수 있습니다
> - 공식 다중 계정 지원 요청: [GitHub Issues](https://github.com/anthropics/claude-code/issues)
> - 환경변수 공식 문서: [Claude Code Settings](https://code.claude.com/docs/en/settings)
