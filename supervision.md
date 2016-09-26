# Supervision
## 개요
- root를 제외한 모든 actor은 부모 actor을 가짐
- 여기에서 부모 actor이 자기 자식 actor들의 supervisor 역할을 수행함
- 만약에, 자식 actor가 작업을 수행하는 도중에 exception 등 실패를 감지하게 되면 즉시 일시정지하고 자신의 supervisor에게 실패에 대한 메세지를 보내게 됨. 이러한 상황에서 supervisor에게는 4가지 선택지가 주어짐
  1. 자식 actor의 현재 상태를 유지하면서 재시작
  2. 자식 actor의 상태를 초기화하여 재시작
  3. 자식 actor을 영구적으로 정지
  4. 자식 actor에 대한 처리를 한 단계 위의 supervisor에게 위임
- `Actor` 클래스의 `preRestart` 의 기본 동작에서 모든 자식 actor을 종료시킴. 그것이 의도치 않은 동작이라면 해당 함수를 override할 필요성이 있음.
- 실패 대응 function은 자식 actor에 대한 정보를 입력 받지 않기 때문에, 실패한 actor의 종류 별로 분류해서 다른 전략으로 대응하고 싶다면 레벨을 더 세분화하는 것 같이 구조적으로 나눠서 대응해야 함. 만약에 하나의 레벨에서 너무 많은 실패에 대응하려고 한다면 원인을 찾는게 어려워지기 때문
- 아카는 "parental supervision" 이라고 할 수 있음. 모든 actor은 부모 actor으로부터 만들어지고, 최상단 root actor은 라이브러리가 생성함.이러한 제약이 위와 같은 actor 구조를 형성해줌. 따라서 부모가 없는 actor나 외부에서 붙여진 actor는 존재하지 않도록 해야 함.

## The Top-Level Supervisors
![](http://doc.akka.io/docs/akka/2.4/_images/guardians.png)
- ActorSystem이 생성되면 위와 같은 최소한 3개의 actor을 가지고 있음

### /user : The Guardian Actor
- 유저가 만든 모든 actor의 최상단 부모
- `system.actorOf()` 를 호출해서 만드는 모든 actor가 이 actor의 자식으로 들어감
- 만약에 이 guardian이 종료되면 모든 일반적인 actor들이 종료된다고 할 수 있음
- default supervisor strategy를 정할 때 활용할 수 있음
- 이 guardian이 어떤 실패를 root로 넘기게 되면 전체 시스템이 종료됨

### /system : The System Guardian
- 체계적으로 shut-down을 진행하기 위해 존재하는 guardian
- 예를 들면 모든 actor가 종료될 때까지 logging이 활성화되어야 하는 상황
- user guardian을 감시하다가 `Terminated` 메세지를 받으면 shut-down을 시도
- root guardian이 감시하며 `ActorInitializationException`, `ActorKilledException` 외에 모든 `Exception`에 대해서 재시작 시도
- 그 외의 모든 throwable가 넘어가게 되면 전체 시스템이 종료됨

### / : The Root Guardian
- 모든 "Top-level" actor들의 부모이며 superviser
- SupervisorStrategy.stoppingStrategy을 통해 종류를 막론하고 `Exception`이 발생하면 자식을 종료하게 만듬

## 재시작의 의미
- 어떠한 작업 도중 실패가 일어나게 되면 보통 그 원인을 세가지로 분류할 수 있음
  - 특정 메세지를 받았을 때 발생하는 프로그래밍적 에러
  - 메세지를 처리하기 위해 외부 자원을 사용할 때 발생하는 에러
  - actor의 내부 상태가 오염되어서 발생하는 에러

