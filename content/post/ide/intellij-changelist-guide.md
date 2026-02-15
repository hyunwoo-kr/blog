---
title: "IntelliJ IDEA Changelist 개발자 가이드"
date: 2026-01-30
categories:
- ide
tags:
- intellij
- changelist
- git
- ide
keywords:
- IntelliJ Changelist
- 커밋 단위 분리
- 작업 분류
- Reselect Files After Commit
author: "hyunwoo"
draft: false
---

Changelist는 변경된 파일을 작업 단위로 분류하는 IntelliJ 기능입니다. 한 브랜치에서 여러 작업을 병행할 때 커밋 단위를 깔끔하게 분리할 수 있으며, Commit 후 파일 선택 해제 문제는 Reselect Files After Commit 설정으로 해결합니다.

<!--more-->

> 대상 버전: IntelliJ IDEA 2025.3.2 (Build #IU-253.30387.90)

---

## Changelist란?

Changelist는 IntelliJ IDEA에서 **변경된 파일을 논리적 그룹으로 분류**하는 기능이다.

Git의 staging area와는 별개로, IDE 레벨에서 변경 파일을 **작업 단위별로 묶어서 관리**할 수 있다.

```
예시) 하나의 브랜치에서 여러 작업을 동시에 진행하는 경우

[Default Changelist]
  └── 아직 분류하지 않은 변경 파일들

[기능A - 제품목록 검색 필터 추가]
  ├── BiocideProdController.java
  ├── biocideProd_SQL.xml
  └── biocideProdList.jsp

[기능B - 승인 상태 버그 수정]
  ├── BiocideProdChgAprvController.java
  └── chgAprvCn.jsp

[리팩토링 - 공통코드 정리]
  └── CodeUtilController.java
```

> **신입 개발자를 위한 비유**: 쇼핑백을 들고 마트에 갔다고 상상해보세요.
> Changelist는 장바구니 안의 **칸막이**입니다. 과일, 육류, 음료를 각각 다른 칸막이에 담듯이,
> 변경된 파일들을 작업 목적별로 나눠 담는 것입니다.

---

## 왜 사용하는가?

### 1. 커밋 단위를 깔끔하게 분리

하나의 브랜치에서 여러 작업을 병행할 때, **관련 파일만 골라서 커밋**할 수 있다.

Changelist 없이 작업하면 커밋 시 수십 개 파일 중 관련된 것만 수동으로 체크해야 한다.

### 2. 작업 맥락 보존

"이 파일을 왜 수정했지?" 하는 상황을 방지한다.

Changelist 이름 자체가 작업 내용을 설명하므로, **파일과 작업 목적의 연결이 명확**해진다.

### 3. 실수 방지

관련 없는 파일을 함께 커밋하는 실수를 줄인다.

특히 `globals.properties` 같은 설정 파일이나 로컬 전용 변경 사항을 분리해둘 수 있다.

### 4. 코드 리뷰 품질 향상

커밋이 논리적 단위로 분리되면, 리뷰어가 변경 의도를 빠르게 파악할 수 있다.

---

## 어떤 때 사용하는가?

| 상황 | 설명 |
|------|------|
| **여러 작업 병행** | 한 브랜치에서 기능 개발 + 버그 수정을 동시에 진행할 때 |
| **커밋하면 안 되는 파일 분리** | 로컬 DB 설정, 테스트용 변경 등을 별도 Changelist로 격리 |
| **단계별 커밋** | 대규모 리팩토링을 여러 커밋으로 나눠야 할 때 |
| **작업 중단/재개** | 급한 작업이 들어와서 현재 작업을 일시 중단할 때 |
| **코드 리뷰 대비** | 리뷰어가 이해하기 쉽도록 변경 사항을 주제별로 분리 |

---

## 기본 사용법

### Changelist 생성

1. **Commit 도구 창** 열기: `Alt + 0` (또는 왼쪽 사이드바의 **Commit** 탭)
2. **Unversioned Files** 또는 **Changes** 영역에서 우클릭
3. **New Changelist...** 선택
4. 이름 입력 (예: `기능A - 제품목록 검색 필터 추가`)

또는 메뉴: **Git > Changelist > New Changelist...**

### 파일을 Changelist로 이동

- **Commit 도구 창**에서 파일을 드래그 & 드롭하여 다른 Changelist로 이동
- 파일 우클릭 > **Move to Another Changelist...** > 대상 Changelist 선택

### Active Changelist 설정

- Changelist 이름을 우클릭 > **Set Active Changelist**
- **Active Changelist**: 이후 새로 수정하는 파일이 자동으로 이 Changelist에 추가됨

### Changelist 단위로 커밋

1. `Ctrl + K` (커밋 창 열기)
2. 상단에서 커밋할 **Changelist 선택**
3. 해당 Changelist의 파일만 표시됨
4. 커밋 메시지 입력 후 **Commit**

---

## 핵심 설정: Commit 후 파일 선택 해제 방지

> **반드시 설정 필요!**
> 기본 설정에서는 커밋 완료 후 Changelist의 파일 체크박스가 모두 해제됩니다.
> 이후 같은 Changelist에서 추가 커밋을 하려면 파일을 다시 선택해야 하는 번거로움이 있습니다.
> 특히 **부분 커밋** 후 나머지 파일의 선택 상태가 풀리면 실수로 빠뜨리기 쉽습니다.

### 방법 1: After Commit 설정 변경 (권장)

1. `Ctrl + Alt + S` > **Settings** 열기
2. **Version Control > Commit** 이동
3. **After Commit** 섹션 찾기
4. 아래 옵션 설정:

| 설정 | 값 | 설명 |
|------|-----|------|
| **Select files after commit** | 체크 | 커밋 후에도 파일 선택 상태 유지 |

> 이 옵션이 보이지 않는 경우, **Use non-modal commit interface** 설정에 따라 UI가 달라질 수 있습니다. 아래 방법 2를 확인하세요.

### 방법 2: Non-modal Commit Interface 사용 시

IntelliJ 2025.x에서는 기본적으로 **Non-modal commit interface**를 사용합니다.

1. `Ctrl + Alt + S` > **Settings** 열기
2. **Version Control > Commit** 이동
3. **Use non-modal commit interface** 가 체크되어 있는지 확인
4. 이 모드에서는 Commit 도구 창(`Alt + 0`)이 항상 왼쪽에 표시됨
5. **Commit 도구 창** 상단의 **톱니바퀴 아이콘** 클릭
6. 드롭다운에서 다음을 확인:

| 설정 | 권장값 | 설명 |
|------|--------|------|
| **Reselect Files After Commit** | 체크 | 커밋 후 파일 재선택 |

> **위치**: Commit 도구 창 > 톱니바퀴 또는 점 세 개 메뉴

### 방법 3: Registry 설정 (고급)

위 방법으로 해결되지 않는 경우:

1. `Ctrl + Shift + A` > **Registry...** 검색하여 열기
2. `vcs.commit.reselect` 검색
3. 해당 키가 있으면 **체크** 활성화
4. IDE 재시작

### 설정 확인 체크리스트

- Settings > Version Control > Commit 에서 After Commit 옵션 확인
- Commit 도구 창(Alt+0) > 톱니바퀴 > Reselect Files After Commit 체크
- 설정 변경 후 테스트 커밋으로 동작 확인

---

## 실전 활용 시나리오

### 시나리오 1: 로컬 설정 파일 격리

개발 중 `globals.properties`의 DB 접속 정보를 로컬 환경에 맞게 변경했지만, 이 변경은 절대 커밋하면 안 되는 경우:

```
[커밋 금지 - 로컬 설정]          <- 이 Changelist는 커밋하지 않음
  └── globals.properties

[기능개발 - 제품 검색 개선]      <- 이것만 커밋
  ├── BiocideProdController.java
  └── biocideProd_SQL.xml
```

> **신입 개발자 팁**: `globals.properties`를 "커밋 금지" Changelist에 넣어두면, 커밋 시 실수로 포함하는 실수를 원천차단할 수 있습니다.

### 시나리오 2: JSP 리팩토링 + 기능 개발 병행

JSP 컴포넌트 분리 리팩토링과 새 기능 개발을 동시에 진행하는 경우:

```
[리팩토링 - mkrInfo 컴포넌트 분리]
  ├── biocideProdCommFrm.jsp      (부모 JSP 수정)
  └── include/mkrInfo.jsp          (새 컴포넌트)

[기능 - 승인 상태 필터 추가]
  ├── BiocideProdChgAprvController.java
  ├── biocideProdChgAprv_SQL.xml
  └── chgAprvList.jsp
```

각각 별도로 커밋하면 Git 히스토리가 깔끔해진다:

```
commit abc1234: refactor: mkrInfo 컴포넌트 분리 (biocideProdComm)
commit def5678: feat: 승인 상태 필터 추가 (biocideProdChgAprv)
```

### 시나리오 3: 긴급 버그 수정 끼어들기

기능 개발 중 긴급 버그 수정 요청이 들어온 경우:

1. 현재 작업 파일들이 이미 Changelist에 분류되어 있으므로 혼동 없음
2. 새 Changelist `긴급 - 로그인 오류 수정` 생성
3. Active Changelist로 설정
4. 버그 수정 > 해당 Changelist만 커밋
5. 원래 작업 Changelist로 돌아가서 계속 개발

---

## 주의사항

### 1. Active Changelist를 항상 확인

새 파일을 수정하면 **현재 Active Changelist**에 자동 추가된다.

작업 전환 시 Active Changelist를 먼저 변경해야 파일이 엉뚱한 곳에 들어가지 않는다.

> **자주 하는 실수**: 작업 전환 후 Active Changelist를 바꾸지 않아서, 버그 수정 파일이 "기능개발" Changelist에 섞여버리는 경우가 많습니다.

### 2. Changelist는 로컬 전용

Changelist는 IntelliJ IDE의 로컬 기능이다. **Git에 저장되지 않으며**, 팀원과 공유되지 않는다.

IDE를 재설치하거나 프로젝트를 다시 열면 Changelist 구성이 초기화될 수 있다.

### 3. 빈 Changelist 자동 삭제

기본 설정에서 커밋 후 파일이 없는 빈 Changelist는 자동 삭제된다.

이를 방지하려면:
- Settings > Version Control > Changelists > **Remove empty changelists** 체크 해제

### 4. Shelve와 함께 활용

작업을 임시 보관하려면 **Shelve** 기능과 함께 사용한다:
- Changelist 우클릭 > **Shelve Changes...**
- 변경 사항이 임시 저장되고 working directory에서 제거됨
- 나중에 **Unshelve**로 복원

### 5. Git Stash와의 차이

| 항목 | Changelist | Git Stash | Shelve |
|------|:----------:|:---------:|:------:|
| 범위 | IDE 로컬 | Git 전체 | IDE 로컬 |
| 분류 | 여러 그룹 가능 | 단일 스택 | 여러 그룹 가능 |
| 파일 유지 | 유지 (분류만) | 제거 후 복원 | 제거 후 복원 |
| 공유 | 불가 | 가능 (같은 repo) | 불가 |
| 용도 | 작업 분류 | 임시 저장 | 임시 저장 |

---

## 단축키 요약

| 동작 | 단축키 |
|------|--------|
| Commit 도구 창 열기 | `Alt + 0` |
| 커밋 | `Ctrl + K` |
| Settings 열기 | `Ctrl + Alt + S` |
| Action 검색 | `Ctrl + Shift + A` |
| 파일을 다른 Changelist로 이동 | 파일 우클릭 > Move to Another Changelist |
| 새 Changelist 생성 | Changes 영역 우클릭 > New Changelist |
