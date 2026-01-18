# Docker Compose 배포 실행

이 문서는 개발 단계에서 `./gradlew bootRun`으로 실행하던 Spring Boot API를
**Docker 이미지로 빌드하고 docker-compose로 실행**하는 배포 실습이다.

이 프로젝트의 배포 구성은 다음 전제를 가진다.

* MySQL은 호스트(Ubuntu)에 apt로 설치되어 실행 중이다
* Redis는 docker-compose 컨테이너로 실행한다
* Nginx는 외부 진입점(Reverse Proxy) 역할을 한다

---

## 전제 확인

### 1) 호스트 MySQL 포트 확인

호스트에서 MySQL이 3308로 열려 있어야 한다.

```bash
ss -lntp | grep 3308 || true
```

### 2) docker-compose 실행 준비

프로젝트 루트에서 다음 파일이 존재한다고 가정한다.

* `docker-compose.yml`
* `Dockerfile`
* `.env`
* `nginx/www.localhost.conf`

---

## 핵심 포인트 (배포에서 반드시 바뀌는 것)

개발 환경에서는 `localhost`가 내 PC(WSL) 자체를 의미했다.

하지만 컨테이너 내부에서의 `localhost`는 **컨테이너 자기 자신**이다.

따라서 배포(컨테이너 실행)에서는 다음 값을 반드시 바꿔야 한다.

* MySQL: `localhost` → `host.docker.internal`
* Redis: `localhost` → `redis` (compose 서비스 이름)


---

## 1) API Docker 이미지 빌드

프로젝트 루트에서 Dockerfile 기준으로 API 이미지를 빌드한다.

```bash
docker build -t web-app-api:1.0 .
```

---

## 2) docker-compose 실행

```bash
docker compose up -d
```

상태 확인:

```bash
docker compose ps
```

로그 확인(문제 발생 시):

```bash
docker compose logs -f api
```

---

## 3) Nginx 프록시 경유 확인

Nginx가 외부 진입점이라면, 브라우저/클라이언트는 80 포트로만 접근한다.

예시(nginx가 `/api`를 API로 전달하는 경우):

```bash
curl http://www.localhost/api/health
curl http://www.localhost/api/news
```

만약 Nginx 설정이 아직 `/api`를 라우팅하지 않는다면,
다음 문서(Reverse Proxy 설정)에서 확정한다.

---

## 5) 자주 터지는 문제

### (1) DB 접속 실패

* 컨테이너에서 `localhost:3308`로 접속하려고 하면 실패한다
* 반드시 `host.docker.internal:3308`이어야 한다

### (2) Redis 접속 실패

* 컨테이너에서 Redis는 `localhost`가 아니다
* compose 서비스 이름인 `redis`로 접근해야 한다

### (3) Nginx에서 API로 연결 실패

* Nginx 설정에서 upstream 대상이 `api:9092`인지 확인한다
* `api`는 compose 서비스 이름이다

---

## 다음 단계

→ [**Nginx Reverse Proxy 설정**](/docs/05-deployment/02-nginx-reverse-proxy.md)
