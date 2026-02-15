---
title: "WSL Ubuntu 24.04 SSH/SFTP 서버 구성 가이드"
date: 2026-01-27
categories:
- infra
tags:
- wsl
- ubuntu
- ssh
- sftp
- openssh
keywords:
- WSL
- SSH 서버
- SFTP
- Ubuntu 24.04
- OpenSSH
author: "hyunwoo"
draft: false
---

WSL(Windows Subsystem for Linux) Ubuntu 24.04에서 SSH 및 SFTP 서버를 구성하는 방법을 안내합니다.

<!--more-->

---

## 1. SSH 서버 설치 및 기본 구성

```bash
# OpenSSH 서버 설치
sudo apt update
sudo apt install openssh-server -y

# SSH 서버 상태 확인
sudo service ssh status
```

---

## 2. SSH 설정 파일 수정

```bash
sudo nano /etc/ssh/sshd_config
```

주요 설정 항목:

```
Port 22
ListenAddress 0.0.0.0
PermitRootLogin no
PasswordAuthentication yes
PubkeyAuthentication yes
```

---

## 3. SSH 서버 시작 및 관리

WSL에서는 systemd 대신 service 명령 사용:

```bash
# SSH 서버 시작
sudo service ssh start

# SSH 서버 재시작
sudo service ssh restart

# SSH 서버 중지
sudo service ssh stop
```

---

## 4. SFTP 서버 구성

SSH 설치 시 SFTP는 기본 포함됩니다. `/etc/ssh/sshd_config`에서 확인:

```
Subsystem sftp /usr/lib/openssh/sftp-server
```

### SFTP 전용 사용자 설정 (선택사항)

```bash
# SFTP 전용 그룹 생성
sudo groupadd sftpusers

# SFTP 전용 사용자 생성
sudo useradd -m -G sftpusers -s /usr/sbin/nologin sftpuser
sudo passwd sftpuser

# Chroot 디렉토리 설정
sudo mkdir -p /home/sftpuser/uploads
sudo chown root:root /home/sftpuser
sudo chmod 755 /home/sftpuser
sudo chown sftpuser:sftpusers /home/sftpuser/uploads
```

sshd_config에 추가:

```
Match Group sftpusers
    ChrootDirectory /home/%u
    ForceCommand internal-sftp
    AllowTcpForwarding no
    X11Forwarding no
```

---

## 5. Windows 방화벽 및 포트 포워딩

PowerShell(관리자 권한)에서 실행:

```powershell
# 방화벽 규칙 추가
New-NetFirewallRule -Name "WSL SSH" -DisplayName "WSL SSH" -Direction Inbound -Protocol TCP -LocalPort 22 -Action Allow

# WSL IP 확인 후 포트 포워딩 (외부 접속 필요 시)
netsh interface portproxy add v4tov4 listenport=22 listenaddress=0.0.0.0 connectport=22 connectaddress=(WSL_IP)
```

WSL IP 확인:

```bash
ip addr show eth0 | grep inet
```

---

## 6. 자동 시작 설정

`/etc/wsl.conf` 파일 생성/수정:

```text
[boot]
command="service ssh start"
```

또는 Windows 시작 시 실행되는 스크립트 생성:

```powershell
wsl -u root service ssh start
```

---

## 7. 접속 테스트

```bash
# 로컬 SSH 접속
ssh username@localhost

# SFTP 접속
sftp username@localhost
```

---

## 8. 문제 해결

**SSH 서버 시작 실패 시:**

```bash
# 호스트 키 재생성
sudo ssh-keygen -A

# 로그 확인
sudo cat /var/log/auth.log
```

**WSL 재시작 후 IP 변경 시:**

포트 포워딩 규칙을 업데이트하는 스크립트를 만들어 사용하세요.

---

> **Tip:** 키 기반 인증 설정이나 특정 보안 강화 설정이 필요하면 추가 구성을 진행하세요.
