# 03. 공통 응답 포맷 적용 (실습)

이 문서는 **02 단계에서 만든 간단한 REST API 응답을 공통 포맷으로 통일**하는 실습이다.

02 단계에서는 문자열("OK")을 그대로 반환했다.
03 단계에서는 모든 API가 동일한 응답 구조를 갖도록 **공통 응답 객체(ApiResponse)** 를 도입한다.

---

## 1. 실습 목표

* `ApiResponse` 공통 응답 객체를 생성한다
* 컨트롤러 반환 타입을 `ApiResponse<T>`로 통일한다
* 응답이 JSON 공통 포맷으로 내려오는지 확인한다

---

## 2. 공통 응답 포맷 규칙

성공 응답은 다음 구조를 기준으로 한다.

```json
{
  "success": true,
  "data": "...",
  "message": null
}
```

실패 응답은 다음 구조를 기준으로 한다.

```json
{
  "success": false,
  "data": null,
  "message": "..."
}
```

> 실패 응답에 `error` 필드를 둘지 여부는 04(전역 예외 처리)에서 확정한다.

---

## 3. ApiResponse.java 생성

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

---

## 4. HealthController 수정

02 단계에서 작성한 `HealthController`의 반환 타입을 변경한다.

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

---

## 5. 실행 및 확인

서버를 실행한다.

```bash
./gradlew bootRun
```

다음 요청으로 결과를 확인한다.

```bash
curl http://localhost:9092/api/health
```

응답 예시는 다음과 같다.

```json
{
  "success": true,
  "data": "OK",
  "message": null
}
```

---

## 6. 이 실습의 의미

이 단계에서 확인한 내용은 다음과 같다.

* 응답 포맷이 컨트롤러마다 달라지지 않는다
* 클라이언트는 항상 동일한 응답 구조를 기대할 수 있다
* 이후 전역 예외 처리에서 실패 응답도 일관되게 만들 수 있다

---

## 다음 단계

→ [**04. 전역 예외 처리 적용**](04-global-exception.md)
