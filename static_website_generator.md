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

#### Meta Data
- 여러 메타 데이터들도 컨텐츠의 제일 앞부분에 붙여 함께 관리
- 일반적으로 YAML 포맷을 사용하곤 함

#### Asset pipeline
- asset의 컴파일러, 변환기 등 여러 툴들을 파이프라인으로 관리
  - 예를 들면 Babel이라던가, LESS라던가, Minifier 따위
- 일부는 Grunt, Gulp 등 기존의 build tool에 기초
- 별도의 복잡한 설정 없이 이용할 수 있게 제공됨

#### Putting it all together
- CLI든 GUI든 웹 인터페이스건 이용하여 build를 하면 바로 배포 가능한 Static website를 만들어줌
