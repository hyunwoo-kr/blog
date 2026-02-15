---
title: "bat 써봤어? cat은 이제 졸업이야"
date: 2026-02-15
categories:
- tools
tags:
- bat
- cat
- terminal
- cli
keywords:
- bat
- cat 대체
- 구문 강조
- syntax highlighting
author: "hyunwoo"
draft: false
---

`bat`은 `cat`의 상위 호환 도구입니다. 구문 강조, 줄 번호, Git 변경 표시, 183개 언어 지원, 28개 테마까지. alias 하나만 걸어두면 평소처럼 `cat` 쓰듯이 자연스럽게 사용할 수 있습니다.

<!--more-->

---

## 도입

> **현우**: 도현아, 터미널에서 파일 내용 볼 때 뭐 써?

> **도현**: `cat` 쓰죠! `cat file.txt` 하면 바로 내용 나오잖아요.

> **현우**: 나도 예전에 그랬는데, 한번 `bat` 써보고 나서는 진짜 못 돌아가겠더라. 구문 강조에 줄 번호에 Git 변경까지 보여주거든.

> **도현**: bat이요? 처음 들어보는데, cat이랑 뭐가 다른 건가요?

> **현우**: 쉽게 말하면 **bat**은 cat의 상위 호환이야. cat이 순수 텍스트만 보여주는 걸, bat은 코드 에디터처럼 예쁘게 보여줘. Rust로 만들어져서 속도도 빠르고.

---

## bat 설치하고 alias 걸기

> **현우**: macOS에서는 Homebrew로 간단하게 설치할 수 있어.

```bash
brew install bat
```

> **도현**: 설치는 쉽네요! 근데 매번 `bat`이라고 치기 귀찮을 것 같은데요.

> **현우**: 그래서 alias를 걸어두는 거야. `.zshrc`에 이렇게 한 줄만 추가해.

```bash
alias cat='bat --paging=never'
```

> **도현**: `--paging=never`는 뭔가요?

> **현우**: bat은 기본적으로 less처럼 페이저를 띄워서 스크롤하게 해주거든. 근데 cat처럼 쓰려면 페이저 없이 바로 출력되는 게 편하잖아. 그래서 꺼주는 거야.

> **도현**: 그러면 이제 `cat file.txt` 치면 자동으로 bat이 실행되는 거네요? 근데 원래 cat이 필요하면 어떡해요?

> **현우**: 좋은 질문이야! alias를 우회하는 방법이 몇 가지 있어. `command cat` 명령으로 호출하거나, 절대 경로로 직접 지정하면 돼. 바이너리 파일 합칠 때나 ANSI 색상 코드 없이 파이프 처리할 때 원본 cat이 필요해.

---

## 구문 강조와 테마

> **현우**: bat의 제일 큰 장점이 바로 **구문 강조**(syntax highlighting)야. 파일 확장자를 보고 자동으로 언어를 감지해서 색칠해줘.

```bash
# 자동 감지 (확장자 기반)
cat main.py          # Python
cat config.yaml      # YAML

# 수동 지정 (-l 옵션)
bat -l python script        # 확장자 없는 파일
bat -l sql query.txt        # .txt지만 SQL로 표시
cat -l json data.log        # JSON 형식 로그
```

> **도현**: 오, 확장자만으로 알아서 해주는 거예요? 몇 개나 지원하나요?

> **현우**: 무려 **183개** 언어를 지원해. Java, JavaScript, SQL, HTML, Python 다 되고, 심지어 `.env` 같은 설정 파일도 매핑할 수 있어.

```bash
bat -m '*.env:Bourne Again Shell (bash)' .env
bat --ignored-suffix ".bak" file.json.bak
```

> **도현**: 테마도 바꿀 수 있어요? 저 Dracula 테마 좋아하는데...

> **현우**: 28개 테마가 내장되어 있어. 인기 있는 것들 보면...

| 테마 | 설명 |
|------|------|
| Dracula | 인기 다크 테마 |
| Monokai Extended | Sublime Text 느낌 |
| Nord | 차분한 파란 톤 |
| GitHub | 밝은 배경 스타일 |
| Catppuccin Mocha | Catppuccin 다크 |

> **현우**: 적용은 이렇게 해.

```bash
# 일회성
bat --theme="Dracula" file.py

# 기본값 (.zshrc에 추가)
export BAT_THEME="Dracula"

# 터미널 배경에 따라 자동 선택
bat --theme-dark="Dracula" --theme-light="GitHub" file.py
```

---

## 실전 활용: 줄 범위, Git 통합, 스타일

> **현우**: 실무에서 자주 쓰는 기능 세 가지를 알려줄게. 첫째, **줄 범위 출력**.

```bash
bat -r 10:20 file.txt     # 10~20번째 줄만
bat -r :30 file.txt       # 처음~30번째 줄
bat -r -10: file.txt      # 마지막 10줄
bat -H 10:20 file.txt     # 10~20번째 줄 강조(하이라이트)
```

> **도현**: 코드 리뷰할 때 "150번째 줄 부근 봐봐" 하면 바로 볼 수 있겠네요!

> **현우**: 정확해! 둘째는 **Git 통합**이야. bat은 Git 저장소에서 파일을 볼 때 좌측에 변경 마크를 자동 표시해줘. `+`는 추가, `~`는 수정, `-`는 삭제.

```bash
bat -d file.py                # 변경된 줄만 표시
bat -d --diff-context=5 file.py    # 변경분 + 전후 5줄
```

> **도현**: git diff 안 치고도 어디가 바뀌었는지 보이는 거네요! 세 번째는 뭔가요?

> **현우**: **스타일 커스터마이징**이야. `--style`로 UI 요소를 조합할 수 있어.

```bash
bat --style=full file.txt           # 모든 요소
bat --style=plain file.txt          # 순수 내용만
bat --style=numbers,changes file.txt # 줄번호 + Git 변경
bat -p file.txt                     # plain 단축 (-pp는 페이저도 끔)
```

---

## 파이프 조합

> **현우**: bat의 진가는 다른 명령어랑 파이프로 조합할 때 나와. 자주 쓰는 것들이야.

```bash
# grep 결과를 구문 강조
grep -n "error" app.log | cat -l log

# JSON API 응답 확인
curl -s https://api.example.com/data | cat -l json

# diff 결과 색상 표시
diff -u old.py new.py | cat -l diff

# 클립보드 내용 구문 강조
pbpaste | cat -l python

# fzf 미리보기
fzf --preview 'bat --color=always --style=numbers --line-range=:500 {}'
```

> **도현**: fzf랑 조합하는 건 뭔가요?

> **현우**: fzf가 파일 검색할 때 미리보기 창을 띄우거든. 거기에 bat을 쓰면 구문 강조된 상태로 미리보기가 되는 거야. 진짜 편해.

> **도현**: 파이프로 보낼 때 색상이 사라지는 경우는 없나요?

> **현우**: 좋은 질문! bat은 파이프로 보내면 자동으로 색상을 끄거든. 유지하고 싶으면 `-f` 옵션을 써.

```bash
bat -f file.py | head -20
```

---

## 설정파일로 기본값 저장하기

> **현우**: 매번 옵션 입력하기 귀찮으면 설정파일에 저장할 수 있어.

```bash
mkdir -p ~/.config/bat
vim ~/.config/bat/config
```

> **도현**: 설정파일에 뭘 적으면 되나요?

> **현우**: 명령줄 옵션이랑 똑같은 형식이야. `#`으로 주석도 달 수 있고.

```bash
# 테마 설정
--theme="Dracula"

# 기본 스타일
--style="numbers,changes,grid"

# 파일 확장자 매핑
--map-syntax "*.conf:INI"
--map-syntax "*.env:Bourne Again Shell (bash)"
```

> **도현**: 환경변수로도 설정할 수 있다고 하셨죠?

> **현우**: 맞아, 주요 환경변수들 정리해보면 이래.

| 환경변수 | 설명 | 예시 |
|----------|------|------|
| BAT_THEME | 기본 테마 | export BAT_THEME="Dracula" |
| BAT_STYLE | 기본 스타일 | export BAT_STYLE="numbers,changes" |
| BAT_PAGER | 페이저 프로그램 | export BAT_PAGER="less -RF" |
| BAT_PAGING | 페이징 모드 | export BAT_PAGING="never" |

---

## 마무리

> **현우**: 정리하면, bat은 cat의 상위 호환이야. 구문 강조, 줄 번호, Git 통합, 테마 지원까지. alias 하나만 걸어두면 평소처럼 `cat` 쓰듯이 자연스럽게 쓸 수 있고.

> **도현**: 저도 당장 설치해야겠어요! 나중에 fzf 조합도 해봐야지.

> **도현**: 오늘 배운 거 정리하면... bat은 cat을 대체하는 도구고, 구문 강조에 줄 번호, Git 변경 표시까지 해주고, 183개 언어 지원, 28개 테마, 설정파일로 기본값 저장 가능!

> **현우**: 완벽해

---

## 핵심 정리

| 항목 | 내용 |
|------|------|
| bat이란? | cat의 상위 호환. 구문 강조 + Git 통합 + 줄 번호 |
| 설치 | brew install bat |
| alias 설정 | alias cat='bat --paging=never' |
| 언어 지정 | bat -l python file (또는 자동 감지) |
| 테마 변경 | bat --theme="Dracula" 또는 BAT_THEME 환경변수 |
| 줄 범위 | bat -r 10:20 file.txt |
| Git 변경 | bat -d file.py |
| 설정파일 | ~/.config/bat/config |
| 원본 cat | command cat 또는 alias 우회 호출 |
