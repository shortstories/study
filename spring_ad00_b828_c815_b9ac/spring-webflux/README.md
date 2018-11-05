# Spring Webflux

spring-webflux는 spring-webmvc의 대체재. 물론 같이 사용할 수도 있음.

새로운 웹 프레임워크 = 논블로킹 I/O + 함수형 프로그래밍

반응형 프로그래밍

* 어떤 변화에 반응해서 동작
* 논블로킹
* 백 프레셔 : 기존 동기식 로직은 블로킹때문에 데이터 양이 알아서 조절이 되었지만 반응형은 프로그램이 지나친 데이터 양에 무너지지 않게 하기 위해서 직접 조절할 필요가 있음

Reactive Streams: JAVA9에서 나온 스펙. 비동기 컴포넌트와 백 프레셔간의 상호작용을 정의

## 선택 기준

### Spring MVC vs Spring WebFlux

1. 이미 잘 돌아가는 Spring MVC 프로그램이 있다? 그럼 굳이 바꿀 필요까진 없음.
2. 좀 더 가벼운 프로그램, 함수형 웹 프레임워크에 관심이 있다면 써봐도 괜찮음.
3. Spring MVC의 전형적인 controller 방식이랑 Spring WebFlux functional endpoints는 둘 다 동시에 써도 되므로 같이 써보면서 비교해보고 골라도 괜찮음.
4. DB API가 블로킹이거나 블로킹인 네트워크 API를 쓰고있다면 Spring MVC가 더 좋음. 일부 블로킹 콜을 쓰레드로 분리해서 쓸 수도 있지만 굳이 그렇게까지 해야될 필요성이 있을지 잘 모르겠음.
5. 그래도 굳이 써보고싶다면 기존의 Spring MVC 프로그램에다가 `WebClient`부터 부분적으로 적용시켜 써보는게 좋음. WebFlux의 HTTP Client임.

### Server

WebFlux는 Netty가 기본 서버. 필요에 따라 비동기 API를 지원하는 다른 서버를 사용하는 것도 물론 가능.

## Reactive Spring web Server

### HttpHandler

`HttpHandler` 는 request를 받아 response를 돌려주는 하나의 함수. 특정 서버에 국한되지 않고 범용성 있게 Reactive Stream API로 HTTP request를 처리할 수 있도록 추상화됨. 각 서버에 따라 알맞은 HttpHandlerAdapter가 존재함.

### WebHandler API

`HttpHandler` 에다가 일반적으로 Web에서 쓰는 세세한 기능들을 부여할 수 있도록 만든 API. 기존 Spring MVC의 Filter나 Session, ExceptionHandler 등의 기능을 추가할 수 있게 만들어져있음. 모든 컴포넌트들은 `ServerWebExchange` 를 받아서 제각기 처리. 보통은 `WebHttpHandlerBuilder` 를 통해 원하는 컴포넌트들을 조합하여 `HttpWebHandlerAdapter` 를 만들어 사용.

### Codecs

코덱 개념 자체는 Netty의 그것과 거의 유사. `DataBuffer`, `Encoder`, `Decoder` 요렇게 세가지가 있는데 `DataBuffer`은 Netty의 `ByteBuf` 랑 비슷하고 `Encoder`, `Decoder`은 말 그대로 그 버퍼를 원하는 특정 Object로 변환하고 다시 바이트 버퍼로 변환하는 역할.

spring-web에서는 HTTP Message와 body를 읽고 쓰기 위해 `HttpMessageReader` `HttpMessageWriter` 두 인터페이스를 제공하는데 위의 `Encoder` 와 `Decoder` 을 한번 감싸서 그대로 사용할 수도 있음.

## DispatcherHandler

`WebHandler`에 Spring MVC에서 `DispatcherServlet` 이 수행하던 기능들을 덧붙인 것. Spring configuration에서 필요한 컴포넌트들을 땡겨와서 세팅된다. 보통은 bean name을 "webHandler" 으로 지어서 `WebHandler` 역할을 수행하도록 한다.

일반적으로 WebFlux 어플리케이션은 다음과 같은 bean들을 포함한다

* 이름이 "webHandler"인 `DispatcherHandler` bean
* `WebFilter`, `WebExceptionHandler` bean
* `HandlerMapping`, `HandlerAdapter`, `HandlerResultHandler`bean

### Special bean types

Spring MVC에서 우리가 controller, service, repository라고 부르던 레이어. `DispatcherHandler` 는 여기에 속하는 bean에게 역할을 위임하여 request를 처리하고 response를 생성한다.

* `HandlerMapping`: request랑 Handler를 매핑해주는 bean. 익숙한 `@RequestMapping` 어노테이션이나 `RouterFunction`, URL 정보 등을 미리 가지고 있다가 request가 들어올 때마다 검사하여 적합한 Handler를 돌려주는 역할
* `HandlerAdapter`: Handler를 실제로 실행하기 위해 필요한 bean. 예를 들어 Handler중 하나인 `HandlerFunction` 은 parameter로 `ServerRequest` 하나를 받는데 `DispatcherHandler`은 `ServerWebExchange` 만 가지고 있음. 그렇다면 실제로 `HandlerFunction` 을 실행하기 위해서는 `ServerWebExchange` 를 `ServerRequest`로 바꾼 다음 parameter로 넣어서 실행하는 과정이 필요한데 이러한 역할을 수행하는 부분.
* `HandlerResultHandler`: Handler 결과값을 바탕으로 response를 처리하기 위해 필요한 bean. 예를 들면 반환값 타입에 따라 `ResponseEntity<?>` , `@ResponseBody` , String으로 된 view name 모두 제각기 구현체가 등록되어있어야 함.

### 설정 방법

세 가지 선택지가 있음

1. 위 세 가지 bean을 직접 스프링 컨테이너에 등록하는 방법
2. `@EnableWebFlux` 어노테이션을 원하는 특정 `@Configuration` 클래스에 붙이고 `WebFluxConfigurer` 인터페이스를 상속, 필요한 부분을 구현해서 쓰는 방법
   1. `@EnableWebFlux` 어노테이션을 발견하면 기본 값으로 초기화
3. `DelegatingWebFluxConfiguration` 인터페이스를 상속해서 처음부터 끝까지 직접 다 세팅하는 방법

### 동작 방식

1. `HandlerMapping` 를 모조리 돌면서 제일 먼저 처리할 수 있는 Handler를 찾음
2. Handler를 찾았으면 거기에 적합한 `HandlerAdapter` 를 찾음. 찾았으면 `HandlerAdapter` 에서 실제 Handler를 실행하고, 그 결과를 `HandlerResult` 로 바꿔서 넘겨줌.
3. 이번에는 `HandlerResult` 를 처리할 수 있는 `HandlerResultHandler` 를 찾아서 결과값을 처리하여 view나 값을 담아 response를 보냄.

## Functional endpoints

spring mvc에서는 `@Controller` 와 `@RequestMapping` 을 써왔다면, spring webflux에서 완전 새롭게 추가된 방법. 물론 spring mvc처럼 컨트롤러를 작성해도 알아서 `RequestMappingHandlerMapping` 으로 변환되서 등록되기 때문에 정상작동하긴 함.

직접 java lambda 등을 써서 함수형 프로그래밍처럼 하고싶다면 `RouterFunction` 과 `HandlerFunction` 을 활용해야함. `RouterFunction`은 request가 왔을 때 request의 정보를 바탕으로 로직을 수행해서 자기가 가지고 있는 `HandlerFunction` 으로 처리할 수 있는지 검사하고, `HandlerFunction` 은 실제로 request를 처리하여 response를 반환. `RouterFunction<ServerResponse>` 형태로 bean을 생성, 등록하면 됨.

```java
  @Bean
  public RouterFunction<ServerResponse> index() {
    final Map<String, ?> emptyModel = new HashMap<>();
    return RouterFunctions.route(
        RequestPredicates.GET("/").and(RequestPredicates.accept(MediaType.TEXT_HTML)),
        req -> ServerResponse.ok().render("index", emptyModel)
    );
  }
```

index.html를 위처럼 출력 가능. 다만 webflux는 viewResolver가 자동으로 등록되지 않기 때문에 `WebFluxConfigurer` 을 사용해서 설정해주는 것이 좋음.

`RouterFunction` 을 여러 개 등록하면 가장 먼저 일치하는 `HandlerFunction` 을 실행하므로 `.path("/**")` , `.path("/somePath")`처럼 중첩된 범위를 가지는 `RouterFunction` 을 여러 개 등록한다면 반드시 좀 더 자세한 범위의 `RouterFunction` 을 먼저 등록해야 함. /\*\* 다음에 /somePath를 등록하게 되면 /somePath에 등록한 `HandlerFunction` 은 절대로 호출되지 않음.

## WebSocket

### 사용법

```java
@Configuration
public class WebSocketConfig {
  @Bean
  public WebSocketHandlerAdapter handlerAdapter() {
    return new WebSocketHandlerAdapter();
  }

  @Bean
  public HandlerMapping webSocketMapping() {
    Map<String, WebSocketHandler> map = Maps.newHashMap();
    map.put("/websocket", session -> {
      /* websocket handling */
      return Mono.never();
    });

    SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
    mapping.setUrlMap(map);
    mapping.setOrder(-2);
    return mapping;
  }
}
```

위처럼 `WebSocketHandlerAdapter` bean을 등록하고 `SimpleUrlHandlerMapping` 을 사용해서 websocket 요청을 매핑하면 끝.

여기서 중요한건 session의 종료 시점. connection이 생성될 때, websocket session을 따로 저장해두고 관리하는 경우가 많은데 그러면 난처한 경우를 겪을 수 있음. 왜냐하면 webflux의 websocket session은 서버에서 직접 close를 호출하지 않아도 Websocket Handler \(위예제에서는 `session -> Mono.never()`\) 가 return하는 `Mono`의 complete에 의해서 close 되어버리기 때문. 즉 `Mono.never()` 대신에 `Mono.empty()` 를 반환해보면 의도와는 달리 session이 멋대로 종료되어버리는 것을 볼 수 있음. `Mono.never()` 를 쓰면 그런 부분은 없어지지만 이 경우도 난감한게 직접 session을 close할 방법이 없음. 그러므로 session을 별도로 관리하려면 `Mono.never()` 대신에 `Processor` 을 반환하고, 반환한 `Processor` 을 session과 함께 저장해둔 다음 필요할 때 `.complete()` 를 호출하는 방향으로 해야할 듯함. 제일 좋은건 역시 handler 내부에서 모든 처리를 끝내는 것이고.

