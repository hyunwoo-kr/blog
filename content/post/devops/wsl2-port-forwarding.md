---
title: "WSL2 포트포워딩 & 방화벽 가이드"
date: 2025-01-28
categories:
- devops
tags:
- wsl2
- port-forwarding
- firewall
- powershell
- networking
keywords:
- WSL2
- 포트포워딩
- 방화벽
- netsh
- PowerShell
author: "hyunwoo"
draft: false
---

WSL2에서 서비스를 실행했는데 같은 네트워크의 다른 PC에서 접속이 안 되는 문제를 해결하는 포트포워딩 및 방화벽 설정 가이드입니다.

<!--more-->

## 빠른 시작

**5분 안에 외부에서 WSL2 서비스 접근 가능하게 만들기**

```powershell
# 0. PowerShell 정책 설정 (최초 1회)
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser

# 1. 관리자 PowerShell 열기: Win + X -> Windows PowerShell (관리자)

# 2. 스크립트 폴더로 이동
cd C:\scripts

# 3. 방화벽 규칙 추가 (최초 1회)
.\wsl2-firewall.ps1

# 4. 포트포워딩 설정 (재부팅할 때마다)
.\wsl2-port-forward.ps1

# 5. 상태 확인
.\wsl2-port-status.ps1
```

---

## 목차

| 파트 | 내용 | 대상 |
|------|------|------|
| Part 1 | 왜 필요한가? WSL2 네트워크 구조 | 처음 접하는 분 |
| Part 2 | 스크립트 사용법 | 바로 적용하고 싶은 분 |
| Part 3 | 트러블슈팅 | 문제가 생겼을 때 |
| Part 4 | 명령어 모음 | 수동 설정/참고용 |

---

# Part 1. 배경 이해

## 문제 상황

WSL2에서 서비스를 실행했는데, **같은 네트워크의 다른 PC에서 접속이 안 됩니다**.

```
[내 PC: 192.168.0.100]
    └── WSL2 (Ubuntu)
           └── MariaDB :3306 실행 중

[동료 PC] ────X────> 192.168.0.100:3306 (접속 실패!)
```

## 원인: WSL2의 가상 네트워크

WSL2는 **Hyper-V 가상 네트워크**를 사용합니다.

- Windows와 WSL2는 **서로 다른 IP 대역**
- 외부에서 `192.168.0.100:3306`으로 접속해도 WSL2까지 도달하지 않음
- **WSL2 IP는 재부팅할 때마다 변경됨!**

## 해결 방법

| 단계 | 설명 | 실행 주기 |
|------|------|-----------|
| 방화벽 | 포트를 열어줌 | **최초 1회** |
| 포트포워딩 | Windows -> WSL2 연결 | **재부팅마다** |

## WSL1 vs WSL2 비교

| 항목 | WSL1 | WSL2 |
|------|------|------|
| 네트워크 | Windows와 동일 IP | 별도 가상 네트워크 |
| 외부 접근 | 바로 가능 | **포트포워딩 필요** |
| IP 주소 | 없음 (공유) | 172.x.x.x (동적) |
| 성능 | 낮음 | 높음 (실제 Linux 커널) |

---

# Part 2. 설정 방법

## 사전 준비

### 1. WSL2 배포판 확인

```powershell
wsl -l -v
```

```
  NAME                   STATE           VERSION
* Ubuntu-24.04           Running         2
  Ubuntu-22.04           Stopped         2
```

- `*` = 기본 배포판
- VERSION이 `2`인지 확인

### 2. WSL2 IP 확인

```powershell
# 기본 배포판
wsl hostname -I

# 특정 배포판 지정 (여러 개 설치 시)
wsl -d Ubuntu-24.04 hostname -I
```

---

## 스크립트 파일 구조

```
C:\scripts\
├── wsl2-ports.conf           # 포트 설정 파일
├── wsl2-firewall.ps1         # 방화벽 규칙 (최초 1회)
├── wsl2-firewall-status.ps1  # 방화벽 상태 조회
├── wsl2-port-forward.ps1     # 포트포워딩 (재부팅마다)
├── wsl2-port-status.ps1      # 포트포워딩 상태 조회
└── wsl2-port-clear.ps1       # 포트포워딩 삭제
```

---

## Step 1. 설정 파일 편집

`wsl2-ports.conf` 파일을 열어 필요한 포트를 설정합니다.

```text
# 형식: Windows포트,WSL포트,서비스명,설명

# SSH (외부 4022 -> WSL2 22)
4022,22,ssh,SSH via port 4022

# MariaDB (동일 포트)
3306,3306,MariaDB,MariaDB Database Server

# 주석은 무시됨
#5678,5678,n8n,n8n Workflow
```

**포트 매핑 전략:**

| 상황 | Windows 포트 | WSL2 포트 | 이유 |
|------|-------------|-----------|------|
| SSH | 4022 | 22 | Windows OpenSSH와 충돌 방지 |
| MariaDB | 3306 | 3306 | 동일 포트 사용 |
| 테스트용 | 13306 | 3306 | 다른 포트로 접속 |

---

## Step 2. 방화벽 규칙 추가 (최초 1회)

```powershell
# 관리자 PowerShell에서 실행
.\wsl2-firewall.ps1
```

**출력 예시:**

```
WSL2 방화벽 규칙 추가 (2개)...
  + WSL2-ssh-4022 (TCP 4022 -> WSL:22)
  + WSL2-MariaDB-3306 (TCP 3306 -> WSL:3306)
완료!
```

**옵션:**

```powershell
# 규칙 제거
.\wsl2-firewall.ps1 -Remove

# 다른 설정 파일 사용
.\wsl2-firewall.ps1 -ConfigFile "D:\my-ports.conf"
```

---

## Step 3. 포트포워딩 설정 (재부팅마다)

```powershell
# 관리자 PowerShell에서 실행
.\wsl2-port-forward.ps1
```

**출력 예시:**

```
========================================
WSL2 포트포워딩 설정 시작
========================================
설정 파일 로드: C:\scripts\wsl2-ports.conf
로드된 포트 수: 2개
WSL2 IP: 172.23.0.149
  + ssh (Win:4022 -> WSL:22) -> 172.23.0.149:22
  + MariaDB (Win:3306 -> WSL:3306) -> 172.23.0.149:3306
========================================
설정 완료: 성공 2개, 실패 0개
========================================
```

---

## Step 4. 상태 확인

```powershell
# 포트포워딩 상태
.\wsl2-port-status.ps1

# 방화벽 상태
.\wsl2-firewall-status.ps1
```

---

## Step 5. 불필요한 규칙 삭제

```powershell
# 특정 포트만 삭제
.\wsl2-port-clear.ps1 -Ports 22,3306

# 전체 삭제
.\wsl2-port-clear.ps1 -All
```

---

## 요약: 일상적인 사용 흐름

1. Windows 재부팅
2. 관리자 PowerShell 열기
3. `cd C:\scripts`
4. `.\wsl2-port-forward.ps1`
5. 외부에서 접속 가능 (예: `ssh user@192.168.0.100 -p 4022`)

---

# Part 3. 문제 해결

## Q1. 스크립트 실행이 안 됨

**증상:**

```
이 시스템에서 스크립트를 실행할 수 없습니다.
```

**해결:**

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

---

## Q2. 관리자 권한 오류

**증상:**

```
관리자 권한이 있는 Windows PowerShell에서 실행해야 합니다.
```

**해결:**

- `Win + X` -> Windows PowerShell (관리자)
- 또는 시작 메뉴에서 "PowerShell" 검색 후 "관리자 권한으로 실행"

---

## Q3. WSL2 IP를 가져올 수 없음

**증상:**

```
WSL2 IP를 확인할 수 없습니다.
```

**해결:**

```powershell
# 1. WSL 상태 확인
wsl -l -v

# 2. WSL이 Stopped면 시작
wsl -d Ubuntu-24.04

# 3. 다시 실행
.\wsl2-port-forward.ps1
```

---

## Q4. 포트포워딩 했는데 외부 접속 안 됨

**체크리스트:**

| 확인 항목 | 명령어 |
|-----------|--------|
| 방화벽 규칙 | `.\wsl2-firewall-status.ps1` |
| 포트포워딩 | `.\wsl2-port-status.ps1` |
| WSL2 서비스 | `sudo netstat -tlnp` |

**흔한 원인: 서비스가 localhost에만 바인딩됨**

```bash
# WSL2에서 확인
sudo netstat -tlnp | grep 3306

# 잘못된 경우: 127.0.0.1:3306 (localhost만 접근 가능)
# 올바른 경우: 0.0.0.0:3306 (외부 접근 가능)
```

**MariaDB 설정 변경 예시:**

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
# bind-address = 127.0.0.1 -> bind-address = 0.0.0.0
sudo systemctl restart mariadb
```

---

# Part 4. 레퍼런스

## 포트포워딩 명령어 (netsh)

```powershell
# 조회
netsh interface portproxy show v4tov4

# 추가
netsh interface portproxy add v4tov4 `
    listenport=3306 listenaddress=0.0.0.0 `
    connectport=3306 connectaddress=172.23.0.149

# 삭제
netsh interface portproxy delete v4tov4 `
    listenport=3306 listenaddress=0.0.0.0

# 전체 초기화
netsh interface portproxy reset
```

## 방화벽 명령어 (PowerShell)

```powershell
# 조회
Get-NetFirewallRule -DisplayName "WSL2-*" | Select DisplayName, Enabled

# 추가
New-NetFirewallRule -DisplayName "WSL2-Test-8080" `
    -Direction Inbound -Action Allow `
    -Protocol TCP -LocalPort 8080

# 삭제
Remove-NetFirewallRule -DisplayName "WSL2-Test-8080"
```

## WSL 명령어

```powershell
# 배포판 목록
wsl -l -v

# IP 확인
wsl hostname -I                      # 기본 배포판
wsl -d Ubuntu-24.04 hostname -I      # 특정 배포판

# 기본 배포판 설정
wsl --set-default Ubuntu-24.04

# 시작/종료
wsl -d Ubuntu-24.04                  # 시작
wsl --shutdown                       # 전체 종료
wsl -t Ubuntu-24.04                  # 특정 배포판 종료
```

## 스크립트 파라미터 요약

| 스크립트 | 파라미터 | 설명 |
|----------|----------|------|
| 공통 | `-ConfigFile` | 설정 파일 경로 (기본: `C:\scripts\wsl2-ports.conf`) |
| `wsl2-firewall.ps1` | `-Remove` | 규칙 제거 |
| `wsl2-port-forward.ps1` | `-WslDistro` | WSL 배포판 이름 |
| `wsl2-port-clear.ps1` | `-Ports` | 삭제할 포트 목록 (예: `22,3306`) |
| `wsl2-port-clear.ps1` | `-All` | 전체 삭제 |

---

## 자동 실행 설정 (선택)

재부팅 후 자동으로 포트포워딩을 설정하려면 **작업 스케줄러**를 사용합니다.

1. `Win + R` -> `taskschd.msc`
2. 작업 만들기
3. 트리거: "시작할 때"
4. 동작: `powershell.exe`
5. 인수: `-ExecutionPolicy Bypass -File C:\scripts\wsl2-port-forward.ps1`
6. **"가장 높은 수준의 권한으로 실행" 체크**

---

*이 문서는 WSL2 네트워크 설정 삽질의 결과물입니다.*
