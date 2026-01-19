# RSS 데이터 파싱 및 가공 (실습)

이 문서는 **외부 시스템(RSS)에서 가져온 원본 XML 데이터를 파싱하여
실제 서비스에서 사용할 수 있는 형태로 가공**하는 실습이다.

이 단계의 핵심은 다음과 같다.

* Repository는 여전히 "가져오기"만 담당한다
* Service에서 파싱과 가공을 수행한다
* Controller는 가공된 결과를 응답으로 반환한다

---

## 실습 목표

* RSS XML 구조를 이해한다
* 필요한 데이터만 추출하여 가공한다
* Raw 데이터와 가공 데이터의 차이를 명확히 구분한다

---

## 전제

* `RssRepository`를 통해 RSS XML 문자열을 이미 가져올 수 있다
* 외부 연동 API(`/api/rss/raw`)가 정상 동작 중이다
* 서버는 9092 포트에서 실행 중이다

---

## 1) RSS XML 구조 간단 이해

Google News RSS의 기본 구조는 다음과 같다.

```xml
<rss>
  <channel>
    <item>
      <title>뉴스 제목</title>
      <link>뉴스 링크</link>
      <pubDate>발행일</pubDate>
    </item>
  </channel>
</rss>
```

이번 실습에서는 `item` 안의 다음 필드만 사용한다.

* `title`
* `link`
* `pubDate`

---

## 2) News DTO 생성

가공된 데이터를 담을 DTO를 만든다.

```text
src/main/java/com/example/demo/service/dto/NewsItem.java
```

```java
package com.example.demo.service.dto;

public class NewsItem {

    public String title;
    public String link;
    public String publishedAt;

    public NewsItem(String title, String link, String publishedAt) {
        this.title = title;
        this.link = link;
        this.publishedAt = publishedAt;
    }
}
```

---

## 3) RssService 수정

```text
src/main/java/com/example/demo/service/RssService.java
```
```java
package com.example.demo.service;

import java.io.ByteArrayInputStream;
import java.nio.charset.StandardCharsets;
import java.util.ArrayList;
import java.util.List;

import javax.xml.XMLConstants;
import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;

import org.springframework.stereotype.Service;
import org.w3c.dom.Document;
import org.w3c.dom.Element;
import org.w3c.dom.NodeList;

import com.example.demo.repository.RssRepository;
import com.example.demo.service.dto.NewsItem;

@Service
public class RssService {

    // URL 중복 제거 (필요하면 application.yml로 빼도 됨)
    private static final String RSS_URL =
            "https://news.google.com/rss/search?q=IT&hl=ko&gl=KR&ceid=KR:ko";

    private final RssRepository rssRepository;

    public RssService(RssRepository rssRepository) {
        this.rssRepository = rssRepository;
    }

    public String getRawRss() {
        return rssRepository.fetchRss(RSS_URL);
    }

    public List<NewsItem> getNewsItems() {
        String xml = rssRepository.fetchRss(RSS_URL);
        return parseNewsItems(xml);
    }

    // XML → NewsItem 리스트 변환 책임 분리
    private List<NewsItem> parseNewsItems(String xml) {
        List<NewsItem> result = new ArrayList<>();

        if (xml == null || xml.isBlank()) {
            return result;
        }

        try {
            Document doc = parseXmlSafely(xml);
            NodeList items = doc.getElementsByTagName("item");

            for (int i = 0; i < items.getLength(); i++) {
                Element item = (Element) items.item(i);

                String title = getText(item, "title");
                String link = getText(item, "link");
                String pubDate = getText(item, "pubDate");

                // 필수값이 없으면 스킵(원하면 예외로 바꿔도 됨)
                if (title == null || link == null) {
                    continue;
                }

                result.add(new NewsItem(title, link, pubDate));
            }

            return result;

        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    // XXE 같은 외부 엔티티 공격 방지 설정
    private Document parseXmlSafely(String xml) throws Exception {
        DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();

        dbf.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);
        dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
        dbf.setFeature("http://xml.org/sax/features/external-general-entities", false);
        dbf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);

        dbf.setXIncludeAware(false);
        dbf.setExpandEntityReferences(false);

        DocumentBuilder builder = dbf.newDocumentBuilder();

        return builder.parse(new ByteArrayInputStream(xml.getBytes(StandardCharsets.UTF_8)));
    }

    // 태그가 없을 수 있으니 안전하게 꺼내기
    private String getText(Element parent, String tagName) {
        NodeList list = parent.getElementsByTagName(tagName);
        if (list == null || list.getLength() == 0 || list.item(0) == null) {
            return null;
        }
        String text = list.item(0).getTextContent();
        return (text == null) ? null : text.trim();
    }
}
```

---

## 4) Controller 수정 (가공 데이터 반환)

기존 Raw RSS 응답 대신
가공된 데이터를 반환하는 API를 추가한다.

```text
src/main/java/com/example/demo/controller/RssController.java
```

```java
@GetMapping("/api/rss/news")
public ApiResponse<List<NewsItem>> news() {
  return ApiResponse.ok(rssService.getNewsItems());
}
```

---

## 5) 실행 및 확인

서버를 실행한다.

```bash
./gradlew bootRun
```

다음 요청으로 결과를 확인한다.

```bash
http://localhost:9092/api/rss/news
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

## 6) 이 실습의 의미

이 단계에서 확인해야 할 핵심은 다음과 같다.

* 외부 시스템 데이터는 그대로 쓰지 않는다
* Service에서 필요한 형태로 가공한다
* Controller는 가공된 결과만 응답한다

이제 외부 연동 데이터도
**하나의 서비스 API**로 완성되었다.

---

## 다음 단계

→ [**통합 API 형태로 정리**](10-integration-api.md)
