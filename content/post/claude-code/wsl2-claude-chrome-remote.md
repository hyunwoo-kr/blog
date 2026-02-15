---
title: "WSL2에서 Claude Code로 Windows Chrome 원격 제어하기"
date: 2026-02-09
categories:
- claude-code
tags:
- wsl2
- claude-code
- chrome
- puppeteer
- remote-debugging
keywords:
- wsl2 chrome 원격 제어
- claude code chrome
- puppeteer-core
author: "hyunwoo"
draft: false
---

WSL2는 별도 VM 네트워크라 Windows의 `localhost:9222`에 직접 접근할 수 없습니다. 이 가이드에서는 Windows PowerShell에서 node.exe와 puppeteer-core를 경유하는 우회 방식으로 Chrome을 원격 제어하는 방법을 설명하고, Claude Code Skill로 등록하여 편리하게 사용하는 방법까지 다룹니다.

<!--more-->

> **TL;DR (3줄 요약)**
> 1. WSL2는 별도 VM 네트워크라 Windows `localhost:9222`에 직접 접근 불가
> 2. **Windows PowerShell -> node.exe -> puppeteer-core** 우회 방식으로 Chrome 원격 제어
> 3. Claude Code Skill(`/chrome`)로 등록하면 매번 긴 명령어 안 쳐도 됨

---

## 이 가이드가 필요한 이유

WSL2 환경에서 Claude Code를 사용할 때, 웹 애플리케이션 화면을 직접 확인해야 할 때가 있다.
예를 들어 JSP 화면이 제대로 렌더링되는지, 버튼 클릭이 동작하는지 등을 확인할 때.

그런데 WSL2에서 이런 시도를 하면 **무조건 실패**한다:

```bash
# 이건 절대 안 된다!
curl localhost:9222          # WSL2 → Windows localhost 접근 불가
curl 127.0.0.1:9222         # 마찬가지로 불가
```

> **왜 안 되는가?**
> WSL2는 Hyper-V 기반 **가상머신**이라 Windows와 **별도의 네트워크 스택**을 사용한다.
> WSL2에서 `localhost`는 WSL2 자체의 localhost이지, Windows의 localhost가 아니다.

이 문제를 해결하는 방법은 **Windows 측의 node.exe를 PowerShell 경유로 실행**하는 것이다.
Windows 내부에서는 `127.0.0.1:9222`에 정상 접근이 가능하다.

```
┌──────────────────┐         ┌──────────────────────────────────┐
│     WSL2         │         │          Windows                  │
│                  │         │                                    │
│  Claude Code     │         │  Chrome (--remote-debugging-port  │
│       │          │         │          =9222)                    │
│       ▼          │         │       ▲                            │
│  powershell.exe ─┼────────>│  node.exe + puppeteer-core        │
│  -Command "..."  │         │       │                            │
│                  │         │       └── 127.0.0.1:9222 ──────┘  │
└──────────────────┘         └──────────────────────────────────┘
```

---

## STEP 1. Windows 사전 준비

### 1-1. 실행 중인 Chrome 모두 종료

디버깅 모드로 Chrome을 새로 시작하려면, **기존 Chrome 프로세스를 모두 종료**해야 한다.
기존 Chrome이 떠 있으면 디버깅 포트가 열리지 않는다.

**Windows PowerShell 또는 CMD에서 실행:**

```shell
taskkill /F /IM chrome.exe /T
```

| 옵션 | 의미 |
|------|------|
| `/F` | 강제 종료 (Force) |
| `/IM chrome.exe` | 이미지 이름으로 지정 |
| `/T` | 하위 프로세스도 함께 종료 (Tree kill) |

WSL2에서 실행하려면:

```bash
powershell.exe -Command "taskkill /F /IM chrome.exe /T"
```

### 1-2. Chrome을 디버깅 모드로 실행

```shell
"C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222 --user-data-dir="C:\temp\chrome-debug"
```

| 옵션 | 의미 |
|------|------|
| `--remote-debugging-port=9222` | CDP(Chrome DevTools Protocol) 포트 열기 |
| `--user-data-dir="C:\temp\chrome-debug"` | 별도 프로필 디렉토리 사용 (기존 프로필과 충돌 방지) |

> **Chrome 설치 경로는 PC마다 다를 수 있다**
> - 64bit: `C:\Program Files\Google\Chrome\Application\chrome.exe`
> - 32bit: `C:\Program Files (x86)\Google\Chrome\Application\chrome.exe`

WSL2에서 한 번에 실행하려면:

```bash
powershell.exe -Command "& 'C:\Program Files (x86)\Google\Chrome\Application\chrome.exe' --remote-debugging-port=9222 --user-data-dir='C:\temp\chrome-debug'"
```

### 1-3. Node.js + puppeteer-core 설치

> **주의**: Windows에 Node.js가 설치되어 있어야 한다. (WSL2의 Node.js가 아님!)

**Windows PowerShell에서 실행:**

```shell
mkdir C:\temp\chrome-ctrl
cd C:\temp\chrome-ctrl
npm init -y
npm install puppeteer-core
```

WSL2에서 실행하려면:

```bash
powershell.exe -Command "mkdir -Force C:\temp\chrome-ctrl; cd C:\temp\chrome-ctrl; npm init -y; npm install puppeteer-core"
```

---

## STEP 2. Claude Code에서 Chrome 제어

### 기본 패턴

모든 명령은 이 패턴으로 실행한다:

```bash
/mnt/c/Windows/System32/WindowsPowerShell/v1.0/powershell.exe -Command "node -e \"<JS_CODE>\""
```

- WSL2에서 Windows PowerShell을 호출 -> PowerShell이 Windows node.exe를 실행
- node.exe는 Windows 네트워크를 사용하므로 `127.0.0.1:9222`에 접근 가능

### 연결 확인 + 탭 목록 보기

```bash
/mnt/c/Windows/System32/WindowsPowerShell/v1.0/powershell.exe -Command "node -e \"const p=require('C:/temp/chrome-ctrl/node_modules/puppeteer-core');(async()=>{const b=await p.connect({browserURL:'http://127.0.0.1:9222'});const pages=await b.pages();console.log('Connected! Tabs:');pages.forEach((pg,i)=>console.log(i+': '+pg.url()));b.disconnect()})().catch(e=>console.error('Error:',e.message))\""
```

출력 예시:

```
Connected! Tabs:
0: http://localhost:8082/login
1: https://www.naver.com/
```

### 페이지 이동

```bash
/mnt/c/Windows/System32/WindowsPowerShell/v1.0/powershell.exe -Command "node -e \"const p=require('C:/temp/chrome-ctrl/node_modules/puppeteer-core');(async()=>{const b=await p.connect({browserURL:'http://127.0.0.1:9222'});const[page]=await b.pages();await page.goto('https://www.naver.com',{waitUntil:'networkidle2',timeout:30000});console.log('Navigated to: '+page.url());b.disconnect()})().catch(e=>console.error('Error:',e.message))\""
```

### 스크린샷 촬영

```bash
/mnt/c/Windows/System32/WindowsPowerShell/v1.0/powershell.exe -Command "node -e \"const p=require('C:/temp/chrome-ctrl/node_modules/puppeteer-core');(async()=>{const b=await p.connect({browserURL:'http://127.0.0.1:9222'});const[page]=await b.pages();await page.screenshot({path:'C:/temp/chrome-ctrl/screenshot.png',fullPage:false});console.log('Screenshot saved');b.disconnect()})().catch(e=>console.error('Error:',e.message))\""
```

촬영한 스크린샷은 WSL2에서 이 경로로 확인:

```
/mnt/c/temp/chrome-ctrl/screenshot.png
```

### 요소 클릭

```bash
/mnt/c/Windows/System32/WindowsPowerShell/v1.0/powershell.exe -Command "node -e \"const p=require('C:/temp/chrome-ctrl/node_modules/puppeteer-core');(async()=>{const b=await p.connect({browserURL:'http://127.0.0.1:9222'});const[page]=await b.pages();await page.click('#loginBtn');console.log('Clicked');b.disconnect()})().catch(e=>console.error('Error:',e.message))\""
```

### 텍스트 입력

```bash
/mnt/c/Windows/System32/WindowsPowerShell/v1.0/powershell.exe -Command "node -e \"const p=require('C:/temp/chrome-ctrl/node_modules/puppeteer-core');(async()=>{const b=await p.connect({browserURL:'http://127.0.0.1:9222'});const[page]=await b.pages();await page.type('#userId','admin');console.log('Typed');b.disconnect()})().catch(e=>console.error('Error:',e.message))\""
```

### JavaScript 실행

```bash
/mnt/c/Windows/System32/WindowsPowerShell/v1.0/powershell.exe -Command "node -e \"const p=require('C:/temp/chrome-ctrl/node_modules/puppeteer-core');(async()=>{const b=await p.connect({browserURL:'http://127.0.0.1:9222'});const[page]=await b.pages();const result=await page.evaluate(()=>{return document.title});console.log('Result:',result);b.disconnect()})().catch(e=>console.error('Error:',e.message))\""
```

---

## STEP 3. Claude Code Skill로 등록 (`/chrome`)

위 과정을 매번 타이핑하기 번거로우므로, Claude Code Skill로 등록하면 `/chrome` 커맨드로 바로 사용할 수 있다.

**Skill 파일 위치**: `.claude/skills/chrome-remote/SKILL.md`

등록 후 사용법:

| 커맨드 | 동작 |
|--------|------|
| `/chrome` | 연결 확인 + 열린 탭 목록 |
| `/chrome goto <URL>` | 페이지 이동 |
| `/chrome screenshot` | 스크린샷 촬영 |
| `/chrome click <selector>` | CSS 셀렉터로 요소 클릭 |
| `/chrome type <selector> <text>` | 텍스트 입력 |
| `/chrome eval <js>` | JavaScript 실행 |

---

## 자주 하는 실수 & 해결법

| 증상 | 원인 | 해결 |
|------|------|------|
| `connect ECONNREFUSED` | Chrome이 디버깅 모드가 아님 | STEP 1-1, 1-2 다시 실행 |
| `Cannot find module 'puppeteer-core'` | Windows에 puppeteer-core 미설치 | STEP 1-3 실행 |
| `timeout` 에러 | 페이지 로딩이 느림 | timeout 값을 60000으로 증가 |
| 기존 Chrome 프로필 사용하고 싶음 | `--user-data-dir` 제거 | 단, 기존 Chrome 모두 종료 필수 |
| PowerShell에서 따옴표 에러 | 이스케이프 문제 | `\"` 사용, 긴 코드는 `.js` 파일로 분리 |

---

## 실전 팁: 긴 스크립트 실행

인라인 코드가 너무 길면 Windows 측에 `.js` 파일을 작성하고 실행하는 게 편하다:

```bash
# 1. WSL2에서 스크립트 파일 작성
cat > /mnt/c/temp/chrome-ctrl/script.js << 'EOF'
const puppeteer = require('C:/temp/chrome-ctrl/node_modules/puppeteer-core');

(async () => {
    const browser = await puppeteer.connect({
        browserURL: 'http://127.0.0.1:9222'
    });
    const [page] = await browser.pages();

    // 여기에 원하는 작업 작성
    await page.goto('http://localhost:8082/login');
    await page.type('#userId', 'admin');
    await page.type('#userPw', 'password');
    await page.click('#loginBtn');

    await page.waitForNavigation();
    console.log('Current URL:', page.url());

    browser.disconnect();
})().catch(e => console.error('Error:', e.message));
EOF

# 2. Windows node.exe로 실행
powershell.exe -Command "node C:/temp/chrome-ctrl/script.js"
```

---

## 핵심 요약

> 1. **WSL2에서 `curl localhost:9222`는 절대 안 된다** (별도 VM 네트워크)
> 2. **Windows PowerShell -> node.exe -> puppeteer-core** 경유로 우회
> 3. Chrome은 `--remote-debugging-port=9222`로 실행해야 함
> 4. `/chrome` Skill을 등록하면 편리하게 사용 가능
