# 데이터베이스 접근 적용 (Repository 실습)

이 문서는 **이미 연동되어 있는 MySQL을 Repository 레이어에서 실제로 사용**하는 실습이다.

중요한 포인트는 다음과 같다.

* DB 연결 설정을 새로 하지 않는다
* 컨트롤러/서비스 코드는 최대한 유지한다
* Repository에서만 "가짜 값"을 "실제 DB 조회"로 교체한다

---

## 실습 목표

* Repository에서 실제 SQL을 실행한다
* Controller → Service → Repository 흐름을 유지한다
* DB 접근 책임이 Repository에만 존재하도록 만든다

---

## 전제

* `application.yml`에 datasource 설정이 이미 존재한다
* MySQL이 실행 중이며, 애플리케이션이 접속 가능한 상태다
* 서버 포트는 9092다

---

## 1) HealthRepository를 DB 기반으로 교체

현재 `HealthRepository`는 다음과 같이 가짜 값을 반환하고 있다.

```java
public String getStatus() {
    return "OK";
}
```

이제 이 부분을 **실제 DB 조회**로 바꾼다.

---

## 2) DataSource 사용

이 프로젝트는 이미 datasource 설정이 되어 있고,
기존 코드도 `DataSource` 기반으로 DB 접근을 하고 있으므로
이번 실습도 **DataSource를 그대로 사용**한다.

> `JdbcTemplate`로 바꾸지 않는다.

---

## 3) HealthRepository 구현

다음 파일을 수정한다.

```text
src/main/java/com/example/demo/repository/HealthRepository.java
```

```java
package com.example.demo.repository;

import org.springframework.stereotype.Repository;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

@Repository
public class HealthRepository {

    private final DataSource dataSource;

    public HealthRepository(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public String getStatus() {
        String sql = "SELECT 1";

        try (Connection conn = dataSource.getConnection();
             PreparedStatement ps = conn.prepareStatement(sql);
             ResultSet rs = ps.executeQuery()) {

            if (rs.next() && rs.getInt(1) == 1) {
                return "OK";
            }
            return "FAIL";

        } catch (Exception e) {
            // 전역 예외 처리로 전달
            throw new RuntimeException(e);
        }
    }
}
```

이제 "OK"는 하드코딩이 아니라
**DB에 실제로 연결되어 쿼리가 성공했을 때만** 반환된다.

---

## 4) 서비스/컨트롤러는 변경하지 않는다

`HealthService`, `HealthController`는
Repository에서 값을 가져오는 구조를 그대로 유지한다.

* Controller: 응답 반환
* Service: 업무 흐름
* Repository: DB 접근

---

## 5) 실행 및 확인

서버를 실행한다.

```bash
./gradlew bootRun
```

다음 요청으로 결과를 확인한다.

```bash
curl http://localhost:9092/api/health
```

정상 응답 예시:

```json
{
  "ok": true,
  "data": "OK",
  "message": null
}
```

만약 DB 접속이 실패하면
전역 예외 처리(04 단계)에서 실패 응답이 내려와야 한다.

---

## 6) 체크 포인트

* Health API는 이제 DB 연결 상태를 실제로 반영한다
* DB 접근 코드는 Repository에만 존재한다
* Service/Controller는 DB를 직접 알지 않는다

---

## 다음 단계

→ [**외부 시스템 연동 API 구현 (RSS)**](08-integration-rss.md)
