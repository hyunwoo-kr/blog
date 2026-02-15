---
title: "DBeaver: 개발자를 위한 무료 데이터베이스 관리 도구"
date: 2026-01-27
categories:
- tools
tags:
- dbeaver
- database
- sql
- gui-tool
keywords:
- DBeaver
- 데이터베이스 관리
- SQL 에디터
- ER 다이어그램
author: "hyunwoo"
draft: false
---

DBeaver는 100개 이상의 DB를 지원하는 무료 오픈소스 데이터베이스 관리 도구입니다. SQL 자동 완성, ER 다이어그램 자동 생성 등 강력한 기능을 제공하며 Windows, macOS, Linux 모두 지원합니다.

<!--more-->

---

## 소개

데이터베이스 작업을 하다 보면 CLI만으로는 한계를 느낄 때가 있습니다. 테이블 구조를 한눈에 파악하고 싶거나, 복잡한 쿼리를 편하게 작성하고 싶을 때 GUI 도구가 필요해집니다.

**DBeaver**는 이런 니즈를 충족시켜주는 무료 오픈소스 데이터베이스 관리 도구입니다. MySQL, MariaDB, PostgreSQL, SQLite, Oracle, SQL Server 등 100개 이상의 데이터베이스를 단일 도구로 관리할 수 있습니다.

---

## 왜 DBeaver인가?

### 1. 완전 무료 (Community Edition)

상용 도구인 DataGrip, Navicat과 달리 DBeaver Community Edition은 **완전 무료**입니다. 개인 개발자부터 스타트업까지 비용 부담 없이 사용할 수 있습니다.

### 2. 크로스 플랫폼

Windows, macOS, Linux 모두 지원합니다. 개발 환경이 바뀌어도 동일한 도구를 사용할 수 있어 학습 비용이 줄어듭니다.

### 3. 강력한 SQL 에디터

- **Syntax Highlighting**: SQL 구문을 색상으로 구분해 가독성 향상
- **Auto Completion**: 테이블명, 컬럼명, SQL 함수를 자동 완성
- **Error Detection**: 실행 전 문법 오류를 미리 감지
- **AI 지원**: OpenAI, Copilot 연동으로 쿼리 자동 생성 가능

```sql
-- 자동 완성 덕분에 긴 테이블명도 빠르게 입력
SELECT * FROM customer_order_history WHERE order_date > '2026-01-01';
```

### 4. ER 다이어그램 자동 생성

복잡한 데이터베이스 스키마를 **시각적으로 파악**할 수 있습니다.

- 테이블 간 관계(FK)를 자동으로 연결선으로 표시
- 단일 테이블 선택 시 참조/피참조 테이블만 필터링 가능
- SELECT, INSERT, UPDATE, DELETE, JOIN 쿼리 자동 생성
- PNG, SVG 등으로 내보내기 -> 문서화에 활용

> **팁**: 신규 프로젝트 투입 시 DB 구조 파악에 매우 유용합니다!

### 5. 데이터 편집의 편리함

- 스프레드시트처럼 셀 단위 편집
- 대량 데이터 Export/Import (CSV, JSON, XML, SQL 등)
- 데이터 필터링, 정렬, 검색
- NULL 값 시각적 구분

### 6. 다중 데이터베이스 동시 연결

하나의 창에서 여러 DB에 동시 접속할 수 있습니다.

```
├── Production (MySQL)
├── Staging (PostgreSQL)
├── Legacy (Oracle)
└── Local (SQLite)
```

### 7. 보안 연결 지원

- SSH 터널링
- SSL/TLS 암호화
- Proxy 연결
- AWS SSM, Kubernetes 지원

운영 서버 DB에 안전하게 접속할 수 있습니다.

---

## 설치 방법

### 시스템 요구사항

| 항목 | 요구사항 |
|------|----------|
| OS | Windows 10 이상, macOS, Linux |
| Java | 내장 (OpenJDK 21 포함, 별도 설치 불필요) |
| 디스크 | 약 500MB |

### Windows 설치

**방법 1: 공식 설치 파일**

1. [DBeaver 다운로드 페이지](https://dbeaver.io/download/) 접속
2. **Windows (installer)** 클릭하여 다운로드
3. 설치 파일 실행
4. 설치 옵션 선택 (기본값 권장)
5. 완료 후 실행

**방법 2: winget (터미널)**

```powershell
winget install dbeaver.dbeaver
```

**방법 3: Chocolatey**

```powershell
choco install dbeaver
```

### macOS 설치

```bash
brew install --cask dbeaver-community
```

### Linux 설치

```bash
# Ubuntu/Debian
sudo snap install dbeaver-ce

# 또는 직접 다운로드
wget https://dbeaver.io/files/dbeaver-ce_latest_amd64.deb
sudo dpkg -i dbeaver-ce_latest_amd64.deb
```

---

## 첫 연결 설정하기

1. DBeaver 실행
2. 좌측 상단 **플러그 아이콘** 클릭 (또는 `Ctrl+Shift+N`)
3. 데이터베이스 종류 선택 (예: MariaDB)
4. 연결 정보 입력:

| 항목 | 예시 |
|------|------|
| Host | `localhost` 또는 IP |
| Port | `3306` (기본) |
| Database | `mydb` |
| Username | `root` |
| Password | `****` |

5. **Test Connection** 클릭하여 연결 확인
6. 드라이버 다운로드 팝업 시 **Download** 클릭 (최초 1회)
7. **Finish**

> **주의**: 최초 연결 시 드라이버 다운로드 팝업이 나타납니다. 인터넷 연결이 필요합니다.

---

## 유용한 단축키

| 단축키 | 기능 |
|--------|------|
| `Ctrl+Enter` | 현재 쿼리 실행 |
| `Ctrl+Shift+Enter` | 전체 스크립트 실행 |
| `Ctrl+Space` | 자동 완성 |
| `Ctrl+Shift+F` | SQL 포맷팅 |
| `Ctrl+/` | 주석 토글 |
| `F4` | 선택한 테이블 상세 보기 |
| `Ctrl+Shift+E` | 실행 계획 보기 |

---

## Community vs Enterprise

| 기능 | Community (무료) | Enterprise (유료) |
|------|:-----------------:|:-----------------:|
| 관계형 DB 지원 | O | O |
| SQL 에디터 | O | O |
| ER 다이어그램 | O | O |
| NoSQL 지원 | X | O |
| 클라우드 DB 연동 | 제한적 | O |
| 기술 지원 | 커뮤니티 | 공식 지원 |

> 대부분의 개발 업무는 **Community Edition으로 충분**합니다.

---

## 마무리

DBeaver는 무료임에도 상용 도구 못지않은 기능을 제공합니다. 특히 여러 종류의 데이터베이스를 다루는 개발자에게 강력히 추천합니다.

CLI에 익숙하더라도 ER 다이어그램이나 데이터 시각화가 필요한 순간이 있습니다. 그럴 때 DBeaver를 꺼내 쓰면 됩니다.

---

## 참고 자료

- [DBeaver 공식 사이트](https://dbeaver.io/)
- [DBeaver 다운로드](https://dbeaver.io/download/)
- [DBeaver 설치 가이드](https://dbeaver.com/docs/dbeaver/Installation/)
- [ER Diagrams 문서](https://dbeaver.com/docs/dbeaver/ER-Diagrams/)
- [DBeaver 주요 기능](https://dbeaver.com/features/)
