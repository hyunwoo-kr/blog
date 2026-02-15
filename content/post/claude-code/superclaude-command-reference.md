---
title: "SuperClaude 명령어 완벽 가이드 - /sc 명령어 레퍼런스"
date: 2026-01-28
categories:
- claude-code
tags:
- superclaude
- claude-code
- slash-commands
- reference
keywords:
- superclaude 명령어
- sc 명령어
- claude code 슬래시 명령어
author: "hyunwoo"
draft: false
---

SuperClaude는 Claude Code에서 사용할 수 있는 `/sc:` 접두사 명령어 모음으로, 코드 분석부터 구현, 테스트, 문서화까지 개발 워크플로우 전체를 지원합니다. 이 가이드에서는 모든 `/sc:` 명령어를 카테고리별로 정리하고, 언제 어떤 명령어를 써야 하는지 실전 예시와 함께 설명합니다.

<!--more-->

> **TL;DR (3줄 요약)**
> - SuperClaude는 Claude Code에서 사용할 수 있는 `/sc:` 접두사 명령어 모음으로, 코드 분석부터 구현, 테스트, 문서화까지 개발 워크플로우 전체를 지원합니다.
> - 상황에 맞는 명령어를 조합하면 생산성이 크게 향상됩니다. (예: 새 기능 개발 = brainstorm -> design -> implement -> test)
> - 이 가이드를 북마크 해두고, 필요한 명령어를 빠르게 찾아 사용하세요.

---

## 이 가이드가 필요한 이유

Claude Code를 사용하다 보면 "어떤 명령어를 써야 하지?" 하는 순간이 옵니다. `/sc:` 명령어는 **20개 이상**의 전문 명령어를 제공하는데, 각각의 용도와 사용 시점을 알아두면 작업 효율이 크게 달라집니다.

이 가이드에서는 모든 `/sc:` 명령어를 **카테고리별로 정리**하고, **언제 어떤 명령어를 써야 하는지** 실전 예시와 함께 설명합니다.

---

## 1. 탐색/분석 명령어

코드를 이해하고 분석하는 데 사용하는 명령어들입니다. "이 코드가 뭐하는 거지?", "품질은 괜찮은가?" 싶을 때 쓰세요.

### `/sc:analyze` -- 코드 종합 분석

> **한 줄 설명**: 코드의 품질, 보안, 성능, 아키텍처를 한 번에 분석합니다.

**언제 쓰나요?**

- PR 올리기 전 자체 코드 리뷰
- 레거시 코드의 현재 상태 파악
- 리팩토링 전 문제점 파악
- 보안 취약점 점검

**사용 예시:**

```
/sc:analyze src/main/java/egovframework/cms/biocide/
/sc:analyze MyController.java
/sc:analyze                    # 현재 디렉토리 전체 분석
```

**출력 내용:** 코드 품질 점수, 보안 취약점, 성능 최적화 제안, 아키텍처 개선 권장사항

---

### `/sc:explain` -- 코드/개념 설명

> **한 줄 설명**: 복잡한 코드나 개념을 교육적 관점에서 쉽게 설명해줍니다.

**언제 쓰나요?**

- 처음 보는 레거시 코드를 분석할 때
- 특정 메서드의 동작 원리가 궁금할 때
- 프레임워크 개념을 이해하고 싶을 때

**사용 예시:**

```
/sc:explain BiocideController.java
/sc:explain "이 프로젝트의 인증 시스템이 어떻게 동작하는지"
/sc:explain Spring Security의 FilterChain
/sc:explain getRequestToHashMap 메서드
```

---

### `/sc:index-repo` -- 저장소 인덱싱 (94% 토큰 절감)

> **한 줄 설명**: 대규모 코드베이스의 전체 구조를 빠르게 파악할 수 있는 인덱스를 생성합니다.

**언제 쓰나요?**

- 새 프로젝트에 투입되었을 때 (첫 번째로 실행 추천!)
- 프로젝트 전체 구조를 한눈에 보고 싶을 때

**사용 예시:**

```
/sc:index-repo
```

> **팁**: 새 프로젝트 시작 시 가장 먼저 실행하면 좋습니다. 이후 `/sc:explain`과 조합하면 코드 이해가 훨씬 빨라집니다.

---

### `/sc:index` -- 프로젝트 문서화 및 지식 베이스 생성

> **한 줄 설명**: 프로젝트 전체를 문서화하고 지식 베이스를 구축합니다.

**사용 예시:**

```
/sc:index
/sc:index src/main/java/egovframework/cms/
```

---

## 2. 설계/계획 명령어

코드를 작성하기 **전에** 사용하는 명령어들입니다. "어떻게 만들지?", "뭘 먼저 해야 하지?" 싶을 때 쓰세요.

### `/sc:brainstorm` -- 요구사항 발굴 (소크라테스식 대화)

> **한 줄 설명**: 질문과 대화를 통해 숨겨진 요구사항을 발굴하고 아이디어를 구체화합니다.

**언제 쓰나요?**

- "이 기능을 만들어야 하는데 뭘 고려해야 할지 모르겠어" 싶을 때
- 요구사항이 막연할 때
- 여러 접근 방식을 비교하고 싶을 때

**사용 예시:**

```
/sc:brainstorm 사용자 인증 시스템 개선
/sc:brainstorm "화학물질 승인 워크플로우 자동화"
/sc:brainstorm 파일 업로드 기능 추가
```

---

### `/sc:design` -- 시스템/API/컴포넌트 설계

> **한 줄 설명**: 아키텍처, API, 데이터 모델 등을 체계적으로 설계합니다.

**사용 예시:**

```
/sc:design 화학제품 승인 API
/sc:design "사용자 권한 관리 모듈"
/sc:design 파일 업로드 서비스 아키텍처
```

**출력 내용:** 아키텍처 설명, API 명세, 컴포넌트 인터페이스, 데이터 흐름

---

### `/sc:workflow` -- 구현 워크플로우 생성

> **한 줄 설명**: 요구사항을 구체적인 구현 단계로 분해하고 작업 순서를 계획합니다.

**사용 예시:**

```
/sc:workflow "제품 목록 검색 기능 구현"
/sc:workflow 파일에서 요구사항 읽어서 워크플로우 생성
```

---

### `/sc:estimate` -- 개발 공수 산정

> **한 줄 설명**: 작업의 복잡도를 분석하고 예상 공수를 산정합니다.

**사용 예시:**

```
/sc:estimate "게시판 모듈 전체 리팩토링"
/sc:estimate 사용자 인증 시스템 구현
```

---

## 3. 구현 명령어

실제 코드를 작성하고 개선하는 명령어들입니다. "코드를 짜줘", "이거 개선해줘" 할 때 쓰세요.

### `/sc:implement` -- 기능/코드 구현

> **한 줄 설명**: 새 기능을 구현하거나 기존 코드를 수정합니다. 프로젝트 패턴을 자동으로 따릅니다.

**사용 예시:**

```
/sc:implement "제품 검색 API 엔드포인트 추가"
/sc:implement CommonController에 페이징 헬퍼 메서드 추가
/sc:implement 파일 다운로드 시 한글 파일명 깨짐 수정
```

---

### `/sc:improve` -- 코드 품질/성능 개선

> **한 줄 설명**: 기존 코드를 리팩토링하거나 성능을 최적화합니다.

**사용 예시:**

```
/sc:improve BiocideService.java
/sc:improve "이 쿼리의 성능을 개선해줘"
/sc:improve src/main/java/egovframework/cms/board/
```

---

### `/sc:cleanup` -- 데드 코드 제거, 구조 정리

> **한 줄 설명**: 사용하지 않는 코드를 제거하고 프로젝트 구조를 정리합니다.

**사용 예시:**

```
/sc:cleanup
/sc:cleanup src/main/java/egovframework/cms/legacy/
```

---

## 4. 테스트/검증 명령어

구현한 코드를 검증하고 문제를 해결하는 명령어들입니다.

### `/sc:test` -- 테스트 실행 및 커버리지 분석

> **한 줄 설명**: 테스트를 실행하고 결과를 분석합니다.

**사용 예시:**

```
/sc:test
/sc:test BiocideServiceTest
/sc:test --coverage
```

---

### `/sc:reflect` -- 작업 자가 검증

> **한 줄 설명**: 완료된 작업을 자체 검토하고 품질을 확인합니다.

**사용 예시:**

```
/sc:reflect "방금 구현한 검색 기능"
/sc:reflect
```

---

### `/sc:troubleshoot` -- 문제 진단 및 해결

> **한 줄 설명**: 오류 원인을 분석하고 해결책을 제시합니다.

**언제 쓰나요?**

- 에러 메시지가 떴는데 원인을 모를 때
- 빌드가 안 될 때
- 런타임에 예상과 다르게 동작할 때

**사용 예시:**

```
/sc:troubleshoot "NullPointerException at line 245"
/sc:troubleshoot "빌드할 때 메모리 부족 오류 발생"
/sc:troubleshoot 로그인 후 세션이 유지되지 않는 문제
```

---

## 5. 문서화 명령어

### `/sc:document` -- 코드/API 문서 생성

> **한 줄 설명**: 코드, API, 기능에 대한 문서를 자동 생성합니다.

**파라미터:**

| 파라미터 | 설명 | 기본값 |
|---------|------|--------|
| `--type` | inline, external, api, guide | external |
| `--style` | brief, detailed | detailed |

**사용 예시:**

```
/sc:document BiocideController --type api
/sc:document src/main/java/egovframework/cms/member/ --type external
/sc:document 인증 시스템 --type guide --style detailed
```

---

### `/sc:research` -- 심층 웹 리서치

> **한 줄 설명**: 기술 관련 주제를 웹에서 조사하여 최신 정보를 제공합니다.

**사용 예시:**

```
/sc:research "Spring Security 6.0 마이그레이션 방법"
/sc:research Java 11에서 17로 업그레이드 시 주의사항
/sc:research "MariaDB 성능 튜닝 베스트 프랙티스"
```

---

## 6. 오케스트레이션 명령어

대규모 작업을 분해하고 관리하는 명령어들입니다. 복잡한 작업일수록 이 명령어들이 빛을 발합니다.

### `/sc:pm` -- 프로젝트 매니저 에이전트

> **한 줄 설명**: 복잡한 다단계 작업을 조율하고 서브 에이전트를 관리합니다.

**사용 예시:**

```
/sc:pm "전체 인증 시스템 리팩토링"
/sc:pm 모듈 A, B, C 동시 업데이트
```

---

### `/sc:spawn` -- 작업 분해 및 병렬 위임

> **한 줄 설명**: 대규모 작업을 분해하여 병렬로 처리합니다.

**사용 예시:**

```
/sc:spawn "10개 모듈 각각에 대해 단위 테스트 작성"
```

---

### `/sc:task` -- 복합 작업 실행

> **한 줄 설명**: 복합적인 작업을 실행하고 워크플로우를 관리합니다.

**사용 예시:**

```
/sc:task "API 엔드포인트 추가 및 테스트 작성"
```

---

## 7. 유틸리티 명령어

### `/sc:help` -- 명령어 목록 보기

```
/sc:help
/sc:help analyze    # 특정 명령어 도움말
```

### `/sc:recommend` -- 상황별 명령어 추천

어떤 명령어를 써야 할지 모르겠을 때 사용하세요.

```
/sc:recommend 이 코드의 성능을 개선하고 싶어
/sc:recommend 새로운 기능을 추가해야 해
/sc:recommend 버그를 찾아서 고치고 싶어
```

### `/sc:git` -- Git 작업 (지능형 커밋)

```
/sc:git commit
/sc:git "변경사항 커밋해줘"
```

### `/sc:build` -- 빌드/컴파일/패키징

```
/sc:build
/sc:build --clean
```

---

## 8. 세션 관리 명령어

### `/sc:load` -- 프로젝트 컨텍스트 로드

세션 시작 시 또는 프로젝트 전환 시 사용합니다.

```
/sc:load
```

### `/sc:save` -- 세션 컨텍스트 저장

세션 종료 전이나 중요한 진행점에서 사용합니다.

```
/sc:save
```

---

## 9. 전문 리뷰 명령어

### `/sc:business-panel` -- 비즈니스 관점 분석

```
/sc:business-panel "화학제품 승인 프로세스"
```

### `/sc:spec-panel` -- 다중 전문가 스펙 리뷰

```
/sc:spec-panel "API 스펙 문서"
/sc:spec-panel requirements.md
```

---

## 실전 조합 가이드

> **상황별 추천 조합** -- 이 조합만 알면 대부분의 개발 워크플로우를 커버할 수 있습니다.

| **상황** | **추천 명령어 조합** |
|---------|---------------------|
| 새 기능 개발 | `brainstorm` -> `design` -> `implement` -> `test` |
| 버그 수정 | `explain` -> `troubleshoot` -> `implement` -> `test` |
| 코드 리팩토링 | `analyze` -> `improve` -> `cleanup` -> `test` |
| 코드 이해 | `index-repo` -> `explain` |
| 문서화 | `analyze` -> `document` |

---

## 자주 하는 실수 & 해결법

> **실수 1: 바로 `/sc:implement`부터 시작하기**
> 복잡한 기능은 `brainstorm` -> `design`을 먼저 거치면 구현 시 시행착오가 줄어듭니다.

> **실수 2: 에러 발생 시 바로 코드를 수정하기**
> `/sc:troubleshoot`를 먼저 사용하면 근본 원인을 파악할 수 있어 같은 문제가 반복되지 않습니다.

> **실수 3: 어떤 명령어를 써야 할지 모를 때 고민하기**
> `/sc:recommend`에 하고 싶은 작업을 자연어로 설명하면 적절한 명령어를 추천해줍니다.

---

> **다음 단계**: SuperClaude 워크플로우 가이드를 통해 더 복잡한 시나리오별 사용법을 익혀보세요.
