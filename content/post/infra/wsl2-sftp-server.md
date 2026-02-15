---
title: "WSL2 SFTP 서버 구성 가이드"
date: 2025-01-28
categories:
- infra
tags:
- wsl2
- sftp
- ssh
- openssh
- ubuntu
keywords:
- WSL2
- SFTP
- SSH
- OpenSSH
- 파일 전송
author: "hyunwoo"
draft: false
---

WSL2에서 SFTP 서버를 구성하는 방법을 단계별로 안내합니다. 선행 조건으로 WSL2 포트포워딩 가이드 완료를 권장합니다.

<!--more-->

## 빠른 시작 - 5분 안에 SFTP 서버 구성하기

```bash
# WSL2 Ubuntu에서 실행

# 1. SSH 서버 설치
sudo apt update && sudo apt install -y openssh-server

# 2. SSH 서비스 시작
sudo service ssh start

# 3. 자동 시작 설정 (wsl.conf)
sudo tee -a /etc/wsl.conf << 'EOF'
[boot]
systemd=true
EOF

# 4. WSL 재시작 (Windows PowerShell에서)
# wsl --shutdown

# 5. WSL 재시작 후 systemd로 자동 시작 활성화
sudo systemctl enable ssh
```

```powershell
# Windows에서 연결 테스트
sftp -P 4022 사용자명@localhost
```

> 기존 포트포워딩(4022->22)이 설정되어 있다면 추가 네트워크 설정 불필요

---

## 목차

| Part | 내용 | 대상 |
|------|------|------|
| Part 1 | SFTP란? SSH와의 관계 | 처음 접하는 분 |
| Part 2 | OpenSSH 설치 및 구성 | 바로 적용하고 싶은 분 |
| Part 3 | Windows 재시작 시 자동 시작 | 재부팅 후에도 사용하고 싶은 분 |
| Part 4 | 보안 강화 (chroot, 전용 사용자) | 보안이 중요한 분 |
| Part 5 | 문제 해결 | 문제가 생겼을 때 |
| Part 6 | 명령어 모음 | 빠른 참조 |

---

# Part 1. SFTP란?

## 1.1 SFTP vs FTP vs SCP 비교

| 항목 | SFTP | FTP | SCP |
|------|------|-----|-----|
| **보안** | SSH 암호화 | 평문 전송 | SSH 암호화 |
| **포트** | 22 (SSH와 동일) | 21 + 데이터 포트 | 22 |
| **방화벽** | 단일 포트 | 복잡 (패시브/액티브) | 단일 포트 |
| **기능** | 디렉토리 탐색, 이어받기 | 풀 기능 | 복사만 |
| **권장** | 권장 | 비권장 | 단순 복사용 |

## 1.2 SFTP는 SSH의 서브시스템

SFTP는 독립된 프로토콜이 아니라 **SSH의 서브시스템**으로 동작합니다:

- SSH 서버(sshd)가 실행 중이면 SFTP도 사용 가능
- 별도의 FTP 서버 설치 불필요
- SSH와 동일한 인증 방식 사용 (비밀번호, SSH 키)

## 1.3 기존 포트포워딩과의 관계

이미 SSH 포트포워딩이 설정되어 있다면:

- 외부 PC -> Windows 4022 -> WSL2 22 (SSH/SFTP)
- 접속 주소: `sftp://user@192.168.0.xxx:4022`

> `wsl2-ports.conf`에 `4022,22,ssh` 설정이 있다면 추가 포트포워딩 불필요

---

# Part 2. OpenSSH 설치 및 구성

## 2.1 사전 확인

```bash
# WSL2 배포판 확인
cat /etc/os-release | grep PRETTY_NAME
# 출력 예: PRETTY_NAME="Ubuntu 24.04 LTS"

# SSH 서버 설치 여부 확인
which sshd
# 설치됨: /usr/sbin/sshd
# 미설치: 출력 없음
```

## 2.2 OpenSSH 서버 설치

```bash
# 패키지 업데이트 및 설치
sudo apt update
sudo apt install -y openssh-server
```

## 2.3 SSH 서비스 시작

```bash
# 서비스 시작
sudo service ssh start

# 상태 확인
sudo service ssh status
# 출력: * sshd is running
```

## 2.4 SFTP 서브시스템 확인

기본적으로 Ubuntu의 OpenSSH는 SFTP가 활성화되어 있습니다:

```bash
# SFTP 설정 확인
grep -i "^Subsystem.*sftp" /etc/ssh/sshd_config
# 출력: Subsystem sftp /usr/lib/openssh/sftp-server
```

## 2.5 주요 sshd_config 설정

| 설정 | 기본값 | 설명 |
|------|--------|------|
| `Port` | 22 | SSH 포트 (변경 비권장) |
| `PermitRootLogin` | prohibit-password | root 로그인 정책 |
| `PasswordAuthentication` | yes | 비밀번호 인증 허용 |
| `PubkeyAuthentication` | yes | SSH 키 인증 허용 |
| `Subsystem sftp` | /usr/lib/openssh/sftp-server | SFTP 서브시스템 |

## 2.6 연결 테스트

### 명령줄에서 테스트

```bash
# WSL2 내부에서 로컬 테스트
sftp localhost
# 비밀번호 입력 후 sftp> 프롬프트 표시되면 성공

# Windows에서 테스트 (포트포워딩 설정된 경우)
sftp -P 4022 사용자명@localhost
```

### FileZilla/WinSCP 설정

| 항목 | 값 |
|------|-----|
| 프로토콜 | SFTP |
| 호스트 | localhost (또는 Windows IP) |
| 포트 | 4022 (포트포워딩 설정 기준) |
| 사용자 | WSL2 사용자명 |
| 비밀번호 | WSL2 사용자 비밀번호 |

---

# Part 3. 자동 시작 설정

Windows 재시작 또는 WSL 종료 후에도 SSH 서비스가 자동으로 시작되도록 설정합니다.

## 3.1 방법 비교

| 방법 | 장점 | 단점 | 권장 |
|------|------|------|------|
| **systemd 활성화** | 표준 Linux 방식, 안정적 | WSL 0.67.6+ 필요 | 권장 |
| wsl.conf [boot] command | 간단한 설정 | 단일 명령어만 가능 | 보통 |
| Windows Task Scheduler | WSL 버전 무관 | Windows 설정 필요 | 보통 |

## 3.2 방법 1: systemd 활성화 (권장)

WSL2 버전 0.67.6 이상에서는 systemd를 활성화할 수 있습니다.

### WSL 버전 확인

```powershell
# Windows PowerShell에서
wsl --version
# 출력 예: WSL 버전: 2.0.9.0
```

### systemd 활성화

```bash
# /etc/wsl.conf 편집
sudo nano /etc/wsl.conf
```

다음 내용 추가:

```text
[boot]
systemd=true
```

### WSL 재시작

```powershell
# Windows PowerShell에서 (관리자 권한 불필요)
wsl --shutdown
```

WSL 터미널을 다시 열면 systemd가 활성화됩니다.

### systemd 활성화 확인

```bash
# PID 1 프로세스 확인
ps --no-headers -o comm 1
# 출력: systemd (활성화됨)
# 출력: init (비활성화)
```

### SSH 자동 시작 설정

```bash
# SSH 서비스 자동 시작 활성화
sudo systemctl enable ssh

# 상태 확인
sudo systemctl status ssh
```

> 이제 Windows 재시작 후에도 WSL을 시작하면 SSH가 자동으로 실행됩니다.

## 3.3 방법 2: wsl.conf [boot] command

systemd를 사용하지 않는 경우의 대안:

```bash
sudo nano /etc/wsl.conf
```

```text
[boot]
command = service ssh start
```

> `command`는 단일 명령어만 지원합니다. 여러 서비스가 필요하면 스크립트 파일을 만들어 실행하세요.

## 3.4 방법 3: Windows Task Scheduler

WSL 버전이 낮거나 wsl.conf가 동작하지 않는 경우:

### 작업 스케줄러 설정

1. `Win + R` -> `taskschd.msc` 실행
2. **작업 만들기** 클릭

#### 일반 탭

- 이름: `WSL SSH Start`
- 보안 옵션: "사용자가 로그온할 때만 실행"

#### 트리거 탭

- 새로 만들기 -> **로그온할 때**

#### 동작 탭

- 새로 만들기
- 프로그램/스크립트: `wsl`
- 인수 추가: `-d Ubuntu-24.04 -u root service ssh start`

### PowerShell로 작업 생성

```powershell
# 관리자 PowerShell에서 실행
$action = New-ScheduledTaskAction -Execute "wsl" `
    -Argument "-d Ubuntu-24.04 -u root service ssh start"
$trigger = New-ScheduledTaskTrigger -AtLogOn
$settings = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries

Register-ScheduledTask -TaskName "WSL SSH Start" `
    -Action $action -Trigger $trigger -Settings $settings
```

---

# Part 4. 보안 강화 (선택)

일반적인 개발 용도로는 기본 설정으로 충분합니다. 외부 접속이 필요하거나 보안이 중요한 경우 추가 설정을 고려합니다.

## 4.1 SFTP 전용 사용자 생성

SSH 쉘 접근 없이 SFTP만 허용하는 사용자:

```bash
# 사용자 생성 (쉘 없음)
sudo adduser sftpuser --shell /usr/sbin/nologin

# SFTP 전용 그룹 생성
sudo groupadd sftponly

# 사용자를 그룹에 추가
sudo usermod -aG sftponly sftpuser
```

## 4.2 Chroot Jail 설정

사용자를 특정 디렉토리에 격리:

```bash
sudo nano /etc/ssh/sshd_config
```

파일 끝에 추가:

```
# SFTP 전용 사용자 설정
Match Group sftponly
    ChrootDirectory /home/%u
    ForceCommand internal-sftp
    AllowTcpForwarding no
    X11Forwarding no
```

### 디렉토리 권한 설정 (중요)

Chroot 디렉토리는 **root 소유**여야 합니다:

```bash
# 홈 디렉토리 소유권 변경
sudo chown root:root /home/sftpuser
sudo chmod 755 /home/sftpuser

# 쓰기 가능한 하위 디렉토리 생성
sudo mkdir /home/sftpuser/uploads
sudo chown sftpuser:sftpuser /home/sftpuser/uploads
```

> Chroot 디렉토리 권한이 잘못되면 "broken pipe" 또는 연결 실패 발생

## 4.3 SSH 키 인증 설정

비밀번호 대신 SSH 키 사용:

### Windows에서 키 생성

```powershell
# Windows PowerShell에서
ssh-keygen -t ed25519 -C "sftp-key"
# 기본 경로: C:\Users\사용자\.ssh\id_ed25519
```

### 공개키를 WSL2로 복사

```powershell
# Windows에서 (포트포워딩 설정 기준)
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh -p 4022 사용자명@localhost "cat >> ~/.ssh/authorized_keys"
```

---

# Part 5. 문제 해결

## Q1. SSH 서비스가 시작되지 않음

### 증상

```
System has not been booted with systemd as init system (PID 1).
```

### 원인

systemd가 활성화되지 않은 상태에서 `systemctl` 명령 사용

### 해결

```bash
# 방법 1: service 명령 사용
sudo service ssh start

# 방법 2: systemd 활성화 (Part 3.2 참조)
```

---

## Q2. SFTP 연결 거부

### 체크리스트

| 확인 항목 | 명령어 | 정상 출력 |
|-----------|--------|-----------|
| SSH 서비스 상태 | `sudo service ssh status` | sshd is running |
| 포트 리스닝 | `ss -tlnp | grep 22` | LISTEN |
| 포트포워딩 | `wsl2-port-status.ps1` (Windows) | 4022 -> 172.x.x.x:22 |
| 방화벽 | `wsl2-firewall-status.ps1` (Windows) | 4022 허용됨 |

---

## Q3. Permission denied

### 원인 1: 비밀번호 오류

```bash
# WSL2에서 비밀번호 재설정
sudo passwd 사용자명
```

### 원인 2: Chroot 디렉토리 권한 문제

```bash
# 디렉토리 소유권 확인
ls -la /home/sftpuser
# 출력: drwxr-xr-x root root /home/sftpuser (올바름)
# 출력: drwxr-xr-x sftpuser sftpuser (잘못됨)

# 수정
sudo chown root:root /home/sftpuser
```

### 원인 3: SSH 키 권한 문제

```bash
# 권한 확인 및 수정
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_*
```

---

## Q4. 연결이 갑자기 끊김

### 원인

- WSL2 IP 변경 (재부팅 시)
- WSL이 유휴 상태로 종료됨

### 해결

```powershell
# Windows에서 WSL 상태 확인
wsl --list --running

# 포트포워딩 재설정
.\wsl2-port-forward.ps1
```

---

## Q5. "broken pipe" 에러

### 원인

Chroot 설정의 디렉토리 권한 문제

### 해결

```bash
# Chroot 디렉토리가 root 소유인지 확인
ls -la /home/ | grep sftpuser

# root:root로 변경
sudo chown root:root /home/sftpuser
sudo chmod 755 /home/sftpuser
```

---

# Part 6. 레퍼런스

## SSH 서비스 관리

```bash
# service 명령 (systemd 비활성화 시)
sudo service ssh start
sudo service ssh stop
sudo service ssh restart
sudo service ssh status

# systemctl 명령 (systemd 활성화 시)
sudo systemctl start ssh
sudo systemctl stop ssh
sudo systemctl restart ssh
sudo systemctl status ssh
sudo systemctl enable ssh    # 자동 시작 활성화
sudo systemctl disable ssh   # 자동 시작 비활성화
```

## SFTP 클라이언트 명령어

```bash
# 접속
sftp -P 4022 user@host

# 기본 명령어
sftp> pwd              # 원격 현재 디렉토리
sftp> lpwd             # 로컬 현재 디렉토리
sftp> ls               # 원격 파일 목록
sftp> lls              # 로컬 파일 목록
sftp> cd dir           # 원격 디렉토리 이동
sftp> lcd dir          # 로컬 디렉토리 이동
sftp> get file         # 다운로드
sftp> put file         # 업로드
sftp> get -r dir       # 디렉토리 다운로드
sftp> put -r dir       # 디렉토리 업로드
sftp> mkdir dir        # 디렉토리 생성
sftp> rm file          # 파일 삭제
sftp> rmdir dir        # 디렉토리 삭제
sftp> exit             # 종료
```

## 설정 파일 경로

| 파일 | 용도 |
|------|------|
| `/etc/ssh/sshd_config` | SSH 서버 설정 |
| `/etc/wsl.conf` | WSL 설정 (자동 시작) |
| `~/.ssh/authorized_keys` | SSH 공개키 |
| `~/.ssh/config` | SSH 클라이언트 설정 |
| `/var/log/auth.log` | 인증 로그 |

---

*이 문서는 WSL2 포트포워딩 시스템의 일부로 작성되었습니다.*
