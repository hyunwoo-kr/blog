---
title: "Caddy 학습 가이드 2회차 - 기본 구성"
date: 2025-02-01
categories:
- devops
tags:
- caddy
- docker
- reverse-proxy
- docker-compose
- https
keywords:
- caddy
- caddyfile
- reverse proxy
- auto https
author: "hyunwoo"
draft: false
---

Caddy는 자동 HTTPS가 기본값인 현대적 웹서버입니다. Caddyfile 하나로 모든 설정이 완료되며, Traefik보다 간결한 설정이 특징입니다.

<!--more-->

WSL2 환경에서 Caddy를 직접 구성하면서 Traefik과의 차이점을 체감하는 실습 가이드입니다.

---

## 학습 목표

| 배울 것 | 설명 |
|---------|------|
| Caddy 기본 개념 | Caddyfile 문법 |
| 자동 HTTPS | Let's Encrypt 내장 원리 |
| 리버스 프록시 | 설정 한 줄로 프록시 |
| Traefik과 비교 | 설정 방식 차이 체감 |

---

## Traefik vs Caddy 미리보기

| 항목 | Traefik | Caddy |
|------|---------|-------|
| 설정 파일 | traefik.yml + Docker 라벨 | Caddyfile 하나 |
| 문법 | YAML | 자체 문법 (더 간결) |
| Docker 연동 | 자동 감지 (라벨 필요) | 수동 설정 |
| HTTPS | 별도 설정 필요 | 자동 (기본값) |
| 대시보드 | 내장 (8080) | 없음 (별도 설치) |

---

## 사전 준비

```bash
# 1회차 정리 (아직 안 했다면)
cd ~/lab/traefik-practice-1
docker compose down

# 2회차 디렉토리 생성
mkdir -p ~/lab/caddy-practice-2
cd ~/lab/caddy-practice-2
```

---

## Step 1: 기본 구조 만들기

```bash
# 필요한 파일 생성
touch docker-compose.yml
touch Caddyfile
```

**현재 구조:**

```
~/lab/caddy-practice-2/
├── docker-compose.yml
└── Caddyfile
```

**Traefik과 비교:**

```
# Traefik (1회차)          # Caddy (2회차)
├── docker-compose.yml     ├── docker-compose.yml
└── traefik/               └── Caddyfile  <- 파일 하나!
    └── traefik.yml
```

---

## Step 2: Caddyfile 작성

`Caddyfile` 파일을 직접 작성해보세요:

```
# Caddy 설정 파일
# Traefik의 traefik.yml + 라벨 역할을 이 파일 하나로

:80 {
    respond "Hello from Caddy!"
}
```

**문법 설명:**

| 부분 | 의미 |
|------|------|
| `:80` | 80번 포트에서 수신 (도메인 없이) |
| `{ }` | 해당 주소에 대한 설정 블록 |
| `respond` | 고정 응답 반환 (테스트용) |

---

## Step 3: Docker Compose 작성

```yaml
services:
  caddy:
    image: caddy:2-alpine
    container_name: caddy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - caddy_data:/data
      - caddy_config:/config
    networks:
      - web

networks:
  web:
    name: web

volumes:
  caddy_data:
  caddy_config:
```

**Traefik과 비교:**

| 항목 | Traefik | Caddy |
|------|---------|-------|
| Docker 소켓 | 마운트 필요 (`docker.sock`) | 불필요 |
| 대시보드 포트 | 8080 | 없음 |
| 볼륨 | 선택사항 | data, config 권장 (인증서 저장) |

**핵심 차이점:**
- Caddy는 `docker.sock`이 필요 없음 -> 보안상 장점
- `caddy_data` 볼륨: Let's Encrypt 인증서 저장 위치

---

## Step 4: 실행 및 테스트

```bash
# 실행
docker compose up -d

# 로그 확인
docker compose logs -f caddy

# 테스트
curl http://localhost
```

**체크포인트**: `Hello from Caddy!` 출력되면 성공

---

## Step 5: 리버스 프록시 설정

whoami 서비스를 Caddy로 프록시해봅니다.

### docker-compose.yml 수정

```yaml
services:
  caddy:
    image: caddy:2-alpine
    container_name: caddy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - caddy_data:/data
      - caddy_config:/config
    networks:
      - web

  whoami:
    image: traefik/whoami
    container_name: whoami
    networks:
      - web

networks:
  web:
    name: web

volumes:
  caddy_data:
  caddy_config:
```

### Caddyfile 수정

```
http://whoami.localhost {
    reverse_proxy whoami:80
}
```

**이게 전부입니다!**

> **중요: `http://` 접두사를 꼭 붙이세요!** Caddy는 도메인만 지정하면 자동으로 HTTPS를 활성화합니다.

---

## 주의: Caddy의 자동 HTTPS

Caddy는 **도메인을 지정하면 자동으로 HTTPS를 활성화**합니다.

### 문제 상황

```
# 이렇게 작성하면
whoami.localhost {
    reverse_proxy whoami:80
}

# Caddy가 자동으로:
# 1. 로컬 인증서 생성 (whoami.localhost용)
# 2. HTTPS(443)로 서비스
# 3. HTTP(80) -> HTTPS 리다이렉트
```

**결과**: `curl http://localhost`가 응답 없음 (HTTPS로 리다이렉트됨)

### 해결 방법

**방법 1: HTTP 명시 (학습용 권장)**

```
http://whoami.localhost {
    reverse_proxy whoami:80
}
```

**방법 2: HTTPS로 테스트**

```bash
curl -k https://whoami.localhost
```

(`-k`는 자체 서명 인증서 허용)

### Traefik vs Caddy 설정 비교 (같은 기능)

```yaml
# Traefik: Docker 라벨 3줄
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.whoami.rule=Host(`whoami.localhost`)"
  - "traefik.http.routers.whoami.entrypoints=web"
```

```
# Caddy: Caddyfile 3줄
http://whoami.localhost {
    reverse_proxy whoami:80
}
```

**체감 포인트:**
- Caddy가 더 직관적이고 읽기 쉬움
- 하지만 Docker 자동 감지는 없음 (수동 추가 필요)

---

## Step 6: 서비스 테스트

```bash
# 변경 적용
docker compose up -d

# 테스트 (http:// 접두사 사용했으므로 HTTP로 테스트)
curl -H "Host: whoami.localhost" http://localhost
```

**체크포인트**: whoami 컨테이너 정보가 출력되면 성공

---

## Step 7: 두 번째 서비스 추가 (직접 해보기)

nginx 서비스를 추가하고 `nginx.localhost`로 접속되게 만들어보세요.

**힌트:**
- docker-compose.yml에 nginx 서비스 추가 (이미지: `nginx:alpine`)
- Caddyfile에 nginx.localhost 블록 추가

**정답 (스스로 시도 후 확인):**

docker-compose.yml에 추가:

```yaml
nginx:
  image: nginx:alpine
  container_name: nginx
  networks:
    - web
```

Caddyfile에 추가:

```
http://nginx.localhost {
    reverse_proxy nginx:80
}
```

---

## 2회차 완료 체크리스트

- [ ] Caddy 기본 응답 테스트 성공 (`Hello from Caddy!`)
- [ ] whoami 리버스 프록시 성공
- [ ] nginx 서비스 직접 추가 성공
- [ ] Traefik과 설정 방식 차이 체감

---

## 핵심 개념 정리

### Caddyfile 기본 구조

```
# 기본 형태
도메인 {
    지시어 인자
}

# 예시
example.com {
    reverse_proxy backend:8080
}

# 포트만 지정 (도메인 없이)
:80 {
    respond "Hello"
}
```

### 자주 쓰는 지시어

| 지시어 | 용도 | 예시 |
|--------|------|------|
| `reverse_proxy` | 리버스 프록시 | `reverse_proxy app:3000` |
| `file_server` | 정적 파일 서빙 | `file_server` |
| `respond` | 고정 응답 | `respond "OK" 200` |
| `redir` | 리다이렉트 | `redir https://{host}{uri}` |
| `encode` | 압축 | `encode gzip` |

---

## 정리 명령어

```bash
# 중지 및 삭제
docker compose down

# 볼륨까지 삭제
docker compose down -v

# 네트워크 삭제
docker network rm web
```

---

## Traefik vs Caddy 최종 비교

| 항목 | Traefik | Caddy |
|------|---------|-------|
| **장점** | Docker 자동 감지, 대시보드 | 설정 간결, 자동 HTTPS |
| **단점** | 설정 복잡, 학습 곡선 | Docker 연동 수동 |
| **적합한 상황** | 컨테이너 많음, 동적 환경 | 서비스 적음, 빠른 설정 |

---

## 다음 단계

3회차에서는 **Caddy + Traefik 하이브리드** 구성을 만들어봅니다.
- Caddy: 외부 진입점 (HTTPS)
- Traefik: 내부 Docker 라우팅

두 도구의 장점을 결합한 구성을 직접 체험해보세요!
