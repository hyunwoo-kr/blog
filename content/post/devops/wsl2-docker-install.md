---
title: "WSL2 Docker Engine 설치 가이드"
date: 2025-01-28
categories:
- devops
tags:
- wsl2
- docker
- docker-engine
- ubuntu
- container
keywords:
- WSL2
- Docker Engine
- Docker Desktop 대안
- Ubuntu 24.04
- 컨테이너
author: "hyunwoo"
draft: false
---

Docker Desktop 없이 WSL2에서 Docker Engine을 직접 설치하면 메모리 1~2GB를 절약할 수 있습니다. Ubuntu 24.04 + systemd 설정으로 Windows 재부팅 후에도 자동 시작되며, 기존 Docker Desktop 이미지/볼륨도 100% 이관 가능합니다.

<!--more-->

## 이 가이드가 필요한 이유

> **왜 Docker Desktop 대신 Docker Engine인가요?**
>
> Docker Desktop은 GUI가 편리하지만:
> - 유휴 상태에서도 **2~3GB 메모리** 점유
> - 기업 사용 시 **유료 라이선스** 필요 (직원 250명 이상 또는 연 매출 $10M 이상)
> - WSL2 통합 시 추가 오버헤드 발생
>
> 반면 Docker Engine 직접 설치 시:
> - **500MB~1GB**로 메모리 절반 이하 사용
> - **완전 무료** (오픈소스)
> - 동일한 Docker 명령어 100% 호환

---

## 아키텍처 이해하기

```
┌─────────────────────────────────────────────────────────┐
│ Windows 11                                              │
│  ┌───────────────────────────────────────────────────┐  │
│  │ WSL2 (Ubuntu 24.04)                               │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │ Docker Engine                               │  │  │
│  │  │  ├── dockerd (데몬)                         │  │  │
│  │  │  ├── containerd                             │  │  │
│  │  │  └── runc                                   │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  │                                                   │  │
│  │  /mnt/c/ <-> C:\  (Windows 파일시스템 마운트)     │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

> **신입 개발자를 위한 설명:**
> - **WSL2**: Windows 안에서 돌아가는 진짜 Linux (가상머신 기반)
> - **dockerd**: Docker 명령어를 받아서 처리하는 백그라운드 서비스
> - **containerd**: 실제 컨테이너 생명주기를 관리
> - **runc**: 컨테이너를 실제로 실행하는 저수준 도구
> - **/mnt/c/**: WSL2에서 C드라이브에 접근하는 경로

---

## 1단계: 사전 요구사항 확인

### 시스템 요구사항

| 항목 | 요구사항 | 확인 방법 |
|------|---------|-----------|
| OS | Windows 10 (2004 이상) 또는 Windows 11 | `winver` 실행 |
| RAM | 최소 8GB (권장 16GB) | 작업관리자 -> 성능 |
| CPU | 가상화 지원 (VT-x/AMD-V) | BIOS에서 활성화 필요 |
| 저장소 | 최소 20GB 여유 공간 | 파일 탐색기 |

### Windows 기능 확인

PowerShell **(관리자 권한)** 에서 실행:

```powershell
# 가상화 지원 확인
systeminfo | findstr /i "Hyper-V"

# WSL 버전 확인
wsl --version
```

### 필수 Windows 기능 활성화

```powershell
# WSL 및 가상 머신 플랫폼 활성화
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# 재부팅 필요
Restart-Computer
```

> **주의:** 위 명령어 실행 후 반드시 재부팅해야 합니다!

---

## 2단계: WSL2 환경 설정

### WSL2 설치 및 설정

PowerShell에서 실행:

```powershell
# WSL 업데이트
wsl --update

# WSL2를 기본 버전으로 설정
wsl --set-default-version 2

# 사용 가능한 배포판 목록
wsl --list --online
```

### Ubuntu 24.04 설치

```powershell
# Ubuntu 24.04 설치
wsl --install -d Ubuntu-24.04

# 설치 확인
wsl -l -v
```

출력 예시:

```
  NAME            STATE           VERSION
* Ubuntu-24.04    Running         2
```

> **VERSION이 2인지 확인하세요!** 1이면 WSL1이라 Docker가 제대로 동작하지 않습니다.

### 기존 WSL1을 WSL2로 변환 (해당되는 경우)

```powershell
# 현재 버전 확인
wsl -l -v

# WSL2로 변환 (시간 소요)
wsl --set-version Ubuntu-24.04 2

# 변환 완료 확인
wsl -l -v
```

### WSL2 리소스 제한 설정 (메모리 관리)

Windows의 `%UserProfile%\.wslconfig` 파일 생성:

```powershell
# PowerShell에서 실행
notepad "$env:USERPROFILE\.wslconfig"
```

파일 내용 (복사해서 붙여넣기):

```text
[wsl2]
memory=4GB
swap=2GB
processors=2
localhostForwarding=true

[experimental]
autoMemoryReclaim=gradual
sparseVhd=true
```

설정 적용:

```powershell
wsl --shutdown
wsl -d Ubuntu-24.04
```

> **autoMemoryReclaim=gradual**: WSL2가 사용하지 않는 메모리를 점진적으로 Windows에 반환합니다. 메모리 부족 문제 해결에 효과적!

---

## 3단계: Docker Engine 설치

> **여기서부터는 WSL2 Ubuntu 터미널에서 실행합니다!**
> PowerShell에서 `wsl` 또는 `wsl -d Ubuntu-24.04` 입력하여 Ubuntu로 진입하세요.

### 기존 패키지 제거

```bash
# 기존 Docker 관련 패키지 제거
sudo apt remove -y docker docker-engine docker.io containerd runc 2>/dev/null || true

# 잔여 파일 정리
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

### 필수 패키지 설치

```bash
# 패키지 업데이트
sudo apt update

# 필수 패키지 설치
sudo apt install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

### Docker 공식 저장소 추가

```bash
# GPG 키 디렉토리 생성
sudo install -m 0755 -d /etc/apt/keyrings

# Docker 공식 GPG 키 추가
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# 저장소 추가
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
    https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Docker Engine 설치

```bash
# 패키지 인덱스 업데이트
sudo apt update

# Docker 패키지 설치
sudo apt install -y \
    docker-ce \
    docker-ce-cli \
    containerd.io \
    docker-buildx-plugin \
    docker-compose-plugin

# 버전 확인
docker --version
docker compose version
```

---

## 4단계: 설치 후 설정

### systemd 활성화 (WSL2)

```bash
# /etc/wsl.conf 설정
sudo tee /etc/wsl.conf <<EOF
[boot]
systemd=true

[interop]
enabled=true
appendWindowsPath=true

[automount]
enabled=true
options="metadata,umask=22,fmask=11"
EOF
```

**PowerShell에서 WSL 재시작:**

```powershell
wsl --shutdown
wsl -d Ubuntu-24.04
```

### /etc/wsl.conf 설정 상세 설명

| 섹션 | 설정 | 의미 |
|------|------|------|
| [boot] | systemd=true | WSL2 시작 시 systemd를 PID 1로 실행. `systemctl` 명령어로 서비스 관리 가능 |
| [interop] | enabled=true | WSL에서 Windows 프로그램 직접 실행 허용 (예: `notepad.exe`, `code .`) |
| [interop] | appendWindowsPath=true | Windows PATH를 WSL PATH에 추가. `.exe` 경로 지정 불필요 |
| [automount] | enabled=true | Windows 드라이브를 `/mnt/c`, `/mnt/d` 등에 자동 마운트 |
| [automount] | metadata | Linux 파일 권한(chmod)을 Windows 파일에 저장. git 권한 문제 해결 |
| [automount] | umask=22 | 디렉토리 기본 권한 755 (rwxr-xr-x) |
| [automount] | fmask=11 | 파일 기본 권한 666 (rw-rw-rw-) |

> **왜 이 설정들이 중요한가요?**
>
> **systemd 없으면:**
> - `systemctl` 명령어 사용 불가
> - Docker를 매번 수동 시작해야 함 (`sudo dockerd &`)
>
> **metadata 없으면:**
> - `chmod +x script.sh` 실행 권한이 재부팅 시 사라짐
> - git에서 파일 권한 변경이 계속 diff로 잡힘
>
> **interop 없으면:**
> - WSL에서 `code .`로 VS Code 열기 불가
> - `explorer.exe .`로 폴더 열기 불가

**appendWindowsPath=true 실전 예시:**

```bash
# 현재 폴더를 Windows 탐색기로 열기
explorer.exe .

# VS Code로 현재 프로젝트 열기
code .

# 텍스트를 Windows 클립보드에 복사
echo "복사할 내용" | clip.exe
cat ~/.ssh/id_rsa.pub | clip.exe   # SSH 공개키 복사

# Windows 기본 브라우저로 URL 열기
cmd.exe /c start http://localhost:8080

# Windows 메모장으로 파일 편집
notepad.exe config.txt
```

이 설정이 없으면 `/mnt/c/Windows/System32/notepad.exe`처럼 전체 경로를 입력해야 합니다.

### Docker 서비스 시작

```bash
# Docker 서비스 활성화 및 시작
sudo systemctl enable docker
sudo systemctl start docker

# 상태 확인
sudo systemctl status docker
```

### 사용자 권한 설정 (sudo 없이 사용)

```bash
# docker 그룹에 현재 사용자 추가
sudo usermod -aG docker $USER

# 변경사항 적용 (재로그인 또는)
newgrp docker

# 권한 확인
docker ps
```

> **newgrp docker** 후에도 permission denied가 나오면, WSL을 완전히 종료했다가 다시 시작하세요: `wsl --shutdown`

### Docker 데몬 설정 (선택사항)

```bash
# /etc/docker/daemon.json 생성
sudo tee /etc/docker/daemon.json <<EOF
{
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "10m",
        "max-file": "3"
    },
    "storage-driver": "overlay2",
    "dns": ["8.8.8.8", "8.8.4.4"]
}
EOF

# Docker 재시작
sudo systemctl restart docker
```

---

## 5단계: Docker Desktop에서 이관

> **이 섹션은 기존 Docker Desktop 사용자만 해당됩니다.**
> 신규 설치라면 다음 "검증 및 테스트" 섹션으로 건너뛰세요.

### 이관 전 체크리스트

- [ ] 실행 중인 컨테이너 목록 확인
- [ ] 이미지 목록 확인
- [ ] 볼륨 데이터 백업
- [ ] docker-compose.yml 파일 백업
- [ ] 환경변수 및 설정 기록

### Docker Desktop에서 이미지 백업

Docker Desktop이 실행 중인 환경에서:

```bash
# 이미지 목록 저장
docker images --format "{{.Repository}}:{{.Tag}}" | grep -v "<none>" > image_list.txt

# 개별 이미지 저장
docker save -o image_backup.tar <image_name>:<tag>

# 모든 이미지 한번에 저장 (시간 소요)
docker images --format "{{.Repository}}:{{.Tag}}" | grep -v "<none>" | \
    xargs docker save -o all_images.tar
```

### 볼륨 데이터 백업

```bash
# 볼륨 목록 확인
docker volume ls

# 볼륨 데이터 백업 (각 볼륨별)
docker run --rm \
    -v <volume_name>:/source:ro \
    -v /mnt/c/docker_backup:/backup \
    alpine tar cvf /backup/<volume_name>.tar -C /source .
```

### WSL2 Docker Engine에서 복원

```bash
# 이미지 로드
docker load -i /mnt/c/docker_backup/image_backup.tar

# 또는 모든 이미지
docker load -i /mnt/c/docker_backup/all_images.tar

# 이미지 확인
docker images
```

### 볼륨 데이터 복원

```bash
# 볼륨 생성
docker volume create <volume_name>

# 데이터 복원
docker run --rm \
    -v <volume_name>:/target \
    -v /mnt/c/docker_backup:/backup:ro \
    alpine sh -c "cd /target && tar xvf /backup/<volume_name>.tar"

# 확인
docker volume inspect <volume_name>
```

### Docker Desktop 제거 (선택)

이관 완료 및 검증 후:

```powershell
# Windows 설정 -> 앱 -> Docker Desktop 제거
# 또는
winget uninstall Docker.DockerDesktop

# 잔여 파일 정리 (선택)
Remove-Item -Recurse -Force "$env:APPDATA\Docker"
Remove-Item -Recurse -Force "$env:LOCALAPPDATA\Docker"
```

---

## 6단계: 검증 및 테스트

### 기본 동작 테스트

```bash
# Docker 정보 확인
docker info

# 테스트 컨테이너 실행
docker run --rm hello-world

# 인터랙티브 테스트
docker run -it --rm alpine sh -c "echo 'Docker is working!'"
```

### 네트워크 테스트

```bash
# 포트 바인딩 테스트
docker run -d --name nginx-test -p 8080:80 nginx

# Windows 브라우저에서 http://localhost:8080 접속 확인

# 정리
docker rm -f nginx-test
```

### 볼륨 마운트 테스트

```bash
# Windows 경로 마운트 테스트
docker run --rm -v /mnt/c/Users:/host alpine ls /host

# Linux 경로 마운트 테스트
docker run --rm -v /tmp/test:/data alpine sh -c "echo 'test' > /data/test.txt && cat /data/test.txt"
```

### Docker Compose 테스트

```bash
# 테스트용 compose 파일 생성
cat > /tmp/docker-compose-test.yml <<EOF
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
  db:
    image: alpine
    command: sleep infinity
EOF

# 실행
docker compose -f /tmp/docker-compose-test.yml up -d

# 확인
docker compose -f /tmp/docker-compose-test.yml ps

# 정리
docker compose -f /tmp/docker-compose-test.yml down
```

---

## 자주 하는 실수 & 해결법

### 실수 1: Docker 서비스가 시작되지 않음

```bash
# 로그 확인
sudo journalctl -u docker.service -n 50

# containerd 상태 확인
sudo systemctl status containerd

# 수동 시작 시도
sudo dockerd --debug
```

### 실수 2: permission denied 오류

```bash
# docker 그룹 확인
groups $USER

# 그룹에 없으면 추가
sudo usermod -aG docker $USER

# 소켓 권한 확인
ls -la /var/run/docker.sock

# 임시 해결 (권장하지 않음)
sudo chmod 666 /var/run/docker.sock
```

### 실수 3: 네트워크 문제 (이미지 pull 실패)

```bash
# DNS 문제 시 daemon.json에 DNS 추가
sudo tee /etc/docker/daemon.json <<EOF
{
    "dns": ["8.8.8.8", "8.8.4.4"]
}
EOF
sudo systemctl restart docker
```

### 실수 4: WSL2 메모리 부족

PowerShell에서:

```powershell
# 현재 메모리 사용량 확인
wsl -- free -h

# .wslconfig 수정 후
wsl --shutdown
```

### 실수 5: systemd가 동작하지 않음

```bash
# systemd가 PID 1인지 확인
ps -p 1 -o comm=

# 출력이 'init'이면 systemd가 아님
# /etc/wsl.conf 재설정 후 WSL 재시작
```

---

## 유용한 명령어 모음

### Docker 명령어

```bash
# 컨테이너 관리
docker ps -a                    # 모든 컨테이너 목록
docker logs -f <container>      # 로그 실시간 확인
docker exec -it <container> sh  # 컨테이너 접속
docker stats                    # 리소스 사용량 모니터링

# 이미지 관리
docker images                   # 이미지 목록
docker image prune -a           # 미사용 이미지 삭제
docker system df                # 디스크 사용량 확인

# 정리
docker system prune -a          # 미사용 리소스 전체 정리
```

### WSL2 명령어 (PowerShell)

```powershell
wsl -l -v                       # 배포판 목록 및 버전
wsl --shutdown                  # WSL 종료
wsl -d Ubuntu-24.04             # 특정 배포판 시작
wsl --export Ubuntu-24.04 backup.tar  # 백업
wsl --import Ubuntu-restore C:\WSL backup.tar  # 복원
```

### 추천 bash alias

```bash
# ~/.bashrc에 추가
cat >> ~/.bashrc <<'EOF'

# Docker aliases
alias d='docker'
alias dc='docker compose'
alias dps='docker ps'
alias dpsa='docker ps -a'
alias di='docker images'
alias dex='docker exec -it'
alias dlog='docker logs -f'
alias dprune='docker system prune -af'
EOF

source ~/.bashrc
```

---

## 빠른 설치 스크립트 (원클릭)

아래 스크립트를 저장하여 한번에 설치:

```bash
#!/bin/bash
# install_docker_wsl2.sh

set -e

echo "=== Docker Engine 설치 시작 ==="

# 기존 패키지 제거
sudo apt remove -y docker docker-engine docker.io containerd runc 2>/dev/null || true

# 필수 패키지 설치
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release

# Docker GPG 키 추가
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# 저장소 추가
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
    https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Docker 설치
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io \
    docker-buildx-plugin docker-compose-plugin

# 사용자 권한 설정
sudo usermod -aG docker $USER

# systemd 설정
sudo tee /etc/wsl.conf <<EOF
[boot]
systemd=true
EOF

echo "=== 설치 완료 ==="
echo "WSL을 재시작하세요: wsl --shutdown (PowerShell에서)"
echo "재시작 후: sudo systemctl enable --now docker"
```

---

## 다음 단계

> **설치 완료! 이제 무엇을 할 수 있나요?**
> 1. **Docker Compose로 개발 환경 구성** - 여러 서비스를 한번에 실행
> 2. **Dockerfile 작성법 학습** - 커스텀 이미지 빌드
> 3. **Docker 네트워크 이해** - 컨테이너 간 통신
> 4. **볼륨 마운트 활용** - 데이터 영속성 확보

### 참고 자료

- [Docker 공식 문서 - Ubuntu 설치](https://docs.docker.com/engine/install/ubuntu/)
- [Microsoft WSL 문서](https://docs.microsoft.com/en-us/windows/wsl/)
- [Docker Compose 문서](https://docs.docker.com/compose/)

---

*문서 작성일: 2025-01-28*
*대상 환경: Windows 11 + WSL2 + Ubuntu 24.04 + Docker Engine*
