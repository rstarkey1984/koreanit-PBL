# 배포 점검 체크리스트

이 문서는 배포 실습에서 자주 발생하는 문제를 빠르게 점검하기 위한 체크리스트다.

---

## 1) 컨테이너 상태

```bash
docker compose ps
```

* nginx / api / redis가 모두 `running` 인가

---

## 2) API 로그

```bash
docker compose logs -f api
```

* Spring Boot가 9092로 정상 기동했는가
* DB/Redis 연결 에러가 있는가

---

## 3) MySQL 연결

컨테이너에서 호스트 MySQL에 접근하는지 확인한다.

* URL은 `host.docker.internal:3308`인가
* 호스트에서 3308 포트가 열려 있는가

```bash
ss -lntp | grep 3308 || true
```

---

## 4) Redis 연결

* Spring 설정의 redis host가 `redis`인가
* redis 컨테이너가 떠 있는가

```bash
docker compose logs -f redis
```

---

## 5) Nginx 프록시

```bash
curl -i http://www.localhost/api/health
```

* 200이 나오면 OK
* 502면 Nginx → API 연결 실패

Nginx 로그:

```bash
docker compose logs -f nginx
```

---

## 6) 최종 확인

* 직접 접근(개발 방식): `http://localhost:9092/api/news`
* 프록시 경유(배포 방식): `http://www.localhost/api/news`

배포에서는 **프록시 경유**가 정답이다.
