---
title: "DBeaver에서 Tibero6 연결 가이드"
date: 2026-01-19
categories:
- database
tags:
- dbeaver
- tibero6
- jdbc
- database
keywords:
- DBeaver
- Tibero6
- JDBC 드라이버
- 데이터베이스 연결
author: "hyunwoo"
draft: false
---

DBeaver에서 Tibero6 데이터베이스에 연결하는 방법을 단계별로 정리합니다. JDBC 드라이버 준비부터 연결 생성, 문제 해결까지 다룹니다.

<!--more-->

## 1. JDBC 드라이버 준비

Tibero6 JDBC 드라이버 파일이 필요합니다. 일반적으로 Tibero가 설치된 서버에서 다음 경로에 있습니다:

```
$TB_HOME/client/lib/tbjdbc6.jar
```

해당 jar 파일을 로컬 PC의 적절한 위치(예: `C:\drivers\tibero\`)에 복사해두세요.

---

## 2. DBeaver에서 드라이버 등록

**Database > Driver Manager** 메뉴를 선택합니다.

**New** 버튼을 클릭하여 새 드라이버를 생성합니다.

**Settings 탭**에서 다음 정보를 입력합니다:

| 항목 | 값 |
|------|-----|
| Driver Name | Tibero6 |
| Driver Type | Generic |
| Class Name | `com.tmax.tibero.jdbc.TbDriver` |
| URL Template | `jdbc:tibero:thin:@{host}:{port}:{database}` |
| Default Port | 8629 |

**Libraries 탭**에서 **Add File** 버튼을 클릭하고 `tbjdbc6.jar` 파일을 선택합니다.

**OK** 버튼으로 저장합니다.

---

## 3. 데이터베이스 연결 생성

**Database > New Database Connection** 메뉴를 선택합니다.

검색창에 **Tibero6**를 입력하고 방금 등록한 드라이버를 선택합니다.

연결 정보를 입력합니다:

| 항목 | 설명 |
|------|------|
| Host | Tibero 서버 IP 또는 호스트명 |
| Port | 8629 (기본값, 환경에 따라 다를 수 있음) |
| Database | SID 또는 서비스명 |
| Username | 접속 계정 |
| Password | 비밀번호 |

**Test Connection** 버튼으로 연결을 테스트합니다.

정상 연결되면 **Finish** 버튼으로 완료합니다.

---

## 4. 연결 문제 해결

**드라이버 클래스를 찾을 수 없는 경우**: jar 파일 경로를 확인하고, DBeaver를 재시작한 후 다시 시도하세요.

**연결 거부 오류**: 방화벽 설정, Tibero 리스너 상태, 포트 번호를 확인하세요.

**인증 실패**: 계정 정보와 권한을 확인하세요.
