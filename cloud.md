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

- spring boot에서 제공하는 각종 기능들의 빌드보다 먼저 빌드되고, 필요하리라 여겨지는 몇몇 기능들 추가

### The Bootstrap Application Context

- `bootstrap` context를 생성하여 동작
  - 일반적인 spring context의 부모로 생성됨
  - 외부 소스로부터 설정을 읽어오기, 암호화된 설정 복호화 등
  - spring context와 `Environment` 공유
  - 높은 우선도를 가지고 있기 때문에 기본적으로는 local configuration에 의해서 override 될 수 없음
  - `application[.yml|.properties]` 대신에 `bootstrap[.yml|.properties]` 사용

### Application Context Hierarchies

- Spring Cloud Config를 추가하여 빌드하게 되면 기존 context에 property source가 추가됨
  1. "bootstrap": Bootstrap context에 `PropertySourceLocator` Bean이 있으면 application context 설정 완료 이후에 `CompositePropertySource`를 높은 우선순위로 추가함
  2. "applicationConfig": `classpath:bootstrap.yml` 에 등록된 값으로 Bootstrap context가 설정 완료된 이후에 `classpath:application.yml` 및 기타 스프링 부트 기본 PropertySource보다 낮은 우선순위로 추가됨

#### 세팅 순서

1. bootstrap.yml 로드
2. bootstrap context 설정
3. bootstrap.yml에서 로드해온 applicationConfig를 낮은 우선순위로 `MutablePropertySources`에 추가
4. application context 설정
5. application.yml에서 로드해온 applicationConfig를 `MutablePropertySources`에 추가
6. `PropertySourceLocator.locate()`를 통해 얻은 `PropertySource`를 높은 우선순위로 `MutablePropertySources`에 추가

+------------------------+
|  PropertySourceLocator |
|   .locate()            |
|                        |
+------------------------+
|                        |
|   application.yml      |
|                        |
|                        |
|          +             |
|                        |
|   spring boot sources  |
|                        |
+------------------------+
|                        |
|  bootstrap.yml         |
|                        |
+------------------------+

