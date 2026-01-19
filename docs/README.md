# 강의 안내

이 문서는 **PBL 서버 프로그램 프로젝트의 전체 학습 흐름을 안내**하는 문서이다.  
요구사항 확인부터 설계, 통합 구현, 테스트, 배포까지  
서버 프로그램이 만들어지고 운영 환경에 배포되기까지의 과정을 단계별로 학습한다.

모든 문서는 **순서대로 학습하는 것을 전제**로 구성되어 있으며,  
강의 진행·실습·과제의 기준 문서로 사용된다.

---

## 문서 구성 및 학습 흐름

### 1. 요구사항 확인

- [01-current-system-analysis.md](./01-requirements/01-current-system-analysis.md)  
  현행 시스템 분석

- [02-requirements.md](./01-requirements/02-requirements.md)  
  요구사항 정의

- [03-analysis-model.md](./01-requirements/03-analysis-model.md)  
  분석 모델 정의

---

### 2. 애플리케이션 설계

- [01-application-architecture.md](./02-application-design/01-application-architecture.md)  
  애플리케이션 설계

- [02-common-module-design.md](./02-application-design/02-common-module-design.md)  
  공통 모듈 및 역할 설계

- [03-integration-design.md](./02-application-design/03-integration-design.md)  
  타 시스템 연동 설계

---

### 3. 통합 구현

- [01-springboot-setup.md](./03-integration-implementation/01-springboot-setup.md)  
  Spring Boot 프로젝트 초기 구성

- [02-rest-controller.md](./03-integration-implementation/02-rest-controller.md)  
  REST Controller 구현

- [03-common-response.md](./03-integration-implementation/03-common-response.md)  
  공통 응답 구조 구현

- [04-global-exception.md](./03-integration-implementation/04-global-exception.md)  
  전역 예외 처리

- [05-service-layer.md](./03-integration-implementation/05-service-layer.md)  
  Service 계층 구현

- [06-repository-layer.md](./03-integration-implementation/06-repository-layer.md)  
  Repository / SQL 계층 구현

- [07-database-connection.md](./03-integration-implementation/07-database-connection.md)  
  데이터베이스 연결 구성

- [08-integration-rss.md](./03-integration-implementation/08-integration-rss.md)  
  RSS 연계 구현

- [09-rss-parsing.md](./03-integration-implementation/09-rss-parsing.md)  
  RSS 파싱 처리

- [10-integration-api.md](./03-integration-implementation/10-integration-api.md)  
  외부 API 통합

---

### 4. 테스트

- [01-api-test.md](./04-testing/01-api-test.md)  
  API 테스트

- [02-error-scenario.md](./04-testing/02-error-scenario.md)  
  오류 시나리오 검증

---

### 5. 배포

- [01-docker-compose-deploy.md](./05-deployment/01-docker-compose-deploy.md)  
  Docker Compose 기반 배포

- [02-nginx-reverse-proxy.md](./05-deployment/02-nginx-reverse-proxy.md)  
  Nginx 리버스 프록시 구성

- [03-deploy-checklist.md](./05-deployment/03-deploy-checklist.md)  
  배포 점검 체크리스트
