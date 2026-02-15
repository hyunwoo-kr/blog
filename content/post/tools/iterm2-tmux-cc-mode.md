---
title: "iTerm2 + Tmux, 대체 뭐가 다른 건데? (-CC 모드까지)"
date: 2026-02-14
categories:
- tools
tags:
- iterm2
- tmux
- macos
- terminal
- productivity
keywords:
- iterm2 tmux
- tmux cc mode
- mac terminal
- terminal multiplexer
author: "hyunwoo"
draft: false
---

iTerm2와 tmux를 함께 쓰면 3가지 레이어가 겹쳐서 혼란스러울 수 있습니다. 이 글에서는 두 도구의 관계를 정리하고, 특히 `-CC` 모드를 활용해 tmux의 세션 유지 능력과 iTerm2의 네이티브 UX를 결합하는 방법을 다룹니다.

<!--more-->

*이 글은 Linux에서 Mac으로 전환한 개발자를 위해, iTerm2 + Tmux의 핵심 개념과 -CC 모드 활용법을 정리한 글입니다.*

---

## 왜 이렇게 헷갈리는 거야? - 3가지 레이어

iTerm2랑 tmux를 같이 쓰면 **3가지 레이어**가 겹칩니다.

```
+─────────────────────────────────────────────────+
|  레이어 3: iTerm2 (Mac 터미널 앱)                   |
|  - 자체 탭(Tab), 자체 분할(Split Pane)              |
|  - Cmd+T(새 탭), Cmd+D(세로분할), Cmd+Shift+D(가로분할)|
+─────────────────────────────────────────────────+
|  레이어 2: Tmux (터미널 멀티플렉서)                   |
|  - 자체 Window, 자체 Pane                         |
|  - Ctrl+B c(새 윈도우), Ctrl+B %(세로), "(가로)     |
+─────────────────────────────────────────────────+
|  레이어 1: Shell (zsh/bash)                       |
|  - 실제 명령 실행                                   |
+─────────────────────────────────────────────────+
```

iTerm2도 탭이 있고, tmux도 윈도우가 있습니다. 둘 다 비슷한 기능인데 왜 같이 쓸까요?

핵심은 **세션 유지**입니다. iTerm2만 쓰면 앱 종료하거나 Mac 재시작하면 다 날아갑니다. 그런데 tmux는 세션이 살아있어서, 다시 접속하면 그대로 복원됩니다. SSH로 서버 작업할 때 연결 끊겨도 tmux 세션은 서버에 살아있습니다.

---

## Mac 전환 첫 번째 관문: 키 매핑

리눅스에서 넘어왔으면 키부터 헷갈립니다. 가장 중요한 건 `Cmd`(Command) 키입니다. 리눅스에서 GUI 앱의 `Ctrl` 역할을 Mac에서는 `Cmd`가 합니다.

| Linux | Mac | 설명 |
|-------|-----|------|
| `Ctrl` | `Control` | 동일 위치, 동일 역할 |
| `Alt` | `Option` | iTerm2에서 Meta키로 설정 필요 |
| (없음) | `Cmd (Command)` | **Mac 전용**, GUI 앱 단축키의 핵심 |
| `Ctrl+C` (복사) | `Cmd+C` | Mac에서 Cmd가 GUI 단축키 |
| `Ctrl+Shift+T` (새탭) | `Cmd+T` | iTerm2 새 탭 |

터미널에서 `Ctrl+C`는 여전히 프로세스 중지고, 텍스트 복사는 `Cmd+C`입니다. Mac에서 `Cmd`는 Linux의 GUI `Ctrl` 역할이고, 터미널의 `Control`은 그대로 시그널 역할입니다.

### iTerm2 기본 단축키

| 단축키 | 동작 |
|--------|------|
| `Cmd+T` | 새 탭 |
| `Cmd+W` | 탭/패인 닫기 |
| `Cmd+D` | 세로 분할 (왼쪽\|오른쪽) |
| `Cmd+Shift+D` | 가로 분할 (위/아래) |
| `Cmd+[` / `Cmd+]` | 분할 패인 간 이동 |
| `Cmd+1~9` | N번째 탭으로 이동 |
| `Cmd+Shift+Enter` | 현재 패인 최대화/복원 |
| `Cmd+F` | 터미널 내 검색 |
| `Cmd+Enter` | 전체화면 토글 |

iTerm2에만 있는 킬러 기능으로 **Hotkey Window**가 있습니다. `Option+Space` 누르면 화면 위에서 터미널이 슬라이드로 내려옵니다. Quake 게임 콘솔처럼! 설정에서 켜면 됩니다.

---

## tmux 기초: Session, Window, Pane

tmux는 **터미널 멀티플렉서**(Terminal Multiplexer)입니다. 쉽게 말하면 "하나의 터미널 안에서 여러 개의 화면을 동시에 띄워놓는 도구"입니다.

```
Session  -+- Window 1 (= "탭" 개념)
           |    ├─ Pane A (분할된 영역)
           |    └─ Pane B
           ├─ Window 2
           |    └─ Pane A
           └─ Window 3
                ├─ Pane A
                ├─ Pane B
                └─ Pane C
```

**Session** 안에 **Window**가 여러 개, Window 안에 **Pane**이 여러 개인 구조입니다. tmux만의 차이점은 **Detach/Attach**입니다. 세션에서 빠져나와도(Detach) 세션은 백그라운드에서 계속 살아있고, 나중에 다시 붙으면(Attach) 그대로 복원됩니다.

### tmux 주요 단축키

tmux 명령은 항상 `Ctrl+B`를 먼저 누르고 그 다음 키를 눌러야 합니다 (Prefix 키):

| 단축키 | 동작 |
|--------|------|
| `Ctrl+B c` | 새 Window |
| `Ctrl+B %` | 세로 분할 |
| `Ctrl+B "` | 가로 분할 |
| `Ctrl+B 방향키` | Pane 간 이동 |
| `Ctrl+B n` / `p` | 다음/이전 Window |
| `Ctrl+B d` | Session Detach (빠져나오기) |
| `Ctrl+B z` | 현재 Pane 줌(최대화/복원) |
| `Ctrl+B [` | 스크롤(복사) 모드 진입 |

### 세션 관리 명령어

```bash
tmux new -s work        # 새 세션 생성 (이름: work)
tmux ls                 # 세션 목록 확인
tmux attach -t work     # 세션 재접속 (줄여서 tmux a -t work)
tmux kill-session -t work  # 세션 강제 종료
```

세션 이름을 안 붙이면 0, 1, 2... 숫자로 자동 이름이 붙습니다. 세션 많아지면 구분이 안 되니 **반드시 이름 붙이는 습관**을 들이는 게 좋습니다.

---

## tmux -CC: 이게 진짜 마법이야

`-CC` 모드(Control-Control Mode)는 tmux가 텍스트 UI 대신 **iTerm2에게 직접 제어 프로토콜을 전송**하는 모드입니다.

쉽게 말하면 일반 tmux를 쓰면 터미널 안에 tmux만의 UI(하단 상태바, 텍스트 분할선)가 보입니다. 그런데 `-CC` 모드를 쓰면 tmux 윈도우가 **iTerm2 네이티브 탭**으로 보이고, tmux 패인이 **iTerm2 네이티브 분할**로 보입니다!

```bash
# 일반 tmux
tmux new -s work          # -> 터미널 안에 tmux UI(상태바 등)가 보임

# -CC 모드
tmux -CC new -s work      # -> iTerm2 네이티브 탭/분할로 tmux가 동작
tmux -CC attach -t work   # -> 기존 세션에 -CC로 재접속
```

탭 타이틀에 `[tmux: 세션이름]`이 표기되니까 그걸로 tmux 안인지 구분할 수 있습니다.

### 일반 tmux vs -CC 모드 비교

| 항목 | 일반 tmux | -CC 모드 |
|------|----------|---------|
| Window 표시 | tmux 하단 상태바 | **iTerm2 네이티브 탭** |
| Pane 표시 | tmux 텍스트 분할선 | **iTerm2 네이티브 분할** |
| 스크롤 | `Ctrl+B [` | **마우스/트랙패드** 그대로! |
| 복사/붙여넣기 | tmux 복사모드 | **Cmd+C / Cmd+V** 네이티브! |
| 탭 이동 | `Ctrl+B n/p` | **Cmd+좌/우** 또는 **Cmd+1~9** |
| 패인 리사이즈 | `Ctrl+B Ctrl+방향키` | **마우스 드래그** |
| 시각적 구분 | 하단에 tmux 상태바 보임 | **tmux 상태바 없음!** |

> **-CC 모드의 최대 장점 요약**
> 1. 네이티브 UX: Mac 유저에게 익숙한 Cmd 단축키로 tmux 조작
> 2. 스크롤이 자연스러움: 트랙패드/마우스 스크롤 그대로 동작
> 3. 복사/붙여넣기: Cmd+C/Cmd+V 그냥 됨
> 4. Session 유지: SSH 끊겨도 `tmux -CC attach`로 원래 레이아웃 복원
> 5. 리사이즈: 마우스로 분할선 드래그

`-CC` 세션에서 Detach 하면 탭이 **"Bury(매장)"**됩니다. 사라진 게 아니라 숨겨진 것입니다. 복원하려면 iTerm2 메뉴에서 `Shell > tmux > Dashboard`로 가거나, 터미널에서 `tmux -CC attach -t 세션이름`을 입력하면 됩니다.

---

## 실전 워크플로우

### 워크플로우 1: 로컬 개발 (추천!)

매일 아침 이것만 치면 됩니다:

```bash
tmux -CC new -A -s work
```

`-A` 옵션은 **세션이 있으면 Attach, 없으면 New**입니다. 이 한 줄이면 매번 "세션 있나? 없나?" 분기할 필요가 없습니다.

```bash
# 아침에 Mac 켜고 iTerm2 실행
tmux -CC new -A -s work     # 한 줄로 끝!

# 탭 구성 (Cmd+T로 추가):
# 탭 1: 코드 작업
# 탭 2: 서버 실행
# 탭 3: 로그 모니터링 (tail -f)
# 탭 4: git 작업

# 퇴근 시 -> 그냥 Mac 닫기 (세션 유지됨)
# 다음 날 -> tmux -CC new -A -s work -> 어제 그대로 복원!
```

프로젝트별로 세션 이름을 다르게 만들 수도 있습니다:

```bash
tmux -CC new -s chemisys       # 화학제품관리시스템
tmux -CC new -s personal       # 개인 프로젝트

# 세션 목록 확인
tmux ls
```

### 워크플로우 2: SSH + tmux (원격 서버)

SSH 너머에서도 `-CC`가 동작합니다:

```bash
# 방법 1: SSH 접속 후 수동으로
ssh user@server
tmux -CC new -A -s work

# 방법 2: iTerm2 프로필에 한 줄로 설정
# iTerm2 > Preferences > Profiles > General > Command:
ssh user@server -t "tmux -CC new -A -s main"
```

방법 2면 프로필 선택만 하면 자동으로 SSH + tmux 접속까지 됩니다. 서버별로 프로필 만들어두면 클릭 한 번입니다. SSH 끊겨도 세션은 서버에 살아있으니까, 다시 접속해서 `tmux -CC attach -t work` 하면 그대로 복원됩니다.

---

## 설정 꿀팁

### ~/.tmux.conf

```bash
# ~/.tmux.conf

# 인덱스 1부터 시작 (0은 키보드 왼쪽 끝이라 불편)
set -g base-index 1
setw -g pane-base-index 1

# 윈도우 번호 자동 정리
set -g renumber-windows on

# 마우스 지원 (패인 선택/리사이즈)
set -g mouse on

# 히스토리 크기
set -g history-limit 50000

# 256 컬러
set -g default-terminal "screen-256color"
set -ga terminal-overrides ",xterm-256color:Tc"

# ESC 키 딜레이 제거 (Vim 사용자 필수!)
set -sg escape-time 0

# 직관적인 분할 키
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"

# 새 윈도우를 현재 경로로
bind c new-window -c "#{pane_current_path}"

# 설정 리로드
bind r source-file ~/.tmux.conf \; display "Config Reloaded!"
```

`bind | split-window`은 tmux에서 세로 분할을 원래 `Ctrl+B %`에서 `|`(파이프)로 바꾼 것입니다. 세로선 모양이니까 기억하기 쉽습니다. 가로 분할도 `-`(대시)로 바꿨습니다.

`-c "#{pane_current_path}"`는 새 패인이나 윈도우를 열 때 **현재 디렉토리에서 열리게** 하는 옵션입니다. 이거 안 하면 매번 홈 디렉토리에서 열려서 `cd`를 다시 해야 합니다.

### iTerm2 설정

> **iTerm2 > Settings > General > tmux**
> - Open tmux windows as: **Native tabs in a new window** (tmux 탭이 별도 iTerm2 윈도우에 열림, 섞이지 않음!)
> - Automatically bury the tmux client session: 체크 (detach 시 자동 숨김)
>
> **Settings > Profiles > Keys > General**
> - Left Option Key: **Esc+** (Vim/Emacs Meta키)
> - Right Option Key: **Normal** (한글 입력과 충돌 방지)

Hotkey Window 설정은 Settings > Keys > Hotkey에서 "Create a Dedicated Hotkey Window" 클릭하고, 핫키를 `Option+Space`로 설정합니다. 프로필에서 Window Style을 "Full-Width Top of Screen"으로 하면 화면 위에서 터미널이 슬라이드로 내려옵니다.

---

## 트러블슈팅

> **Q: -CC 탭이랑 일반 탭이 섞여서 구분이 안 돼요**
> A: iTerm2 설정에서 "Open tmux windows as" -> **"Native tabs in a new window"**로 바꿔. tmux 탭이 별도 윈도우에 열려서 섞이지 않습니다.

> **Q: Detach 했는데 세션이 사라진 것 같아요**
> A: Bury된 것입니다! `Shell > tmux > Dashboard`에서 세션 클릭하거나, 터미널에서 `tmux -CC attach -t 세션이름` 치면 복원됩니다.

> **Q: SSH 연결이 끊겼어요!**
> A: 당황하지 마세요! 서버에 다시 SSH 접속하고 `tmux -CC attach -t work` 하면 그대로 살아있습니다.

> **Q: 한글이 깨져요**
> A: `.zshrc`에 이거 추가하세요:
> `export LANG=ko_KR.UTF-8`
> `export LC_ALL=ko_KR.UTF-8`

---

## 마무리

| 상황 | 이렇게 하면 됩니다 |
|------|-------------------|
| 로컬 작업 시작 | `tmux -CC new -A -s 이름` |
| SSH 원격 작업 | SSH 접속 후 `tmux -CC new -A -s 이름` |
| 세션 복원 | `tmux -CC attach -t 이름` |
| 세션 확인 | `tmux ls` |
| 탭/분할 조작 | -CC면 `Cmd` 키, 일반이면 `Ctrl+B` |
| Detach | `Ctrl+B d` (일반/-CC 모두) |
| 전부 종료 | `tmux kill-server` |

**한 줄로 요약하면:** -CC 모드 = tmux의 세션 유지 능력 + iTerm2의 네이티브 UX. 평소엔 `Cmd` 키만 쓰고, detach/attach만 tmux 명령으로 하면 됩니다.
