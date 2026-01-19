# 통합 API 형태로 정리

이 문서는 지금까지 구현한 RSS 연동 기능을
**실제 서비스에서 사용하는 API 형태로 정리**하는 단계다.

이 단계의 목적은
"기능을 더 만드는 것"이 아니라,
**API를 서비스 관점에서 정리하는 것**이다.

---

## 실습 목표

* 실습용 API 경로를 서비스용 API로 정리한다
* 응답 구조를 최종 형태로 고정한다
* Controller의 책임을 명확히 한다

---

## API 정리 방향

기존 실습용 API는 다음과 같다.

* `/api/rss/raw`
* `/api/rss/news`

이제 외부 시스템(RSS)을 직접 드러내지 않고,
서비스 기준 API 하나로 통합한다.

정리 후 API:

* `/api/news`

> 외부 시스템의 존재는 API 밖으로 숨긴다.

---

## NewsController 생성

다음 파일을 생성한다.

```text
src/main/java/com/example/demo/controller/NewsController.java
```

```java
package com.example.demo.controller;

import com.example.demo.common.ApiResponse;
import com.example.demo.service.RssService;
import com.example.demo.service.dto.NewsItem;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
public class NewsController {

    private final RssService rssService;

    public NewsController(RssService rssService) {
        this.rssService = rssService;
    }

    @GetMapping("/api/news")
    public ApiResponse<List<NewsItem>> list() {
        return ApiResponse.ok(rssService.getNewsItems());
    }
}
```

---

## 실행 및 확인

서버를 실행한다.

```bash
./gradlew bootRun
```

다음 요청으로 결과를 확인한다.

```bash
curl http://localhost:9092/api/news
```

응답 예시는 다음과 같다.

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

## 이 단계의 의미

이 단계에서 확인해야 할 핵심은 다음과 같다.

* 외부 시스템 연동이 내부 구현으로 완전히 숨겨졌다
* API는 서비스 기준으로 정리되었다
* 클라이언트는 데이터 출처를 알 필요가 없다

이제 이 프로젝트는
**실제 서비스 API 형태를 갖춘 상태**가 된다.

---

## 다음 단계

→ [**애플리케이션 테스트 (기본 호출)**](/docs/04-testing/01-api-test.md)
