# 애플리케이션 테스트 (기본 호출)

이 문서는 지금까지 구현한 API를 **실행 중인 상태에서 빠르게 검증**하기 위한 테스트 문서다.

이 강의의 테스트는 자동화(JUnit)보다

* 요청을 직접 보내고
* 응답을 확인하며
* 흐름을 검증하는
  방식에 초점을 둔다.

---

## 전제

* 서버는 `./gradlew bootRun`으로 실행한다
* 서버 포트는 `9092`다
* 모든 API 응답은 `ApiResponse` 포맷을 따른다

---

## 1) 서버 실행

```bash
./gradlew bootRun
```

---

## 2) Health API 확인

```bash
curl http://localhost:9092/api/health
```

기대 결과(예시):

```json
{
  "ok": true,
  "data": "OK",
  "message": null
}
```

---

## 3) 뉴스 API 확인 (RSS 연동 + 파싱 + 통합 API)

```bash
http://localhost:9092/api/news
```

기대 결과(형태):

```json
{
  "ok": true,
  "data": [
    {
      "title": "...",
      "link": "...",
      "publishedAt": "..."
    }
  ],
  "message": null
}
```

---

## 4) 포인트

* `ok=true/false`로 성공/실패를 판단한다
* 실제 데이터의 개수/내용보다 **응답 구조와 흐름**을 확인한다
* 문제가 생기면 다음을 먼저 확인한다

체크리스트:

* 포트가 9092가 맞는가
* `application.yml`의 datasource/redis 설정이 현재 환경과 맞는가
* RSS 요청이 외부 네트워크에서 차단되지 않았는가

---

## 다음 단계

→ [**오류 시나리오 테스트**](02-error-scenario.md)
