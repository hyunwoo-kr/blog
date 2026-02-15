---
title: "PowerShell 7 설치 및 Windows PowerShell과의 차이점"
date: 2025-01-28
categories:
- devops
tags:
- powershell
- windows
- installation
- shell
keywords:
- PowerShell 7
- Windows PowerShell
- pwsh
- 설치
author: "hyunwoo"
draft: false
---

Windows에서 PowerShell을 실행하면 두 가지 버전이 있다는 걸 알게 됩니다. `powershell.exe`(5.1)는 Windows 기본 내장이고, `pwsh.exe`(7+)는 별도 설치가 필요합니다. PowerShell 7은 UTF-8 기본 지원, 새로운 연산자(`??`, `?.`, `&&`), 병렬처리 내장 등 다양한 개선을 포함합니다.

<!--more-->

## 이 가이드가 필요한 이유

Windows에서 PowerShell을 실행하면 **두 가지 버전**이 있습니다.

- **Windows PowerShell** (파란 아이콘) - Windows에 기본 설치됨
- **PowerShell 7** (검은 아이콘) - 별도 설치 필요

> **신입 개발자가 자주 하는 질문**
> "둘 다 PowerShell인데 뭐가 다른 거죠?"
> "어떤 걸 써야 하나요?"
> "둘 다 설치해도 되나요?"

이 가이드에서 모든 궁금증을 해결해 드립니다!

---

## 1. 핵심 비교: 5.1 vs 7

### 한눈에 보는 차이점

```
Windows PowerShell 5.1          PowerShell 7+
─────────────────────          ─────────────────
.NET Framework 기반            .NET Core/5+ 기반
Windows 전용                   크로스 플랫폼 (Win/Mac/Linux)
Windows 기본 포함              별도 설치 필요
유지보수 모드                  활발히 개발 중
powershell.exe                 pwsh.exe
```

> **쉽게 이해하기**
> Windows PowerShell 5.1은 "레거시(구버전)"이고, PowerShell 7은 "신버전"입니다.
> Microsoft는 더 이상 5.1에 새 기능을 추가하지 않고, 7에만 집중하고 있습니다.

### 상세 비교표

| 항목 | Windows PowerShell 5.1 | PowerShell 7+ |
|------|----------------------|---------------|
| 기반 런타임 | .NET Framework 4.x | .NET 6/7/8+ |
| 실행 파일 | `powershell.exe` | `pwsh.exe` |
| 설치 경로 | `C:\Windows\System32\WindowsPowerShell` | `C:\Program Files\PowerShell\7` |
| 지원 OS | Windows만 | Windows, macOS, Linux |
| 기본 인코딩 | 시스템 로캘 (CP949) | UTF-8 (BOM 없음) |
| 개발 상태 | 유지보수만 (신기능 없음) | 활발히 개발 중 |
| Windows 포함 | 기본 설치됨 | 별도 설치 필요 |

### 버전 확인하는 방법

```powershell
# 현재 사용 중인 PowerShell 버전 확인
$PSVersionTable.PSVersion

# Windows PowerShell이면: 5.1.xxxxx
# PowerShell 7이면: 7.x.x
```

---

## 2. 주요 차이점 상세

### 2.1 인코딩 (한글 처리) - 가장 체감되는 차이!

> **Windows PowerShell 5.1의 한글 문제**
> 기본 인코딩이 CP949(한국어 Windows 로캘)라서 UTF-8 파일을 읽으면 한글이 깨집니다.
> 스크립트 상단에 인코딩 설정을 추가해야 합니다.

**Windows PowerShell 5.1에서 한글 깨짐 해결:**

```powershell
# 스크립트 맨 위에 이 두 줄 추가 (필수!)
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
$OutputEncoding = [System.Text.Encoding]::UTF8

# 그리고 파일은 UTF-8 BOM으로 저장해야 함
```

PowerShell 7은 기본이 UTF-8이라 별도 설정 없이 한글이 잘 나옵니다.

```powershell
# PowerShell 7에서는 그냥 쓰면 됨
Write-Output "안녕하세요"  # 잘 나옴!
```

### 2.2 새로운 연산자 (코드가 짧아진다!)

**파이프라인 체인 연산자 `&&`, `||`**

Bash/Linux에서 쓰던 그 연산자! 이제 PowerShell 7에서도 사용 가능합니다.

```powershell
# PowerShell 7: 앞 명령 성공하면 다음 실행
Get-Process notepad && Write-Output "메모장 실행 중!"

# PowerShell 7: 앞 명령 실패하면 다음 실행
Get-Process notepad || Write-Output "메모장 안 켜져 있음"

# Windows PowerShell 5.1에서는 이렇게 길게 써야 함:
if (Get-Process notepad -ErrorAction SilentlyContinue) {
    Write-Output "메모장 실행 중!"
}
```

**삼항 연산자 `? :`**

조건문을 한 줄로 쓸 수 있습니다.

```powershell
# PowerShell 7: 깔끔한 한 줄
$result = $value -gt 10 ? "크다" : "작다"

# Windows PowerShell 5.1: 복잡한 if문
if ($value -gt 10) { $result = "크다" } else { $result = "작다" }
```

**Null 조건 연산자 `?.`, `??`**

JavaScript/C#에서 쓰던 것과 동일합니다.

```powershell
# PowerShell 7
$value = $null
$length = $value?.Length       # null이면 오류 없이 null 반환
$default = $value ?? "기본값"  # null이면 "기본값" 사용

# Windows PowerShell 5.1: 길고 복잡함
if ($null -ne $value) { $length = $value.Length }
if ($null -eq $value) { $default = "기본값" } else { $default = $value }
```

### 2.3 병렬 처리 (속도 향상!)

> **ForEach-Object -Parallel**
> PowerShell 7의 킬러 기능! 반복 작업을 병렬로 처리해서 속도가 훨씬 빨라집니다.

```powershell
# PowerShell 7: 병렬 처리 (5개씩 동시 실행)
1..10 | ForEach-Object -Parallel {
    Start-Sleep -Seconds 1
    "처리 완료: $_"
} -ThrottleLimit 5
# 결과: 10개 작업이 약 2초만에 완료!

# Windows PowerShell 5.1: Job 사용 (복잡함)
$jobs = 1..10 | ForEach-Object {
    Start-Job -ScriptBlock { Start-Sleep -Seconds 1; "처리 완료: $using:_" }
}
$jobs | Wait-Job | Receive-Job
```

### 2.4 성능 비교

| 작업 | Windows PowerShell 5.1 | PowerShell 7 | 개선율 |
|------|----------------------|--------------|--------|
| 시작 시간 | ~600ms | ~400ms | ~33% 빠름 |
| 대용량 파일 처리 | 기준 | 2-3배 빠름 | 100-200% |
| JSON 파싱 | 기준 | 2-5배 빠름 | 100-400% |
| 정규식 처리 | 기준 | 1.5-2배 빠름 | 50-100% |

### 2.5 호환성 주의사항

> **PowerShell 7에서 안 되는 것들**
> - `Get-WmiObject` -> `Get-CimInstance`로 대체
> - `workflow` 키워드 -> 제거됨
> - 스냅인(Snap-in) -> 모듈(Module)로 대체

---

## 3. PowerShell 7 설치 방법

### 방법 1: winget (가장 쉬움, 권장!)

```powershell
# 이 한 줄이면 끝!
winget install Microsoft.PowerShell
```

> **winget 장점**
> - 가장 간단한 설치 방법
> - PATH 자동 설정
> - 업데이트도 간편: `winget upgrade Microsoft.PowerShell`

### 방법 2: MSI 설치 프로그램

1. [GitHub PowerShell Releases](https://github.com/PowerShell/PowerShell/releases) 방문
2. `PowerShell-7.x.x-win-x64.msi` 다운로드
3. 설치 시 다음 옵션 체크:
   - [x] Add PowerShell to Path Environment Variable
   - [x] Add 'Run with PowerShell 7' context menu

### 방법 3: Microsoft Store

Microsoft Store에서 "PowerShell" 검색 후 설치

- 장점: 자동 업데이트
- 단점: 일부 시스템 기능 제한

### 방법 4: Chocolatey

```powershell
choco install powershell-core
```

### 설치 방법 비교

| 방법 | 난이도 | 자동 업데이트 | 권장 대상 |
|------|--------|-------------|-----------|
| **winget** | 쉬움 | 수동 | **일반 개발자 (권장)** |
| MSI | 쉬움 | 수동 | 기업 환경 |
| Store | 가장 쉬움 | 자동 | 일반 사용자 |
| Chocolatey | 중간 | 수동 | DevOps |

---

## 4. 설치 확인 및 기본 설정

### 설치 확인

```powershell
# PowerShell 7 실행
pwsh

# 버전 확인
$PSVersionTable

# 출력 예시:
# PSVersion      7.5.0
# PSEdition      Core
# Platform       Win32NT
```

### 프로필 설정 (선택사항)

```powershell
# 프로필 경로 확인
$PROFILE

# 프로필 파일 생성
if (!(Test-Path -Path $PROFILE)) {
    New-Item -ItemType File -Path $PROFILE -Force
}

# 프로필 편집
notepad $PROFILE
```

**추천 프로필 설정:**

```powershell
# 유용한 별칭
Set-Alias -Name ll -Value Get-ChildItem
Set-Alias -Name touch -Value New-Item

# which 명령어 (Linux처럼)
function which($command) {
    Get-Command -Name $command -ErrorAction SilentlyContinue |
        Select-Object -ExpandProperty Definition
}
```

### Windows Terminal 기본 셸로 설정

1. Windows Terminal 실행
2. 설정 (Ctrl + ,)
3. "시작" -> "기본 프로필" -> "PowerShell" 선택

---

## 5. 두 버전 함께 사용하기

> **두 버전은 완전히 별개!**
> PowerShell 7을 설치해도 Windows PowerShell 5.1은 그대로 남아있습니다.
> 두 버전을 동시에 사용할 수 있습니다.

### 실행 파일 구분

```
powershell.exe  -> Windows PowerShell 5.1
pwsh.exe        -> PowerShell 7
```

### 스크립트에서 버전 강제하기

```powershell
#Requires -Version 7.0
# 이 스크립트는 PowerShell 7.0 이상에서만 실행됨

$result = $value -gt 10 ? "크다" : "작다"  # 삼항 연산자 사용
```

---

## 6. 자주 하는 실수 & 해결법

### 실수 1: Windows PowerShell을 삭제하려고 함

Windows PowerShell 5.1은 Windows 시스템 구성 요소입니다. 삭제할 수 없고, 삭제할 필요도 없습니다.

### 실수 2: 기존 스크립트가 안 돌아감

**확인해볼 것:**

```powershell
# WMI 사용하는지 확인
Select-String -Path .\*.ps1 -Pattern "Get-WmiObject"

# 있다면 Get-CimInstance로 변경
# Before
Get-WmiObject -Class Win32_OperatingSystem
# After
Get-CimInstance -ClassName Win32_OperatingSystem
```

### 실수 3: pwsh 명령어가 안 됨

**원인:** PATH에 PowerShell 7이 없음

**해결:**

```powershell
# 재설치하면서 PATH 옵션 체크
winget install Microsoft.PowerShell

# 또는 수동으로 PATH 추가
$env:PATH += ";C:\Program Files\PowerShell\7"
```

---

## 7. 자주 묻는 질문 (FAQ)

**Q1: 어떤 버전을 써야 하나요?**

- **새 프로젝트** -> PowerShell 7 권장
- **기존 레거시 스크립트** -> 호환성 테스트 후 결정
- **WSL2 관련 스크립트** -> Windows PowerShell 5.1도 OK

**Q2: PowerShell 7 업데이트는?**

```powershell
# winget으로 설치했다면
winget upgrade Microsoft.PowerShell
```

**Q3: VS Code에서 버전 선택하려면?**

VS Code 하단 상태 표시줄에서 PowerShell 버전 클릭 -> 원하는 버전 선택

---

## 요약: 빠른 시작 가이드

**1단계: 설치**

```powershell
winget install Microsoft.PowerShell
```

**2단계: 확인**

```powershell
pwsh
$PSVersionTable.PSVersion
```

**3단계: 사용**

- 새 터미널 열 때: `pwsh`
- 레거시 스크립트 실행: `powershell`

---

*작성일: 2025-01-28 | 대상: Windows PowerShell 사용 경험이 있는 개발자*
