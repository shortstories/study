# Spring Test

스프링 테스트 작성 중 참고사항 정리

## Spring boot 1.4 테스트 주요 변경사항

1. `@RunWith(SpringJUnit4ClassRunner.class)` -> `@RunWith(SpringRunner.class)`
2. `@SpringApplicationConfiguration(MyApplication.class)` -> `@SpringBootTest`
3. `@WebIntegrationTest` + `@SpringApplicationConfiguration` -> `@WebMvcTest`
4. `@MockBean` 어노테이션을 사용해서 특정 Dependency를 mock 객체로 바꿔끼우는 것이 가능
5. 좀 더 사용하기 편한 **AssertJ** 기본 탑재
6. `@DataJpaTest`, `@WebMvcTest`, `@JsonTest` 등 어노테이션을 사용하면 기존처럼 Spring context 전체를 생성하는게 아니라 필요한 일부만 생성해서 테스트를 진행할 수 있음
