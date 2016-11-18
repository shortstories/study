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

- 캐시를 사용할 메소드 표시
  - 이 어노테이션이 붙어있는 메소드는 결과가 캐시에 저장됨
- 캐시 이름은 여러 개를 동시에 입력할 수 있음
  - 최소한 하나 이상 hit가 있으면 캐시 값 반환

#### 키 생성

##### 기본 키 생성

- `KeyGenerator` 사용
  - 패러미터가 없을 땐 `SimpleKey.EMPTY`
  - 패러미터가 한 개 있을 땐 패러미터 인스턴스 사용
  - 패러미터가 여러 개 있을 땐 모든 패러미터를 담은 `SimpleKey` 사용
- 패러미터 객체에 적절한 `hashCode()`, `equals()` 구현이 있다면 수정할 필요 없음
- 기본 키 생성 방식을 바꾸고 싶으면 `org.springframework.cache.interceptor.KeyGenerator` 인터페이스를 구현
- 스프링 4.0 이전 버전에서는 `org.springframework.cache.interceptor.DefaultKeyGenerator` 사용
  - 여러 패러미터를 가진 키를 생성할 때 `hashCode()`만 사용하여 예기치 않은 키 충돌을 일으킴
  - 4.0 이후로는 `SimpleKeyGenerator` 사용


##### 커스텀 키 생성

- 메소드의 모든 패러미터를 써서 캐시 키를 만들기엔 적절하지 않을 때 사용
  - 일부만 캐시 키로 쓰고, 나머지는 메소드 내부 로직에만 사용하는 경우
- SpEL을 사용해서 특정 패러미터, 혹은 그 패러미터의 프로퍼티를 지정
- 코드 베이스가 확장됨에 따라 메소드의 패러미터도 복잡해지기 때문에 추천되는 방법
  - 기본 키 생성 전략이 모든 메소드에 적용되는 일은 드뭄
- ex)

``` java
@Cacheable(cacheNames="testCache", key="#param")
public Object test(Object param, Object param2)
```

- 정말 특별한 키 생성 알고리즘을 쓰거나 샤딩이 필요하면 `keyGenerator`에 별도의 bean 정의
  - `key`, `keyGenerator` 두개는 동시에 적용할 수 없음

#### 캐시 탐색

##### 기본 캐시 탐색

- `CacheResolver` 에서 `CacheManager` 의 설정을 보고 캐시 탐색
  - 마찬가지로 기본 `CacheResolver`을 바꾸고 싶으면 `org.springframework.cache.interceptor.CacheResolver` 구현

##### 커스텀 캐시 탐색

- `cacheManager` 에 별도의 CacheManager bean을 정의하거나 아예 `cacheResolver` 에 CacheResolver bean을 정의해서 갈아끼워도 됨
  - 둘 중 하나만 선택

#### 캐싱 동기화

- 캐시 구현체가 thread safe 하지 않을 때, 자체적으로 캐시 접근에 동기화를 거는 방법
  - `@Cacheable(..., sync="/* true or false */")`

#### 조건부 캐싱

- 특별한 조건을 만족할 때만 캐싱하도록 제약을 거는 방법

##### `condition`
- `condition` 패러미터에 SpEL 방식으로 작성
- 연산 결과가 true일 때 캐싱
- false일 땐 캐싱되지 않았을 때와 동일하게 동작
- 현재 캐시 상태, 패러미터와 상관없이 항상 실행됨
- ex) `condition="#name.length < 32"`


##### `unless`
- 

### `@CacheEvict`

- 

### `@CachePut`
### `@Cacheing`
### `@CacheConfig`