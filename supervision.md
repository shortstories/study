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
- 