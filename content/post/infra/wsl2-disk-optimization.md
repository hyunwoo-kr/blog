---
title: "WSL2 서버 운영을 위한 디스크 제한 및 최적화 가이드"
date: 2026-02-01
categories:
- infra
tags:
- wsl2
- windows
- disk-optimization
- vhdx
- server
keywords:
- WSL2
- 디스크 최적화
- VHDX 압축
- wslconfig
author: "hyunwoo"
draft: false
---

WSL2를 개발 서버로 장기 운영하다 보면 VHDX 파일이 계속 커지는 문제를 경험하게 됩니다. 파일을 삭제해도 가상 디스크 크기는 줄어들지 않고, 어느새 수십 GB를 차지하고 있죠.

이 글에서는 WSL2를 서버 용도로 안정적으로 운영하기 위한 핵심 설정들을 정리합니다.

<!--more-->

---

## 1. 디스크 사용량 제한 (.wslconfig)

Windows 사용자 폴더에 `.wslconfig` 파일을 생성합니다.

**경로**: `%USERPROFILE%\.wslconfig` (예: `C:\Users\사용자명\.wslconfig`)

```text
[wsl2]
# 메모리 제한 (호스트 RAM의 50% 권장)
memory=16GB

# CPU 코어 제한
processors=4

# 스왑 파일 크기
swap=8GB

# 스왑 파일 위치 (SSD 권장)
swapFile=C:\\wsl-swap.vhdx

[experimental]
# 자동 메모리 회수 - 서버 운영 시 필수!
autoMemoryReclaim=gradual

# 스파스 VHD - 미사용 공간 자동 반환
sparseVhd=true
```

> **핵심 포인트**: `sparseVhd=true`와 `autoMemoryReclaim=gradual` 설정이 장기 운영의 핵심입니다. 이 설정 없이는 VHDX가 계속 커지기만 합니다.

---

## 2. 기존 VHDX 압축하기

이미 커져버린 VHDX 파일은 수동으로 압축해야 합니다.

### PowerShell에서 실행

```powershell
# 1단계: WSL 완전 종료
wsl --shutdown

# 2단계: VHDX 위치 확인
Get-ChildItem -Path "$env:LOCALAPPDATA\Packages" -Recurse -Filter "ext4.vhdx" | Select-Object FullName

# 3단계: diskpart로 압축
diskpart
```

### diskpart 명령어

```
select vdisk file="C:\Users\사용자명\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu..\LocalState\ext4.vhdx"
attach vdisk readonly
compact vdisk
detach vdisk
exit
```

> **주의**: 압축 전 반드시 `wsl --shutdown`으로 완전히 종료하세요. 실행 중 압축하면 데이터가 손상될 수 있습니다.

---

## 3. WSL 내부 설정 (wsl.conf)

WSL 배포판 내부의 `/etc/wsl.conf` 파일을 설정합니다.

```text
[boot]
# systemd 활성화 - 서비스 관리 필수
systemd=true

[automount]
enabled=true
options="metadata,umask=22,fmask=11"

[network]
generateResolvConf=true
hostname=wsl-server

[interop]
enabled=true
# Windows PATH 오염 방지 (서버 용도 시 권장)
appendWindowsPath=false
```

> **appendWindowsPath=false**: 서버 운영 시 Windows 경로가 섞이면 예상치 못한 문제가 발생할 수 있습니다. 필요한 Windows 명령어는 절대 경로로 호출하세요.

---

## 4. 네트워크 설정 (외부 접속용)

### .wslconfig에 추가

```text
[wsl2]
# 미러 모드 - 호스트와 동일한 IP 사용
networkingMode=mirrored

# localhost 포워딩
localhostForwarding=true
```

### 포트 포워딩 (기존 NAT 모드 사용 시)

```powershell
# WSL IP 확인
$wslIP = (wsl hostname -I).Trim().Split(" ")[0]

# 포트 포워딩 설정 (예: SSH 22번 포트)
netsh interface portproxy add v4tov4 listenport=22 listenaddress=0.0.0.0 connectport=22 connectaddress=$wslIP

# 방화벽 규칙 추가
New-NetFirewallRule -DisplayName "WSL SSH" -Direction Inbound -LocalPort 22 -Protocol TCP -Action Allow
```

---

## 5. 자동 시작 설정

Windows 부팅 시 WSL 서비스가 자동으로 시작되도록 설정합니다.

### 시작 스크립트 생성

**파일**: `C:\Scripts\start-wsl-services.ps1`

```powershell
# WSL 서비스 자동 시작 스크립트
$logFile = "C:\Scripts\wsl-startup.log"
$timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"

Add-Content $logFile "[$timestamp] WSL 서비스 시작 중..."

# SSH 서비스 시작
wsl -d Ubuntu -u root -- service ssh start

# Docker 서비스 시작 (설치된 경우)
wsl -d Ubuntu -u root -- service docker start

# 필요한 서비스 추가
# wsl -d Ubuntu -u root -- service nginx start
# wsl -d Ubuntu -u root -- service mysql start

Add-Content $logFile "[$timestamp] WSL 서비스 시작 완료"
```

### 작업 스케줄러 등록

```powershell
$action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-WindowStyle Hidden -ExecutionPolicy Bypass -File C:\Scripts\start-wsl-services.ps1"
$trigger = New-ScheduledTaskTrigger -AtStartup -RandomDelay (New-TimeSpan -Seconds 30)
$settings = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries
$principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" -RunLevel Highest

Register-ScheduledTask -TaskName "WSL2 Server Startup" -Action $action -Trigger $trigger -Settings $settings -Principal $principal
```

---

## 6. 보안 설정

서버로 운영한다면 기본적인 보안 설정은 필수입니다.

### UFW 방화벽

```bash
sudo apt update && sudo apt install -y ufw

# 기본 정책
sudo ufw default deny incoming
sudo ufw default allow outgoing

# 필요한 포트만 허용
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
sudo ufw allow 3000/tcp  # 개발 서버

# 방화벽 활성화
sudo ufw enable
sudo ufw status verbose
```

### Fail2ban (SSH 무차별 대입 방지)

```bash
sudo apt install -y fail2ban

# 설정 파일 생성
sudo tee /etc/fail2ban/jail.local << 'EOF'
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
findtime = 600
EOF

sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

---

## 7. 모니터링 설정

서버 상태를 주기적으로 확인할 수 있는 도구를 설치합니다.

```bash
# htop - 실시간 프로세스 모니터
sudo apt install -y htop

# ncdu - 디스크 사용량 분석
sudo apt install -y ncdu

# 간단한 상태 확인 alias 추가
echo 'alias sysinfo="echo \"=== Memory ===\" && free -h && echo \"\n=== Disk ===\" && df -h / && echo \"\n=== Load ===\" && uptime"' >> ~/.bashrc
```

---

## 최종 체크리스트

| 항목 | 설정 파일 | 필수 여부 |
|------|-----------|-----------|
| 메모리 제한 | .wslconfig | 필수 |
| 스파스 VHD | .wslconfig | 필수 |
| 메모리 자동 회수 | .wslconfig | 필수 |
| systemd 활성화 | wsl.conf | 필수 |
| Windows PATH 분리 | wsl.conf | 권장 |
| 자동 시작 스크립트 | 작업 스케줄러 | 권장 |
| UFW 방화벽 | Ubuntu 내부 | 권장 |
| Fail2ban | Ubuntu 내부 | 선택 |

---

## 설정 적용

모든 설정을 마친 후 WSL을 재시작합니다.

```powershell
# PowerShell에서
wsl --shutdown
wsl
```

---

## 마치며

WSL2는 편리하지만 서버로 장기 운영하려면 세심한 설정이 필요합니다. 특히 **디스크 관리**와 **메모리 회수** 설정을 빠뜨리면 나중에 디스크 용량 부족으로 고생하게 됩니다.

이 가이드가 WSL2로 개발 서버를 구축하시는 분들께 도움이 되길 바랍니다.

> **관련 문서**
> - WSL2 Docker Engine 설치 가이드
> - WSL2 포트포워딩 & 방화벽 가이드
> - Tailscale 기반 원격 개발환경 구축 가이드
