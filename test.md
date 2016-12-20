# Test

스프링 테스트 작성 중 참고사항 정리



## Spring boot 1.4 테스트 주요 변경사항

1. `@RunWith(SpringJUnit4ClassRunner.class)` -&gt; `@RunWith(SpringRunner.class)`
2. `@SpringApplicationConfiguration(MyApplication.class)` -&gt; `@SpringBootTest`
3. `@WebIntegrationTest` + `@SpringApplicationConfiguration` -&gt; `@WebMvcTest`
4. `@MockBean` 어노테이션을 사용해서 특정 Dependency를 mock 객체로 바꿔끼우는 것이 가능



## `DirtiesContext`

스프링 어플리케이션에서 단위테스트를 작성하다보면 이상한 일이 발생할 때가 있다. 

