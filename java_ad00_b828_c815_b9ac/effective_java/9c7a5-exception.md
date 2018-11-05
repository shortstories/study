# 9장 - Exception

## 쓸데없이 Exception 쓰지 않기

예를 들어 반복문 빠져나오기위해 Exception을 쓴다던가 하는 경우. 정상적인 흐름에 Exception을 전제로 깔고 코드 짜지 말라는 뜻. 비슷한 관점에서 API를 쓸 때 반드시 Exception을 catch해야만 쓸 수 있다면 좋은 API가 아니다. `Iterator` 은 그래서 `hasNext()`를 가지고 있고 `Optional` 은 `isPresent()` 를 가지고 있다. 얘들을 보고 state-testing 메소드라고 부르는데 state-dependent한 `Iterator.next()` 또는 `Optional.get()` 을 호출하기 전에 미리 검사해서 Exception을 피할 수 있도록 해준다.

1. JVM이 정상적으로 최적화할 수 없는 경우가 발생할 수 있음 -&gt; 성능 저하
2. 의도된 상황과 또 다른 예외 케이스가 발생한다거나 할 때 디버깅이 아주 어려워질 수 있음
3. 당연히 코드 알아보기도 어려움

## Exception vs RuntimeException vs Error

### Exception

checked exception이라고 부르기도 함. 만약에 이 Exception을 throw하면 반드시 try-catch문을 사용해서 처리해야만 함. 즉, API의 사용자에게 강력한 경고 또는 암시를 줄 수 있음. try-catch문을 사용해서 어떤 특별한 조치를 취하면 그대로 쓰레드를 이어나갈 수 있는 그런 케이스에 사용해야 함. 물론 API 사용자가 무시할 수도 있겠지만 이거는 사용자 문제고.

### RuntimeException

unchecked exception이라고 부르기도 함. try-catch로 꼭 감싸줄 필요는 없음. 오히려 catch를 안 하는게 좀 더 바람직한 상황이 많음. RuntimeException을 throw하는건 발생한 상황이 복구도 불가능하고 계속 실행해봤자 득될게 없는 상황일 때여야 함. 달리 말하면 에러메세지를 보여주면서 쓰레드가 종료되는게 더 좋은 상황임. 만약에 복구가 가능한지 불가능한지 긴가민가하다면 RuntimeException이 더 좋은 선택임.

RuntimeException은 주로 프로그래밍 에러를 나타냄. 주로 validation에 실패한 경우가 많음. 가령 예를 들어 배열에서 어떤 원소를 호출할 때 그 index는 0부터 length - 1이어야 하는데 음수거나 length보다 큰 숫자를 넣으면 RuntimeException을 던지는 것과 비슷함.

### Error

마찬가지로 unchecked exception인데, JVM에서 리소스 부족이나 불변 규칙 위반에 따른 상황, 실행이 불가능항 상황에 사용함. 그러므로 프로그래머가 사용하는 일은 없다고 봐도 됨.

## 쓸데없이 Checked Exception 쓰지않기

checked exception이 던져지면 api 사용자 입장에서는 try-catch로 처리하던가 다시 throw로 한번 더 던져야하는데 코드도 덕지덕지 늘어나고 이래저래 부담이 큼. 그러므로 가급적이면 안 쓰는게 좋음. 만약에 catch문의 내용으로 아예 프로그램을 종료하는 코드거나 다시 throw하는 코드가 많다면 checked보단 unchecked가 더 나을 수 있다는 뜻임. 그리고 그냥 checked를 unchecked로 바꾸고 싶은 케이스도 있을건데 그럴 땐 아까 위에서 언급한 state-testing, state-dependent 메소드처럼 두개로 쪼개는 방법이 있음.

