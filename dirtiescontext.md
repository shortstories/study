# `DirtiesContext`

스프링 어플리케이션에서 단위테스트를 작성하다보면 이상한 일이 발생할 때가 있다. 단독으로 돌리게 되면 멀쩡하게 수행되는 케이스가 인텔리제이에서 "Run all Tests"로 모든 단위테스트를 한번에 돌릴 때는 실패하는 것이다.

알고보니 Spring test에서 같은 context를 사용하는 테스트\(같은 context.xml 파일을 이용해서 생성되거나, 같은 SpringBootApplication 이용\)가 여러 개 있을 때 각각의 테스트마다 새로운 context를 생성하는게 아니라 기존의 context를 재활용하기 때문에 발생하는 문제였다. 앞선 테스트에서 특정 Bean의 속성값을 바꾸거나, 제거하거나, 추가하게 되면 다음에 오는 테스트를 실패하게 만들 수도 있는 것이다.

그에 따른 해결책이 바로 `@DirtiesContext`이다. 이 어노테이션을 통해 테스트를 수행하기 전, 수행한 이후, 그리고 테스트의 각 테스트 케이스마다 수행하기 전, 수행한 이후에 context를 다시 생성하도록 지시할 수 있다.

[http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/\#\_\_dirtiescontext](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/\#\_\_dirtiescontext)

## 사용법

메소드나 클래스레벨에 어노테이션을 붙이는 것으로 동작하며, methodMode, classMode에 따라 각각 다르게 동작하게 설정할 수 있다.

1. 클래스의 테스트가 시작하기 전에 context 재생성
   ```java
   @DirtiesContext(classMode = BEFORE_CLASS)
   public class FreshContextTests {
    // 테스트 케이스들이 새로운 context에서 실행됨
   }
   ```
2. 클래스의 테스트가 모두 끝난 다음 context 재생성 \(기본값\)
   ```java
   @DirtiesContext
   public class ContextDirtyingTests {
    // 테스트 케이스가 context의 @Bean의 상태에 영향을 끼침
   }
   ```
3. 클래스의 모든 테스트 케이스마다 시작하기 이전에 context 재생성
   ```java
   @DirtiesContex(classMode = BEFORE_EACH_TEST_METHOD)
   public class FreshContextTests {
    // 모든 케이스에서 새로운 context가 필요함
   }
   ```
4. 클래스의 모든 테스트 케이스가 끝날 때 마다 context 재생성
   ```java
   @DirtiesContext(classMode = AFTER_EACH_TEST_METHOD)
   public class ContextDirtyingTests {
    // 모든 케이스가 context의 상태에 영향을 끼침
   }
   ```
5. 특정 케이스를 시작하기 전에 context 재생성
   ```java
   @DirtiesContext(methodMode = BEFORE_METHOD)
   @Test
   public void testProcessWhichRequiresFreshAppCtx() {
    // 새로운 context가 필요한 어떤 로직
   }
   ```
6. 특정 케이스를 시작한 이후 context 재생성
   ```java
   @DirtiesContext
   @Test
   public void testProcessWhichDirtiesAppCtx() {
    // context의 상태를 변경하는 어떤 로직
   }
   ```