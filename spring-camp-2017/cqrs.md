# CQRS

## 이벤트

* 불변 객체
* 오직 추가만 가능하게 보관
* 이벤트 스토어는 persistance하기만 하다면 무엇이든 상관없음

### 이벤트 리플레이

* 현재 상태를 복원하는 것
* 초기값으로부터 지금까지 발생한 이벤트를 모두 적용하여 마지막 결과를 이끌어냄
* 언제나 해야만 한다?
* 대량의 이벤트가 있다면 성능상 문제가 발생한다.

### 스냅샷

* 이벤트 식별자, 버전, 리플레이 끝난 마지막 도메인 객체를 저장
* 이벤트 저장 시점에 생성
* 시간 내지는 이벤트 발생 개수를 기준으로 생성
* 보통 in-memory 저장소 사용
* 이렇게 하면 리플레이로 인한 비용은 줄일 수 있겠지만, 특정 쿼리에서 너무 느려진다

## CQRS 란?

* 이러한 이벤트 소싱의 성능 한계를 해결하기위해 나옴
* 명령과 조회의 책임 분리
* 커맨드 모델과 쿼리 모델 분리
* 사실상 이벤트 소싱에서 필수라고 보면 됨

### Event Projection

* 커맨드 모델로 이벤트가 발생하면 이벤트 스토어에 저장하고 쿼리 모델로 보내서 쿼리  모델의 저장소를 업데이트 하는 과정

### Aggregate

* 연관있는 엔티티의 집합. 여기에서는 특정 도메인 오브젝트에 대한 이벤트의 리스트
* 이 객체에서 버전을 가지고 있어서 이벤트를 저장할 때 race condition이 발생하면 버전 비교해서 예기치못한 상태 변경을 피할 수 있다. 마치 칼레이도처럼.
* 이벤트마다 처리하는 apply\(\) 메소드 만들어서 처리?
* 비지니스 로직에서는 도메인 오브젝트의 상태를 변경하지 않는다. 오직 이벤트로만.

### MSA + Event Source?

* 장애 전파? 아니 근데 어차피 다른 시스템이 정상적으로 작동하지 않는데 결과가 정상적으로 리턴되는게 맞나????


