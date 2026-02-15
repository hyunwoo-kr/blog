---
title: "Caddy + Traefik 하이브리드 학습 가이드 3회차"
date: 2025-02-01
categories:
- devops
tags:
- caddy
- traefik
- docker
- reverse-proxy
- hybrid-architecture
keywords:
- caddy traefik hybrid
- edge proxy
- docker routing
- basic auth
author: "hyunwoo"
draft: false
---

Caddy를 외부 HTTPS 종단으로, Traefik을 내부 Docker 자동 감지 및 동적 라우팅으로 사용하는 하이브리드 아키텍처를 구성합니다. 각 도구의 강점만 활용하는 실전 구성입니다.

<!--more-->

WSL2 환경에서 Caddy와 Traefik을 결합한 하이브리드 구성을 직접 만들어보는 실습 가이드입니다.

---

## 학습 목표

| 배울 것 | 설명 |
|---------|------|
| 하이브리드 아키텍처 | 두 도구의 역할 분담 이해 |
| Caddy -> Traefik 연동 | 외부 프록시에서 내부 프록시로 트래픽 전달 |
| 복잡도 체감 | 단일 도구 vs 하이브리드의 차이 |
| 최종 선택 기준 | 어떤 구성이 나에게 맞는지 판단 |

---

## 아키텍처 개요

```
+─────────────────────────────────────────────────────────+
|  인터넷 / 외부 요청                                      |
|                          |                              |
|                     :80, :443                           |
|                          v                              |
|  +───────────────────────────────────────────+          |
|  |              CADDY (Edge)                  |          |
|  |  - 외부 HTTPS 종단                         |          |
|  |  - 도메인 기반 라우팅                      |          |
|  |  - 보안 헤더                               |          |
|  +─────────────────────┬─────────────────────+          |
|                        |                                |
|                   HTTP (내부)                           |
|                        v                                |
|  +───────────────────────────────────────────+          |
|  |            TRAEFIK (Internal)              |          |
|  |  - Docker 서비스 자동 발견                 |          |
|  |  - 라벨 기반 동적 라우팅                   |          |
|  |  - 대시보드                                |          |
|  +──────┬──────────────┬──────────────┬──────+          |
|         |              |              |                 |
|         v              v              v                 |
|   +──────────+   +──────────+   +──────────+           |
|   |  whoami  |   |  nginx   |   |  기타    |           |
|   +──────────+   +──────────+   +──────────+           |
+─────────────────────────────────────────────────────────+
```

---

## 왜 하이브리드인가?

| Caddy 담당 | Traefik 담당 |
|-----------|-------------|
| 외부 HTTPS (Let's Encrypt) | Docker 자동 감지 |
| 보안 헤더 | 라벨 기반 라우팅 |
| Rate Limiting | 대시보드 모니터링 |
| 단순한 Edge 설정 | 동적 서비스 관리 |

**장점**: 각 도구의 강점만 활용
**단점**: 관리 포인트 2개, 복잡도 증가

---

## 사전 준비

```bash
# 2회차 정리
cd ~/lab/caddy-practice-2
docker compose down

# 3회차 디렉토리 생성
mkdir -p ~/lab/hybrid-practice-3
cd ~/lab/hybrid-practice-3
```

---

## Step 1: 디렉토리 구조 만들기

```bash
mkdir -p traefik
touch docker-compose.yml
touch Caddyfile
touch traefik/traefik.yml
```

**구조:**

```
~/lab/hybrid-practice-3/
├── docker-compose.yml
├── Caddyfile           # Caddy 설정 (외부)
└── traefik/
    └── traefik.yml     # Traefik 설정 (내부)
```

---

## Step 2: Traefik 설정 (내부 라우터)

`traefik/traefik.yml` 작성:

```yaml
api:
  dashboard: true
  insecure: true

entryPoints:
  web:
    address: ":80"

providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
    network: internal

log:
  level: INFO
```

**1, 2회차와 다른 점:**
- `network: internal` 추가 - 내부 네트워크 지정
- 외부 포트 노출 안 함 (Caddy가 담당)

---

## Step 3: Caddyfile 설정 (외부 Edge)

`Caddyfile` 작성:

```
# 모든 요청을 Traefik으로 전달
http://*.localhost {
    reverse_proxy traefik:80
}

# Traefik 대시보드 접근용
http://traefik.localhost {
    reverse_proxy traefik:8080
}
```

**핵심 포인트:**
- `*.localhost`: 모든 서브도메인을 Traefik으로 전달
- `traefik.localhost`: 대시보드 접근 경로
- `http://` 접두사로 자동 HTTPS 비활성화 (학습용)

---

## Step 4: Docker Compose 작성

```yaml
services:
  # ========== Edge Proxy (외부) ==========
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
      - external
      - internal  # Traefik과 통신용
    depends_on:
      - traefik

  # ========== Internal Router ==========
  traefik:
    image: traefik:v2.11
    container_name: traefik
    # ports 없음! Caddy를 통해서만 접근
    expose:
      - "80"
      - "8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./traefik/traefik.yml:/etc/traefik/traefik.yml:ro
    networks:
      - internal

  # ========== 서비스들 ==========
  whoami:
    image: traefik/whoami
    container_name: whoami
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.whoami.rule=Host(`whoami.localhost`)"
      - "traefik.http.routers.whoami.entrypoints=web"
    networks:
      - internal

networks:
  external:
    name: external
  internal:
    name: internal

volumes:
  caddy_data:
  caddy_config:
```

**핵심 차이점:**

| 항목 | 1회차 (Traefik) | 2회차 (Caddy) | 3회차 (하이브리드) |
|------|----------------|--------------|------------------|
| 외부 포트 | Traefik 80 | Caddy 80 | **Caddy만** 80 |
| Traefik 포트 | 80, 8080 노출 | 없음 | **expose만** (내부) |
| 네트워크 | 1개 | 1개 | **2개** (분리) |

---

## Step 5: 실행 및 테스트

```bash
# 실행
docker compose up -d

# 로그 확인
docker compose logs -f
```

### 테스트 1: whoami 서비스

```bash
curl -H "Host: whoami.localhost" http://localhost
```

**트래픽 흐름:**

```
curl -> Caddy(:80) -> Traefik(:80) -> whoami(:80)
```

### 테스트 2: Traefik 대시보드

```bash
curl -H "Host: traefik.localhost" http://localhost
```

또는 브라우저에서: `/etc/hosts`에 `127.0.0.1 traefik.localhost` 추가 후 접속

---

## Step 6: nginx 서비스 추가 (직접 해보기)

nginx를 추가하고 `nginx.localhost`로 접속되게 만들어보세요.

**힌트:**
- docker-compose.yml에 nginx 서비스 추가
- Traefik 라벨 추가 (1회차와 동일)
- Caddyfile 수정 불필요 (`*.localhost`가 이미 처리)

**정답 (스스로 시도 후 확인):**

docker-compose.yml에 추가:

```yaml
nginx:
  image: nginx:alpine
  container_name: nginx
  labels:
    - "traefik.enable=true"
    - "traefik.http.routers.nginx.rule=Host(`nginx.localhost`)"
    - "traefik.http.routers.nginx.entrypoints=web"
  networks:
    - internal
```

**Caddy 수정이 필요 없는 이유:**
- `*.localhost`가 모든 서브도메인을 Traefik으로 전달
- 새 서비스는 Traefik 라벨만 추가하면 자동 등록

---

## 3회차 완료 체크리스트

- [ ] Caddy -> Traefik 연동 성공
- [ ] whoami 서비스 접근 성공
- [ ] Traefik 대시보드 접근 성공
- [ ] nginx 서비스 직접 추가 성공
- [ ] 하이브리드 복잡도 체감

---

## 프로덕션 보안 설정: Basic Auth

프로덕션 환경에서는 Traefik 대시보드와 민감한 서비스에 **인증을 추가**해야 합니다.

### Step 1: 비밀번호 해시 생성

Caddy 컨테이너에서 bcrypt 해시를 생성합니다:

```bash
# Caddy 컨테이너에서 비밀번호 해시 생성
docker exec caddy caddy hash-password --plaintext 'your-secure-password'

# 출력 예시:
# $2a$14$Hc6Y4qCld5NdPqhYffIWN.qJp1Qf4UA5L/FlgVRmUkLvUI2I1MD3K
```

> **중요**: `your-secure-password`를 실제 안전한 비밀번호로 변경하세요!

### Step 2: Caddyfile에 Basic Auth 추가

```
# Traefik 대시보드 (Basic Auth 보호)
http://traefik.localhost {
    route {
        basicauth {
            admin "$2a$14$Hc6Y4qCld5NdPqhYffIWN.qJp1Qf4UA5L/FlgVRmUkLvUI2I1MD3K"
        }
        reverse_proxy traefik:8080
    }
}

# whoami 서비스 (Basic Auth 보호)
http://whoami.localhost {
    route {
        basicauth {
            admin "$2a$14$Hc6Y4qCld5NdPqhYffIWN.qJp1Qf4UA5L/FlgVRmUkLvUI2I1MD3K"
        }
        reverse_proxy traefik:80
    }
}
```

**핵심 포인트:**
- `route { }` 블록으로 처리 순서 명시 (basicauth -> reverse_proxy)
- 비밀번호 해시는 `"따옴표"`로 감싸야 함 (`$` 기호 해석 방지)
- 사용자명:비밀번호 형식 (예: `admin:your-password`)

### Step 3: 설정 적용

```bash
# Caddy 재시작 (reload가 아닌 restart 필요)
docker compose restart caddy

# 테스트: 인증 없이 접근 시 401 반환
curl -I http://traefik.localhost
# HTTP/1.1 401 Unauthorized

# 테스트: 인증 포함 시 200 반환
curl -I -u admin:your-password http://traefik.localhost
# HTTP/1.1 200 OK
```

### 보안 설정 비교

| 항목 | 학습용 설정 | 프로덕션 설정 |
|------|-----------|-------------|
| 인증 | 없음 | Basic Auth 필수 |
| HTTPS | `http://` (비활성화) | 도메인 지정 (자동 HTTPS) |
| 비밀번호 | - | bcrypt 해시 사용 |
| 접근 제어 | 모두 허용 | 인증된 사용자만 |

### 프로덕션 Caddyfile 예시 (실제 도메인)

```
# 보안 헤더 스니펫
(security_headers) {
    header {
        Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
        X-Content-Type-Options "nosniff"
        X-Frame-Options "SAMEORIGIN"
        -Server
    }
}

# Traefik 대시보드 (HTTPS + Basic Auth)
traefik.example.com {
    import security_headers
    route {
        basicauth {
            admin "$2a$14$..."
        }
        reverse_proxy traefik:8080
    }
}

# 공개 서비스 (인증 없이)
public.example.com {
    import security_headers
    reverse_proxy traefik:80
}
```

### 자주 하는 실수

| 실수 | 증상 | 해결 |
|------|------|------|
| 따옴표 없이 해시 사용 | Caddy 설정 에러 | `"$2a$14$..."` 따옴표로 감싸기 |
| `reload` 대신 사용 | 인증 미적용 | `docker compose restart caddy` |
| `route` 블록 누락 | 인증 우회됨 | `route { basicauth ... }` 사용 |

---

## 트러블슈팅

### 문제: 502 Bad Gateway

**원인**: Caddy가 Traefik에 연결 못 함

**해결**:

```bash
# 네트워크 확인
docker network ls
docker network inspect internal

# Traefik이 internal 네트워크에 있는지 확인
docker inspect traefik | grep -A 10 Networks
```

### 문제: Traefik 대시보드 접근 불가

**원인**: 8080 포트가 expose만 되어 있음

**해결**: Caddyfile에서 `traefik:8080`으로 프록시 중인지 확인

### 문제: 새 서비스가 라우팅 안 됨

**원인**: 서비스가 `internal` 네트워크에 없음

**해결**: `networks: - internal` 추가 확인

---

## 최종 비교: 어떤 구성을 선택할까?

| 구성 | 장점 | 단점 | 추천 상황 |
|------|------|------|----------|
| **Traefik 단독** | Docker 자동화, 대시보드 | 설정 복잡 | 컨테이너 많은 동적 환경 |
| **Caddy 단독** | 설정 간결, 자동 HTTPS | Docker 연동 수동 | 서비스 적은 정적 환경 |
| **하이브리드** | 각 도구의 장점 결합 | 관리 포인트 2개 | 보안/성능 최적화 필요 시 |

### 나의 선택 가이드

```
Q: 서비스가 자주 추가/삭제되나요?
├── Yes -> Traefik (자동 감지)
└── No -> Caddy (간단한 설정)

Q: 자동 HTTPS가 중요한가요?
├── Yes -> Caddy 포함 (단독 또는 하이브리드)
└── No -> Traefik 단독 OK

Q: 대시보드 모니터링이 필요한가요?
├── Yes -> Traefik 포함
└── No -> Caddy 단독 OK

Q: DMZ/네트워크 분리가 필요한가요?
├── Yes -> 하이브리드
└── No -> 단일 도구 선택
```

---

## 정리 명령어

```bash
# 중지 및 삭제
docker compose down

# 볼륨까지 삭제
docker compose down -v

# 네트워크 삭제
docker network rm external internal
```

---

## 다음 단계

3회 연습을 모두 마쳤습니다! 이제 결정할 차례입니다:

1. **세 가지 구성을 모두 체험**했으니 어떤 것이 가장 편했는지 생각해보세요
2. **실제 homelab에 적용**할 구성을 선택하세요
3. **프로덕션 설정** (실제 도메인, Let's Encrypt 인증서 등)을 추가하세요
