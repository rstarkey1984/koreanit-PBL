# 외부 시스템 연동 API 구현 (RSS)

이 문서는 **외부 시스템(RSS)을 연동하여 데이터를 조회하고 응답하는 API를 구현**하는 실습이다.

이번 단계에서는

* 데이터베이스가 아닌 외부 시스템을 데이터 소스로 사용하고
* Repository는 "어디에서 가져오는가"만 책임지며
* Service는 데이터를 가공하고
* Controller는 응답만 반환한다.

---

## 실습 목표

* 외부 시스템 연동을 Repository 레이어로 분리한다
* HTTP 호출 책임을 명확히 분리한다
* Controller → Service → Repository 흐름을 유지한다

---

## 전제

* 서버는 `9092` 포트에서 실행 중이다
* 기존 DB 연동 및 전역 예외 처리가 적용되어 있다
* 외부 연동 대상은 **Google News RSS** 이다

---

## 1) 외부 연동 구조 이해

외부 시스템 연동도 구조는 동일하다.

```
Controller → Service → Repository → 외부 시스템(RSS)
```

차이점은 **데이터 소스가 DB가 아니라 HTTP**라는 점뿐이다.

---

## 2) Repository 역할 정의

RSS Repository의 책임은 다음과 같다.

* RSS URL 관리
* HTTP 요청 수행
* 응답 문자열(XML) 반환

> 파싱이나 가공은 Service에서 수행한다.

---

## 3) RssRepository 생성

다음 파일을 생성한다.

```text
src/main/java/com/example/demo/repository/RssRepository.java
```

```java
package com.example.demo.repository;

import org.springframework.stereotype.Repository;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URI;
import java.net.URL;

@Repository
public class RssRepository {

  public String fetchRss(String rssUrl) {
    try {
      URI uri = URI.create(rssUrl);
      URL url = uri.toURL();
      HttpURLConnection conn = (HttpURLConnection) url.openConnection();
      conn.setRequestMethod("GET");
      conn.setConnectTimeout(3000);
      conn.setReadTimeout(3000);

      try (BufferedReader br = new BufferedReader(
          new InputStreamReader(conn.getInputStream()))) {

        StringBuilder sb = new StringBuilder();
        String line;
        while ((line = br.readLine()) != null) {
          sb.append(line);
        }
        return sb.toString();
      }
    } catch (Exception e) {
      throw new RuntimeException(e);
    }
  }
}
```

---

## 4) Service 레이어에서 RSS 가공

다음 파일을 생성한다.

```text
src/main/java/com/example/demo/service/RssService.java
```

```java
package com.example.demo.service;

import com.example.demo.repository.RssRepository;
import org.springframework.stereotype.Service;

@Service
public class RssService {

    private final RssRepository rssRepository;

    public RssService(RssRepository rssRepository) {
        this.rssRepository = rssRepository;
    }

    public String getRawRss() {
        String rssUrl = "https://news.google.com/rss/search?q=IT&hl=ko&gl=KR&ceid=KR:ko";
        return rssRepository.fetchRss(rssUrl);
    }
}
```

---

## 5) Controller 생성

다음 파일을 생성한다.

```text
src/main/java/com/example/demo/controller/RssController.java
```

```java
package com.example.demo.controller;

import com.example.demo.common.ApiResponse;
import com.example.demo.service.RssService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class RssController {

    private final RssService rssService;

    public RssController(RssService rssService) {
        this.rssService = rssService;
    }

    @GetMapping("/api/rss/raw")
    public ApiResponse<String> rssRaw() {
        return ApiResponse.ok(rssService.getRawRss());
    }
}
```

---

## 6) 실행 및 확인

서버를 실행한다.

```bash
./gradlew bootRun
```

다음 요청으로 결과를 확인한다.

```bash
http://localhost:9092/api/rss/raw
```

응답으로 RSS XML 문자열이 내려오면 정상이다.

---

## 7) 이 실습의 의미

이 단계에서 확인해야 할 핵심은 다음과 같다.

* 외부 시스템도 Repository 책임으로 관리한다
* Service는 데이터 가공 위치다
* Controller는 응답 규격만 책임진다

DB든 외부 API든
**구조는 동일하다**는 점이 중요하다.

---

## 다음 단계

→ [**RSS 데이터 파싱 및 가공**](09-rss-parsing.md)
