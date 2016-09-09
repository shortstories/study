# Actor 모델
- 사람과 비슷한 'Actor' 를 기반으로 동작하는 디자인 패턴
- Java가 객체끼리 function call로 통신한다면 Actor 모델에서는 오직 'message' 로 상호 소통함

## Message
- string, int 등 primitive type 뿐만 아니라 개발자가 정의한 특정 클래스까지 가릴 것 없음
- 한 Actor는 미리 정해진 type 메세지만 처리할 수 있음

### Immutable
- message 객체는 생성된 이후에 그 상태를 변경할 수 없음
- Immutable Message는 thread-safe 하도록 도와줌. 한 thread에서 다른 thread로 message를 보낸다고 가정할 때, 두 번째 thread가 뭘 하더라도 첫 번째 thread에게 영향을 미치지 않음.

### passing
- message는 비동기로 전송함
- message는 기록을 저장해둘 수도 있고, 어떤 특정 message들은 미래에 실행하도록 미룰 수도 있게 됨

### address
