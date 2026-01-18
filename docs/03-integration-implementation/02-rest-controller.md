# REST 컨트롤러 기본 분리 실습

이 문서는 **이미 실행 중인 Spring Boot API 프로젝트에**
기존에 사용하던 `ApiController.java`와 분리된
**첫 번째 단일 책임 REST 컨트롤러를 추가**하는 실습이다.

이번 단계의 목적은
"컨트롤러를 이렇게 나누기 시작하는구나"를 체감하는 것이다.

---

## 1. 실습 목표

* 기존 `ApiController.java`는 그대로 유지한다
* 새로운 단일 책임 컨트롤러를 추가한다
* 공통 응답 포맷을 적용한 API 응답을 확인한다

---

## 2. 전제 (현재 프로젝트 상태)

현재 프로젝트에는 다음과 같은 컨트롤러가 이미 존재한다.

* `ApiController.java`
  → 기존에 사용하던 **통합(몰빵) 컨트롤러**
* `HelloController.java`
* `RedisPingController.java`

이 중 **`ApiController.java`는 현행 시스템의 기준 코드**로 남겨둔다.
이번 실습에서는 해당 파일을 수정하지 않는다.

---

## 3. 컨트롤러 패키지 위치

컨트롤러는 다음 패키지 아래에 위치한다.

```text
src/main/java/com/example/demo/controller/
```

이번 실습에서는
**패키지 경로를 변경하지 않는다.**

---

## 4. HealthController 생성

다음 파일을 생성한다.

```text
src/main/java/com/example/demo/controller/HealthController.java
```

```java
package com.example.demo.controller;

import com.example.demo.common.ApiResponse;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HealthController {

    @GetMapping("/api/health")
    public ApiResponse<String> health() {
        return ApiResponse.ok("OK");
    }
}
```

이 컨트롤러는 다음 특징을 가진다.

* 하나의 API만 담당한다
* 비즈니스 로직이 없다
* 공통 응답 포맷(`ApiResponse`)을 사용한다

---

## 5. 애플리케이션 실행 (bootRun)

프로젝트 루트에서 다음 명령어로 서버를 실행한다.

```bash
./gradlew bootRun
```

---

## 6. API 응답 확인

Spring Boot 설정에 따라
서버는 **9092 포트**에서 실행된다.

다음 요청으로 결과를 확인한다.

```bash
curl http://localhost:9092/api/health
```

응답 예시는 다음과 같다.

```json
OK
```

---

## 7. 이 실습의 의미

이 단계에서 확인해야 할 핵심은 다음과 같다.

* 기존 `ApiController.java`를 건드리지 않고도
* 새로운 컨트롤러를 추가할 수 있다
* 구조를 깨지 않고 점진적으로 분리가 가능하다

이제부터는
`ApiController.java` 안의 기능을
**하나씩 새로운 컨트롤러로 옮기는 작업**을 진행한다.

---

## 다음 단계

→ [**공통 응답 포맷 적용**](03-common-response.md)
