---
title: "systemctl reset-failed 완벽 가이드"
date: 2025-01-28
categories:
- devops
tags:
- systemctl
- systemd
- linux
- docker
- troubleshooting
keywords:
- systemctl reset-failed
- Start request repeated too quickly
- systemd rate limiting
author: "hyunwoo"
draft: false
---

서비스가 여러 번 실패하면 systemd가 재시작을 차단하고, 설정 파일을 수정해도 실패 카운터가 초기화되지 않습니다. `sudo systemctl reset-failed <service>`로 초기화 후 재시작할 수 있습니다.

<!--more-->

---

## 이 가이드가 필요한 이유

> **이런 상황을 겪어본 적 있나요?**
> 1. Docker 설정 변경하려고 `systemctl stop docker`
> 2. `/etc/docker/daemon.json` 수정
> 3. `systemctl start docker` 했는데 실패
> 4. 설정 파일 오류 수정 후 다시 시도해도 **계속 실패**
> 5. "Start request repeated too quickly" 메시지만 반복...
>
> 이 가이드는 이 문제의 **원인과 해결법**을 설명합니다.

---

## 문제 상황

### 증상

서비스를 시작하려고 할 때 다음과 같은 오류 발생:

```
x docker.service - Docker Application Container Engine
     Active: failed (Result: exit-code)
...
Jan 28 03:02:07 systemd[1]: docker.service: Start request repeated too quickly.
Jan 28 03:02:07 systemd[1]: docker.service: Failed with result 'exit-code'.
Jan 28 03:02:07 systemd[1]: Failed to start docker.service - Docker Application Container Engine.
```

### 발생 시나리오

1. `systemctl stop docker`로 서비스 중지
2. 설정 파일 수정 (예: `/etc/docker/daemon.json`)
3. 설정 파일에 오류가 있는 상태로 `systemctl start docker` 시도
4. systemd가 자동으로 3회 재시작 시도 -> 모두 실패
5. **"Start request repeated too quickly"** 로 서비스 차단됨
6. 설정 파일을 올바르게 수정해도 **여전히 시작 불가**

---

## 원인 이해하기

### systemd의 Rate Limiting (재시작 제한)

systemd는 서비스가 반복적으로 실패할 때 시스템 보호를 위해 **재시작 제한**을 적용합니다.

```
[Service]
Restart=on-failure          # 실패 시 자동 재시작
RestartSec=2s               # 재시작 간격: 2초
StartLimitIntervalSec=10s   # 10초 내에
StartLimitBurst=3           # 3번까지만 재시작 허용
```

### 동작 방식 시각화

```
시간 ─────────────────────────────────────────────>
     | 실패 | 2초 | 실패 | 2초 | 실패 | 차단!
     └──────┴─────┴──────┴─────┴──────┴────────
          1회      2회      3회    "too quickly"

     └─────────── 10초 이내 ───────────┘
```

> **핵심:** systemd는 **실패 카운터**를 메모리에 유지합니다.
> - 설정 파일을 수정해도 실패 카운터는 초기화되지 않음
> - 카운터가 3에 도달하면 계속 차단됨
> - **명시적으로 초기화해야 함**

---

## 해결 방법

### 방법 1: reset-failed (권장)

```bash
# 특정 서비스의 실패 상태 초기화
sudo systemctl reset-failed docker.service

# 서비스 시작
sudo systemctl start docker
```

> **reset-failed가 하는 일:**
> - 실패 카운터를 0으로 초기화
> - 서비스 상태를 "failed"에서 "inactive"로 변경
> - 재시작 제한 타이머 초기화

### 방법 2: 모든 서비스 실패 상태 초기화

```bash
# 모든 서비스의 실패 상태 초기화
sudo systemctl reset-failed
```

### 방법 3: systemd 데몬 재로드 (설정 변경 시)

```bash
# .service 파일을 수정한 경우
sudo systemctl daemon-reload
sudo systemctl reset-failed docker.service
sudo systemctl start docker
```

---

## 실전 사례: Docker 서비스

### 전체 문제 해결 흐름

```bash
# 1. 문제 상황: Docker 설정 변경 중 오류 발생
sudo systemctl stop docker
sudo vi /etc/docker/daemon.json    # 실수로 잘못된 JSON 입력
sudo systemctl start docker        # 실패 시작...

# 2. 오류 확인
sudo systemctl status docker
# -> "Start request repeated too quickly" 메시지

# 3. 로그에서 근본 원인 확인
sudo journalctl -u docker.service -n 20 --no-pager
# -> "invalid character '}' looking for beginning of object key string"

# 4. 설정 파일 수정
sudo vi /etc/docker/daemon.json    # JSON 오류 수정

# 5. 실패 상태 초기화 및 재시작
sudo systemctl reset-failed docker.service
sudo systemctl start docker

# 6. 정상 동작 확인
sudo systemctl status docker
docker ps
```

### daemon.json JSON 오류 예방

```bash
# 수정 전 JSON 문법 검증
cat /etc/docker/daemon.json | python3 -m json.tool

# 또는 jq 사용
cat /etc/docker/daemon.json | jq .
```

---

## 자주 하는 실수 & 해결법

### 실수 1: JSON trailing comma (마지막 쉼표)

```json
// 잘못된 예
{
    "dns": ["8.8.8.8"],
    "storage-driver": "overlay2",  // <- 마지막에 쉼표!
}

// 올바른 예
{
    "dns": ["8.8.8.8"],
    "storage-driver": "overlay2"
}
```

### 실수 2: 작은따옴표 사용

```json
// 잘못된 예
{
    'dns': ['8.8.8.8']  // <- JSON은 큰따옴표만 허용
}

// 올바른 예
{
    "dns": ["8.8.8.8"]
}
```

### 실수 3: reset-failed 없이 계속 재시도

```bash
# 잘못된 접근
sudo systemctl start docker   # 실패
sudo systemctl start docker   # 또 실패
sudo systemctl start docker   # 계속 실패...

# 올바른 접근
sudo systemctl reset-failed docker.service
sudo systemctl start docker
```

---

## 예방 및 모범 사례

### 설정 변경 시 안전한 절차

```bash
# 1. 현재 설정 백업
sudo cp /etc/docker/daemon.json /etc/docker/daemon.json.bak

# 2. 설정 수정
sudo vi /etc/docker/daemon.json

# 3. JSON 문법 검증
cat /etc/docker/daemon.json | python3 -m json.tool
# 오류 시: "Expecting property name enclosed in double quotes"

# 4. 문제없으면 서비스 재시작
sudo systemctl restart docker

# 5. 문제 발생 시 백업에서 복원
sudo cp /etc/docker/daemon.json.bak /etc/docker/daemon.json
sudo systemctl reset-failed docker.service
sudo systemctl start docker
```

---

## 관련 명령어 정리

### 기본 서비스 관리

| 명령어 | 설명 |
|--------|------|
| `systemctl start <service>` | 서비스 시작 |
| `systemctl stop <service>` | 서비스 중지 |
| `systemctl restart <service>` | 서비스 재시작 |
| `systemctl status <service>` | 서비스 상태 확인 |

### 실패 상태 관리

| 명령어 | 설명 |
|--------|------|
| `systemctl reset-failed` | 모든 서비스 실패 상태 초기화 |
| `systemctl reset-failed <service>` | 특정 서비스 실패 상태 초기화 |
| `systemctl --failed` | 실패한 서비스 목록 |

### 로그 및 디버깅

| 명령어 | 설명 |
|--------|------|
| `journalctl -u <service>` | 서비스 로그 전체 |
| `journalctl -u <service> -n 50` | 최근 50줄 |
| `journalctl -u <service> -f` | 실시간 로그 |
| `journalctl -u <service> --since "5 min ago"` | 최근 5분 로그 |

---

## 핵심 요약

> **문제:** "Start request repeated too quickly"
> **원인:** systemd의 재시작 제한 (10초 내 3회 실패 시 차단)
> **해결:**
> 1. `journalctl`로 근본 원인 확인
> 2. 원인 해결 (설정 파일 수정 등)
> 3. `sudo systemctl reset-failed <service>`
> 4. `sudo systemctl start <service>`

---

## 다음 단계 / 추가 학습 자료

> **더 알아보기:**
> - [systemd 공식 문서](https://www.freedesktop.org/software/systemd/man/systemd.service.html)
> - [journalctl 사용법](https://www.freedesktop.org/software/systemd/man/journalctl.html)
> - Docker 데몬 설정: `/etc/docker/daemon.json`

---

*문서 작성일: 2025-01-28*
*환경: Ubuntu 24.04 / WSL2 / systemd*
