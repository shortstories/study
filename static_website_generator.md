# Static Website

* 순위 참고
  * [https://www.staticgen.com/](https://www.staticgen.com/)

## Dynamic Website

### 문제점

* 보안 이슈 발생
* Scaling 비용 비쌈
* 불필요한 자원 소모 발생
* 웹사이트를 유지보수하기 위해 복잡한 코드들을 파악해야 함

### 대안

* 캐싱
  * 사용하기 어려움
    * 특히 cache invalidation가 매우 어려운 작업
  * 캐싱을 해도 결국 static website에 비해 느릴 수 밖에 없음

## Static Website Generator

* 물론 Static website를 만드는 것이 dynamic으로 만드는 것에 비해서 수고가 많이 들어가는 것은 사실
* 그러나 최근에 Static website generator을 사용하게 된다면 이러한 문제점의 상당수가 해소 가능

### 특징

#### Templating

* 각 페이지를 레이아웃으로 나눔
* 중복되는 코드를 제거
* logic-less, mixture of template and code 등 다양

#### Markdown support

* 모든 Static website generator들이 Markdown을 지원함
* 디자인 요소와 컨텐츠를 분리하여 관리할 수 있게 도와줌
* 모든 컨텐츠를 순수한 텍스트로 저장할 수 있음

#### Meta Data

* 여러 메타 데이터들도 컨텐츠의 제일 앞부분에 붙여 함께 관리
* 일반적으로 YAML 포맷을 사용하곤 함

#### Asset pipeline

* asset의 컴파일러, 변환기 등 여러 툴들을 파이프라인으로 관리
  * 예를 들면 Babel이라던가, LESS라던가, Minifier 따위
* 일부는 Grunt, Gulp 등 기존의 build tool에 기초
* 별도의 복잡한 설정 없이 이용할 수 있게 제공됨

#### Putting it all together

* CLI든 GUI든 웹 인터페이스건 이용하여 build를 하면 바로 배포 가능한 Static website를 만들어줌

### 지금에야 화제가 된 이유

#### 브라우저의 발달

* 현대의 브라우저는 일종의 자체적인 OS처럼 동작
* 따라서 하나의 완전한 Web Application을 실행할 수 있게 됨
* 서버 사이드의 코드로만 가능하던 일들을 이제는 클라이언트 사이드에서 처리하는 것이 가능
  * 수많은 add-ons로 필요한 기능을 모두 붙일 수 있음

#### CDN의 대두

* 예전에는 규모가 큰 엔터프라이즈들이나 쓰던 CDN이었지만 지금은 모두가 쓸 수 있게 진입 장벽이 낮아짐
* Static website는 CDN에서 바로 제공하는 것이 가능

#### 중요해진 성능

* 모바일 환경의 규모가 거대해짐에 따라 성능의 중요성이 급상승
* 아무리 Dynamic website의 성능을 최적화 하더라도 Static website를 따라잡기 힘듬
* 개발 단계에서 성능 최적화에 들어가는 비용이 확연히 줄어듬

#### 어디에서나 쓰이게 된 build tool

* 예전엔 build tool은 백엔드 개발자들이나 신경쓰던 물건
* 그러나 지금은 프론트엔드 개발자들도 수많은 build tool들을 사용하고 거기에 익숙해진 상황
* 그에 따라 쉽게 플로우에 적응할 수 있게 됨

