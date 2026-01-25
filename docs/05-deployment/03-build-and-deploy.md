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

---

## 7. Docker Compose 자주 쓰는 명령어 정리

> 실습/강의에서 **가장 많이 쓰는 docker compose 명령어**를 목적별로 정리한 표다.
> (Docker Compose v2 기준: `docker compose`)


### 7-1. 기본 실행 / 중지

| 명령어                      | 설명                  | 비고            |
| ------------------------ | ------------------- | ------------- |
| `docker compose up`      | 컨테이너 생성 + 실행        | 로그가 터미널에 출력됨  |
| `docker compose up -d`   | 백그라운드 실행 (detached) | 실무에서 가장 많이 사용 |
| `docker compose down`    | 컨테이너 중지 + 삭제        | 네트워크도 함께 제거   |
| `docker compose stop`    | 컨테이너만 중지            | 삭제는 안 함       |
| `docker compose start`   | 중지된 컨테이너 재시작        | 기존 컨테이너 사용    |
| `docker compose restart` | 컨테이너 재시작            | 설정 변경 없을 때    |

## 7-2. 상태 확인

| 명령어                     | 설명              |
| ----------------------- | --------------- |
| `docker compose ps`     | 컨테이너 상태 목록      |
| `docker compose ls`     | compose 프로젝트 목록 |
| `docker compose images` | 사용 중인 이미지 목록    |


## 7-3. 로그 확인

| 명령어                              | 설명         | 팁        |
| -------------------------------- | ---------- | -------- |
| `docker compose logs`            | 전체 서비스 로그  | 길면 너무 많음 |
| `docker compose logs nginx`      | 특정 서비스 로그  | 서비스명 사용  |
| `docker compose logs -f`         | 실시간 로그 확인  | 서버 실행 감시 |
| `docker compose logs --tail=100` | 마지막 N줄만 출력 | 디버깅용     |


## 7-4. 컨테이너 접속

| 명령어                             | 설명        | 예시            |
| ------------------------------- | --------- | ------------- |
| `docker compose exec 서비스명 sh`   | 컨테이너 쉘 접속 | alpine 기반     |


## 7-5. 강제 정리 / 초기화

| 명령어                             | 설명         | 주의         |
| ------------------------------- | ---------- | ---------- |
| `docker compose down -v`        | 볼륨까지 삭제    | DB 데이터 날아감 |
| `docker compose down --rmi all` | 이미지까지 삭제   | 재빌드 필요     |
| `docker compose rm -f`          | 컨테이너 강제 삭제 | 중지 상태에서    |

---

## 다음 단계

→ [**산출물 제출 안내**](04-outro.md)