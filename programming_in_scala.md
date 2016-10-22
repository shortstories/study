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

## 5장
### 객체 동일성
- 자바랑 다르게 `==`로 객체의 비교도 가능
- primitive type은 자바와 동일하게 동작
- reference type에서 자바랑 다르게 동작
  - 우선 null 체크
  - 그 다음 `equals` 호출
- 만약에 인스턴스 자체를 비교하고자 한다면 `eq`, `nq` 사용
  - 자바 객체에 직접 매핑했을 때만 동작

## 6장
### Immutable Object
#### 장점
- 코드 흐름의 추론이 쉬움
- 객체를 전달하는데 자유로움. 변경이 가능하다면 기존 값을 미리 복사해놓는 등의 방어조치가 필요
- 병렬 프로그래밍 환경에서 동시성 보장
- HashSet 등에서 상태 변경으로 키값도 같이 변하는 불상사를 막을 수 있음

#### 단점
- 간단한 값 하나를 바꾸기 위해서 거대한 객체 그래프를 통째로 복사해야되는 케이스가 생길 수 있음
  - 성능상 병목 가능
  - 때문에 변경가능한 객체를 같이 제공해주는게 좋음

### 클래스 패러미터 조건 검사
- `require` 사용
``` scala
class Rational(n: Int, d: Int) {
    require(d != 0)
}
```

### 필드 추가
- 클래스 패러미터에 `val`, `var` 둘 다 붙어있지 않는 경우에는 해당 스코프에서만 참조가능한 private 변수가 됨

### 보조 생성자
- `def this(...) = {...}`
- 모든 보조 생성자는 다른 `this(...)`를 호출하며 시작
- 따라서 결국 기본 생성자를 호출하게 되어있음

### 식별자
- 기본적으로 Camel case
- 띄워쓰기를 _로 구분할 수 있긴 한데 되도록 말길
- $로 시작할 수도 있긴 한데 스칼라 내부에서만 사용하니까 쓰지말길
- 상수는 첫글자만 대문자로 (자바의 `final String SOME_VAR` 이렇게 쓰는게 아니라 `val SomeVar`)

#### 연산자 식별자
- +, -, :, ?, ~, # 등 출력 가능한 아스키 문자
- 내부적으로 $를 이용하여 해체하고 적합한 자바 식별자로 재생성함
  - ++는 $plus$plus로 바꿈

#### 혼합 식별자
- 영어, 숫자 뒤에 밑줄이 오고 연산자 식별자가 옴
  - ex) `unary_+`, `myvar_=`

### 리터럴 식별자
- ```...` ``  같은 형식
- 예약어를 포함해서 모든 문자열을 쓸 수 있음

## 7장
- 스칼라에서 할당의 결과는 할당의 값이 아니라 `Unit`
- 되도록이면 `while`문은 지양할 것

### for
#### 컬렉션 순회
- 제네레이터 문법: `for (element <- collection)`
  - Range 타입도 사용 가능 ex) `i <- 1 to 5`, `i <- 1 until 6`
  - 물론 스칼라에선 인덱스를 바탕으로 순회하는 일은 잘 없음

#### 필터링
- collection에 if문을 넣어서 필터링 가능 ex) `for (element <- collection if ...)`

#### 중첩 순회
- 여러 개의 제네레이터를 세미콜론이나 중괄호로 추가하면 중첩 순회도 가능
- ex) 
``` scala
for (
    element <- collection
    if ...;
    subElement <- element.getSubCollection()
    if ...
) { 
    sumFunc(subElement)
}
```

#### 루프문 도중에 변수 바인딩하기
- 별도의 val이나 var 없이 선언하면 됨
- ex)
``` scala
for (
    element <- collection
    if ...;
    subElement <- element.getSubCollection(
    affectedSubElement = affect(subElement)
    if ...
) { 
    sumFunc(affectedSubElement)
}
```

#### 새로운 collection 반환하기
- for 루프문 코드 블록의 중괄호 앞에 `yield` 선언
- ex) `for (...) yield {...}`, `for () yield ...`

### try-catch-finally
- 자바랑 다르게 checked excetpion도 별도의 throws를 붙이거나 반드시 catch할 필요가 없음
- 결과로 값을 반환. try가 성공하면 당연히 try의 반환값이고, 만약에 도중에 예외가 발생하였다면 case 수행 결과가 반환값
- finally에서 return을 명시적으로 호출하지 않는 이상 반환값은 버려짐. finally에서 try나 catch의 결과를 바꾸지 않도록 주의해야 함
#### 패턴 매치로 예외 처리
- `try {...} catch {case ex: Exception => {...} ...}`

### match-case
- 자바의 switch-case문과 유사
- 자바의 switch-case는 정수나 Enum만 되는 반면 모든 상수 사용 가능
- break문이 암시적으로 존재함
- 다른 표현식들과 마찬가지로 값을 반환