# 빌드 및 배포

이 문서는 검증된 소스를 실제 실행 환경에 배포하는 절차를 정의한다.
이 단계에서는 **명령어의 의미와 결과 확인**이 핵심이다.

---

## 학습 목표

* Docker 이미지 빌드 흐름을 이해한다
* docker compose 기반 배포 절차를 수행한다
* 로그를 통해 배포 성공 여부를 판단한다

---

## 1. 빌드 단계

### 1-1. API 이미지 빌드

```bash
cd web-app/demo
```

```bash
docker build -t web-app-api:1.0 .
```

이미지 생성 여부 확인:

```bash
docker image ls
```

---

## 2. 배포 단계

### 2-1. docker compose 기동

```bash
docker compose up -d
```

컨테이너 상태 확인:

```bash
docker compose ps
```

---

## 3. 로그 기반 검증

### 3-1. API 로그

```bash
docker compose logs api
```

확인 항목:

* 서버가 정상 기동되었는가
* DB / Redis 연결 에러가 없는가

---

### 3-2. Nginx / Redis 로그

```bash
docker compose logs nginx

docker compose logs redis
```

---

## 4. 통신 테스트

### 4-1. 외부 접근 테스트

```bash
curl http://www.localhost/api/health
```

정상 응답 시 배포 성공으로 판단한다.

---

## 5. 수정 후 재배포 규칙

### 코드 수정 시

```bash
docker build -t web-app-api:1.0 .
docker compose up -d --force-recreate api
```

### Nginx 설정 수정 시

```bash
docker compose restart nginx
```

---

## 6. 이 단계의 완료 기준

* 모든 컨테이너가 running 상태다
* 외부 요청이 정상 응답을 반환한다
* 로그에 치명적인 에러가 없다

이로써 배포 실습 단계를 완료한다.


