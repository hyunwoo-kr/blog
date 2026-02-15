---
title: "Caddy SSL 인증서 자동 갱신 가이드"
date: 2025-02-01
categories:
- devops
tags:
- caddy
- ssl
- letsencrypt
- https
- certificate
keywords:
- caddy ssl
- 인증서 자동 갱신
- lets encrypt
- acme
author: "hyunwoo"
draft: false
---

Caddy는 Let's Encrypt 인증서를 완전 자동으로 발급하고 갱신합니다. 만료 30일 전부터 자동 갱신을 시도하며, 별도의 cron job이나 스크립트가 필요 없습니다.

<!--more-->

---

## 인증서 갱신 프로세스

```
+─────────────────────────────────────────────────────────────+
|                    Caddy 인증서 생명주기                      |
+─────────────────────────────────────────────────────────────+
|                                                             |
|  [시작] Caddy 실행                                          |
|     |                                                       |
|     v                                                       |
|  Caddyfile에서 도메인 감지                                   |
|     |                                                       |
|     v                                                       |
|  +─────────────────────────────────────+                    |
|  | 인증서 존재?                         |                    |
|  +──────────┬──────────────────────────+                    |
|        No   |   Yes                                         |
|     +───────┴───────+                                       |
|     v               v                                       |
|  새 인증서      만료까지 30일 이내?                          |
|  발급 요청          |                                       |
|     |          No   |   Yes                                 |
|     |       +───────┴───────+                               |
|     |       v               v                               |
|     |    대기 (주기적     갱신 요청                          |
|     |     체크 계속)         |                               |
|     |       |               |                               |
|     +───────┴───────────────+                               |
|             |                                               |
|             v                                               |
|  Let's Encrypt ACME 서버                                    |
|     |                                                       |
|     v                                                       |
|  HTTP-01 Challenge (포트 80)                                |
|     |                                                       |
|     v                                                       |
|  인증서 저장 (/data/caddy/certificates/)                    |
|     |                                                       |
|     v                                                       |
|  [반복] 갱신 체크 계속...                                    |
|                                                             |
+─────────────────────────────────────────────────────────────+
```

---

## 갱신 타이밍

| 항목 | 값 |
|------|---|
| 인증서 유효 기간 | 90일 |
| 갱신 시작 시점 | 만료 30일 전 |
| 갱신 체크 주기 | 약 12시간마다 |
| 갱신 시간대 | 랜덤 (서버 부하 분산) |

---

## 인증서 저장 위치

Docker 환경에서 인증서는 `caddy_data` 볼륨에 저장됩니다.

```bash
# 볼륨 위치 확인
docker volume inspect caddy_data

# 인증서 파일 확인
docker exec caddy ls -la /data/caddy/certificates/acme-v02.api.letsencrypt.org-directory/
```

**디렉토리 구조:**

```
/data/caddy/
├── certificates/
|   └── acme-v02.api.letsencrypt.org-directory/
|       ├── example.com/
|       |   ├── example.com.crt    # 인증서
|       |   ├── example.com.key    # 개인키
|       |   └── example.com.json   # 메타데이터
|       ├── traefik.example.com/
|       └── whoami.example.com/
└── locks/
```

---

## 인증서 상태 확인

### 방법 1: 브라우저에서 확인

브라우저 주소창 자물쇠 아이콘 클릭 -> 인증서 정보 확인

### 방법 2: CLI로 확인

```bash
# 인증서 만료일 확인 (your-domain.com을 실제 도메인으로 변경)
echo | openssl s_client -servername your-domain.com -connect your-domain.com:443 2>/dev/null | openssl x509 -noout -dates

# 출력 예시:
# notBefore=Feb  1 00:00:00 2026 GMT
# notAfter=May  2 00:00:00 2026 GMT
```

### 방법 3: Caddy 로그 확인

```bash
# 인증서 관련 로그 필터링
docker compose logs caddy | grep -E "(certificate|tls|acme)"

# 실시간 로그 모니터링
docker compose logs -f caddy | grep -E "(obtain|renew)"
```

---

## 갱신 성공 로그 예시

```json
{"level":"info","msg":"certificate obtained successfully","identifier":"example.com"}
{"level":"info","msg":"certificate renewed successfully","identifier":"example.com"}
```

---

## 갱신 실패 시 동작

1. **자동 재시도**: 실패 시 지수 백오프로 재시도
2. **대체 CA**: Let's Encrypt 실패 시 ZeroSSL 시도 (Caddy 기본 설정)
3. **기존 인증서 유지**: 갱신 실패해도 만료 전까지 기존 인증서 사용

---

## 수동 갱신 (필요시)

일반적으로 불필요하지만, 강제 갱신이 필요한 경우:

```bash
# Caddy 재시작 (인증서 체크 트리거)
docker compose restart caddy

# 또는 인증서 삭제 후 재발급 (your-domain.com을 실제 도메인으로 변경)
docker exec caddy rm -rf /data/caddy/certificates/acme-v02.api.letsencrypt.org-directory/your-domain.com
docker compose restart caddy
```

---

## 트러블슈팅

### 문제: 인증서 갱신 실패

**원인 1: 포트 80 차단**

```bash
# 포트 80 접근 테스트
curl -I http://your-domain.com/.well-known/acme-challenge/test
```

-> 방화벽에서 포트 80 허용 필요

**원인 2: DNS 문제**

```bash
# DNS 확인
nslookup your-domain.com
```

-> A 레코드가 서버 IP를 가리키는지 확인

**원인 3: Rate Limit**

- Let's Encrypt는 주당 50개 인증서 제한
- 동일 도메인 실패 시 1시간 대기 필요

### 문제: 인증서가 갱신되지 않음

```bash
# Caddy 로그에서 에러 확인
docker compose logs caddy 2>&1 | grep -i error

# 인증서 메타데이터 확인
docker exec caddy cat /data/caddy/certificates/acme-v02.api.letsencrypt.org-directory/your-domain.com/your-domain.com.json
```

---

## 모니터링 권장사항

### 간단한 체크 스크립트

```bash
#!/bin/bash
# check-ssl-expiry.sh

DOMAIN="your-domain.com"  # 실제 도메인으로 변경
DAYS_WARNING=14

expiry_date=$(echo | openssl s_client -servername $DOMAIN -connect $DOMAIN:443 2>/dev/null | openssl x509 -noout -enddate | cut -d= -f2)
expiry_epoch=$(date -d "$expiry_date" +%s)
current_epoch=$(date +%s)
days_left=$(( ($expiry_epoch - $current_epoch) / 86400 ))

if [ $days_left -lt $DAYS_WARNING ]; then
    echo "WARNING: $DOMAIN certificate expires in $days_left days"
    exit 1
else
    echo "OK: $DOMAIN certificate valid for $days_left days"
    exit 0
fi
```

### Cron으로 주기적 체크 (선택사항)

```bash
# 매일 체크 (보통 불필요 - Caddy가 자동 관리)
0 9 * * * /path/to/check-ssl-expiry.sh
```

---

## 요약

| 항목 | 설명 |
|------|------|
| 자동 발급 | Caddyfile에 도메인 추가만 하면 자동 |
| 자동 갱신 | 만료 30일 전 자동 시작 |
| 수동 작업 | 필요 없음 |
| 다운타임 | 없음 (핫 리로드) |
| 백업 필요 | caddy_data 볼륨 백업 권장 |

> Caddy의 자동 인증서 관리 덕분에 SSL 관련 유지보수가 거의 필요 없습니다.
