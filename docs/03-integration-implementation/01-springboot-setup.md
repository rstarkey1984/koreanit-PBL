# Spring Boot 개발환경 확인 및 실행 (실습)

이 문서는 **이미 구성된 Gradle 기반 Spring Boot 프로젝트를 기준으로**
개발환경을 확인하고, 서버를 실제로 실행해보는 실습 단계이다.

새 프로젝트를 생성하지 않으며,
**기존 프로젝트를 수정·확장해 나가는 방식**으로 진행한다.

---

## 1. 실습 목표

* JDK 17 환경에서 프로젝트가 정상 동작하는지 확인한다
* Gradle Wrapper를 통해 서버를 실행한다
* VSCode 기준 개발 흐름에 익숙해진다

---

## 2. 개발 환경 기준

본 실습은 다음 환경을 전제로 한다.

* JDK 17
* Gradle Wrapper 사용
* IDE: VSCode
* 실행 환경: 로컬 또는 Docker

> 이미 프로젝트가 구성되어 있으며, 새로 생성하지 않는다.

---

## 3. 프로젝트 기본 확인

프로젝트 루트에서 다음 파일들의 존재를 확인한다.

* build.gradle
* settings.gradle
* gradlew / gradlew.bat
* src/main/java/**/Application.java
* src/main/resources/application.yml (또는 profile 별 yml)

이 중 하나라도 누락되면 이후 실습을 진행할 수 없다.

---

## 4. JDK 버전 확인

VSCode 터미널에서 다음 명령어를 실행한다.

```bash
java -version
```

출력 결과에 `17`이 포함되어야 한다.

---

## 5. Gradle 실행 확인

프로젝트 루트에서 다음 명령어를 실행한다.

```bash
./gradlew --version
```

Gradle 정보가 정상 출력되면 다음 단계로 진행한다.

---

## 6. 애플리케이션 실행

### 로컬 실행

```bash
./gradlew bootRun
```

실행 방식은 프로젝트 구성에 따라 하나만 선택해도 된다.

---

## 7. 실행 확인

서버가 기동되면 다음을 확인한다.

* Spring Boot 시작 로그가 출력된다
* 설정된 포트에서 서버가 대기 상태가 된다

---

## 8. 이 단계의 정리

이 단계에서는

* 기존 프로젝트 환경 확인
* 서버 실행

까지만 수행한다.

다음 단계부터는
실제 코드를 수정하며
**컨트롤러를 하나씩 분리하는 작업**을 시작한다.

---

## 다음 단계

→ [**REST 컨트롤러 기본 분리 실습**](02-rest-controller.md)
