# 레포지토리 레이어 분리 (실습)

이 문서는 **서비스 레이어에서 데이터 접근 책임을 분리하여 레포지토리 레이어로 이동**하는 실습이다.

이 단계부터는

* 컨트롤러는 요청/응답만 담당하고
* 서비스는 업무 흐름만 담당하며
* 레포지토리는 **데이터 접근(SQL 실행)** 만 담당한다.

---

## 실습 목표

* 데이터 접근 책임을 명확히 분리한다
* 서비스에서 직접 SQL 또는 DB 접근 코드를 제거한다
* Controller → Service → Repository 흐름을 완성한다

---

## 왜 레포지토리 레이어가 필요한가

서비스에서 직접 DB에 접근하면 다음과 같은 문제가 생긴다.

* SQL과 비즈니스 로직이 섞인다
* 테스트와 유지보수가 어려워진다
* DB 변경 시 영향 범위가 커진다

레포지토리는 **데이터 접근 전용 레이어**로,
"어떻게 저장하고 가져오는지"만 책임진다.

---

## 실습 대상 API

이번 실습에서는 **Health API를 기준**으로
"DB 접근이 있는 것처럼" 구조를 먼저 분리한다.

> 실제 SQL을 바로 작성하지 않고,
> 레이어 분리 흐름을 먼저 완성하는 것이 목적이다.

---

## 레포지토리 패키지 생성

다음 패키지를 확인하거나 생성한다.

```text
src/main/java/com/example/demo/repository/
```

---

## HealthRepository 생성

다음 파일을 생성한다.

```text
src/main/java/com/example/demo/repository/HealthRepository.java
```

```java
package com.example.demo.repository;

import org.springframework.stereotype.Repository;

@Repository
public class HealthRepository {

    public String getStatus() {
        // 실제로는 DB 조회 로직이 위치할 자리
        return "OK";
    }
}
```

---

## HealthService 수정

서비스가 직접 데이터를 만들지 않고
레포지토리를 통해 가져오도록 수정한다.

```java
package com.example.demo.service;

import com.example.demo.repository.HealthRepository;
import org.springframework.stereotype.Service;

@Service
public class HealthService {

    private final HealthRepository healthRepository;

    public HealthService(HealthRepository healthRepository) {
        this.healthRepository = healthRepository;
    }

    public String check() {
        return healthRepository.getStatus();
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
curl http://localhost:9092/api/health
```

응답은 이전 단계와 동일해야 한다.

```json
{
  "ok": true,
  "data": "OK",
  "message": null
}
```

---

## 이 실습의 의미

이 단계에서 확인해야 할 핵심은 다음과 같다.

* 각 레이어의 책임이 명확해졌다
* 데이터 접근 위치가 한 곳으로 모였다
* 이후 실제 DB 연동을 자연스럽게 추가할 수 있다

이제 실제 데이터베이스(MySQL) 연동은
이 레포지토리 레이어에 추가하게 된다.

---

## 다음 단계

→ [**07. 데이터베이스 연동 설정**](07-database-connection.md)
