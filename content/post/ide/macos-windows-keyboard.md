---
title: "macOS에서 Windows 키보드 습관 유지하기"
date: 2026-02-14
categories:
- ide
tags:
- karabiner
- macos
- keyboard
- shortcut
keywords:
- Karabiner-Elements
- Shift Space 한영전환
- Home End 줄 이동
- Ctrl C V 복사 붙여넣기
author: "hyunwoo"
draft: false
---

Windows에서 macOS로 전환할 때 가장 불편한 세 가지 -- Shift+Space 한영 전환, Home/End 줄 이동, Ctrl+C/V 복사/붙여넣기 -- 를 Karabiner-Elements로 해결하는 방법을 정리합니다.

<!--more-->

---

## 오늘 다룰 내용

| 문제 | Windows 동작 | macOS 기본 동작 | 해결 |
|------|-------------|----------------|------|
| 한영 전환 | `Shift+Space` | 미지원 | Karabiner로 `Ctrl+Space`에 매핑 |
| Home 키 | 줄의 처음으로 | 페이지 맨 위로 | Karabiner로 `Cmd+Left`에 매핑 |
| End 키 | 줄의 끝으로 | 페이지 맨 아래로 | Karabiner로 `Cmd+Right`에 매핑 |
| 복사/붙여넣기 등 | `Ctrl+C/V/X/Z/A/S/F` | `Cmd+C/V/X/Z/A/S/F` | Karabiner로 `Ctrl` -> `Cmd`에 매핑 (터미널 제외) |

---

## 도입

> **현우**: 도현아, 맥북 받고 적응은 좀 됐어?

> **도현**: 대부분은 괜찮은데요... 한영 전환이 너무 불편해요. 윈도우에서는 `Shift+Space`로 전환했는데, 맥은 `Caps Lock`이나 `Ctrl+Space`라서 손이 꼬여요.

> **현우**: 나도 맥 처음 왔을 때 그거 때문에 미칠 뻔했어. 윈도우 10년 넘게 쓰다가 오면 진짜 적응이 안 되거든.

> **도현**: 방법이 없는 건가요?

> **현우**: 있어! **Karabiner-Elements**라는 프로그램을 쓰면 `Shift+Space`를 그대로 살릴 수 있어. 원리도 간단하고.

---

## Karabiner-Elements가 뭔데?

> **현우**: **Karabiner-Elements**는 macOS 전용 키 매핑 프로그램이야. 키보드 입력을 가로채서 다른 키로 바꿔주는 건데, 쉽게 말하면 "키보드 번역기"야.

> **도현**: 아, 그러면 `Shift+Space`를 누르면 macOS가 알아듣는 다른 키로 바꿔주는 건가요?

> **현우**: 정확해! 흐름을 보면 이래.

```
Shift+Space 입력
    -> (Karabiner-Elements가 가로채서 변환)
Control+Space 전달
    -> (macOS가 입력 소스 전환 단축키로 인식)
한영 전환 실행
```

> **도현**: 오 깔끔하네요. 설치는 어떻게 해요?

---

## 1단계: 설치

> **현우**: Homebrew로 한 줄이면 돼.

```bash
brew install --cask karabiner-elements
```

> **현우**: 설치할 때 sudo 비밀번호 입력하라고 나와. 입력하면 끝이야. 설치 후에 한 가지 중요한 게 있는데, **시스템 설정 -> 개인정보 보호 및 보안 -> 입력 모니터링**에서 Karabiner 권한을 허용해줘야 해.

> **도현**: 권한을 안 주면요?

> **현우**: 키 입력을 가로채는 프로그램이니까, 권한 없으면 아무것도 못 해. macOS 보안 정책상 필수야.

---

## 2단계: 키 매핑 설정 파일 만들기

> **현우**: 이제 Karabiner에게 "Shift+Space를 Control+Space로 바꿔라"는 규칙을 알려줘야 해. JSON 파일로 만들면 돼.

> **도현**: 어디에 만들어요?

> **현우**: 이 경로에 넣으면 Karabiner가 자동으로 인식해.

```bash
mkdir -p ~/.config/karabiner/assets/complex_modifications
```

> **현우**: 그리고 이 경로에 `shift_space_input_switch.json` 파일을 만들어.

```json
{
  "title": "Shift+Space로 한영 전환",
  "rules": [
    {
      "description": "Shift+Space로 한영 전환 (left_shift + space)",
      "manipulators": [
        {
          "type": "basic",
          "from": {
            "key_code": "spacebar",
            "modifiers": {
              "mandatory": ["left_shift"],
              "optional": ["any"]
            }
          },
          "to": [
            {
              "key_code": "spacebar",
              "modifiers": ["left_control"]
            }
          ]
        },
        {
          "type": "basic",
          "from": {
            "key_code": "spacebar",
            "modifiers": {
              "mandatory": ["right_shift"],
              "optional": ["any"]
            }
          },
          "to": [
            {
              "key_code": "spacebar",
              "modifiers": ["left_control"]
            }
          ]
        }
      ]
    }
  ]
}
```

> **도현**: `left_shift`랑 `right_shift` 둘 다 넣었네요?

> **현우**: 응, 왼쪽 Shift든 오른쪽 Shift든 상관없이 동작하게 한 거야. 하나만 넣으면 반대쪽 Shift에서는 안 되거든. 나도 처음에 left만 넣었다가 "왜 오른손으로는 안 돼?" 하고 한참 삽질했어.

---

## 3단계: Karabiner에서 규칙 활성화

> **현우**: 파일을 만들었으면 Karabiner에서 활성화해야 해.

> **도현**: 어떻게요?

> **현우**: 순서대로 해봐.

1. **Karabiner-Elements** 실행
2. **Complex Modifications** 탭 클릭
3. 좌측 하단 **Add rule** 클릭
4. **"Shift+Space로 한영 전환"** 항목이 보이면 **Enable** 클릭

> **도현**: 항목이 안 보이면요?

> **현우**: JSON 파일 경로가 잘못됐거나 JSON 문법 에러가 있는 거야. 경로가 `~/.config/karabiner/assets/complex_modifications/` 안에 있는지 다시 확인해봐.

---

## 4단계: macOS 입력 소스 단축키 설정

> **현우**: 마지막으로 macOS 쪽에서 `Control+Space`가 한영 전환으로 잡혀 있는지 확인해야 해.

> **도현**: 어디서 보면 돼요?

> **현우**: **시스템 설정 -> 키보드 -> 키보드 단축키 -> 입력 소스**에서 **"이전 입력 소스 선택"**이 `Ctrl+Space`(Control+Space)로 되어 있으면 끝이야.

> **도현**: 아, Karabiner가 Shift+Space를 Control+Space로 바꿔주고, macOS는 Control+Space를 한영 전환으로 인식하는 거군요!

> **현우**: 완벽하게 이해했네.

---

## 블루투스 키보드에서 안 될 때

> **현우**: 그런데 하나 함정이 있어. 맥북 자체 키보드에서는 되는데, **블루투스 키보드에서 안 되는 경우**가 있거든.

> **도현**: 왜요? 같은 키보드 아닌가요?

> **현우**: Karabiner는 키보드를 장치별로 따로 관리해. 블루투스 키보드는 별도 장치로 인식되기 때문에, 해당 장치에 대해 키 매핑을 허용해줘야 해.

> **도현**: 그럼 어떻게 해요?

> **현우**: Karabiner-Elements -> **Devices** 탭에 가면 연결된 키보드 목록이 보여. 거기서 블루투스 키보드 찾아서 **"Modify events"** 체크박스를 **ON**으로 켜주면 돼.

> **도현**: 그게 꺼져 있으면 Karabiner가 그 키보드 입력을 무시하는 거군요.

> **현우**: 맞아! 이거 모르면 "왜 내 키보드만 안 돼?" 하고 한참 헤매게 돼. 목록에 아예 안 보이면 블루투스 연결을 한번 끊었다 다시 연결하면 나타나.

---

## Home/End 키를 Windows처럼 줄 이동으로 변경하기

> **도현**: 선배, 그런데 하나 더 불편한 게 있어요. `Home`/`End` 키가 줄의 처음/끝으로 안 가고 페이지 맨 위/아래로 가버려요.

> **현우**: 아 그거! 나도 19년 동안 Home/End로 줄 이동 하다가 맥에서 페이지가 확 날아가니까 너무 당황스러웠어. macOS에서 줄의 처음/끝은 `Cmd+Left`/`Cmd+Right`거든.

> **도현**: 그러면 Home을 `Cmd+Left`으로 매핑하면 되는 건가요?

> **현우**: 정확해! 아까 한영 전환이랑 똑같은 원리야. Karabiner로 키를 번역해주는 거지. 설정 파일을 하나 더 만들면 돼.

---

### Home/End 설정 파일 만들기

> **현우**: 아까와 같은 경로에 `home_end_windows_style.json` 파일을 만들어.

```json
{
  "title": "Home/End를 Windows처럼 줄의 처음/끝으로 이동",
  "rules": [
    {
      "description": "Home -> 줄의 처음 (Cmd+Left), End -> 줄의 끝 (Cmd+Right)",
      "manipulators": [
        {
          "type": "basic",
          "from": {
            "key_code": "home",
            "modifiers": {
              "mandatory": ["shift"],
              "optional": ["any"]
            }
          },
          "to": [
            {
              "key_code": "left_arrow",
              "modifiers": ["left_command", "left_shift"]
            }
          ]
        },
        {
          "type": "basic",
          "from": {
            "key_code": "end",
            "modifiers": {
              "mandatory": ["shift"],
              "optional": ["any"]
            }
          },
          "to": [
            {
              "key_code": "right_arrow",
              "modifiers": ["left_command", "left_shift"]
            }
          ]
        },
        {
          "type": "basic",
          "from": {
            "key_code": "home"
          },
          "to": [
            {
              "key_code": "left_arrow",
              "modifiers": ["left_command"]
            }
          ]
        },
        {
          "type": "basic",
          "from": {
            "key_code": "end"
          },
          "to": [
            {
              "key_code": "right_arrow",
              "modifiers": ["left_command"]
            }
          ]
        }
      ]
    }
  ]
}
```

> **도현**: `Shift+Home`이랑 `Shift+End`도 있네요?

> **현우**: 응, 텍스트 선택할 때 쓰잖아. `Shift+Home`은 커서에서 줄의 처음까지 선택, `Shift+End`는 줄의 끝까지 선택. 이것도 Windows랑 똑같이 동작하게 매핑한 거야.

> **도현**: 오, 세심하네요. 이거 없으면 코드 작성할 때 진짜 불편하겠다.

---

### Home/End 규칙 활성화

> **현우**: 아까와 똑같은 방법이야.

1. **Karabiner-Elements** -> **Complex Modifications** 탭
2. **Add rule** 클릭
3. **"Home -> 줄의 처음 (Cmd+Left), End -> 줄의 끝 (Cmd+Right)"** -> **Enable**

> **도현**: 와 진짜 되네요! Home/End가 줄 이동으로 바뀌니까 살 것 같아요.

> **현우**: 맞아. 이 두 개만 설정하면 Windows에서 불편 없이 맥을 쓸 수 있어.

### Home/End 매핑 결과

| 키 입력 | macOS 기본 동작 | 변경 후 (Windows 동작) |
|---------|----------------|----------------------|
| `Home` | 페이지 맨 위로 | 줄의 처음으로 |
| `End` | 페이지 맨 아래로 | 줄의 끝으로 |
| `Shift+Home` | 페이지 맨 위까지 선택 | 줄의 처음까지 선택 |
| `Shift+End` | 페이지 맨 아래까지 선택 | 줄의 끝까지 선택 |

---

## Ctrl+C/V를 Windows처럼 복사/붙여넣기로 사용하기

> **도현**: 선배, 이거 하나만 더요... `Ctrl+C`, `Ctrl+V`로 복사/붙여넣기 하고 싶은데, 맥은 `Cmd+C`, `Cmd+V`라서 손이 자꾸 헷갈려요.

> **현우**: 그거 진짜 Windows 사용자들 맥 오면 다 겪는 관문이야. 나도 처음에 복사한다고 `Ctrl+C` 누르니까 아무 것도 안 되서 당황했어.

> **도현**: 이것도 Karabiner로 해결할 수 있어요?

> **현우**: 당연하지! `Ctrl+C`를 `Cmd+C`로 바꿔주면 돼. 근데 여기서 **하나 중요한 주의사항**이 있어.

> **도현**: 뭐요?

> **현우**: **터미널에서 `Ctrl+C`는 프로세스 중단** 명령이거든. 이걸 복사로 바꿔버리면 터미널에서 실행 중인 프로그램을 끝낼 수가 없어져. 그래서 터미널 앱만 제외하고 매핑해야 해.

> **도현**: 아, 그렇군요! 그거 모르고 전체 적용했으면 큰일 날 뻔했네요.

> **현우**: 맞아. 이런 게 경험에서 나오는 노하우지.

---

### Ctrl+C/V 설정 파일 만들기

> **현우**: 같은 경로에 `ctrl_copy_paste_windows_style.json` 파일을 만들어. 핵심은 `conditions`에서 터미널 앱을 제외하는 부분이야.

```json
{
  "title": "Ctrl+C/V/X/Z/A/S/F를 Windows처럼 사용 (터미널 제외)",
  "rules": [
    {
      "description": "Ctrl+C -> Cmd+C, Ctrl+V -> Cmd+V 등 Windows 단축키 (터미널 제외)",
      "manipulators": [
        {
          "type": "basic",
          "from": { "key_code": "c", "modifiers": { "mandatory": ["control"], "optional": ["shift"] } },
          "to": [{ "key_code": "c", "modifiers": ["left_command"] }],
          "conditions": [{ "type": "frontmost_application_unless", "bundle_identifiers": ["^com\\.apple\\.Terminal$", "^com\\.googlecode\\.iterm2$", "^io\\.alacritty$", "^com\\.mitchellh\\.ghostty$"] }]
        },
        {
          "type": "basic",
          "from": { "key_code": "v", "modifiers": { "mandatory": ["control"], "optional": ["shift"] } },
          "to": [{ "key_code": "v", "modifiers": ["left_command"] }],
          "conditions": [{ "type": "frontmost_application_unless", "bundle_identifiers": ["^com\\.apple\\.Terminal$", "^com\\.googlecode\\.iterm2$", "^io\\.alacritty$", "^com\\.mitchellh\\.ghostty$"] }]
        },
        {
          "type": "basic",
          "from": { "key_code": "x", "modifiers": { "mandatory": ["control"], "optional": ["shift"] } },
          "to": [{ "key_code": "x", "modifiers": ["left_command"] }],
          "conditions": [{ "type": "frontmost_application_unless", "bundle_identifiers": ["^com\\.apple\\.Terminal$", "^com\\.googlecode\\.iterm2$", "^io\\.alacritty$", "^com\\.mitchellh\\.ghostty$"] }]
        },
        {
          "type": "basic",
          "from": { "key_code": "z", "modifiers": { "mandatory": ["control"], "optional": ["shift"] } },
          "to": [{ "key_code": "z", "modifiers": ["left_command"] }],
          "conditions": [{ "type": "frontmost_application_unless", "bundle_identifiers": ["^com\\.apple\\.Terminal$", "^com\\.googlecode\\.iterm2$", "^io\\.alacritty$", "^com\\.mitchellh\\.ghostty$"] }]
        },
        {
          "type": "basic",
          "from": { "key_code": "a", "modifiers": { "mandatory": ["control"], "optional": ["shift"] } },
          "to": [{ "key_code": "a", "modifiers": ["left_command"] }],
          "conditions": [{ "type": "frontmost_application_unless", "bundle_identifiers": ["^com\\.apple\\.Terminal$", "^com\\.googlecode\\.iterm2$", "^io\\.alacritty$", "^com\\.mitchellh\\.ghostty$"] }]
        },
        {
          "type": "basic",
          "from": { "key_code": "s", "modifiers": { "mandatory": ["control"], "optional": ["shift"] } },
          "to": [{ "key_code": "s", "modifiers": ["left_command"] }],
          "conditions": [{ "type": "frontmost_application_unless", "bundle_identifiers": ["^com\\.apple\\.Terminal$", "^com\\.googlecode\\.iterm2$", "^io\\.alacritty$", "^com\\.mitchellh\\.ghostty$"] }]
        },
        {
          "type": "basic",
          "from": { "key_code": "f", "modifiers": { "mandatory": ["control"], "optional": ["shift"] } },
          "to": [{ "key_code": "f", "modifiers": ["left_command"] }],
          "conditions": [{ "type": "frontmost_application_unless", "bundle_identifiers": ["^com\\.apple\\.Terminal$", "^com\\.googlecode\\.iterm2$", "^io\\.alacritty$", "^com\\.mitchellh\\.ghostty$"] }]
        }
      ]
    }
  ]
}
```

> **도현**: `frontmost_application_unless`가 "이 앱이 앞에 있을 때는 적용하지 마라"는 뜻인 거죠?

> **현우**: 정확해! Terminal, iTerm2, Alacritty, Ghostty 네 개 터미널 앱을 제외했어. 이 앱들에서는 `Ctrl+C`가 원래대로 프로세스 중단으로 동작해.

---

### Ctrl+C/V 규칙 활성화

> **현우**: 똑같은 방법이야.

1. **Karabiner-Elements** -> **Complex Modifications** 탭
2. **Add rule** 클릭
3. **"Ctrl+C -> Cmd+C, Ctrl+V -> Cmd+V 등 Windows 단축키 (터미널 제외)"** -> **Enable**

> **도현**: 와, `Ctrl+C`로 복사되니까 진짜 Windows 쓰는 것 같아요!

### Ctrl+C/V 매핑 결과

| Windows 단축키 | macOS 변환 | 기능 |
|---------------|-----------|------|
| `Ctrl+C` | `Cmd+C` | 복사 |
| `Ctrl+V` | `Cmd+V` | 붙여넣기 |
| `Ctrl+X` | `Cmd+X` | 잘라내기 |
| `Ctrl+Z` | `Cmd+Z` | 실행취소 |
| `Ctrl+A` | `Cmd+A` | 전체선택 |
| `Ctrl+S` | `Cmd+S` | 저장 |
| `Ctrl+F` | `Cmd+F` | 찾기 |

> **현우**: 터미널(Terminal, iTerm2, Alacritty, Ghostty)에서는 기존 `Ctrl+C`(프로세스 중단) 동작이 그대로 유지돼.

---

## 보너스: IntelliJ 단축키 충돌 해결

> **도현**: 선배, 그런데 IntelliJ에서 단축키 충돌 경고가 엄청 많이 뜨는데 이것도 관련 있어요?

> **현우**: 아 그거! macOS 시스템 단축키가 IntelliJ 단축키를 가로채는 거야. 한영 전환이랑은 별개 문제인데, 같이 해결하면 편하지.

> **도현**: 어떻게 하면 돼요?

> **현우**: **시스템 설정 -> 키보드 -> 키보드 단축키**에서 안 쓰는 것들 꺼버려. 특히 이것들이 주범이야.

| 항목 | 충돌 단축키 | 조치 |
|------|-----------|------|
| Mission Control | `Ctrl+Up` `Ctrl+Down` `Ctrl+Left` `Ctrl+Right` | 체크 해제 |
| Mission Control | `Ctrl+1` `Ctrl+2` `Ctrl+3` ... | 체크 해제 |
| 입력 소스 | `Ctrl+Option+Space` | 체크 해제 |
| 서비스 | 각종 숨은 단축키 | 전부 체크 해제 |

> **도현**: 서비스 안에 숨은 단축키가 있었군요. 몰랐어요.

> **현우**: 그게 제일 찾기 힘들어. 서비스 항목 펼쳐보면 의외로 많이 잡혀있거든. 다 끄는 게 편해.

---

## 핵심 정리

| 단계 | 내용 |
|------|------|
| 설치 | `brew install --cask karabiner-elements` |
| 한영 전환 설정 | `shift_space_input_switch.json` 생성 -> Enable |
| Home/End 설정 | `home_end_windows_style.json` 생성 -> Enable |
| Ctrl+C/V 설정 | `ctrl_copy_paste_windows_style.json` 생성 -> Enable (터미널 제외) |
| macOS 설정 | 입력 소스 단축키 -> `Ctrl+Space` 확인 |
| 블루투스 키보드 | Devices 탭 -> 해당 키보드 Modify events **ON** |
| 한영 전환 원리 | `Shift+Space` -> (Karabiner) -> `Ctrl+Space` -> (macOS) -> 한영 전환 |
| Home/End 원리 | `Home` -> `Cmd+Left` -> 줄의 처음 / `End` -> `Cmd+Right` -> 줄의 끝 |
| Ctrl+C/V 원리 | `Ctrl+C` -> `Cmd+C` (복사) / `Ctrl+V` -> `Cmd+V` (붙여넣기) 등 7개 단축키 |

## 참고 자료

- [Karabiner-Elements 공식 사이트](https://karabiner-elements.pqrs.org/)
- 설정 파일 위치: `~/.config/karabiner/assets/complex_modifications/`
- 환경: macOS (Apple Silicon), Karabiner-Elements 15.9.0
