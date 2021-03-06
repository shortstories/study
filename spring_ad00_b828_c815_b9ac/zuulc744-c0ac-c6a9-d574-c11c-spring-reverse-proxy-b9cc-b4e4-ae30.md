# Zuul을 사용해서 Spring Reverse proxy 만들기

프로젝트 진행 중에 약간 귀찮은 부분이 생겼다.

하나는 프론트엔드 javascript 단에서 외부 REST API를 호출해야되는 부분이었다. 당연히 CORS에 의해서 막히기 때문에 처음에는 해당 API 서버의 CORS 설정을 직접 만져 수정했었는데, 하다보니 한계가 있었다. CORS 허용 도메인 하나 추가해야 할 때마다 전체 모듈을 재배포해야될 판국인 것이다. 이유는 하나였다. CORS는 sub domain 전체를 허용하는 것이 불가능한 것. 만약에 \`[http://\*.navercorp.com\`](http://*.navercorp.com`) 같은 식으로 만드는게 가능하다면 편리했겠지만 불가능했다. 만약 꼭 하겠다면 아예 \*을 써서 모든 CORS 요청을 허락하는 방법은 있지만 이는 보안적인 측면에서 불가능한 방법이었다.

다른 하나는 iframe 태그를 쓰다가 생긴 것이었다. 현재 서비스가 https로 제공되고 있는데 iframe으로 가져오려는 페이지는 http만 제공되고 있었던 것. 마찬가지로 브라우저의 보안 정책 상 이것은 허용되지 않는 부분이었다. 그래서 처음엔 iframe으로 끌어오는 페이지에다가 https를 적용시킬까 하니 도메인도 넣어야되고, 인증서도 얻어야되고, 여러모로 골치아픈 부분들이 많았다.

이 전에 node.js에서 위와 같은 문제들을 해결하기 위해 프록시 모듈을 사용한 적이 있었다. 그래서 혹시 spring에도 비슷한 기능을 하는 모듈이 있지 않을까 싶어 검색을 했고, Spring-cloud-zuul을 찾을 수 있었다.

### 사용해서 좋았던 부분

* 위에서 언급한 것 처럼 브라우저의 보안 정책을 우회할 수 있었다.
  * 물론 아무데나 남용하다가는 보안 이슈가 생기겠지만!
* 비교적 세팅도 간단. properties 두어개 추가하는 걸로 간단하게 적용할 수 있었다.

### 아쉬웠던 부분

* 문서가 중구난방. 커스텀해서 쓰려고 하니 결국 코드 뜯어보게 되더라.
* 동적으로 라우팅 추가하기가 엄청 귀찮다. 원칙은 리본이나 유레카 붙이라나? 닭잡는데 소잡는 칼 들이미는 격이다. 내가 현재 쓰고 있는 consul로도 돌릴 순 있지만 service discovery 쓸 때만 적용 가능하더라. 결국 특정 kv에 맞춰서 하려면 직접 코드를 짜는 수 밖에 없었다.
* 물론 이건 내가 무리한 요구를 한 것일 수도 있는데, Zuul로 프록시하는 path에다가 cors를 허용하게 만들기가 좀 귀찮았다. 그래서 spring의 `CorsFilter`를 추가해서 해결했다.
* 예외를 `ControllerAdvice`로 처리하기가 쉽지않다...

## 사용법

maven, spring-boot 1.4 이상 기준.

### dependency 추가

```markup
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zuul</artifactId>
    <version>1.3.1.RELEASE</version>
</dependency>
```

### annotation 추가

```java
@Configuration
@EnableZuulProxy
public class ZuulConfig {
}
```

### properties 추가

나는 spring의 프로퍼티를 yaml으로 관리하고 있다.

```yaml
zuul:
  routes:
    myApiRoute:
      path: /proxy/**
      url: localhost:8080
```

이렇게 세 가지 세팅을 처리해주면, /proxy/ path 아래로 가는 모든 요청은 `localhost:8080` 아래로 가게 된다. 물론 옵션에 따라 달라지기는 하지만 path도, body도, header도 그대로 포워딩되는게 좋았다.

물론 필요하다면 Pre Filter 등을 추가해서 원하는 대로 인증을 한다던가 하는 방법도 존재한다.

## 동적으로 Routing 추가하는 방법

Zuul은 라우팅 과정에서 어떤 주소로 포워딩할건지 결정하기 위해 `RouteLocator` 이라는 인터페이스를 사용한다. 이 인터페이스를 구현한 클래스들이 여러 개 존재하는데 모든 `RouteLocator` 구현 클래스들은 `CompositeRouteLocator` 이라는 구현 클래스 안에 담겨지게 된다. \(만약에 직접 상속해서 구현한 클래스를 @Primary bean으로 만들 경우, 에러가 발생한다.\) 그리고 포워딩 url 요청이 올 때마다 모든 `RouteLocator` 구현체들을 주어진 순서에 따라 돌면서 탐색하며, 가장 먼저 알맞은 path가 발견되면 그 path에 매핑된 url을 돌려주게 된다.

직접 라우팅을 동적으로 관리하기 위해서는 클래스를 작성해야되는데, 여기에 필요한 것은 다음과 같다.

1. 라우팅이 추가, 삭제될 수 있으므로 `RefreshableRouteLocator` 를 구현해야 한다.
2. 대부분의 로직은 그대로 사용할 수 있으므로 기존의 `SimpleRouteLocator` 을 상속해서 사용한다.
3. `SimpleRouteLocator` 은 `ZuulProperties` 의 `ZuulRoute` 목록을 받아다가 쓴다. 그러므로 우리는 저 `ZuulRoute` 맵을 관리하면 된다.

그에 따라 나는 다음과 같은 클래스를 작성했다.

```java
public class MyDynamicRouteLocator extends SimpleRouteLocator implements RefreshableRouteLocator {
  private static final Logger logger = LoggerFactory.getLogger(MyDynamicRouteLocator.class);

  private final ZuulProperties properties;
  private final Map<String, ZuulProperties.ZuulRoute> staticRoutes;

  public MyDynamicRouteLocator(final String servletPath,
                               final ZuulProperties properties) {
    super(servletPath, properties);

    this.properties = properties;

    // properties 파일에 등록된 zuul 옵션들
    this.staticRoutes = Maps.newLinkedHashMap();
    this.staticRoutes.putAll(properties.getRoutes());
  }

  public void refreshWithRoutesMap(final Map<String, ZuulProperties.ZuulRoute> newRoutesMap) {
    newRoutesMap.putAll(staticRoutes);

    final ZuulProperties.ZuulRoute defaultRoute =
        properties.getRoutes().get(DiscoveryClientRouteLocator.DEFAULT_ROUTE);
    if (defaultRoute != null) {
      newRoutesMap.put(DiscoveryClientRouteLocator.DEFAULT_ROUTE, defaultRoute);
    }

    properties.setRoutes(newRoutesMap);
    refresh();
  }

  @Override
  public void refresh() {
    doRefresh();
  }
}
```

그리고 이 클래스를 아까 만든 ZuulConfig에서 bean으로 등록하되, 다른 `RouteLocator`보다 우선될 수 있도록 order를 -1로 지정하였다.

```java
@Configuration
@EnableZuulProxy
public class ZuulConfig {
  @Bean
  @ConditionalOnBean(ZuulProperties.class)
  public RouteLocator myDynamicRouteLocator(final ServerProperties server,
                                            final ZuulProperties zuulProperties) {
    MyDynamicRouteLocator myDynamicRouteLocator = new MyDynamicRouteLocator(server.getServletPath(), zuulProperties);
    myDynamicRouteLocator.setOrder(-1);
    return myDynamicRouteLocator;
  }
}
```

그 결과, 원하는 대로 적절하게 추가되고 삭제됨을 확인할 수 있었다.

