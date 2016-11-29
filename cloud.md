# Cloud

- 분산 시스템에서 흔히 나타나는 패턴을 빠르게 빌드할 수 있도록 스프링에서 제공하는 툴

## 특징

- Distributed/versioned configuration
- Service registration and discovery
- Routing
- Service-to-service calls
- Load balancing
- Circuit Breakers
  - Remote call에서 사용하는 패턴. 정해진 기준 이상 요청이 실패하거나 응답이 없을 때, 요청이 쌓여서 전체 프로그램에 장애를 일으키지 않도록 바로 바로 에러 메세지를 되돌려주도록 차단 
- Distributed messaging

## Cloud Native Applications

- continuous delivery, value-driven development 에서 권장되는 모범 사례들을 쉽게 사용할 수 있게 도와주는 개발 스타일
- https://12factor.net/

## Spring Cloud Context