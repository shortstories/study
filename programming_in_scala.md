# Programming in Scala
## 1장
- 스킵

## 2장
- 스킵

## 3장
### List
- list의 제일 앞에 원소 추가하는건 O(1), 뒤에 추가하는건 O(n)
- 그러므로 앞에 추가하는게 좋음. 만약에 뒤에 추가하고 싶다면 우선 뒤집고 앞에 추가한 다음에 다시 뒤집거나 `ListBuffer` 사용

### Tuple
- tuple은 `tuple._i` 같은 형태로 원소 호출
- i는 1부터 시작 (하스켈, ML 등 정적 타입 언어의 영향)
- list와 tuple의 차이점은 여러 자료형을 한번에 담을 수 있느냐의 차이

### Map
- map의 각 원소는 tuple

### 함수형 스타일
- var보단 val을 더 권장
- side effect를 최소화
  - 만약에 Unit을 반환하는 함수가 있다면 주의할 것

## 4장
### Class, Field, Method
- Scala의 모든 필드는 기본적으로 public
- 메소드 패러미터는 기본적으로 val
- return을 되도록 쓰지말고, 쓰더라도 1번 이상 쓰지 않을 것. 각 메소드가 한 값을 계산하는 표현식이 되도록 구현
- side effect을 위한 메소드라면 결과타입을 생략하고 = 도 떼어서 프로시저처럼 보이는게 좋음
  ```scala
  def procedure(params: Any) {
    {// something
  }
  ```
- =를 빼먹으면 무조건 `Unit`을 반환하므로 주의

### Semicolon
- 한 줄을 여러 개로 나누려면 사용 ex) `val s = "hello"; println(s)`
- 기본적으론 쓸 필요 없음
- 그런데 연산자가 있을 때 오류가 나올 수 있음
  ```scala
  x
  + y //// 이 경우 x + y가 아니라 x와 +y로 파싱됨
  ```
- 이럴 땐 괄호로 감싸도 되지만 일반적으론 연산자를 줄 맨 끝에 붙이는걸 추천함
  ```scala
  x +
  y
  ```
  
### Singleton object
- 자바의 static을 대체
- 같은 이름의 class와 object가 동시에 존재할 때, 이들을 각각 companion class, companion object라고 부르며 언제나 같은 파일에 위치해야 함
- companion class, companion object은 서로의 private field에 접근 가능
- singleton object는 패러미터를 받을 수 없음
- companion class가 없는 singleton object는 standalone object라고 부름
