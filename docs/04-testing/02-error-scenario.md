# 오류 시나리오 테스트 (전역 예외 처리 검증)

이 문서는 전역 예외 처리(GlobalExceptionHandler)가
**실제로 동작하여 실패 응답을 공통 포맷으로 내려주는지** 확인한다.

---

## 전제

* 전역 예외 처리가 적용되어 있다
* 실패 응답은 `ApiResponse.fail(message)` 포맷을 따른다

---

## 1) 테스트용 에러 엔드포인트 추가

학습용으로만 사용하는 테스트 엔드포인트를 추가한다.

다음 파일에 메서드를 추가한다.

```text
src/main/java/com/example/demo/controller/HealthController.java
```

```java
@GetMapping("/api/test/error")
public ApiResponse<String> error() {
    throw new RuntimeException("테스트 오류");
}
```

> 이 엔드포인트는 강의/실습용이다.
> 실서비스에서는 제거한다.

---

## 2) 실행 및 확인

서버를 실행한다.

```bash
./gradlew bootRun
```

요청:

```bash
curl http://localhost:9092/api/test/error
```

기대 결과(예시):

```json
{
  "ok": false,
  "data": null,
  "message": "테스트 오류"
}
```

---

## 3) 뉴스 API 실패 케이스 확인 (선택)

네트워크가 차단되었거나 RSS 응답이 비정상인 경우,
뉴스 API도 실패 응답이 동일한 포맷으로 내려와야 한다.

```bash
curl http://localhost:9092/api/news
```

* 성공 시: `ok: true`
* 실패 시: `ok: false` + `message`에 원인

---

## 4) 체크 포인트

* 컨트롤러에서 try-catch 없이도 실패 응답이 공통 포맷으로 내려오는가
* 실패 응답이 API마다 다른 형태로 섞이지 않는가

---

## 다음 단계

→ [**배포 구조 이해**](/docs/05-deployment/01-docker-compose-deploy)
