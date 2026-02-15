---
title: "SuperClaude 워크플로우 실전 가이드 - 상황별 /sc 명령어 조합법"
date: 2026-01-28
categories:
- ai
tags:
- superclaude
- claude-code
- workflow
- best-practices
keywords:
- superclaude 워크플로우
- sc 명령어 조합
- 개발 워크플로우
author: "hyunwoo"
draft: false
---

SuperClaude 명령어는 단독으로도 강력하지만, 상황에 맞게 조합하면 훨씬 효과적입니다. 이 가이드에서는 새 기능 개발, 버그 수정, 리팩토링 등 10가지 실전 시나리오별로 최적의 명령어 조합을 정리했습니다.

<!--more-->

> **TL;DR (3줄 요약)**
> - SuperClaude 명령어는 단독으로도 강력하지만, **상황에 맞게 조합**하면 훨씬 효과적입니다.
> - 새 기능 개발, 버그 수정, 리팩토링 등 10가지 실전 시나리오별로 최적의 명령어 조합을 정리했습니다.
> - "뭘 쓸지 모르겠으면" -> 페이지 맨 아래 **빠른 선택 가이드** 표를 확인하세요.

---

## 이 가이드가 필요한 이유

`/sc:implement`만 알고 있다면, SuperClaude의 10%만 활용하는 것입니다. 실제 개발은 "분석 -> 설계 -> 구현 -> 검증" 흐름으로 이루어지는데, 각 단계에 맞는 명령어를 조합하면 **작업 품질과 속도가 모두 향상**됩니다.

### 워크플로우 표기법

```
/sc:명령어1 → /sc:명령어2 → /sc:명령어3
```

- `→` : 순서대로 실행
- `⟷` : 반복 가능
- `[ ]` : 선택적 실행 (필요할 때만)

---

## 1. 새 기능 개발 워크플로우

> **흐름**: `brainstorm` -> `design` -> `workflow` -> `implement` -> `test` -> [`document`]

### 단계별 설명

**Step 1: 요구사항 발굴** (`brainstorm`)

기능의 범위를 정의하고, 엣지 케이스를 발굴합니다.

```
/sc:brainstorm 사용자가 화학제품을 등록할 때 첨부파일을 업로드할 수 있는 기능
```

**Step 2: 설계** (`design`)

API 인터페이스, 데이터 모델, 컴포넌트 구조를 설계합니다.

```
/sc:design 화학제품 첨부파일 업로드 API
```

**Step 3: 구현 계획** (`workflow`)

구현 단계를 분해하고 의존성을 파악합니다.

```
/sc:workflow 파일 업로드 기능 구현
```

**Step 4: 구현** (`implement`)

실제 코드를 작성합니다.

```
/sc:implement FileUploadController에 화학제품 첨부파일 업로드 엔드포인트 추가
```

**Step 5: 테스트** (`test`)

단위/통합 테스트로 검증합니다.

```
/sc:test FileUploadController
```

**Step 6: 문서화** (`document`) [선택]

```
/sc:document FileUploadController --type api
```

### 실전 예시: 게시판 댓글 기능 추가

```
# 1. 요구사항 논의
/sc:brainstorm 게시판에 댓글 기능을 추가하려고 해. 대댓글도 지원해야 해.

# 2. 설계
/sc:design 게시판 댓글/대댓글 시스템

# 3. 구현 계획
/sc:workflow 댓글 기능 구현

# 4. 구현 (여러 단계로 나눠서)
/sc:implement 댓글 테이블 생성 SQL
/sc:implement 댓글 Service 클래스 작성
/sc:implement 댓글 Controller 작성
/sc:implement 댓글 JSP 화면 작성

# 5. 테스트
/sc:test BoardCommentService

# 6. 문서화
/sc:document 댓글 API --type api
```

---

## 2. 버그 수정 워크플로우

> **흐름**: `explain` -> `troubleshoot` -> `implement` -> `test`

### 단계별 설명

**Step 1: 현황 파악** (`explain`)

문제가 발생한 코드를 먼저 이해합니다.

```
/sc:explain BiocideController의 getList 메서드
```

**Step 2: 문제 진단** (`troubleshoot`)

근본 원인을 분석하고 해결책을 제시받습니다.

```
/sc:troubleshoot "검색 결과가 0건으로 나오는 문제"
```

**Step 3: 수정** (`implement`)

```
/sc:implement 검색 조건 null 체크 추가
```

**Step 4: 검증** (`test`)

```
/sc:test BiocideServiceTest
```

### 실전 예시: 로그인 세션 유지 버그

```
# 1. 관련 코드 이해
/sc:explain LoginController의 로그인 처리 로직

# 2. 문제 진단
/sc:troubleshoot "로그인 후 다른 페이지로 이동하면 세션이 끊어지는 문제"

# 3. 수정
/sc:implement 세션 쿠키 설정 수정

# 4. 테스트
/sc:test 로그인 기능
```

> **자주 하는 실수**: 바로 `implement`부터 시작하지 마세요! `explain` -> `troubleshoot`를 먼저 거치면 근본 원인을 파악할 수 있어 같은 버그가 반복되지 않습니다.

---

## 3. 코드 리팩토링 워크플로우

> **흐름**: `analyze` -> `improve` <-> `cleanup` -> `test`
> `improve`와 `cleanup`은 필요에 따라 반복할 수 있습니다.

### 단계별 설명

**Step 1: 현황 분석** (`analyze`)

코드 품질 점수를 확인하고 개선 영역을 파악합니다.

```
/sc:analyze src/main/java/egovframework/cms/biocide/
```

**Step 2: 코드 개선** (`improve`)

리팩토링, 성능 최적화, 가독성 향상을 수행합니다.

```
/sc:improve BiocideService.java
```

**Step 3: 정리** (`cleanup`)

데드 코드 제거, 불필요한 의존성 제거를 수행합니다.

```
/sc:cleanup
```

**Step 4: 회귀 테스트** (`test`)

기존 기능이 정상 동작하는지 확인합니다.

```
/sc:test
```

### 실전 예시: 게시판 모듈 리팩토링

```
# 1. 현황 파악
/sc:analyze src/main/java/egovframework/cms/board/

# 2. 분석 결과 확인 후 개선 (반복)
/sc:improve BoardService.java
/sc:improve BoardController.java

# 3. 정리
/sc:cleanup src/main/java/egovframework/cms/board/

# 4. 테스트
/sc:test BoardServiceTest
```

---

## 4. 코드 리뷰 워크플로우

> **흐름**: `analyze` -> `explain` -> [`improve`] -> `document`

1. `analyze` -- 코드 품질 검토, 잠재적 문제 식별
2. `explain` -- 변경 내용 이해, 영향 범위 파악
3. `improve` -- 리뷰 중 발견된 이슈 수정 (선택)
4. `document` -- 리뷰 결과 정리

---

## 5. 기술 조사 워크플로우

> **흐름**: `research` -> `brainstorm` -> `design`

1. `research` -- 기술 트렌드 조사, 장단점 비교
2. `brainstorm` -- 프로젝트 상황에 맞게 조정, 리스크 평가
3. `design` -- 구체적인 적용 계획 수립

### 실전 예시: 캐시 시스템 도입 검토

```
# 1. 기술 조사
/sc:research "Spring Cache vs Redis vs Caffeine 비교"

# 2. 적용 논의
/sc:brainstorm 우리 시스템에 어떤 캐시가 적합할까

# 3. 설계
/sc:design 캐시 레이어 아키텍처
```

---

## 6. 프로젝트 온보딩 워크플로우

> **흐름**: `index-repo` -> `explain` -> `analyze`
> 새 프로젝트에 투입됐을 때 가장 먼저 실행하세요!

1. `index-repo` -- 전체 구조 맵 생성, 주요 모듈 파악
2. `explain` -- 핵심 기능과 아키텍처 이해
3. `analyze` -- 현재 코드 상태 파악, 주의 영역 확인

### 실전 예시: chemiSys 프로젝트 온보딩

```
# 1. 전체 구조 파악
/sc:index-repo

# 2. 핵심 시스템 이해
/sc:explain "전자정부프레임워크 구조"
/sc:explain "화학제품 승인 워크플로우"
/sc:explain "사용자 인증 및 권한 시스템"

# 3. 코드 품질 확인
/sc:analyze src/main/java/egovframework/
```

---

## 7. 복잡한 작업 처리 워크플로우

> **흐름**: `pm` -> `spawn` -> `task` -> `reflect`
> 여러 모듈에 걸친 대규모 작업에 적합합니다.

1. `pm` -- 복잡한 작업 조율 및 전체 계획 수립
2. `spawn` -- 병렬 처리 가능한 작업 분리
3. `task` -- 분해된 개별 작업 실행
4. `reflect` -- 완료된 작업 검토 및 품질 확인

---

## 8. 문서화 워크플로우

> **흐름**: `analyze` -> `explain` -> `document`

### 실전 예시: API 문서 작성

```
# 1. API 분석
/sc:analyze src/main/java/egovframework/cms/biocide/controller/

# 2. 동작 이해
/sc:explain BiocideController의 각 엔드포인트

# 3. 문서 생성
/sc:document BiocideController --type api --style detailed
```

---

## 9. 성능 최적화 워크플로우

> **흐름**: `analyze` -> `research` -> `improve` -> `test`

### 실전 예시: 쿼리 성능 개선

```
# 1. 성능 분석
/sc:analyze "느린 쿼리 찾기"

# 2. 최적화 방법 조사
/sc:research "MyBatis 쿼리 최적화 기법"

# 3. 개선
/sc:improve BiocideMapper.xml

# 4. 성능 테스트
/sc:test --performance
```

---

## 10. 보안 점검 워크플로우

> **흐름**: `analyze` -> `troubleshoot` -> `improve` -> `test`

### 실전 예시: 보안 취약점 점검

```
# 1. 보안 분석
/sc:analyze --security

# 2. 취약점 진단
/sc:troubleshoot "XSS 취약점 가능성 검토"

# 3. 보안 강화
/sc:improve "입력 값 검증 추가"

# 4. 보안 테스트
/sc:test --security
```

---

## 상황별 빠른 선택 가이드

> **뭘 쓸지 모를 때는 이 표를 보세요!** 상황에 맞는 시작 명령어와 전체 흐름을 한눈에 확인할 수 있습니다.

| **상황** | **시작 명령어** | **전체 흐름** |
|---------|----------------|--------------|
| "코드가 이해가 안 돼" | `/sc:explain` | `explain` |
| "버그가 있어" | `/sc:troubleshoot` | `explain` -> `troubleshoot` -> `implement` -> `test` |
| "새 기능 만들어야 해" | `/sc:brainstorm` | `brainstorm` -> `design` -> `implement` -> `test` |
| "코드 정리하고 싶어" | `/sc:analyze` | `analyze` -> `improve` -> `cleanup` -> `test` |
| "뭘 어떻게 해야 할지 모르겠어" | `/sc:recommend` | `recommend` -> (추천된 명령어) |
| "이 기술 도입해도 될까?" | `/sc:research` | `research` -> `brainstorm` -> `design` |
| "프로젝트 처음이야" | `/sc:index-repo` | `index-repo` -> `explain` |
| "성능이 느려" | `/sc:analyze` | `analyze` -> `research` -> `improve` -> `test` |
| "문서 만들어야 해" | `/sc:document` | `analyze` -> `document` |
| "복잡한 작업이야" | `/sc:pm` | `pm` -> `spawn` -> `task` -> `reflect` |

---

## 실전 팁

> **Tip 1: 단순한 작업은 단일 명령어로**
> 모든 작업에 전체 워크플로우를 적용할 필요는 없습니다.
> `/sc:explain "이 함수가 뭐 하는 거야?"` 이것만으로 충분할 때가 많습니다.

> **Tip 2: 반복적인 개선 작업**
> `improve`와 `test`는 필요에 따라 반복합니다.
> `improve` -> `test` -> `improve` -> `test` -> ...

> **Tip 3: 모를 때는 `/sc:recommend`**
> 어떤 명령어를 써야 할지 모를 때는 항상 `/sc:recommend`로 시작하세요.
> `/sc:recommend [하고 싶은 일을 자연어로 설명]`

> **Tip 4: 큰 작업은 `/sc:pm`으로**
> 여러 모듈에 걸친 작업이나 장기 작업은 `/sc:pm`으로 시작하면 체계적으로 관리할 수 있습니다.
