# ApiController – ApiService – ApiRepository 쪼개기

## 목표

* **Controller는 성공 응답만 리턴**한다: `return ApiResponse.ok(...)`
* 실패는 전부 **예외(ApiException)** 로 통일하고 `GlobalExceptionHandler`가 응답 포맷을 만든다.

---

## 1. 계층별 책임(역할 분리 기준)

### 1-1. ApiController 책임(HTTP 전용)

Controller는 **HTTP와 프레임워크 처리**만 담당한다.

* URL 매핑: `@GetMapping / @PostMapping / @PutMapping / @DeleteMapping`
* 입력 바인딩: `@RequestBody / @PathVariable / @RequestParam`
* 세션/요청 객체 전달: `HttpSession`, `HttpServletRequest`
* 성공 응답 포장: `ApiResponse.ok(service.xxx(...))`

**Controller가 하면 안 되는 것**

* SQL/JDBC 코드
* 권한 체크(작성자 비교)
* 입력 검증 규칙(길이/패턴/필수 등)

---

### 1-2. ApiService 책임(업무 규칙/정책)

Service는 **비즈니스 규칙과 흐름을 책임**진다.

* 로그인 필요 여부 판단: `requireLogin(session)`
* 권한 체크: 작성자만 수정/삭제
* 입력값 검증: 필수/길이/형식
* 여러 작업 조합(정합성): 예) 댓글 insert + posts 카운트 증가
* 트랜잭션 경계 설정(한 요청에서 DB를 여러 번 만질 때)
* Repository 결과/DB예외를 **ApiException(ErrorCode)** 로 변환

**Service가 결정하는 것(중요)**

* `UNAUTHORIZED / FORBIDDEN / NOT_FOUND / INVALID_REQUEST / INTERNAL_ERROR` 같은 **ErrorCode 선택**

---

### 1-3. ApiRepository 책임(DB 접근 전용)

Repository는 **SQL 실행과 결과 매핑**만 담당한다.

* SQL 보관/실행
* `Connection/PreparedStatement/ResultSet` 처리
* 결과를 자바 값으로 반환

  * `Map<String,Object>` / `List<Map<String,Object>>`
  * `int/long`(영향 행, count)
  * `generated key`(insert 후 id)

**Repository가 하면 안 되는 것**

* “로그인 필요”, “권한 없음” 같은 API 의미 판단
* ErrorCode 선택
* 트랜잭션 경계 설정(원칙적으로 Service)

---

## 2. 컨트롤러 최종 형태

* 입력 받고 → 서비스 호출하고 → ok로 감싸서 리턴

예(형태 예시)

```java
@PostMapping("/signup")
public ApiResponse<Map<String, Object>> signup(@RequestBody Map<String, Object> body) throws Exception {    
  return ApiResponse.ok(apiService.signup(body));
}
```

---

## 3. Repository 메서드는 ‘SQL 단위’로 쪼갠다

Repository 메서드 기준은 “SQL 한 덩어리(한 책임)”이다.

예시(개념)

* 사용자

  * `findUserByUsername(username)`
  * `insertUser(...) -> userId`
  * `findUserWithProfile(userId)`
  * `updateProfile(userId, ...) -> updatedRows`
  * `insertProfile(userId, ...) -> insertedRows`

* 게시글

  * `countPosts(type, keyword)`
  * `findPosts(page, pageSize, type, keyword)`
  * `findPostById(postId)`
  * `insertPost(userId, title, content) -> postId`
  * `findPostOwnerId(postId) -> ownerId`
  * `updatePost(postId, title, content) -> updatedRows`
  * `deletePost(postId) -> deletedRows`

* 조회수(중복 방지)

  * `insertViewLog(postId, viewerKey) -> insertedTrueFalse(또는 예외)`
  * `incrementViewCount(postId) -> updatedRows`

* 댓글

  * `findCommentsByPostId(postId)`
  * `insertComment(postId, userId, comment) -> commentId`
  * `findCommentOwnerAndPostId(commentId)`
  * `updateComment(commentId, comment) -> updatedRows`
  * `deleteComment(commentId) -> deletedRows`
  * `incrementCommentsCnt(postId)`
  * `decrementCommentsCnt(postId)`

---

## 4. 예외 처리 규칙(구조를 깔끔하게 만드는 핵심)

### 4-1. Repository

* DB 사실만 표현

  * `SQLException`류를 그대로 던지거나
  * `null/0` 같은 결과로 반환

### 4-2. Service

* Repository 결과를 해석해서 의미를 부여

  * 없음 → `NOT_FOUND`
  * 로그인 없음 → `UNAUTHORIZED`
  * 작성자 불일치 → `FORBIDDEN`
  * 입력값 문제 → `INVALID_REQUEST`
  * 그 외 → `INTERNAL_ERROR`

**즉, ErrorCode 선택은 Service 책임**

---

## 5. 실습 진행 순서

1. Controller에서 SQL을 모두 제거하고 Service 호출만 남긴다.
2. Service로 검증/권한/트랜잭션을 옮긴다.
3. Repository로 SQL을 옮기고 “SQL 단위 메서드”로 쪼갠다.
4. Service에서 모든 ErrorCode를 결정하도록 통일한다.
5. Controller는 `ApiResponse.ok(...)`만 남긴다.

---

## 체크리스트

* Controller에 `Connection/PreparedStatement/ResultSet`이 없다.

* Repository는 SQL 실행/매핑만 한다.

* `ApiException(ErrorCode...)`는 Service에서만 만든다(권장).

* 성공 응답은 Controller에서 `ApiResponse.ok(...)`로 통일된다.

---

## 다음 단계

→ [**애플리케이션 테스트**](/docs/04-testing/01-api-test.md)