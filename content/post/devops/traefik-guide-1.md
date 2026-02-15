---
title: "Traefik 학습 가이드 1회차 - 기본 구성"
date: 2025-02-01
categories:
- devops
tags:
- traefik
- docker
- reverse-proxy
- docker-compose
keywords:
- traefik
- reverse proxy
- docker provider
- 리버스 프록시
author: "hyunwoo"
draft: false
---

Traefik은 Docker 컨테이너를 자동 감지하여 라우팅하는 리버스 프록시입니다. Docker 라벨로 라우팅 규칙을 설정하고, 대시보드로 실시간 모니터링할 수 있습니다.

<!--more-->

WSL2 환경에서 Traefik을 직접 구성하면서 리버스 프록시의 기본 개념을 익히는 실습 가이드입니다.

---

## 학습 목표

| 배울 것 | 설명 |
|---------|------|
| Traefik 기본 개념 | Entrypoint, Router, Service, Provider |
| Docker Provider | 컨테이너 자동 발견 원리 |
| 라벨 기반 라우팅 | Docker 라벨로 라우팅 설정하는 법 |
| 대시보드 | Traefik 상태 모니터링 |

---

## 사전 준비

시작 전에 확인하세요:

```bash
# Docker 설치 확인
docker --version

# Docker Compose 확인
docker compose version

# 작업 디렉토리 생성
mkdir -p ~/lab/traefik-practice-1
cd ~/lab/traefik-practice-1
```

> **체크포인트**: 두 명령 모두 버전이 출력되면 OK

---

## Step 1: 기본 구조 만들기

직접 타이핑해보세요:

```bash
# 디렉토리 구조 생성
mkdir -p traefik

# 필요한 파일 생성 (빈 파일)
touch docker-compose.yml
touch traefik/traefik.yml
```

**현재 구조:**

```
~/lab/traefik-practice-1/
├── docker-compose.yml
└── traefik/
    └── traefik.yml
```

---

## Step 2: Traefik 정적 설정

`traefik/traefik.yml` 파일을 직접 작성해보세요.

```yaml
# Traefik 정적 설정 파일
# 이 파일은 Traefik 시작 시 한 번만 읽힘

api:
  dashboard: true    # 대시보드 활성화
  insecure: true     # 인증 없이 접근 (연습용)

entryPoints:
  web:
    address: ":80"   # HTTP 트래픽 받을 포트

providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false   # 명시적 라벨 있는 것만 노출

log:
  level: DEBUG       # 연습 중엔 DEBUG로 상세 로그
```

**핵심 개념 설명:**

| 설정 | 역할 |
|------|------|
| `entryPoints` | 트래픽이 들어오는 문 (포트) |
| `providers.docker` | Docker 컨테이너를 자동 감지하는 기능 |
| `exposedByDefault: false` | 라벨 없는 컨테이너는 무시 (보안) |

---

## Step 3: Docker Compose 작성

`docker-compose.yml`을 직접 작성:

```yaml
services:
  traefik:
    image: traefik:v2.11
    container_name: traefik
    ports:
      - "80:80"      # HTTP
      - "8080:8080"  # Dashboard
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./traefik/traefik.yml:/etc/traefik/traefik.yml:ro
    networks:
      - web

networks:
  web:
    name: web
```

> **주의**: Traefik v3.0은 Docker API 버전 요구사항이 높아 일부 환경에서 호환 문제가 있습니다. **v2.11 사용을 권장**합니다.

**핵심 개념 설명:**

| 설정 | 역할 |
|------|------|
| `docker.sock` 마운트 | Traefik이 다른 컨테이너 정보를 읽을 수 있게 함 |
| `8080` 포트 | 대시보드 접속용 |
| `networks: web` | 서비스들이 통신할 공유 네트워크 |

---

## Step 4: 실행 및 확인

```bash
# 실행
docker compose up -d

# 로그 확인
docker compose logs -f traefik
```

**체크포인트 1**: 브라우저에서 `http://localhost:8080` 접속
- Traefik 대시보드가 보이면 성공

**체크포인트 2**: 대시보드에서 확인할 것
- Entrypoints: `web` 이 보이는지
- Providers: `docker` 가 연결됐는지

---

## Step 5: 테스트 서비스 추가

이제 실제 서비스를 Traefik에 연결해봅니다.

`docker-compose.yml`을 아래 내용으로 수정:

```yaml
services:
  traefik:
    image: traefik:v2.11
    container_name: traefik
    ports:
      - "80:80"
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./traefik/traefik.yml:/etc/traefik/traefik.yml:ro
    networks:
      - web

  # 테스트용 서비스
  whoami:
    image: traefik/whoami
    container_name: whoami
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.whoami.rule=Host(`whoami.localhost`)"
      - "traefik.http.routers.whoami.entrypoints=web"
    networks:
      - web

networks:
  web:
    name: web
```

**라벨 의미 해석:**

| 라벨 | 의미 |
|------|------|
| `traefik.enable=true` | 이 컨테이너를 Traefik에 등록 |
| `routers.whoami.rule=Host(...)` | whoami.localhost로 오면 여기로 라우팅 |
| `routers.whoami.entrypoints=web` | 80번 포트(web)로 들어온 트래픽 처리 |

---

## Step 6: 서비스 테스트

```bash
# 변경 적용
docker compose up -d

# 테스트 (Host 헤더로 도메인 시뮬레이션)
curl -H "Host: whoami.localhost" http://localhost
```

**체크포인트**: 응답에 컨테이너 정보가 표시되면 성공

```
Hostname: xxxxxx
IP: 172.x.x.x
RemoteAddr: 172.x.x.x:xxxxx
...
```

---

## Step 7: 두 번째 서비스 추가 (직접 해보기)

nginx 서비스를 추가하고 `nginx.localhost`로 접속되게 만들어보세요.

**힌트:**
- 이미지: `nginx:alpine`
- 라벨 구조는 whoami와 동일, 이름만 변경

**정답 (스스로 시도 후 확인):**

```yaml
nginx:
  image: nginx:alpine
  container_name: nginx
  labels:
    - "traefik.enable=true"
    - "traefik.http.routers.nginx.rule=Host(`nginx.localhost`)"
    - "traefik.http.routers.nginx.entrypoints=web"
  networks:
    - web
```

---

## 1회차 완료 체크리스트

- [ ] Traefik 대시보드 접속 성공
- [ ] whoami 서비스 curl 테스트 성공
- [ ] 대시보드에서 Router/Service 확인
- [ ] nginx 서비스 직접 추가 성공

---

## 핵심 개념 정리

### Traefik 구조

```
                  +─────────────────────────────────+
외부 요청 ───────>|         Entrypoints             |
(80, 443)         |    (트래픽 진입점)              |
                  +──────────────┬──────────────────+
                                 |
                  +──────────────v──────────────────+
                  |           Routers               |
                  |  (규칙에 따라 트래픽 분배)       |
                  |  Host(), Path(), Headers()...   |
                  +──────────────┬──────────────────+
                                 |
                  +──────────────v──────────────────+
                  |          Services               |
                  |   (실제 컨테이너로 전달)         |
                  +─────────────────────────────────+
```

### Docker Provider 작동 원리

1. Traefik이 Docker 소켓에 연결
2. 컨테이너 생성/삭제 이벤트 감지
3. `traefik.enable=true` 라벨이 있는 컨테이너만 등록
4. 라벨에서 라우팅 규칙 읽어서 자동 설정

---

## 정리 명령어

```bash
# 중지 및 삭제
docker compose down

# 볼륨까지 삭제 (깔끔하게 정리)
docker compose down -v

# 네트워크 삭제
docker network rm web
```

---

## 다음 단계

2회차에서는 **Caddy**를 사용해 같은 구성을 만들어봅니다.
두 도구의 설정 방식 차이를 직접 체감해보세요.
