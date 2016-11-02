# Static Website
- 순위 참고
  - https://www.staticgen.com/

## Dynamic Website
### 문제점
  - 보안 이슈 발생
  - Scaling 비용 비쌈
  - 불필요한 자원 소모 발생
  - 웹사이트를 유지보수하기 위해 복잡한 코드들을 파악해야 함

### 대안
- 캐싱
  - 사용하기 어려움
    - 특히 cache invalidation가 매우 어려운 작업
  - 캐싱을 해도 결국 static website에 비해 느릴 수 밖에 없음

## Static Website Generator
- 물론 Static website를 만드는 것이 dynamic으로 만드는 것에 비해서 수고가 많이 들어가는 것은 사실
- 그러나 최근에 Static website generator을 사용하게 된다면 이러한 문제점의 상당수가 해소 가능

### 특징
#### Templating
- 각 페이지를 레이아웃으로 나눔
- 중복되는 코드를 제거
- logic-less, mixture of template and code 등 다양

#### Markdown support
- 모든 Static website generator들이 Markdown을 지원함
- 디자인 요소와 컨텐츠를 분리하여 관리할 수 있게 도와줌
- 모든 컨텐츠를 순수한 텍스트로 저장할 수 있음

