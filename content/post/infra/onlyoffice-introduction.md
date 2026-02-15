---
title: "ONLYOFFICE 도입기"
date: 2025-05-02
categories:
- infra
tags:
- onlyoffice
- office-suite
- docker
- self-hosted
- document-server
keywords:
- ONLYOFFICE
- 오픈소스 오피스
- Docker
- 문서 편집
author: "hyunwoo"
draft: false
---

ONLYOFFICE는 오픈소스 온라인 오피스 스위트로, Microsoft Office와 호환되는 문서 편집 기능을 제공합니다. 워드(Writer), 엑셀(Spreadsheet), 파워포인트(Presentation) 형식의 문서를 웹 브라우저에서 실시간으로 편집할 수 있습니다.

<!--more-->

## 주요 특징

- Microsoft Office 형식과 높은 호환성 제공
- 실시간 공동 편집 기능 지원
- 문서 버전 관리 기능
- Docker를 통한 간편한 설치와 구축
- 오픈소스 라이선스로 제공

## Docker를 이용한 로컬 설치 방법

### 1. 사전 준비사항

- Windows 10/11 Pro 이상
- Docker Desktop 설치
- 최소 4GB RAM, 2 CPU 코어 이상 권장

### 2. 설치 단계

1. Docker Desktop 실행 및 정상 작동 확인
2. PowerShell 또는 명령 프롬프트를 관리자 권한으로 실행
3. ONLYOFFICE Document Server 이미지 다운로드 및 실행:

```bash
docker run -i -t -d -p 80:80 --restart=always onlyoffice/documentserver
```

### 3. 설치 확인

웹 브라우저에서 http://localhost 접속하여 ONLYOFFICE Document Server가 정상적으로 실행되는지 확인합니다.

## 사용 방법

- 웹 브라우저를 통해 접속 (기본 포트: 80)
- 지원 문서 형식: docx, xlsx, pptx, pdf 등
- 문서 편집기 API를 통한 타 시스템 연동 가능

## 주의사항

- 프로덕션 환경에서는 보안을 위해 HTTPS 설정 필요
- 정기적인 백업 설정 권장
- 시스템 리소스 모니터링 필요

## 문제해결

### Docker 컨테이너가 시작되지 않는 경우

1. Docker 로그 확인
2. 포트 충돌 여부 확인
3. 시스템 리소스 사용량 체크

### 문서가 열리지 않는 경우

1. 브라우저 캐시 삭제
2. 지원되는 문서 형식인지 확인
3. 문서 권한 설정 확인

## 참고 자료

- [ONLYOFFICE 공식 문서](https://www.onlyoffice.com/docs.aspx)
- [ONLYOFFICE GitHub 저장소](https://github.com/ONLYOFFICE/DocumentServer)
- [Docker Hub - ONLYOFFICE](https://hub.docker.com/r/onlyoffice/documentserver)

## n8n과 통합하기

ONLYOFFICE를 n8n 워크플로우와 통합하여 문서 자동화를 구현할 수 있습니다.

### 1. 통합 준비사항

- ONLYOFFICE Document Server가 설치되어 있어야 함
- n8n 설치 및 실행 환경 구성
- ONLYOFFICE API 접근 토큰 발급

### 2. n8n 워크플로우 설정

1. HTTP Request 노드 추가
2. ONLYOFFICE API 엔드포인트 설정:

```json
{
  "url": "http://your-onlyoffice-server/api/2.0",
  "headers": {
    "Authorization": "Bearer YOUR_ACCESS_TOKEN"
  }
}
```

### 3. 주요 활용 사례

- 문서 자동 생성 및 변환
- 템플릿 기반 문서 작성 자동화
- 문서 처리 결과 이메일 발송
- 정기 보고서 자동 생성

### 4. 워크플로우 예시

문서 변환 워크플로우:

```
트리거 시작 -> 문서 업로드 -> ONLYOFFICE API 호출 -> 문서 변환 -> 결과 저장/전송
```

이러한 통합을 통해 문서 처리 작업을 자동화하고 업무 효율성을 높일 수 있습니다.
