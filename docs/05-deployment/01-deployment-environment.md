# 애플리케이션 배포 환경구성

## 학습 목표

* Docker 기반 배포 환경의 전체 구조를 이해한다
* 컨테이너와 호스트 간 네트워크 규칙을 설명할 수 있다
* 배포 실패를 유발하는 환경 설정 오류를 사전에 제거한다

---

## 1. 배포 아키텍처 개요

요청 흐름은 반드시 다음 구조를 따른다.

브라우저 → Nginx 컨테이너(80) → API 컨테이너(9092) → MySQL(호스트) / Redis(컨테이너)

```
(외부)
┌───────────────┐
│   Browser     │
│ Cookie: SESSION
└───────┬───────┘
        │  HTTP :80
        ▼
┌──────────────────────────────┐
│        Nginx (Docker)        │
│  container: web-nginx        │
│  host port: 80 -> 80         │
│  역할: reverse proxy         │
└───────────┬──────────────────┘
            │ proxy_pass (docker network)
            ▼
┌──────────────────────────────┐
│    Spring Boot API (Docker)   │
│  container: api               │
│  internal port: 9092          │
│  인증: Session (Spring Session)│
│  - JSESSIONID로 Redis 조회     │
└───────────┬─────────────┬────┘
            │             │
            │ TCP 3306    │ TCP 6379
            ▼             ▼
┌──────────────────────┐  ┌─────────────────────────┐
│   MySQL (HOST)       │  │     Redis (Docker)      │
│  host: 127.0.0.1     │  │  container: redis       │
│  host: host.docker...│  │  internal port: 6379    │
│  port: 3308          │  │  세션 저장소            │
└──────────────────────┘  └─────────────────────────┘
```

1. 브라우저가 `http://서버주소로` 요청

2. 요청은 먼저 Nginx 컨테이너(80) 에 도착

3. Nginx가

    - 정적 파일은 직접 응답

    - /api/* 요청은 API 컨테이너(9092) 로 프록시

4. Spring Boot API가

    - 세션 값 저장

    - 비즈니스 로직 수행

5. 데이터는

    - MySQL(호스트) : 영구 저장

    - Redis(컨테이너) : 캐시 / 인증 보조

6. 응답이 다시 Nginx → 브라우저로 반환

---

## 2. 컨테이너 네트워크 규칙

### 2-1. localhost 규칙

* 컨테이너 내부에서 `localhost`는 자기 자신을 의미한다
* 컨테이너에서 호스트 DB에 접근할 때 `localhost`를 사용하면 안 된다

### 2-2. 접근 규칙 정리

* 컨테이너 → 호스트 : `host.docker.internal`
* 컨테이너 → 컨테이너 : docker compose 서비스명
* 외부 → 서비스 : Nginx(80)

---

## 3. MySQL / Redis 환경 전제

### 3-1. MySQL

* MySQL은 호스트에서 실행된다
* 포트는 3308을 사용한다
* 컨테이너에서 접근 시 `host.docker.internal:3308`을 사용한다

### 3-2. Redis

* Redis는 docker compose로 실행된다
* API 컨테이너에서는 `redis:6379`로 접근한다

---

## 4. hosts 설정

배포 테스트를 위해 도메인을 고정한다.

```
127.0.0.1 www.localhost
```

이를 통해 실제 서비스와 유사한 접근 구조를 만든다.

---

## 5. 이 단계의 완료 기준

* docker compose가 정상 기동된다
* 컨테이너에서 호스트 MySQL 접근이 가능하다
* [www.localhost](http://www.localhost) 요청이 Nginx까지 도달한다

## 다음 단계

→ [**소스 검증**](/docs/05-deployment/02-source-verification.md)

