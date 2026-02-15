---
title: "WSL2 Ubuntu 24.04에서 Claude Code 전역 설치 가이드"
date: 2026-01-27
categories:
- infra
tags:
- wsl2
- claude-code
- nodejs
- ubuntu
- installation
keywords:
- Claude Code
- WSL2
- Ubuntu 24.04
- Node.js
author: "hyunwoo"
draft: false
---

Windows 10의 WSL2(Ubuntu 24.04) 환경에서 Claude Code를 전역으로 설치하는 완벽 가이드입니다. 모든 사용자가 사용할 수 있도록 설정하는 방법을 단계별로 안내합니다.

<!--more-->

---

## 사전 요구사항

시작하기 전에 다음 항목을 준비해주세요.

- Windows 10 (버전 2004 이상)
- WSL2 활성화 및 Ubuntu 24.04 설치 완료
- 인터넷 연결
- Anthropic API 키

---

## 1단계: root 계정으로 전환

시스템 전역 설치를 위해 관리자 권한이 필요합니다.

```bash
sudo -i
```

> `sudo`는 관리자(superuser) 권한으로 명령을 실행하며, `-i`는 root 사용자의 환경으로 완전히 전환(login shell)합니다.

---

## 2단계: 시스템 패키지 업데이트

설치 전에 패키지 목록을 최신 상태로 갱신합니다.

```bash
apt-get update && apt-get upgrade -y
```

> - `apt-get update`: 사용 가능한 패키지 목록을 인터넷에서 다운로드하여 갱신
> - `apt-get upgrade -y`: 설치된 패키지들을 최신 버전으로 업그레이드
> - `-y`: 설치 확인 질문에 자동으로 "yes" 응답

---

## 3단계: Node.js 설치

Claude Code는 Node.js 18 버전 이상이 필요합니다. NodeSource 공식 저장소를 추가해서 최신 LTS 버전을 설치합니다.

### 3-1. 필수 패키지 설치

```bash
apt-get install -y ca-certificates curl gnupg
```

### 3-2. NodeSource GPG 키 등록

```bash
mkdir -p /etc/apt/keyrings

curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
```

> GPG 키란 패키지가 공식 배포자(NodeSource)가 만든 것인지 검증하는 디지털 서명입니다. 위변조된 패키지 설치를 방지합니다.

### 3-3. NodeSource 저장소 추가

```bash
echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_20.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list
```

### 3-4. 패키지 목록 갱신 및 Node.js 설치

```bash
apt-get update

apt-get install -y nodejs
```

### 3-5. 설치 확인

```bash
node -v
npm -v
```

Node.js가 18 이상이어야 Claude Code가 정상 작동합니다.

---

## 4단계: Claude Code 설치

npm을 사용해서 Claude Code를 시스템 전역에 설치합니다.

```bash
npm install -g @anthropic-ai/claude-code
```

> `-g` 옵션은 global(전역) 설치로, 시스템 전체에서 사용 가능하도록 `/usr/lib/node_modules/`에 설치합니다.

### 설치 확인

```bash
which claude
claude --version
```

---

## 5단계: 환경변수 설정 (API 키)

Claude Code를 실제로 사용하려면 API 키가 필요합니다.

### 방법 A: 시스템 전역 설정 (모든 사용자 공용)

모든 사용자가 동일한 API 키를 사용하는 방법입니다.

```bash
cat > /etc/profile.d/claude-code.sh << 'EOF'
# Claude Code 환경 설정
export ANTHROPIC_API_KEY="sk-ant-xxxxx-your-api-key-here"
EOF

chmod 644 /etc/profile.d/claude-code.sh
```

### 방법 B: 사용자별 개별 설정 (권장)

각 사용자가 자신의 API 키를 사용하는 방식입니다. 사용량 추적과 보안 측면에서 이 방법을 권장합니다.

```bash
# root에서 나간 후, 각 사용자 계정에서 실행
exit

# 일반 사용자로 로그인 후
echo 'export ANTHROPIC_API_KEY="sk-ant-xxxxx-your-personal-key"' >> ~/.bashrc
source ~/.bashrc
```

---

## 6단계: 설치 확인 테스트

### root에서 나가기 (아직 root인 경우)

```bash
exit
```

### 일반 사용자로 테스트

```bash
# 환경변수 확인
echo $ANTHROPIC_API_KEY

# Claude Code 실행 테스트
claude --version

# 실제 사용 테스트 (프로젝트 디렉토리에서)
cd ~/your-project
claude
```

---

## 7단계: 업데이트 방법

새 버전이 나오면 다음과 같이 업데이트합니다.

```bash
sudo npm update -g @anthropic-ai/claude-code
```

---

## 설치 결과 요약

| 항목 | 위치/값 |
|------|---------|
| Node.js | `/usr/bin/node` |
| npm | `/usr/bin/npm` |
| Claude Code 실행파일 | `/usr/bin/claude` |
| Claude Code 모듈 | `/usr/lib/node_modules/@anthropic-ai/claude-code/` |
| 전역 환경변수 | `/etc/profile.d/claude-code.sh` |
| 사용자별 환경변수 | `~/.bashrc` |

---

## 문제 해결

### 패키지 설치 오류 시

```bash
# 패키지 캐시 정리 후 재시도
apt-get clean
apt-get update
apt-get install -y nodejs
```

### npm 권한 오류 시

```bash
# npm 캐시 정리
npm cache clean --force
```

### 환경변수가 적용되지 않을 때

```bash
# 현재 터미널에 즉시 적용
source /etc/profile.d/claude-code.sh

# 또는 터미널을 완전히 종료 후 다시 열기
```

---

## API 키 설정 방식 비교

| 구분 | 전역 설정 (방법 A) | 사용자별 설정 (방법 B) |
|------|-------------------|----------------------|
| 설정 위치 | `/etc/profile.d/claude-code.sh` | `~/.bashrc` |
| 설정 권한 | root 필요 | 각 사용자 |
| API 키 | 모든 사용자 동일 | 사용자마다 다름 |
| 사용량 추적 | 어려움 | 사용자별 추적 가능 |
| 보안 | 키 노출 위험 높음 | 상대적으로 안전 |
| 관리 편의성 | 한 곳에서 관리 | 각 사용자가 관리 |

---

## 참고 링크

- [Claude Code 공식 문서](https://docs.anthropic.com/en/docs/claude-code)
- [Anthropic API 키 발급](https://console.anthropic.com/)
- [NodeSource 공식 저장소](https://github.com/nodesource/distributions)
