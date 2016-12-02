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

```
+------------------------+
|  PropertySourceLocator |
|   .locate()            |
+------------------------+
|   application.yml      |
|          +             |
|   spring boot sources  |
+------------------------+
|  bootstrap.yml         |
+------------------------+
MutablePropertySources (last state)
```

#### 세팅 순서

1. bootstrap.yml 로드
2. bootstrap context 설정
3. bootstrap.yml에서 로드해온 applicationConfig를 낮은 우선순위로 `MutablePropertySources`에 추가
4. application context 설정
5. application.yml에서 로드해온 applicationConfig를 `MutablePropertySources`에 추가
6. `PropertySourceLocator.locate()`를 통해 얻은 `PropertySource`를 높은 우선순위로 `MutablePropertySources`에 추가

#### Bootstrap Properties 파일 위치 변경

- 시스템 환경변수에다가 `spring.cloud.bootstrap.name`, `spring.cloud.bootstrap.location` 적절하게 설정

#### Remote Properties 덮어 씌우기

- 외부 Cloud 저장소 (Consul 등) 에서 가져온 Properties 값들을 로컬에서 덮어씌울 수 있게 할 것인지 설정하는 부분
- `spring.cloud.config.allowOverride`
  - false이면 Remote Properties를 덮어 씌울 수 없게 됨
  - true이면 세부 옵션 추가 (기본값)
    - `spring.cloud.config.overrideNone`: true이면 모든 로컬 `PropertySource`가 Remote Properties를 덮어쓰게 됨 (기본값 false)
    - `spring.cloud.config.overrideSystemProperties`: false이면 시스템 환경설정으로는 덮어 씌울 수 있게 됨 (기본값 true)

#### Bootstrap context Configuration

- 리소스 폴더에 `/META-INF/spring.factories` 파일 생성하여 설정
  - `org.springframework.boot.autoconfigure.EnableAutoConfiguration` 키에 원하는 `@Configuration` 클래스 리스트들을 각각 ","로 구분하여 값 작성
- 이 configuration에서 생성한 bean들을 main application context에서 autowired 할 수 있음
  - `ApplicationContextInitializer` bean도 생성 가능
  - `@Order` annotation으로 실행 순서 지정 가능 (기본 값은 "last")
- 이 BootstrapConfiguration 클래스들은 꼭 필요한 경우가 아니라면 `@ComponentScan`이나 `@SpringBootApplication`에 걸리지 않도록 별도의 패키지로 관리하거나 아예 `@Configuration` annotation을 안 붙이는 것을 추천
- bootstrap 과정이 모두 끝나고 나면 main application을 시작하기 전에 `ApplicationContextInitializer` bean들을 추가함

#### Bootstrap Property Sources 추가하기

- `spring.factories` 파일에 알맞은 `PropertySourceLocator` 클래스를 등록해도 되고, 아니면 Configuration 클래스에서 Locator 클래스 인스턴스를 생성하여 bean으로 등록하게 하여도 됨
- ex)

``` java
@Configuration
public class CustomPropertySourceLocator implements PropertySourceLocator {
  @Override
  public PropertySource<?> locate(Environment environment) {
    return new MapPropertySource(
          "customMapProperty",
          Collections.singletonMap("custom.property.key", "custom property value")
        );
  }
}
```

#### Environment Change 이벤트 처리 

- Environment가 변경되면 어플리케이션 전역에 `EnvironmentChangedEvent` publish 됨
  - 변경된 값들의 key가 담겨있음
  - 자동적으로 `@ConfigurationProperties` 다시 바인딩하게 만듬
  - log 레벨을 프로퍼티 `logging.level.*` 에 맞춰서 재설정
- 일반적으로는 `ApplicationListeners` bean 을 만들어서 처리하도록 함
  - Config Client가 `Environment` 변경사항을 폴링하는 것보다 나음
