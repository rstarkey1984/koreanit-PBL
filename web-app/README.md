# 개발 환경 및 프로젝트 실행 가이드

이 문서는
**PBL 프로젝트를 실행하기 위한 사전 환경 준비부터
Spring Boot API 서버와 Docker Compose 기반 실행까지의 전체 흐름**을 정리한다.

---

## 1. 사전 준비 사항

다음 도구들이 사전에 설치되어 있어야 한다.

* Git
* Docker / Docker Compose
* WSL2 (Ubuntu)

---

## 2. 프로젝트 디렉터리 구조

프로젝트는 다음과 같은 구조를 가진다.

```text
/projects/koreanit-pbl/web-app
  demo        # Spring Boot API 서버
  nginx       # Nginx 설정 파일
  var/www     # Nginx document root (html, css, js, php 등)
```

* `demo`

  * Java 기반 Spring Boot API 서버
* `nginx`

  * Reverse Proxy 및 정적 리소스 설정
* `var/www`

  * PHP / HTML / 정적 파일 영역

---

## 3. JDK 17 설치

Spring Boot 서버 실행을 위해 JDK 17을 설치한다.

```bash
sudo apt install openjdk-17-jdk
```

설치 확인:

```bash
java -version
```

---

## 4. 소스코드 다운로드 (Git sparse-checkout)

이 저장소에는 **강의안(md) + 전체 소스코드**가 함께 포함되어 있다.
WSL 환경에서는 **실습에 필요한 `web-app` 디렉터리만** 받는다.

### 4-1. 저장소 클론 및 sparse-checkout 설정

```bash
git clone --filter=blob:none --no-checkout https://github.com/rstarkey1984/koreanit-PBL.git
cd koreanit-PBL
git sparse-checkout init --cone
git sparse-checkout set web-app
git checkout HEAD -- web-app
```

### 4-2. Git 메타데이터 제거 (선택)

실습용 환경에서는 Git 관리가 필요 없으므로 `.git` 디렉터리를 제거한다.

```bash
rm -rf .git
```

---

## 5. PHP-FPM Docker 이미지 빌드
```bash
cd web-app/php
```

PHP 실행을 위한 커스텀 PHP-FPM 이미지를 빌드한다.

```bash
docker build -f Dockerfile -t custom-php-fpm:8.3-alpine .
```

---

## 6. Spring Boot 프로젝트 이미지 빌드

프로젝트 경로로 이동:
```bash
cd web-app/demo
```

이미지 빌드:
```bash
docker build -t web-app-api:1.0 .
```

이미지 확인:

```bash
docker images | grep web-app-api
```

---

## 7. Docker Compose 기반 컨테이너 관리

Docker Compose를 통해
Nginx / PHP-FPM / API 서버를 함께 관리한다.

### 7-1. 실행 중인 컨테이너 중지

```bash
docker compose down
```

### 7-2. 중단된 컨테이너 정리 (선택)

```bash
docker rm $(docker ps -aq -f status=exited)
```

---

## 8. Docker Compose 실행

```bash
docker compose up -d
```

서비스 확인:

```text
http://www.localhost/
```

중지:

```bash
docker compose down
```

---

## 10. 컨테이너 로그 확인

### Nginx 로그

```bash
docker compose logs nginx
```

### 스프링부트 api 로그

```bash
docker compose logs api
```
