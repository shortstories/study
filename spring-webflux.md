# Spring webflux

## 토비님 강연

### 사전 지식

* 스프링은 DI가 런타임에 결정되므로 별도의 컨테이너가 있어야함 -&gt; 3
* 동기 비동기는 2+
  * 동기는 시간을 맞추고
  * 비동기는 시간을 안 맞추고
  * 중요한건 대상, 시간
  * A, B 시작시간 == 종료시간 -&gt; 동기. 
  * 멀티쓰레드라도 두 쓰레드가  동시에 작업을 시작하면 동기
  * 메소드 리턴 시간과 결과를 전달받는 시간이 일치하면 동기
  * A가 끝나는 시간과 B가 시작하는 시간이 같아도 동기
* 블록킹, 논블록킹

  * 내가 직접 제어할 수 없는 대상을 상대하는 방법
  * IO, 네트워크, 멀티스레드 동기화 등

* Servlet 3.0

  * 비동기 요청 처리

* Spring 3.2부터 비동기 지원

* CompletableFuture, CompletionStage

  * ListenableFuture의 개선인가?

* 스프링 웹 처리

  * * 리퀘스트 매핑 \(라우팅\): 어느 핸들러로 보낼지

    * 리퀘스트 바인딩: 핸들러에 전달할 argument 준비

    * 핸들러 실행: 로직 수행

    * 핸들러 리턴 결과로 응답 생성: response 생성 json 변환 뷰리졸버 등

### 특징

* 서블릿 스택, API 벗어남

  * 리액티브 함수형하고 안 어울림
  * ServerRequest, ServerResponse

* Reactor 스트림을 그대로 리턴하는게 가능

### 관련 API

* RouterFunction

  * 리퀘스트를 바탕으로 어느 HandlerFunction이 처리할지 찾음 \(디스패처?\)

* HandlerFunction

  * 리퀘스트를 처리해서 결과값 반환 \(= 컨트롤러\)

* 좀 노드 js같기도

* RouteFunctions.route\(조건, HandlerFunction\)

* RouterFunction 등록

  * @Bean으로 만들기
  * 여러 조건을 and\(\), or\(\) 등으로 엮을 수 있고
  * @RequestMapping을 클래스레벨로 걸고 메소드레벨로 걸던 것을 nest\(\)로 중복되게 할 수 있고

### 장점

* 어노테이션으로 감춰진 부분 없이 모든 웹 처리 과정을 명시적인 코드로 작성?
* 함수 조합이라 추상화에 유리
* 테스트 작성 편리
  * 모든 로직을 내가 처리하므로 단위테스트로 작성 가능

### WebFlux + Spring Mvc

* 그냥 어노테이션 기반 대부분 그대로 쓰고 값을 리턴할 때만 Reactor 클래스에 담아서 리턴해야됨

* @RequestBody의 경우 Mono&lt;T&gt;, Flux&lt;T&gt; 안에 들어가있는 형식으로 받을 수도 있다.

  * HTTP Stream 스펙을 쓸 때, Flux&lt;T&gt;를 쓰면 스트리밍 형태로 계속 하나하나 받아서 리턴할 수도 있다.

* @ResponseBody는 Mono, Flux, ServerSideEvent?

### 성능 개선?

* DB 연결
  * JDBC API에 블로킹 메소드가 넘쳐나서 답이 없음
  * 일부 DB에 논블로킹 드라이버가 존재하지만 지원하지 않고 있음
  * 일단 지금 가능한건 쓰레드풀 관리를 효율적으로 하는 정도에 불과.
  * NoSQL은 Spring data reactive 로 만들어진걸 쓸 수 있음.
* HTTP API
  * AsyncRestTemplate 사용
* 중요한 것

  * WebFlux + 리액티브 레포지터리, API 호출, 지원하는 외부 서비스

  * 만약에 리액티브를 지원하지 않는 블록킹 API라면 @Async 써서 다른 쓰레드에서 돌도록. 이건 노드js랑 비슷하네.

### + 알파

* 함수형 스타일 쓰고싶으면
* 데이터 흐름에 다양한 오퍼레이터
* 연산을 조합해서 추상회되어 동시성 정보를 노출하지 않음
* 데이터 흐름의 속도 제어 가능
  * 이게 아마 백프레셔였던가...

---

## 공식 문서

spring-webflux는 spring-webmvc의 대체재. 물론 같이 사용할 수도 있음.

새로운 웹 프레임워크 = 논블로킹 I/O + 함수형 프로그래밍

반응형 프로그래밍

* 어떤 변화에 반응해서 동작
* 논블로킹
* 백 프레셔 : 기존 동기식 로직은 블로킹때문에 데이터 양이 알아서 조절이 되었지만 반응형은 프로그램이 지나친 데이터 양에 무너지지 않게 하기 위해서 직접 조절할 필요가 있음

Reactive Streams: JAVA9에서 나온 스펙. 비동기 컴포넌트와 백 프레셔간의 상호작용을 정의

### 선택 기준

#### Spring MVC vs Spring WebFlux

1. 이미 잘 돌아가는 Spring MVC 프로그램이 있다? 그럼 굳이 바꿀 필요까진 없음.
2. 좀 더 가벼운 프로그램, 함수형 웹 프레임워크에 관심이 있다면 써봐도 괜찮음.
3. Spring MVC의 전형적인 controller 방식이랑 Spring WebFlux functional endpoints는 둘 다 동시에 써도 되므로 같이 써보면서 비교해보고 골라도 괜찮음.
4. DB API가 블로킹이거나 블로킹인 네트워크 API를 쓰고있다면 Spring MVC가 더 좋음. 일부 블로킹 콜을 쓰레드로 분리해서 쓸 수도 있지만 굳이 그렇게까지 해야될 필요성이 있을지 잘 모르겠음.
5. 그래도 굳이 써보고싶다면 기존의 Spring MVC 프로그램에다가 `WebClient`부터 부분적으로 적용시켜 써보는게 좋음. WebFlux의 HTTP Client임.

#### Server

WebFlux는 Netty가 기본 서버. 필요에 따라 비동기 API를 지원하는 다른 서버를 사용하는 것도 물론 가능.

### Reactive Spring web Server

#### HttpHandler

`HttpHandler` 는 request를 받아 response를 돌려주는 하나의 함수. 특정 서버에 국한되지 않고 범용성 있게 Reactive Stream API로 HTTP request를 처리할 수 있도록 추상화됨. 각 서버에 따라 알맞은 Adapter가 존재함.

#### WebHandler API

`HttpHandler` 에다가 일반적으로 Web에서 쓰는 세세한 기능들을 부여할 수 있도록 만든 API. 기존 Spring MVC의 Filter나 Session, ExceptionHandler 등의 기능을 추가할 수 있게 만들어져있음. 모든 컴포넌트들은 `ServerWebExchange` 를 받아서 제각기 처리. 보통은 `WebHttpHandlerBuilder` 를 통해 원하는 컴포넌트들을 조합하여 `HttpWebHandlerAdapter` 를 만들어 사용.

#### Codecs

코덱 개념 자체는 Netty의 그것과 거의 유사. `DataBuffer`, `Encoder`, `Decoder` 요렇게 세가지가 있는데 `DataBuffer`은 Netty의 `ByteBuf` 랑 비슷하고 `Encoder`, `Decoder`은 말 그대로 그 버퍼를 원하는 특정 Object로 변환하고 다시 바이트 버퍼로 변환하는 역할.

spring-web에서는 HTTP Message와 body를 읽고 쓰기 위해 `HttpMessageReader` `HttpMessageWriter` 두 인터페이스를 제공.  

