# 서비스 레이어 분리 

이 문서는 **컨트롤러에 직접 작성되어 있던 로직을 서비스 레이어로 이동**하는 실습이다.

이전 단계에서

* 응답 포맷을 공통화했고
* 예외 처리를 전역으로 분리했기 때문에

이제 컨트롤러는 **요청/응답만 담당**하고,
실제 로직은 서비스가 담당하도록 구조를 나눈다.

---

## 1. 실습 목표

* 컨트롤러의 책임을 최소화한다
* 비즈니스 로직을 서비스 클래스로 이동한다
* 컨트롤러 → 서비스 호출 흐름을 이해한다

---

## 2. 대상 API 선정

이번 실습에서는 기존에 사용하던 API 중
가장 단순한 기능 하나를 대상으로 한다.

예시:

* health 체크
* hello 메시지 반환
* ping 테스트

여기서는 **Health API**를 기준으로 진행한다.

---

## 3. 서비스 패키지 확인

서비스 클래스는 다음 패키지에 위치한다.

```text
src/main/java/com/example/demo/service/
```

해당 패키지가 없다면 생성한다.

---

## 4. HealthService 생성

다음 파일을 생성한다.

```text
src/main/java/com/example/demo/service/HealthService.java
```

```java
package com.example.demo.service;

import org.springframework.stereotype.Service;

@Service
public class HealthService {

    public String check() {
        return "HealthService OK";
    }
}
```

---

## 5. HealthController 수정

컨트롤러에서 직접 처리하던 로직을
서비스 호출로 변경한다.

```java
package com.example.demo.controller;

import com.example.demo.common.ApiResponse;
import com.example.demo.service.HealthService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/** 상태 확인 요청을 처리하는 컨트롤러 */
@RestController
public class HealthController {

    /** 상태 확인 로직을 담당하는 Service */
    private final HealthService healthService;

    /** 생성자 주입 */
    public HealthController(HealthService healthService) {
        this.healthService = healthService;
    }

    /** 헬스 체크 API */
    @GetMapping("/api/health")
    public ApiResponse<String> health() {
        return ApiResponse.ok(healthService.check());
    }
}
```

## Spring 프레임워크 구조 (어노테이션 · 생성자 주입)

* `@Controller`, `@Service`, `@Repository` 는 **서버 구성 요소 역할 표시**

* Spring은 해당 클래스를 **서버 실행 시 자동으로 생성·관리**

* **생성자 주입**은 서버 구성 요소 간 의존 관계를 연결하는 방식

* 필요한 객체를 생성자 파라미터로 자동 전달받음

> 서버 구성 요소는 역할에 따라 분리되고,
> 생성자를 통해 흐름이 연결된다.


---

## 6. 실행 및 확인

서버를 실행한다.

```bash
./gradlew bootRun
```

다음 요청으로 결과를 확인한다.

```bash
curl http://localhost:9092/api/health
```

응답은 이전과 동일해야 한다.

```json
{
  "ok": true,
  "data": "OK",
  "message": null
}
```

---

## 이 실습의 의미

이 단계에서 확인해야 할 핵심은 다음과 같다.

* 컨트롤러는 요청을 받고 응답을 반환한다
* 서비스는 로직만 담당한다
* 구조를 분리해도 기능은 변하지 않는다

이제 기존 `ApiController.java`에 있는 로직도
같은 방식으로 하나씩 서비스로 옮길 수 있다.

---

## 다음 단계

→ [**레포지토리 레이어 분리**](06-repository-layer.md)
