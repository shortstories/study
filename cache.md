# Cache

- 스프링 3.1버전 이후부터 지원
- 스프링 4.1 버전부터 [JCache Annotations](https://jcp.org/en/jsr/detail?id=107)에 맞춰서 기능이 개선됨

## Cache Abstraction

- 자바 메소드에 캐싱 적용
  - 메소드를 실행하기 전에 캐시를 먼저 거치게 하여 메소드의 실행 횟수를 줄여줌
  - 주어진 argument를 기준으로 판단
    - 따라서 같은 input이 주어지면 언제나 같은 output이 나와야함
- 캐시를 지우거나 수정할 수 있는 기능 제공
- 추상체기 때문에 실제 캐시 저장소 구현체중 하나를 선택하여 사용해야 함
  - `org.springframework.cache.Cache`, `org.springframework.cache.CacheManager` 인터페이스를 사용해서 접근
- 멀티 쓰레드, 멀티 프로세스 관련 처리는 캐시 구현체에서 처리해야 됨
  - 특히 멀티 프로세스의 경우 여러 노드에서 같은 데이터를 가질 수 있도록 처리해야 됨
- 
### Cache vs Buffer

- Cache와 Buffer은 서로 바꿔 쓸 수 있는 단어
- Buffer는 전통적으로 연산 속도가 서로 다른 객체 사이에 임시로 데이터를 저장하는 저장소 지칭
  - 연산 속도가 빠른 쪽이 불필요하게 대기하지 않도록 도와 성능 향상에 도움을 줌
  - 작은 데이터 조각들을 모아서 한번에 움직일 수 있게 도와줌
  - 적어도 한 객체는 버퍼의 존재를 알아야함
- Cache는 보이지 않고 또한 두 객체 모두 그 존재를 알 필요가 없음
  - 같은 데이터를 여러 번 읽으려고 할 때 상대적으로 빠른 방법을 제공하여 성능 향상에 도움을 줌

## Annotation

### `@Cacheable` 



### `@CacheEvict`
### `@CachePut`
### `@Cacheing`
### `@CacheConfig`