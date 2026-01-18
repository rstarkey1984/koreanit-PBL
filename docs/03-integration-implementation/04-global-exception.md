# 전역 예외 처리 적용 (실습)

이 문서는 **컨트롤러에서 직접 try-catch를 제거하고**,
애플리케이션 전반의 예외를 한 곳에서 처리하도록
**전역 예외 처리(Global Exception Handling)** 를 적용하는 실습이다.

이 단계부터는

* 정상 응답: `ApiResponse.ok(...)`
* 실패 응답: `ApiResponse.fail(...)`

이라는 규칙이 명확해진다.

---

## 1. 실습 목표

* 컨트롤러에서 try-catch를 제거한다
* 공통 예외 처리 클래스를 만든다
* 모든 오류 응답을 동일한 포맷으로 반환한다

---

## 2. 왜 전역 예외 처리가 필요한가

기존 컨트롤러 코드에서는 다음과 같은 형태가 흔하다.

```java
try {
    // 로직 처리
    return ApiResponse.ok(result);
} catch (Exception e) {
    return ApiResponse.fail("에러 발생");
}
```

이 방식의 문제점은 다음과 같다.

* 컨트롤러마다 try-catch가 반복된다
* 예외 처리 방식이 컨트롤러마다 달라질 수 있다
* 응답 포맷을 통일하기 어렵다

그래서 예외 처리는 **한 곳에서만** 처리하도록 한다.

---

## 3. 전역 예외 처리 클래스 생성

다음 파일을 생성한다.

```text
src/main/java/com/example/demo/common/GlobalExceptionHandler.java
```

```java
package com.example.demo.common;

import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public ApiResponse<Void> handleException(Exception e) {
        return ApiResponse.fail(e.getMessage());
    }
}
```

이 클래스의 역할은 다음과 같다.

* 애플리케이션 전체에서 발생하는 예외를 가로챈다
* 예외를 공통 응답 포맷으로 변환한다

---

## 4. 컨트롤러 코드 단순화

이제 컨트롤러에서는 try-catch를 사용하지 않는다.

```java
@GetMapping("/api/health")
public ApiResponse<String> health() {
    if (true) {
        throw new RuntimeException("강제 오류 테스트");
    }
    return ApiResponse.ok("OK");
}
```

---

## 5. 실행 및 확인

서버를 실행한다.

```bash
./gradlew bootRun
```

오류 상황을 호출한다.

```bash
curl http://localhost:9092/api/health
```

응답 예시는 다음과 같다.

```json
{
  "ok": false,
  "data": null,
  "message": "강제 오류 테스트"
}
```

---

## 6. 이 실습의 의미

이 단계에서 확인해야 할 핵심은 다음과 같다.

* 컨트롤러는 **정상 흐름만** 신경 쓴다
* 예외 처리는 공통 모듈이 담당한다
* 응답 포맷이 성공/실패 모두 일관된다

이제 컨트롤러는
"요청 → 응답"의 역할만 수행하게 된다.

---

## 다음 단계

→ [**서비스 레이어 분리**](05-service-layer.md)
