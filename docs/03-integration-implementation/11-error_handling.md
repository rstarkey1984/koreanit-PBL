# 에러 Exception 처리 구조

> **에러가 발생하면 `return` 하지 않는다.**
> 실패는 전부 `throw` 하고, 응답은 전역 핸들러가 만든다.

---

## 0. 핵심 원칙 요약

* Controller / Service에서는 **성공만 return**
* 실패는 **의미 있는 예외(ApiException)** 를 `throw`
* 응답 포맷은 **GlobalExceptionHandler에서만 생성**

---

## 1. 에러 표현 전략

### ErrorCode 

```java
package com.example.demo.common;

import org.springframework.http.HttpStatus;

public enum ErrorCode {

    INVALID_REQUEST(HttpStatus.BAD_REQUEST, "입력값 오류"),
    UNAUTHORIZED(HttpStatus.UNAUTHORIZED, "로그인 필요"),
    FORBIDDEN(HttpStatus.FORBIDDEN, "권한 없음"),
    NOT_FOUND(HttpStatus.NOT_FOUND, "대상 없음"),
    DUPLICATE(HttpStatus.CONFLICT, "중복 데이터"),
    DB_ERROR(HttpStatus.INTERNAL_SERVER_ERROR, "DB 오류"),
    INTERNAL_ERROR(HttpStatus.INTERNAL_SERVER_ERROR, "서버 오류");

    private final HttpStatus status;
    private final String defaultMessage;

    ErrorCode(HttpStatus status, String defaultMessage) {
        this.status = status;
        this.defaultMessage = defaultMessage;
    }

    public HttpStatus getStatus() {
        return status;
    }

    public String getDefaultMessage() {
        return defaultMessage;
    }
}
```

### ErrorCode (실패 유형 식별자)

| 코드              | 의미                  |
| --------------- | ------------------- |
| INVALID_REQUEST | 입력값 오류              |
| UNAUTHORIZED    | 로그인 필요              |
| FORBIDDEN       | 권한 없음               |
| NOT_FOUND       | 대상 없음               |
| DUPLICATE       | 중복(업무 규칙상 의미 있는 중복) |
| DB_ERROR        | DB 관련 모든 오류         |
| INTERNAL_ERROR  | 서버 내부 오류            |

* **프론트 분기 기준은 메시지가 아니라 code**
* 메시지는 사람이 읽기 위한 부가 정보

---

## 2. ApiException 규칙

### 2-1. 직접 throw 하는 유일한 예외

```java
package com.example.demo.common;

public class ApiException extends RuntimeException {

    private final ErrorCode code;

    public ApiException(ErrorCode code) {
        super(code.getDefaultMessage());
        this.code = code;
    }

    public ApiException(ErrorCode code, String message) {
        super(message);
        this.code = code;
    }

    public ErrorCode getCode() {
        return code;
    }
}
```


### 실패는 예외로 표현한다

```
실패 = throw
응답 = 전역 핸들러
```

* `return ApiResponse.fail(...)` 사용 금지
* 실패 분기는 모두 `throw new ApiException(...)`
* 상태 코드는 `ErrorCode`가 책임진다

---

## 3. ApiResponse 규칙

### 3-1. 성공 / 실패 형태 고정

```java
package com.example.demo.common;

public class ApiResponse<T> {

    private final boolean success;
    private final T data;
    private final String code;
    private final String message;

    private ApiResponse(boolean success, T data, String code, String message) {
        this.success = success;
        this.data = data;
        this.code = code;
        this.message = message;
    }

    public static <T> ApiResponse<T> ok(T data) {
        return new ApiResponse<>(true, data, null, null);
    }

    public static <T> ApiResponse<T> fail(String code, String message) {
        return new ApiResponse<>(false, null, code, message);
    }

    public boolean isSuccess() {
        return success;
    }

    public T getData() {
        return data;
    }

    public String getCode() {
        return code;
    }

    public String getMessage() {
        return message;
    }
}
```

* 실패 응답에는 **반드시 code 포함**
* HTTP 상태코드 + 내부 code **이중 구조**

---

## 4. GlobalExceptionHandler 정책

> **예외 → ApiResponse 변환은 이곳에서만 한다**

```java
package com.example.demo.common;

import org.springframework.dao.DataAccessException;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {

    // 1) 우리가 의도적으로 던진 예외
    @ExceptionHandler(ApiException.class)
    public ResponseEntity<ApiResponse<Object>> handleApi(ApiException e) {
        ErrorCode code = e.getCode();
        return ResponseEntity
                .status(code.getStatus())
                .body(ApiResponse.fail(code.name(), e.getMessage()));
    }

    // 2) DB 예외는 전부 DB_ERROR로 통일
    @ExceptionHandler(DataAccessException.class)
    public ResponseEntity<ApiResponse<Object>> handleDb(DataAccessException e) {
        ErrorCode code = ErrorCode.DB_ERROR;
        return ResponseEntity
                .status(code.getStatus())
                .body(ApiResponse.fail(code.name(), code.getDefaultMessage()));
    }

    // 3) 그 외 모든 예외
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ApiResponse<Object>> handleAny(Exception e) {
        ErrorCode code = ErrorCode.INTERNAL_ERROR;
        return ResponseEntity
                .status(code.getStatus())
                .body(ApiResponse.fail(code.name(), code.getDefaultMessage()));
    }
}
```

### 처리 규칙 요약

* `ApiException` → 상태/메시지 그대로 사용
* `DataAccessException` → `DB_ERROR`, 메시지 고정
* 나머지 예외 → `INTERNAL_ERROR`

---

## 5. 리팩토링 기준표

| 상황     | 기존 방식            | 변경 후                                     |
| ------ | ---------------- | ---------------------------------------- |
| 로그인 필요 | return fail(...) | throw ApiException(UNAUTHORIZED, ...)    |
| 입력값 오류 | return fail(...) | throw ApiException(INVALID_REQUEST, ...) |
| 대상 없음  | return fail(...) | throw ApiException(NOT_FOUND, ...)       |
| 권한 없음  | return fail(...) | throw ApiException(FORBIDDEN, ...)       |

Controller는
**검사 → 실패면 throw → 성공이면 ok 반환**
이 패턴만 남긴다.

---

## 이 구조의 장점

* 실패 흐름이 명확함
* 응답 포맷 단일화
* 보안 사고 방지 (에러 메시지 노출 차단)
* 프론트엔드 분기 안정성 확보

---

## 한 줄 요약

> **실패는 throw,
> 의미 있는 실패만 ApiException,
> DB 오류는 DB_ERROR,
> 응답은 전역 핸들러가 만든다.**

---

## 다음 단계

→ [**ApiController – ApiService – ApiRepository 쪼개기**](12-refactoring.md)
