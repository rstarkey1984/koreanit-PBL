# 사전 준비 사항
- git
- mysql
- docker

## 프로젝트 디렉터리

```
/projects/koreanit-pbl/web-app 
  demo -- 스프링부트 api 서버
  nginx -- nginx 설정파일
  var/www -- nginx document root 폴더 ( html,css,js,php 등 )
```

## jdk17 설치
```bash
sudo apt install openjdk-17-jdk
```

# 소스코드 github에서 다운받기

이 저장소는 **강의안(md) + 소스코드**가 함께 들어 있다.
WSL 환경에서는 **소스코드(`web-app`)만** 받기 위해
Git의 **sparse-checkout** 기능을 사용한다.

```bash
git clone --filter=blob:none --no-checkout https://github.com/rstarkey1984/koreanit-PBL.git && cd koreanit-PBL && git sparse-checkout init --cone && git sparse-checkout set web-app && git checkout HEAD -- web-app
```
```bash
rm -rf .git
```

# PHP-FPM 이미지 빌드
```bash
docker build -f web-app/Dockerfile -t custom-php-fpm:8.3-alpine .
```

---

# 스프링부트 프로젝트 실행

```bash
cd web-app/demo
```
### gradlew 실행권한 +
```bash
chmod +x gradlew
```

### 실행:
```bash
./gradlew bootRun
```

### 확인:

`http://localhost:9092/`

---
# Docker Compose 설정을 기반으로 컨테이너 관리

### docker compose 로 실행된 컨테이너 중지
```bash
docker compose down
```

### 중단된 모든 컨테이너 삭제
```bash
docker rm $(docker ps -aq -f status=exited)
```

### 스프링부트 API Docker 이미지 빌드:
```bash
cd demo
```
```bash
docker build -t web-app-api:1.0 .
```

빌드 성공시:
```bash
docker images | grep web-app-api
```

Docker Compose 실행:
```bash
docker compose up -d
```

확인:
`http://www.localhost/`

Docker Compose 중지:
```bash
docker compose down
```

NGINX 로그확인:
```bash
docker logs -f web-nginx
```

PHP-FPM 로그확인:
```bash
docker logs -f web-php
```
