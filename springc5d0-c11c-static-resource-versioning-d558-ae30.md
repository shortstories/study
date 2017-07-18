# Spring에서 효율적으로 Static resource 관리하기

이전에는 static resource들을 관리할 때 HTTP cache header를 \(`Expires`, `Cache-Control`, `Last-Modified`\) 통해서 관리하는게 일반적이었다. 스프링에서 설정하기도 쉬웠다. 그냥 Java Config든 XML이든 하나 써서 설정하면 된다. 그러나 이런 방법을 사용하면 많은 문제들이 남아있게 된다. 우선, 만약에 리소스에다가 minify, uglify 같은 변환을 적용하고 싶다면 어떻게 할 것인가? 그리고, less, sass, gzip같은 리소스들은 언제 어떻게 컴파일해서 제공할 것인가? 그 외에 새로운 스크립트를 배포하였지만 클라이언트 브라우저에 남아있는 캐시때문에 문제가 생기는 일도 있을 것이다. 이에 따라 스프링 4.1부터 위와 같은 방법에 대응할 수 있는 수단이 추가되었다.

## Resource versioning

이것을 Fingerprinting URL이라고 부르기도 한다. 스프링에 추가된 기능은 RoR의 asset pipeline에 기초를 둔다. 이 기능의 요점은 다음과 같다. 가령, `/css/mystyle.css` 이라는 파일이 있다고 하면 자동적으로 `/css/mystyle-f1l521f.css` 같은 이름으로 바꿔서 제공한다. 이렇게 되면 HTTP header의 캐시를 얼마든지 길게 하더라도 언제나 최신의 리소스를 제공할 수 있게 된다. 특히, 저 버전은 css파일의 내용을 hash한 값이므로 파일의 내용이 변경되지 않는다면 파일 이름에도 변경이 생기지 않아 캐시가 효율적으로 동작한다.

### ResourceResolver, ResourceTransformer

스프링 4.1에서 추가된 클래스들이다. 일반적으로 스프링에서 리소스를 제공할때 호출되는 \`ResourceHttpRequestHandler\` 사이에 끼워넣어 동작하게 만들 수 있다.

```
        +------------+
        |HTTP Request|
        +-----+------+
              |
              |
              |
              v
+-------------+--------------+
|                            |
|   ResourceResolver Chain   | 체인을 하나하나 통과하면서 적절한 리소스 탐색
|                            |
+-------------+--------------+
              |
              |
              v
+-------------+--------------+
|                            |
| ResourceTransformer Chain  | 체인을 하나하나 통과하면서 리소스를 변경
|                            |
+-------------+--------------+
              |
              |
              |
              v
        +-----+-------+
        |HTTP Response|
        +-------------+

```



