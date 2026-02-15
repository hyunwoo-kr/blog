---
title: "Tibero6 MCP 등록 및 Claude Code 연동 가이드"
date: 2026-01-21
categories:
- ai
tags:
- mcp
- tibero6
- claude-code
- jdbc
- database
keywords:
- MCP
- Tibero6
- Claude Code
- JDBC MCP Server
author: "hyunwoo"
draft: false
---

Tibero6 전용 MCP 서버는 존재하지 않지만, 범용 JDBC MCP 서버를 활용하여 Tibero6에 연결할 수 있습니다. 이 가이드에서는 Claude Code에서 Tibero6 데이터베이스에 접근하여 쿼리 실행, 스키마 조회 등을 수행하는 방법을 설명합니다.

<!--more-->

---

## 1. Tibero6 JDBC 드라이버 정보

| 항목 | 값 |
|------|-----|
| JAR 파일 위치 | `$TB_HOME/client/lib/jar/tibero6-jdbc.jar` |
| Driver Class | `com.tmax.tibero.jdbc.TbDriver` |
| JDBC URL 형식 | `jdbc:tibero:thin:@<IP>:<PORT>:<DATABASE>` |
| 기본 포트 | 8629 |

---

## 2. 추천 JDBC MCP 서버

### 방법 1: Quarkiverse JDBC MCP Server (가장 간편)

**jbang 설치:**

```bash
# Mac
brew install jbang

# Linux/Windows
curl -Ls https://sh.jbang.dev | bash -s - app setup
```

**Claude Code에 MCP 등록:**

```bash
claude mcp add tibero -s user -- jbang \
  -cp /path/to/tibero6-jdbc.jar \
  jdbc@quarkiverse/quarkus-mcp-servers \
  jdbc:tibero:thin:@192.168.0.254:8629:tibero \
  -u sys -p tibero
```

---

### 방법 2: OpenLink JDBC MCP Server

**1. 서버 다운로드:**

```bash
git clone https://github.com/OpenLinkSoftware/mcp-jdbc-server
cd mcp-jdbc-server
mvn clean package
```

**2. Claude Code에 등록:**

```bash
claude mcp add-json tibero '{
  "command": "java",
  "args": [
    "-cp",
    "/path/to/mcp-jdbc-server/target/MCPServer-1.0.0-runner.jar:/path/to/tibero6-jdbc.jar",
    "io.quarkus.runner.GeneratedMain"
  ],
  "env": {
    "jdbc.url": "jdbc:tibero:thin:@192.168.0.254:8629:tibero",
    "jdbc.user": "sys",
    "jdbc.password": "tibero"
  }
}'
```

---

### 방법 3: DBMCP Server (Java/Maven 기반)

**1. 서버 다운로드 및 빌드:**

```bash
git clone https://github.com/skanga/dbmcp
cd dbmcp
mvn clean package
```

**2. Claude Code에 등록:**

```bash
claude mcp add-json tibero '{
  "command": "java",
  "args": ["-jar", "/path/to/dbmcp/target/dbmcp-1.0.0.jar"],
  "env": {
    "DB_URL": "jdbc:tibero:thin:@192.168.0.254:8629:tibero",
    "DB_USER": "sys",
    "DB_PASSWORD": "tibero",
    "DB_DRIVER": "com.tmax.tibero.jdbc.TbDriver"
  }
}'
```

---

## 3. 프로젝트 설정 파일로 관리

프로젝트 루트에 `.mcp.json` 파일 생성:

```json
{
  "mcpServers": {
    "tibero": {
      "command": "jbang",
      "args": [
        "-cp", "/path/to/tibero6-jdbc.jar",
        "jdbc@quarkiverse/quarkus-mcp-servers",
        "jdbc:tibero:thin:@192.168.0.254:8629:tibero",
        "-u", "chemp",
        "-p", "your_password"
      ]
    }
  }
}
```

---

## 4. Claude Code에서 사용하기

**등록 확인:**

```bash
claude mcp list
```

**MCP 상태 확인 (Claude Code 내부):**

```
/mcp
```

**사용 예시:**

```
> tibero 데이터베이스에서 사용자 테이블 목록 조회해줘

> BIOCIDE_APPL 테이블의 스키마 구조를 보여줘

> 최근 등록된 살생물제 신청 건수를 조회하는 쿼리 작성해줘
```

---

## 5. MCP 범위(Scope) 설명

| 범위 | 설명 | 저장 위치 |
|------|------|-----------|
| `local` (기본) | 현재 프로젝트에서만 사용 | 프로젝트 내 설정 |
| `project` | 프로젝트 전체 공유 (.mcp.json) | 프로젝트 루트 |
| `user` | 모든 프로젝트에서 사용 가능 | ~/.claude.json |

---

## 6. 주의사항

> **설정 시 주의할 점**
> - Tibero JDBC JAR 경로는 반드시 **절대 경로**로 지정
> - Tibero 서버 포트(기본 8629) **방화벽** 접근 가능 확인
> - 연결 계정의 **테이블 조회 권한** 확인
> - MCP 설정 변경 후 **Claude Code 재시작** 필수

---

## 7. 문제 해결

**연결 실패 시:**

```bash
# 직접 JDBC 연결 테스트
java -cp tibero6-jdbc.jar com.tmax.tibero.jdbc.TbDriver

# 로그 활성화
jbang -Dquarkus.log.file.enable=true \
  -Dquarkus.log.file.path=~/mcp-tibero.log \
  jdbc@quarkiverse/quarkus-mcp-servers ...
```

**Connection closed 오류 (Windows):**

```bash
# Windows에서는 cmd /c 래퍼 사용
claude mcp add tibero -- cmd /c jbang ...
```

---

## 참고 링크

- [Quarkiverse JDBC MCP Server](https://github.com/quarkiverse/quarkus-mcp-servers/tree/main/jdbc)
- [OpenLink JDBC MCP Server](https://github.com/OpenLinkSoftware/mcp-jdbc-server)
- [DBMCP Server](https://github.com/skanga/dbmcp)
- [Claude Code MCP 공식 문서](https://docs.anthropic.com/ko/docs/claude-code/mcp)
- [Tibero JDBC 가이드](https://technet.tmaxsoft.com/upload/download/online/tibero/pver-20170217-000001/tibero_jdbc/ch01.html)

---

> **활용 팁**: CHEMP 시스템의 Tibero6와 연동하면 Claude Code에서 직접 DB 조회 및 분석이 가능합니다. 복잡한 SQL 작성, 테이블 구조 파악, 데이터 분석 등을 자연어로 요청할 수 있어 개발 생산성이 크게 향상됩니다.
