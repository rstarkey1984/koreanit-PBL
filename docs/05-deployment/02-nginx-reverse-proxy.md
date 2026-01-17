# Nginx Reverse Proxy 설정

이 문서는 docker-compose로 실행 중인 Nginx가
**외부 요청을 Spring Boot API로 전달**하도록 Reverse Proxy를 설정하는 단계다.

목표는 다음과 같다.

* 클라이언트는 `http://www.localhost`로만 접근한다
* `/api/*` 요청은 Spring Boot(API)로 전달된다
* 정적 파일/PHP 등은 기존 설정을 유지한다

---

## 전제

* docker-compose로 `nginx`, `api`, `redis`가 실행 중이다
* API는 컨테이너 내부 포트 `9092`로 실행된다
* Nginx 컨테이너는 같은 compose 네트워크에서 `api:9092`로 접근 가능하다

---

## 1) Nginx 설정 파일 위치

프로젝트 기준:

```text
nginx/www.localhost.conf
```

그리고 docker-compose에서 다음처럼 마운트된다고 가정한다.

```yaml
nginx:
  volumes:
    - ./nginx:/etc/nginx/conf.d:ro
```

---

## 2) /api 프록시 설정 추가

`www.localhost.conf`의 server 블록에 다음 location을 추가한다.

```nginx
location /api/ {
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    proxy_pass http://api:9092;
}
```

핵심은 이 한 줄이다.

* `api`는 docker-compose 서비스 이름
* `9092`는 Spring Boot 포트

---

## 3) Nginx 리로드

설정 변경 후 Nginx 컨테이너를 재시작하거나 리로드한다.

재시작:

```bash
docker compose restart nginx
```

또는 컨테이너 내부에서 리로드:

```bash
docker exec -it web-nginx nginx -s reload
```

---

## 4) 프록시 동작 확인

```bash
curl http://www.localhost/api/health
curl http://www.localhost/api/news
```

정상이라면 `ApiResponse` 포맷이 내려온다.

---

## 5) 자주 터지는 문제

### (1) 502 Bad Gateway

* Nginx가 `api:9092`에 연결 실패

체크:

```bash
docker compose ps
```

* api 컨테이너가 떠 있는지
* api 로그에 에러가 없는지

```bash
docker compose logs -f api
```

### (2) /api 라우팅이 다른 location에 먹힘

* `/` 또는 정적/PHP location이 먼저 매칭되는 경우
* `/api/` location을 server 블록 상단에 배치한다

---

## 다음 단계

→ [**배포 점검 체크리스트**](/docs/05-deployment/03-deploy-checklist.md)
