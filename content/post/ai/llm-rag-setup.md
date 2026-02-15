---
title: "LLM + RAG 환경구성 하기"
date: 2025-09-16
categories:
- ai
tags:
- llm
- rag
- pyenv
- poetry
- python
keywords:
- LLM
- RAG
- pyenv
- poetry
- python 환경구성
author: "hyunwoo"
draft: false
---

LLM + RAG 환경을 구성하기 위해 Python 버전 관리 도구인 pyenv와 패키지 관리 도구인 Poetry를 설치하고 설정하는 방법을 정리합니다.

<!--more-->

## 1. pyenv 란?

pyenv는 Python의 여러 버전을 설치하고 관리할 수 있는 도구입니다. 프로젝트별로 특정 Python 버전을 지정하거나, 시스템 전역 Python 버전을 변경할 수 있습니다.

Windows에서는 `pyenv-win`을 사용하며, 이는 Windows 환경에 최적화된 pyenv입니다.

## 2. 사전 요구사항

- Git : 이미 설치되어 있다고 가정 (`git --version` 명령어로 확인 가능)
- Window 10/11
- PowerShell 또는 CMD: 관리자 권한으로 실행
- 인터넷 연결

## 3. 윈도우에서 pyenv-win Install 하기

PowerShell 열고, 홈 디렉토리로 이동하기.

```bash
cd ~/
```

Git을 사용해 `pyenv-win` repository clone

```bash
git clone https://github.com/pyenv-win/pyenv-win.git "$env:USERPROFILE\.pyenv"
```

- .pyenv : `$env:USERPROFILE` 경로 아래에 pyenv라는 이름의 숨김 폴더를 생성하고, 그 안에 저장소 내용을 복제하겠다는 의미입니다. 윈도우에서 폴더나 파일 이름 앞에 마침표(.)를 붙이면 숨김 속성이 부여됩니다.
- `$env:USERPROFILE` : 환경 변수. PowerShell에서 사용자 프로필 디렉토리의 경로를 나타내며, 보통 `c:\Users\<사용자이름>` 과 같은 경로를 가리킵니다.

환경 변수 설정

pyenv 명령어를 사용하려면 환경 변수에 경로를 추가해야 합니다.

PowerShell에서:

```bash
[System.Environment]::SetEnvironmentVariable('PYENV', "$env:USERPROFILE\.pyenv\pyenv-win\", 'User')
[System.Environment]::SetEnvironmentVariable('PYENV_ROOT', "$env:USERPROFILE\.pyenv\pyenv-win\", 'User')
[System.Environment]::SetEnvironmentVariable('PYENV_HOME', "$env:USERPROFILE\.pyenv\pyenv-win\", 'User')
[System.Environment]::SetEnvironmentVariable('Path', "$env:USERPROFILE\.pyenv\pyenv-win\bin;" + "$env:USERPROFILE\.pyenv\pyenv-win\shims;" + [System.Environment]::GetEnvironmentVariable('Path', 'User'), 'User')
```

현재의 PowerShell을 종료 후 다시 실행합니다.

다음의 명령어를 입력하여 정상 동작하는지 확인합니다.

```bash
pyenv
```

Python Install

예를 들어, Python 3.9.11을 설치하려면:

```powershell
pyenv install 3.9.11

# 설치된 Python은 ~/.pyenv/pyenv-win/versions/ 디렉토리에 저장됩니다.
```

## Python 설치하기

pyenv

```powershell
pyenv install 3.9.11
```

3.9.11 버전의 python 설정

```powershell
pyenv global 3.9.11
```

파이썬 버전 확인

```powershell
pyenv version
```

## Poetry 설치하기

- `pip + virtualenv` 같은 전통적인 방식보다 더 체계적으로 패키지 관리
- 프로젝트별로 **가상환경 자동 관리**
- **의존성 충돌 해결**
- 패키지 **빌드 및 PyPI 배포 지원**

**Poetry는** Node.js 세계의 **npm/yarn**, Java 세계의 **Maven/Gradle** 같은 역할을 Python에서 수행한다고 보면 됩니다.

아래의 명령어를 실행하여 Poetry 패키지 관리 도구를 설치

```powershell
pip3 install poetry=1.8.5
```

### Poetry 주요 기능

1. **의존성 관리**

- `pyproject.toml` 파일에 패키지와 버전이 기록됨
- `poetry add` 명령으로 패키지를 추가하면 자동으로 기록 + 설치

```bash
poetry add requests
```

`pyproject.toml`과 `poetry.lock`이 업데이트됨

2. **가상환경 관리**

- 프로젝트마다 독립된 가상환경 생성

현재 프로젝트 전용 환경에 진입하기

```bash
poetry shell
```

3. **실행**

- 가상환경 안에서 명령 실행

```bash
poetry run python app.py
```

4. **패키지 빌드 & 배포**

- Python 라이브러리를 만들어 PyPI에 배포할 때 사용

```bash
# 빌드
poetry build

# 배포
poetry publish
```
