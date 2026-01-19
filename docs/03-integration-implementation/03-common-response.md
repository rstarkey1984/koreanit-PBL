# 공통 응답 포맷 적용

이 문서는 **통합 구현 단계의 03번 실습**으로,
지금까지 만든 API들의 응답 형식을
하나의 공통 규칙으로 통일하는 단계다.

앞 단계에서 우리는

* 컨트롤러를 분리했고
* 구조를 깨지 않고 기능을 확장할 수 있음을 확인했다

이제는 각 API가 **각각 다른 형태로 응답하지 않도록**
시스템 차원의 응답 규칙을 적용한다.

---

## 1. 이 단계의 목적

이 단계의 목적은 단순하다.

> **"모든 API는 같은 형태로 응답한다"**

이를 통해 다음을 보장한다.

* 클라이언트는 응답 구조를 예측할 수 있다
* 성공/실패 여부를 일관되게 판단할 수 있다
* 이후 전역 예외 처리와 자연스럽게 연결된다

---

## 2. 공통 응답 포맷이 필요한 이유

지금까지의 API는 다음과 같은 문제를 가질 수 있다.

* 어떤 API는 문자열을 반환하고
* 어떤 API는 JSON 객체를 반환한다
* 오류가 발생했을 때 응답 형식이 제각각이다

이 상태에서는
클라이언트가 API마다 다른 처리를 해야 한다.

> 공통 응답 포맷은
> **서버와 클라이언트 간의 최소한의 약속**이다.

---

## 3. 공통 응답 포맷 규칙

이 프로젝트에서는 다음 구조를 공통 응답 포맷으로 사용한다.

### 성공 응답

```json
{
  "ok": true,
  "data": "...",
  "message": null
}
```

### 실패 응답

```json
{
  "ok": false,
  "data": null,
  "message": "에러 메시지"
}
```

> 실패 응답의 세부 정책은
> 다음 단계(전역 예외 처리)에서 확정한다.

---

## 4. ApiResponse 공통 클래스 생성

다음 파일을 생성한다.

```text
src/main/java/com/example/demo/common/ApiResponse.java
```

```java
package com.example.demo.common;

public class ApiResponse<T> {

    public boolean ok;
    public T data;
    public String message;

    private ApiResponse(boolean ok, T data, String message) {
        this.ok = ok;
        this.data = data;
        this.message = message;
    }

    public static <T> ApiResponse<T> ok(T data) {
        return new ApiResponse<>(true, data, null);
    }

    public static <T> ApiResponse<T> fail(String message) {
        return new ApiResponse<>(false, null, message);
    }
}
```

이 클래스는
모든 API 응답의 **최상위 컨테이너 역할**을 한다.

---

## 5. HealthController 수정

02 단계에서 만든 `HealthController`의 반환 타입을
공통 응답 포맷으로 변경한다.

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

> 이 단계에서는
> **로직은 바꾸지 않고 응답 형태만 변경**한다.

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

정상 응답 예시는 다음과 같다.

```json
{
  "ok": true,
  "data": "OK",
  "message": null
}
```

---

## 이 실습의 핵심 의미

이 단계에서 반드시 이해해야 할 점은 다음이다.

* 응답 포맷은 API마다 달라지면 안 된다
* 컨트롤러는 응답 규칙을 직접 만들지 않는다
* 공통 규칙은 **공통 모듈**로 관리한다

이 기준이 있어야
다음 단계의 전역 예외 처리가 깔끔해진다.

---

## 다음 단계

이제 정상 응답의 규칙이 정해졌다.

다음 단계에서는
모든 오류 상황을 한 곳에서 처리하는
**전역 예외 처리**를 적용한다.

→ [**전역 예외 처리 적용**](04-global-exception.md)
