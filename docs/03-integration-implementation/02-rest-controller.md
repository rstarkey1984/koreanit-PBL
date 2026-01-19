# REST 컨트롤러 기본 분리

이 문서는 **통합 구현 단계에서 진행하는 첫 번째 구조 변경 실습**이다.

이미 실행 중인 Spring Boot 프로젝트에서
기존에 사용하던 **통합 컨트롤러(ApiController)** 를 유지한 채,
**단일 책임을 가지는 REST 컨트롤러를 하나 추가**한다.

이 실습의 목적은

> **"컨트롤러를 이렇게 나누기 시작하는구나"**

를 코드 수준에서 체감하는 것이다.

아직 구조를 완성하는 단계가 아니라,
**분리를 시작하는 기준을 익히는 단계**임을 명확히 한다.

---

## 1. 실습 목표

이 단계에서 달성해야 할 목표는 다음과 같다.

* 기존 `ApiController.java`는 그대로 유지한다
* 새로운 단일 책임 REST 컨트롤러를 추가한다
* 공통 응답 포맷을 적용한 API 응답을 직접 확인한다

> 이 실습에서는
> "기존 코드를 고치는 것"이 아니라
> **"구조를 깨지 않고 확장하는 방법"**을 익히는 것이 핵심이다.

---

## 2. 전제 (현재 프로젝트 상태)

현재 프로젝트에는 다음과 같은 컨트롤러가 이미 존재한다.

* `ApiController.java`

  * 기존에 사용하던 **통합(몰빵) 컨트롤러**
* `HelloController.java`
* `RedisPingController.java`

이 중 **`ApiController.java`는 현행 시스템의 기준 코드**로 남겨둔다.

이번 실습에서는
해당 파일을 수정하지 않는다.

> 기존 코드를 유지한 채
> 새 구조를 덧붙이는 방식이
> 실제 현업에서 가장 안전한 접근이다.

---

## 3. 컨트롤러 패키지 위치

컨트롤러는 다음 패키지 아래에 위치한다.

```text
src/main/java/com/example/demo/controller/
```

이번 실습에서는
패키지 경로를 변경하지 않는다.

> 구조 변경은
> **책임 분리부터 시작**하고,
> 패키지 이동은 그 다음 단계에서 다룬다.

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
    public String health() {
        return "OK";
    }
}
```

이 컨트롤러는 다음 특징을 가진다.

* 하나의 API만 담당한다
* 비즈니스 로직이 없다

> 이 컨트롤러는
> **"단일 책임 컨트롤러의 최소 형태"** 다.

---

## 5. 애플리케이션 실행

프로젝트 루트에서
다음 명령어로 서버를 실행한다.

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

응답 결과가 정상적으로 반환되면
컨트롤러 분리가 성공한 것이다.

---

## 이 실습의 핵심 의미

이 단계에서 반드시 이해해야 할 점은 다음과 같다.

* 기존 `ApiController.java`를 건드리지 않고도
* 새로운 컨트롤러를 추가할 수 있다
* 구조를 깨지 않고 점진적으로 분리가 가능하다

즉,

> **리팩토링은 한 번에 바꾸는 작업이 아니라
> 안전하게 쌓아가는 작업**이다.

---

## 다음 단계

이제 컨트롤러 분리의 감각을 잡았다.

다음 단계에서는
모든 API에 공통으로 적용되는
**공통 응답 포맷**을 정식으로 적용한다.

→ [**공통 응답 포맷 적용**](03-common-response.md)
