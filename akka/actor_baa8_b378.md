# Actor 모델

* 사람과 비슷한 'Actor' 를 기반으로 동작하는 디자인 패턴
* Java가 객체끼리 function call로 통신한다면 Actor 모델에서는 오직 'message' 로 상호 소통함

## Message

* string, int 등 primitive type 뿐만 아니라 개발자가 정의한 특정 클래스까지 가릴 것 없음
* 한 Actor는 미리 정해진 type 메세지만 처리할 수 있음

### Immutable

* message 객체는 생성된 이후에 그 상태를 변경할 수 없음
* Immutable Message는 thread-safe 하도록 도와줌. 한 thread에서 다른 thread로 message를 보낸다고 가정할 때, 두 번째 thread가 뭘 하더라도 첫 번째 thread에게 영향을 미치지 않음.

### passing

* message는 비동기로 전송함
* message는 기록을 저장해둘 수도 있고, 어떤 특정 message들은 미래에 실행하도록 미룰 수도 있게 됨

## address

* Actor는 'Location transparency' 를 가짐
  * 실제 위치를 몰라도 네트워크 자원에 접근할 수 있도록 별도의 이름을 붙여 쓰는 것

## Mailbox

* message를 보내면 직접적으로 actor의 `onReceive`으로 전달되는 것이 아니라 'Mailbox'라는 일종의 큐에 쌓임.
* mailbox는 오직 message를 받고 actor가 작업을 처리할 수 있게 될 때 까지 잠시 보관하는 역할만 수행

## Serial process

* Actor은 순서대로 언제나 한번에 한 동작만 수행함
* single thread 일 때처럼 코드를 작성할 수 있게 도와줌
* Immutable message와 더불어 Actor을 thread-safe하게 만드는 이유 중 하나

## Internal State

* Actor는 각자의 properties와 fields를 가짐.
* Actor는 재시작할 때마다 언제나 새로운 인스턴스를 생성함

## Life-cycle

![](https://petabridge.com/images/2015/what-is-an-actor/akka-actor-lifecycle.png)

* 만약에 actor가 예상치 못하게 죽으면 \(Exception을 던진다면\) actor의 supervisor가 scratch로부터 자동적으로 life cycle를 재시작함. 이 경우 mailbox에 남아있던 데이터를 유실하지 않음.

## Hierarchy

![](https://petabridge.com/images/2015/what-is-an-actor/actor-family-tree.png)

* root actor을 제외한 모든 actor는 자신의 부모를 가짐.
* 정확히는, 모든 actor는 부모로부터 만들어짐.

## Supervisor

* 모든 부모 actor는 자신의 자식 actor들을 감시함.
* 이 때, 그 전략은 `SuperviserStrategy` 객체를 바탕으로 함.
  1. Restart : 일정 시간을 간격으로 계속해서 재시작 시도함
  2. Stop : 영구적으로 제거함
  3. Escalate : 자신의 부모에게 supervision 책임을 넘김

## 장점

### Cheap

* Actor 하나가 메모리에서 차지하는 크기가 작음

### Scale Out

* 하나의 Actor는 하나의 작업밖에 처리할 수 없지만, 위에서 말한 것 처럼 저렴한 특성 덕분에 대량의 Actor을 가질 수 있음
* 반대로 말하면 하나의 Actor로 작업을 처리하게 되면 비효율적이고 느리다는 뜻
* Actor 모델을 이용하고 싶다면 scale out을 필수적으로 해야만 최대 이득을 얻을 수 있음

### Lazy

* Mailbox에 아무런 message가 없는 Actor는 시스템 자원을 사용하지 않음

