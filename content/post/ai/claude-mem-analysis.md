---
title: "Claude-Mem 플러그인 분석 - 설치할까, 말까?"
date: 2026-02-06
categories:
- ai
tags:
- claude-mem
- claude-code
- plugin
- memory
- analysis
keywords:
- claude-mem
- claude code 메모리
- claude code 플러그인
author: "hyunwoo"
draft: false
---

Claude-Mem은 Claude Code의 모든 작업을 자동 캡처하고, 다음 세션에 관련 컨텍스트를 주입하는 메모리 플러그인입니다. 하지만 Windows에서 핵심 기능인 ChromaDB 벡터 검색이 비활성화 상태이고, 현재 CLAUDE.md + MEMORY.md 체계가 잘 동작 중이라면 설치를 보류하는 것이 합리적입니다.

<!--more-->

작성일: 2026-02-06 | 최신 버전: v9.0.16 | 라이선스: AGPL-3.0

---

## 이 분석이 필요한 이유

Claude Code를 사용하다 보면 이런 경험을 합니다:

- "지난 세션에서 어디까지 했더라?" 매번 다시 설명하는 번거로움
- 프로젝트가 복잡해질수록 컨텍스트가 부족해지는 느낌
- MEMORY.md에 수동으로 기록하는 게 귀찮음

Claude-Mem은 이 문제를 **자동화**로 해결하겠다는 플러그인입니다. 하지만 정말 설치할 가치가 있을까요? 현재 내장 메모리 시스템과 비교해서 판단해 봅시다.

---

## 1. Claude-Mem이란?

Claude-Mem은 Claude Code용 오픈소스 플러그인으로, 코딩 세션에서 Claude가 수행한 **모든 작업을 자동으로 캡처**하고, AI로 압축한 뒤, **다음 세션에 관련 컨텍스트를 자동 주입**하는 영구 메모리 시스템입니다.

- **개발자**: Alex Newman (@thedotmack)
- **GitHub**: [thedotmack/claude-mem](https://github.com/thedotmack/claude-mem)
- **공식 문서**: [docs.claude-mem.ai](http://docs.claude-mem.ai)
- 2026년 2월 3일 GitHub 트렌딩 1위 (24시간 만에 1,739 Stars)

> **쉽게 말하면?**
> Claude Code를 쓸 때 내가 한 작업(파일 수정, 검색, 명령 실행 등)을 백그라운드에서 몰래 다 기록해두고, 다음에 Claude Code를 열면 "지난번에 이런 작업 하셨죠?" 하면서 자동으로 알려주는 플러그인입니다.

---

## 2. 핵심 기능

| **기능** | **설명** | **신입 개발자를 위한 해설** |
|---------|---------|--------------------------|
| 자동 캡처 | 파일 읽기/쓰기/검색 등 모든 도구 사용을 자동 기록 | 내가 아무것도 안 해도 알아서 기록함 |
| AI 압축 | Agent SDK로 관찰 데이터를 시맨틱 요약으로 압축 | "파일 100개 수정" -> "인증 모듈 리팩토링" 수준으로 요약 |
| 3-Layer 검색 | search(인덱스) -> timeline(시간순) -> get_observations(상세) | 필요한 만큼만 단계적으로 불러와서 토큰 절약 |
| 토큰 절약 | Progressive Disclosure로 ~10x 토큰 절감 | 전체를 한번에 로드하지 않고 필요한 것만 로드 |
| 자연어 검색 | mem-search 스킬로 과거 작업 검색 | "지난주에 수정한 JSP 파일" 같은 검색 가능 |
| Web Viewer UI | localhost:37777에서 브라우저로 메모리 조회 | 브라우저에서 기록 내역을 시각적으로 확인 |
| Privacy 태그 | `<private>` 태그로 민감 데이터 제외 | 비밀번호, API 키 등은 기록에서 제외 가능 |
| Endless Mode (Beta) | ~95% 컨텍스트 윈도우 절감 | 장시간 세션에서도 컨텍스트가 부족해지지 않음 |

---

## 3. 아키텍처 (어떻게 동작하는가)

> **동작 원리 요약**
> Claude Code가 도구를 사용할 때마다 Hook이 발동 -> Worker 서비스(포트 37777)로 데이터 전송 -> AI가 요약 -> SQLite/ChromaDB에 저장 -> 다음 세션 시작 시 자동 주입

### 5개 Lifecycle Hook

| **Hook** | **이벤트** | **역할** |
|---------|----------|---------|
| `context-hook.js` | SessionStart | 이전 세션 컨텍스트 자동 주입 |
| `new-hook.js` | UserPromptSubmit | 세션 초기화 |
| `save-hook.js` | PostToolUse | 도구 사용 결과를 큐에 저장 |
| `summary-hook.js` | Stop | AI로 세션 요약 생성 |
| (SessionEnd) | SessionEnd | 세션 정리 |

> **Hook이란?** Claude Code의 특정 이벤트(세션 시작, 도구 사용 후 등)에 자동으로 실행되는 스크립트입니다. Claude-Mem은 5개의 Hook을 등록해서 Claude Code의 모든 생명주기에 개입합니다.

### AI Provider (3가지 선택 가능)

| **Provider** | **설명** | **비용** |
|-------------|---------|---------|
| **SDKAgent** (기본) | Claude Agent SDK | Claude Code 구독에 포함 |
| GeminiAgent | Google Gemini API | 별도 API 키 필요 |
| OpenRouterAgent | 100+ 모델 프록시 | 별도 API 키 필요 |

---

## 4. 설치 방법

### 요구사항

| **항목** | **요구 버전** | **비고** |
|---------|-------------|---------|
| Node.js | 18.0.0+ | 필수 |
| Claude Code | 최신 (플러그인 지원) | 필수 |
| Bun | 자동 설치 | JS 런타임 |
| uv (Python) | 자동 설치 | ChromaDB용 |
| SQLite 3 | 번들 포함 | 데이터 저장 |

### 설치 명령

```bash
/plugin marketplace add thedotmack/claude-mem
/plugin install claude-mem
# Claude Code 재시작 후 자동 동작
```

### 데이터 저장 위치

```
~/.claude-mem/
├── claude-mem.db       # SQLite 메인 DB
├── vector-db/          # ChromaDB 벡터 DB
├── settings.json       # 설정 파일 (자동 생성)
├── logs/               # 로그
└── claude-mem.pid      # 프로세스 ID
```

---

## 5. 내장 메모리 vs Claude-Mem 비교 (핵심)

### 현재 사용 중인 Claude Code 내장 메모리 시스템

```
[우선순위 높음 → 낮음]

1. 조직 정책       C:\Program Files\ClaudeCode\CLAUDE.md (미사용)
2. 프로젝트 메모리  CLAUDE.md (950+ 줄, 매우 상세)
3. 프로젝트 규칙    .claude/rules/*.md (미사용)
4. 사용자 메모리    ~/.claude-work/CLAUDE.md (PowerShell 규칙)
5. 프로젝트 로컬    CLAUDE.local.md (미사용)
6. Auto Memory     memory/MEMORY.md (프로젝트 진행 메모)
```

### 상세 비교표

| **비교 항목** | **내장 메모리 (현재)** | **Claude-Mem 플러그인** |
|-------------|---------------------|----------------------|
| 기록 방식 | 수동 (사용자/Claude가 직접 편집) | **자동 (모든 도구 사용 자동 캡처)** |
| 저장 형식 | Markdown 파일 | SQLite + ChromaDB |
| 검색 | 없음 (전체 로드) | **자연어 시맨틱 + 키워드 검색** |
| 토큰 사용 | 세션 시작 시 전체 로드 (고정 비용) | **Progressive Disclosure (~10x 절감)** |
| 세션 간 연속성 | MEMORY.md에 수동 기록한 것만 | **모든 작업 자동 기록 및 주입** |
| 버전 관리 | **Git으로 관리 가능** | DB 파일 (Git 불가) |
| 팀 공유 | **CLAUDE.md를 Git에 커밋** | 개인 로컬만 (공유 불가) |
| 편집 용이성 | **텍스트 편집기로 직접 편집** | Web UI 또는 API |
| 투명성 | **무엇이 로드되는지 명확** | 블랙박스 (주입 컨텍스트 불투명) |
| 추가 리소스 | **없음** | Worker 프로세스 (port 37777) |
| 설정 복잡도 | **없음 (파일만 작성)** | 설정 파일, 환경변수, 프로세스 관리 |
| 안정성 | **매우 안정 (단순 파일)** | 프로세스 크래시, 포트 충돌 가능 |

> **한 줄 요약**: Claude-Mem은 "자동화"와 "검색"에서 우위, 내장 메모리는 "안정성", "투명성", "팀 공유"에서 우위입니다.

---

## 6. Windows 환경 이슈 (중요)

> **현재 PC 환경**: Windows (MINGW32_NT-6.2)
> Windows에서 Claude-Mem을 사용할 때 알아야 할 주요 이슈들입니다.

| **이슈** | **상태** | **설명** |
|---------|---------|---------|
| 콘솔 팝업 | v9.0.6 수정 | 백그라운드 프로세스 생성 시 CMD 창이 뜨던 문제 |
| **ChromaDB 비활성화** | **일시적 비활성화** | Windows에서 벡터 검색 비활성화 (키워드 검색만 동작) |
| Worker 시작 실패 | v9.0.16 수정 | 15초 타임아웃 에러 해결 |
| PID 파일 경쟁 | v9.0.6 수정 | 프로세스 추적 안정성 개선 |
| WMIC 호환 | 수정됨 | Windows 11 WMIC 제거 대응 (PowerShell 대체) |
| 플러그인 비활성화 불완전 | **미해결 (#781)** | 비활성화 후에도 수동 개입 필요 |

> **Windows 핵심 제약**: ChromaDB(벡터 검색)가 비활성화되어 **시맨틱 검색 불가, 키워드 검색만 동작**합니다. 이는 Claude-Mem의 핵심 차별점인 "자연어 검색"이 Windows에서는 사실상 사용 불가능함을 의미합니다.

---

## 7. 장단점 종합

### 장점

1. **완전 자동화** -- 세션 기록이 자동으로 캡처/압축/주입
2. **토큰 효율** -- 3-Layer Progressive Disclosure로 필요한 것만 로드
3. **과거 작업 검색** -- 자연어로 "지난주에 수정한 파일" 같은 검색 가능
4. **세션 연속성** -- "어디까지 했었지?" 문제 해결
5. **Endless Mode (Beta)** -- 장시간 세션에서 컨텍스트 윈도우 절약

### 단점

1. **Windows 호환성 불완전** -- ChromaDB 비활성화, 과거 팝업 이슈
2. **백그라운드 프로세스** -- Worker 서비스가 항상 실행 (포트 37777)
3. **블랙박스** -- 어떤 컨텍스트가 주입되는지 직접 확인하기 어려움
4. **추가 의존성** -- Bun, uv(Python), SQLite, ChromaDB
5. **AGPL-3.0 라이선스** -- 상용 환경에서 제약 가능
6. **디스크 사용** -- SQLite DB + 벡터 DB가 지속적으로 증가
7. **안정성** -- 프로세스 크래시, 타임아웃 등 이슈 지속 발생
8. **내장 메모리와 중복** -- Auto Memory (MEMORY.md)와 역할 겹침

---

## 8. 현재 환경에 대한 판단

### 현재 잘 동작하고 있는 것

| **항목** | **현재 방식** | **평가** |
|---------|-------------|---------|
| 프로젝트 규칙 | CLAUDE.md (950+ 줄) | 매우 잘 정리됨 |
| 진행 상황 기록 | MEMORY.md (Auto Memory) | 수동이지만 정확 |
| 스킬/에이전트 | `.claude/skills/`, `.claude/agents/` | 11개 커스텀 스킬 |
| PowerShell 규칙 | 글로벌 CLAUDE.md | 환경 특화 규칙 |

### Claude-Mem이 추가로 해결할 수 있는 것

| **문제** | **Claude-Mem 해결** | **현재 대안** |
|---------|-------------------|-------------|
| "지난 세션에서 뭘 했지?" | 자동 기록 + 자동 주입 | MEMORY.md에 수동 기록 |
| 특정 과거 작업 검색 | 자연어 검색 | Grep/Read로 직접 검색 |
| 세션 시작 시 컨텍스트 복원 | 자동 | MEMORY.md가 시스템 프롬프트에 로드 |
| 장시간 세션 토큰 관리 | Endless Mode | 현재 대안 없음 |

---

## 9. 결론: 설치를 보류하자

> **현재 상황에서는 설치를 보류하는 것을 권장합니다**

### 보류 이유 4가지

1. **Windows 호환성 미성숙** -- ChromaDB(핵심 기능) 비활성화 상태. 키워드 검색만으로는 내장 메모리 대비 큰 이점 없음
2. **현재 메모리 체계가 충분** -- CLAUDE.md(950+ 줄) + MEMORY.md(Auto Memory)로 프로젝트 컨텍스트가 잘 관리되고 있음
3. **추가 복잡성 대비 이점 불명확** -- Worker 프로세스 관리, 포트 충돌, 프로세스 크래시 등의 리스크를 감수할 만한 이점이 현 시점에서는 부족
4. **안정성 우려** -- v9.0.x에서도 타임아웃, 인증 실패, 좀비 프로세스 등 버그가 계속 수정되고 있음 (빠른 릴리즈 주기 = 아직 안정화 중)

### 재검토 시점

- Windows에서 ChromaDB가 재활성화될 때
- v10.x 등 메이저 안정화 릴리즈 이후
- 프로젝트가 더 복잡해져서 MEMORY.md 수동 관리가 한계에 도달할 때
- Endless Mode가 정식 릴리즈될 때

---

## 10. 대안: 현재 메모리 체계 강화

> Claude-Mem 없이도 현재 내장 메모리를 더 효과적으로 활용할 수 있습니다.

### (1) `.claude/rules/` 디렉토리 활용

주제별로 규칙 파일을 분리하면 관리가 쉬워집니다:

```
.claude/rules/
├── database.md        # DB 관련 규칙
├── jsp-rules.md       # JSP 컴포넌트 규칙
├── security.md        # 보안 규칙
└── testing.md         # 테스트 규칙
```

### (2) CLAUDE.md @import 활용

외부 문서를 참조하여 CLAUDE.md를 깔끔하게 유지:

```markdown
# 프로젝트 규칙
- JSP 컴포넌트 규칙 @doc/JSP_COMPONENT_GUIDELINES.md
- 보안 가이드 @tips/guides/core/security_guide.md
```

### (3) MEMORY.md 구조화

프로젝트별, 주제별 섹션으로 더 세분화하여 활용도를 높일 수 있습니다.

### (4) CLAUDE.local.md 활용

개인 환경 설정 (DB 접속정보, 테스트 데이터 등)을 별도로 관리:

```markdown
# 나만의 설정
- 로컬 DB: localhost:3306/chemisys_dev
- 테스트 계정: test_user / test1234
```

---

## 참고 자료

- [Claude-Mem GitHub](https://github.com/thedotmack/claude-mem)
- [Claude-Mem 공식 문서](https://docs.claude-mem.ai/installation)
- [Claude-Mem DeepWiki 아키텍처](https://deepwiki.com/thedotmack/claude-mem)
- [Claude Code 내장 메모리 공식 문서](https://code.claude.com/docs/en/memory)
- [Claude-Mem CHANGELOG](https://github.com/thedotmack/claude-mem/blob/main/CHANGELOG.md)
- [Claude-Mem 리뷰 (byteiota)](https://byteiota.com/claude-mem-persistent-memory-for-claude-code/)
- [Top 10 Claude Code 플러그인 2026](https://www.firecrawl.dev/blog/best-claude-code-plugins)
- [Claude-Mem Windows 이슈 #367](https://github.com/thedotmack/claude-mem/issues/367)
- [Claude-Mem 비활성화 이슈 #781](https://github.com/thedotmack/claude-mem/issues/781)
