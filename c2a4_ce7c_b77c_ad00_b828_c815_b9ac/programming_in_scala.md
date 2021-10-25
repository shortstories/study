# Programming in Scala

## 1장

* 스킵

## 2장

* 스킵

## 3장

### List

* list의 제일 앞에 원소 추가하는건 O(1), 뒤에 추가하는건 O(n)
* 그러므로 앞에 추가하는게 좋음. 만약에 뒤에 추가하고 싶다면 우선 뒤집고 앞에 추가한 다음에 다시 뒤집거나 `ListBuffer` 사용

### Tuple

* tuple은 `tuple._i` 같은 형태로 원소 호출
* i는 1부터 시작 (하스켈, ML 등 정적 타입 언어의 영향)
* list와 tuple의 차이점은 여러 자료형을 한번에 담을 수 있느냐의 차이

### Map

* map의 각 원소는 tuple

### 함수형 스타일

* var보단 val을 더 권장
* side effect를 최소화
  * 만약에 Unit을 반환하는 함수가 있다면 주의할 것

## 4장

### Class, Field, Method

* Scala의 모든 필드는 기본적으로 public
* 메소드 패러미터는 기본적으로 val
* return을 되도록 쓰지말고, 쓰더라도 1번 이상 쓰지 않을 것. 각 메소드가 한 값을 계산하는 표현식이 되도록 구현
*   side effect을 위한 메소드라면 결과타입을 생략하고 = 도 떼어서 프로시저처럼 보이는게 좋음

    ```scala
    def procedure(params: Any) {
      {// something
    }
    ```
* \=를 빼먹으면 무조건 `Unit`을 반환하므로 주의

### Semicolon

* 한 줄을 여러 개로 나누려면 사용 ex) `val s = "hello"; println(s)`
* 기본적으론 쓸 필요 없음
*   그런데 연산자가 있을 때 오류가 나올 수 있음

    ```scala
    x
    + y //// 이 경우 x + y가 아니라 x와 +y로 파싱됨
    ```
*   이럴 땐 괄호로 감싸도 되지만 일반적으론 연산자를 줄 맨 끝에 붙이는걸 추천함

    ```scala
    x +
    y
    ```

### Singleton object

* 자바의 static을 대체
* 같은 이름의 class와 object가 동시에 존재할 때, 이들을 각각 companion class, companion object라고 부르며 언제나 같은 파일에 위치해야 함
* companion class, companion object은 서로의 private field에 접근 가능
* singleton object는 패러미터를 받을 수 없음
* companion class가 없는 singleton object는 standalone object라고 부름

## 5장

### 객체 동일성

* 자바랑 다르게 `==`로 객체의 비교도 가능
* primitive type은 자바와 동일하게 동작
* reference type에서 자바랑 다르게 동작
  * 우선 null 체크
  * 그 다음 `equals` 호출
* 만약에 인스턴스 자체를 비교하고자 한다면 `eq`, `nq` 사용
  * 자바 객체에 직접 매핑했을 때만 동작

## 6장

### Immutable Object

#### 장점

* 코드 흐름의 추론이 쉬움
* 객체를 전달하는데 자유로움. 변경이 가능하다면 기존 값을 미리 복사해놓는 등의 방어조치가 필요
* 병렬 프로그래밍 환경에서 동시성 보장
* HashSet 등에서 상태 변경으로 키값도 같이 변하는 불상사를 막을 수 있음

#### 단점

* 간단한 값 하나를 바꾸기 위해서 거대한 객체 그래프를 통째로 복사해야되는 케이스가 생길 수 있음
  * 성능상 병목 가능
  * 때문에 변경가능한 객체를 같이 제공해주는게 좋음

### 클래스 패러미터 조건 검사

*   `require` 사용

    ```scala
    class Rational(n: Int, d: Int) {
      require(d != 0)
    }
    ```

### 필드 추가

* 클래스 패러미터에 `val`, `var` 둘 다 붙어있지 않는 경우에는 해당 스코프에서만 참조가능한 private 변수가 됨

### 보조 생성자

* `def this(...) = {...}`
* 모든 보조 생성자는 다른 `this(...)`를 호출하며 시작
* 따라서 결국 기본 생성자를 호출하게 되어있음

### 식별자

* 기본적으로 Camel case
* 띄워쓰기를 \_로 구분할 수 있긴 한데 되도록 말길
* $로 시작할 수도 있긴 한데 스칼라 내부에서만 사용하니까 쓰지말길
* 상수는 첫글자만 대문자로 (자바의 `final String SOME_VAR` 이렇게 쓰는게 아니라 `val SomeVar`)

#### 연산자 식별자

* \+, -, :, ?, \~, # 등 출력 가능한 아스키 문자
* 내부적으로 $를 이용하여 해체하고 적합한 자바 식별자로 재생성함
  * \++는 $plus$plus로 바꿈

#### 혼합 식별자

* 영어, 숫자 뒤에 밑줄이 오고 연산자 식별자가 옴
  * ex) `unary_+`, `myvar_=`

### 리터럴 식별자

* `` `...`  ``  같은 형식
* 예약어를 포함해서 모든 문자열을 쓸 수 있음

## 7장

* 스칼라에서 할당의 결과는 할당의 값이 아니라 `Unit`
* 되도록이면 `while`문은 지양할 것

### for

#### 컬렉션 순회

* 제네레이터 문법: `for (element <- collection)`
  * Range 타입도 사용 가능 ex) `i <- 1 to 5`, `i <- 1 until 6`
  * 물론 스칼라에선 인덱스를 바탕으로 순회하는 일은 잘 없음

#### 필터링

* collection에 if문을 넣어서 필터링 가능 ex) `for (element <- collection if ...)`

#### 중첩 순회

* 여러 개의 제네레이터를 세미콜론이나 중괄호로 추가하면 중첩 순회도 가능
*   ex)&#x20;

    ```scala
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

* 별도의 val이나 var 없이 선언하면 됨
*   ex)

    ```scala
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

* for 루프문 코드 블록의 중괄호 앞에 `yield` 선언
* ex) `for (...) yield {...}`, `for () yield ...`

### try-catch-finally

* 자바랑 다르게 checked excetpion도 별도의 throws를 붙이거나 반드시 catch할 필요가 없음
* 결과로 값을 반환. try가 성공하면 당연히 try의 반환값이고, 만약에 도중에 예외가 발생하였다면 case 수행 결과가 반환값
*   finally에서 return을 명시적으로 호출하지 않는 이상 반환값은 버려짐. finally에서 try나 catch의 결과를 바꾸지 않도록 주의해야 함

    **패턴 매치로 예외 처리**
* `try {...} catch {case ex: Exception => {...} ...}`

### match-case

* 자바의 switch-case문과 유사
* 자바의 switch-case는 정수나 Enum만 되는 반면 모든 상수 사용 가능
* break문이 암시적으로 존재함
* 다른 표현식들과 마찬가지로 값을 반환

### break, continue

* 스칼라에서는 두 구문 모두 지원하지 않음. 대신에 재귀로 코드 짜는 걸 추천
  * 스칼라의 모든 재귀함수는 꼬리재귀
* 만약에 굳이 break를 써야겠다면 `scala.util.control.Breaks` 클래스를 사용할 수도 있긴 함

## 9장

### 접근 제한

* `private` 키워드를 메소드 정의 앞에 붙이는 방법
* local function으로 만드는 방법도 있음
  *   이 경우 클로져 이용 가능

      ```scala
      def outerFunc() : Unit = {
        val outer = "outer"
        def innerFunc() : Unit = {
          println(outer)
        }
      }
      ```

### 위치 표시자

* `_` 혹은 `_ : Type` 형태로 사용
* 패러미터를 순서대로 하나씩 채워라는 의미

### 부분 적용 함수

* `myFunc _`, `myFunc(1, _: Int, 2)` 같은 형태로 사용
* \_의 갯수만큼 패러미터를 받는 새로운 함수를 자동 생성하여 반환
* 이후 패러미터를 넣어 함수를 호출하면 원본 함수를 재호출
* 패러미터가 함수를 받는다면 \_를 생략하는 것도 가능

### 클로져

* 주어진 함수 리터럴로부터 실행 시점에 만들어낸 객체
* 함수 본문에 존재하는 모든 free variable binding을 capturing 하여 더 이상 free varibale가 남지 않게 close 한다하여 붙여진 이름
  * 함수 스코프 밖에 있는 변수를 free variable, 스코프 내에 있는 변수를 bound variable라고 부름
  * 함수 리터럴에 free variable가 없다면 closed term, 있다면 open term 이라고 부름
* 스칼라의 클로져는 변수의 값이 아니라 변수 그 자체를 포획
  * free variable의 값을 수정하면 스코프에 상관없이 모두 수정됨
* 기본적으로 method parameter은 stack 공간에 존재하지만 클로져가 포획하게 되면 자동으로 heap 공간으로 재배치

### 반복 패러미터

* 마지막 패러미터에 \*를 붙여서 사용 가능
* 실제로는 `Array[Type]` 이 됨
* 배열을 전달하고 싶으면 `_*`의 형태로 사용

### 패러미터에 이름 붙이기

* `myFunc(param1 = 1, param2 = 2)` 와 같은 형태로 사용
* 순서를 바꾸어도 문제 없음
* `_`와 혼용 가능. 이 경우 `_`를 먼저 넣어야함

### 디폴트 패러미터

* `def myFunc(param = "default") ...` 형태로 사용

### 꼬리 재귀

*   스칼라에서 재귀 호출로 함수가 끝나면 꼬리 재귀를 사용

    ```scala
    def myFunc(x: Int): Int = {
      if (x <= 0) 0
      else myFunc(x - 1)
    }
    ```
* 루프문을 사용하는 것과 비용 측면에서 별 차이가 없음
* 다만 스택 트레이스에서 추적할 때에도 한 스택밖에 안 보이므로 `-g:notailcalls` 옵션으로 꼬리 재귀 끄는 것도 가능
* 간접 재귀 호출에선 꼬리 재귀를 이용할 수 없음. 다시 말해 마지막 연산으로 자기 자신을 직접 호출할 때만 가능

## 9장

### 커링

* loan pattern: 함수를 제공하는 쪽에서 resource의 open, close 등 라이프사이클은 관리하고, 사용할 수 있게끔 빌려주는 패턴&#x20;
* 스칼라에서는 패러미터가 한 개인 함수를 호출할 때 중괄호로 감싸서 호출할 수 있음.&#x20;
  * 이를 이용하여 사용자가 마치 기본 문법인 것처럼 사용하는 것이 가능

```scala
def customFunc(op: Int => Int) : Unit = {
 //...
}

// call
customFunc { x: Int => {
    //...
}}
```

### 이름에 의한 호출 사용

* `()` 없이 바로 `=>` 사용
* ex) `=> Boolean`, `=> Int`
* 바로 Boolean을 패러미터로 쓸 때와 차이점은 계산 시점의 차이이다

```scala
def byName(isTrue: => Boolean) {
  if (isTrue) { // 여기에서 계산이 일어남
    println("true")
  } else {
    println("false")
  }
}

def byBoolean(isTrue: Boolean) {
  if (isTrue) {
    println("true")
  } else {
    println("false")
  }
}

byName(1 > 0)
byBoolean(1 > 0) // 여기에서 계산이 일어남
```

## 10장

### Field vs Method

```scala
class MyClass {
  val asField: Int = 1 + 1
  def asMethod: Int = 1 + 1
}
```

* 차이점은 클래스 초기화때 미리 값을 계산해 저장해둘 것인가 매 호출마다 계산할 것인가 뿐
  * 미리 값을 계산해두려면 저장공간을 소모하게 됨
* 단일 접근 원칙: 필드든 메소드든 어떤 방식으로 정의하더라도 클라이언트의 코드를 수정하게끔 해서는 안 됨
  * 괄호 없는 메소드를 사용할 것이라면 부수 효과도 없어야 함은 물론 변경 가능한 상태에 의존해서도 안 됨
  * 클라이언트는 자신이 사용하는 필드가 말 그대로 필드인지 괄호 없는 메소드인지 굳이 구분할 필요가 없어야함
* 스칼라에서는 필드와 메소드가 같은 네임스페이스

### Super class 생성자 호출

* `class MyClass extends SuperClass(params)`
* extends에서 바로 호출하면 됨

## 11장

### 바닥에 있는 타입

#### Nothing

* Null이랑 비슷하지만 값이 존재하지 않는 타입을 의미
* Null과 달리 Int, Boolean등 값 타입과 호환됨
* 반환 타입이 Nothing인 것은 이 메소드가 정상적으로 값을 반환하지 않을 것이란 뜻. 즉, 비정상적인 종료를 표시

## 12장 - Trait

### 사용목적

* rich interface
* stackable modification

### 특징

* 일반적인 클래스에서 할 수 있는 대부분이 가능함
* 클래스 변수를 가질 수 없음 ex) `trait MyTrait(x: Int) // 컴파일 에러`
* 믹스인 할 때 클래스에 따라 `super` 가 동적으로 바인딩됨

### 믹스인

* 이미 다른 클래스나 트레이트를 상속한 상황에서 추가로 `with` 구문과 함께 트레이트를 믹스인할 수 있음

### Thin vs Rich Interface

* Thin 인터페이스는 구현하긴 쉬운데 선택지가 적어 용도에 딱 맞는걸 고르기 힘듬
* Rich 인터페이스는 선택지가 많은데 불필요한 코드가 많아져 구현하기 귀찮음
* 자바는 보통 Thin 인터페이스로 되어있음
* 트레이트를 보통 이 Rich 인터페이스로 만들면서 코드 관리는 쉽도록 사용
  * 메소드를 트레이트 안에서 한번만 구현해도 됨
  * 아니면 트레이트 안에 abstract method로 추가하여 구현은 떠넘겨도 됨

### 변경 쌓아올리기

* 트레이트의 super은 동적으로 바인딩됨
* 그에 따라 trait에서도 super을 쓰는 것이 가능
* trait에서 super을 쓰면 메소드 앞에 `abstract override`를 붙여줘야 함
* `abstract override` 메소드가 있는 트레이트는 반드시 해당 메소드의 구현이 존재하는 클래스에 믹스인되어야 함

## 13장

### 패키지

* 만약에 한 파일 안에 여러 패키지가 있어야된다면 중괄호로 감싸고 중첩시키는 것도 가능
  * 이렇게 중첩시키게 되면 바깥 스코프로 바로 접근 가능
  * 중괄호로 감싼게 아니라 자바처럼 한 파일에 한 패키지로 하면 자기 패키지만 보임

```scala
package com {
  package navercorp {
    class Naver {
    }

    package npush {
      class Npush {
        // 중첩된 바깥 스코프에도 접근 가능
        // 덕분에 패키지 이름을 쓰거나 import 할 필요가 없음
        val naver = new Naver() 
      }

    }

    package session-io {
    }
  }
}

package com.navercorp.test {
  class Test {
    // 중괄호로 중첩되어있지 않은 상황에선 자기 패키지만 보이므로
    // 이렇게 사용할 땐 컴파일 오류가 나게 됨
    val naver = new Naver()
  }
}
```

#### Chained package clause

* 중괄호 없이 나열하여 중첩 가능
* 코드가 오른쪽으로 밀리는게 싫을 때 사용

```scala
package com
package navercorp
package npush {
  class Npush {
    // com.navercorp 패키지에 Naver 클래스가 있다면
    // 여기에서도 마찬가지로 import나 패키지 이름 없이 사용 가능
    val naver = new Naver()   
  }
}
```

#### \_root\_

* 같은 이름을 가진 패키지가 섞여있을 때, 최상단 패키지에 접근하기 위해서 사용

```scala
package test {
  class MyTest1 {
  }
}

package hello
package world {
  package test {
    class MyTest2 {
    }
  }

  package ex {
    class Ex {
      // 같은 이름을 가진 두 패키지를 구분해서 접근 가능
      val test1 = new _root_.test.MyTest1
      val test2 = new test.MyTest2
    }
  }
}
```

### Import

* 모든 멤버에 접근하기 위해 `._` 사용
* 코드의 어느 부분에서나 import 가능
* 클래스에 국한되지 않고 일반 객체의 import도 가능
* 패키지 그 자체를 import 하는 것도 가능

```scala
import com.navercorp._ // 전체 가져오기

class Point(val x: Int, val y: Int) {
}

class Test {
  import com.navercorp.npush._ // 언제 어디서나 import 가능

  def myMethod(point: Point) {
    import point._ // 일반 객체의 import도 가능함. 이렇게 하면 x와 y에 바로 접근 가능해짐
    println(x + ", " + y)
  }
}
```

#### Import Selector

* 멤버를 감추거나 이름을 바꾸기 위해 사용
* `import packagename.{member1, member2 => anotherName}` 같은 형식으로 사용
* `{member => anotherName, _}` 특정 멤버는 이름을 바꾸고 모두 불러오기
* `{member => _, _}` 특정 멤버는 제외하고 모두 불러오기
  * import 하고싶은 패키지 사이에 같은 이름의 멤버가 있을 때 모호함을 피할 수 있음

#### Implicit import

* `java.lang._`
* `scala._`
* `Predef._`

### 접근 수식자

#### Private

* inner class는 자기 부모 class의 private에 접근할 수 있음

#### Protected

* 자바와 달리 subclass 에서만 접근 가능
  * 자바에서는 같은 패키지 안에 있으면 접근 가능했음

#### 보호 스코프

* `private[x]`, `protected[x]` 같은 형태로 좀 더 상세하게 정의 가능
  * x까지는 제외하고 비공개, 보호라는 뜻
  * x는 패키지, 클래스, 또는 오브젝트
* `private[this]` 같은 형식으로 쓰면 인스턴스 내부에서만 접근 가능. private보다 좀 더 제한적

```scala
package com.navercorp
package test {
  package test1 {
    private[test] class Test1 {}
  }

  package test2 {
    class Test2 {
      val test1 = new Test1 // [test] 덕분에 접근 가능
    }
  }
}
```

### 패키지 객체

* `package object Name` 형식으로 작성
* 자유롭게 import해서 사용 가능
* package 디렉토리에 package.scala 라는 파일 이름으로 하는 것이 좋음

## 15장

### 케이스 클래스

* class 선언 앞에 `case` 가 붙어있는 서브 클래스
* 클래스 이름과 같은 팩토리 메소드 추가
  * `new MyClass()` 대신에 `MyClass()`로 생성 가능
* 모든 클래스 패러미터에 val 접두사를 붙여줌
  * 자동으로 필드로 만들어줌
* `toString`, `hashCode`, `equals`에 자연스러운 구현 추가
  * 항상 전체를 구조적으로 비교함
* `copy` 메소드 추가
  * 이름있는 패러미터를 이용해서 바꿀 필드를 정의

```scala
case class Ex(pa1: Int, pa2: Int)

val ex = Ex(1, 2)
val copyEx = ex.copy(pa2 = 3)
```

* 패턴 매치 지원

### 패턴 매치

* 자바의 switch 문과 유사한 기능
* `Selector match {case ...}` 형태로 사용
* 각 case는 =>로 패턴과 계산식 분리
* 코드에 쓰인 순서대로 패턴을 하나씩 검사 후 매치되는 첫 번째 패턴을 선택하여 화살표 뒤의 계산식 수행

#### 와일드카드 패턴

* `_` 와일드카드 패턴은 모든 값과 매치되지만 계산식 안에서 사용할 수&#x20;
* 주로 필요없는 부분을 무시하기위해 사용

#### 상수 패턴

* `"+"`, `1` 같은 상수 패턴은 == 연산자 적용하여 매치
* val이나 싱글톤 object도 매치됨 ex) `Nil`

#### 변수 패턴

* `e` 같은 변수 패턴은 모든 값과 매칭 가능, 매칭된 이후 계산식이 수행될 때 같은 이름을 가진 변수가 있다면 매치된 값 참조
* 와일드카드 패턴과 마찬가지로 자바의 `default:` 처럼 사용 가능
* 이름이 소문자로 시작하면 변수, 그 외에는 모두 상수로 간주
  * 만약에 소문자로 시작하는 상수를 쓰고싶으면 패키지부터 이름을 모두 써주거나 역따옴표 \`로 감싸면 됨

#### 생성자 패턴

* `MyCaseClass('test', e)` 같은 생성자 패턴은 먼저 타입을 체크하고 패러미터를 하나하나 체크하여 패턴 매칭
  * 패러미터도 패턴의 일부(deep match)인 덕분에 `MyCaseClass('test', MyCaseClass('nested', e))` 처럼 중첩이 가능함

#### 시퀀스 패턴

* `List`, `Array` 등 시퀀스와 매칭
* 길이에 구애받고싶지 않다면 `List(0, _*) => //...` 같은 형식으로 작성

#### 튜플 패턴

* `Tuple`과 매칭

#### 타입 지정 패턴

* `case e: String => //...`처럼 타입을 지정하면 특정 타입유형만 매칭, e가 계산식 내에서 해당 타입으로 들어옴

#### 타입 소거

* 스칼라도 런타임에선 제네릭의 타입을 소거
* 따라서 `case e: Map[Int, Int] => //...`처럼 특정 타입 제네릭만 매칭하려 해보면 unchecked warning과 함께 모든 타입에 매칭됨
  * 배열만은 원소 타입과 값을 함께 저장하므로 매칭 가능

#### 변수 바인딩

* 깊은 매치에서 매칭된 결과 일부를 변수로 바인딩하는 것
* `case myFunc("const", e @ myFunc("const1", _)) => //...`
  * 먼저 `myFunc("const1", _)`을 매칭하여 e에 담고
  * 다음으로 `myFunc("const", e)` 매칭
* 계산식에서 e를 호출하면 `myFunc("const1", _)`이 됨

#### 패턴 가드

* 변수 패턴에 조건 적용
* `case myFunc2(x, y) if x == y => //...`
* 가드가 true가 될 때에만 매칭

#### 패턴 겹침

* 더 자세한 매칭이 위에 오도록 배치

#### 봉인한 클래스

* 모든 패턴에 대응했는지 확인하기 쉽게 만들어줌
* 다른 컴파일 단위에서 함부로 클래스를 새로 정의하는걸 막아줌
* `sealed abstract class MyClass` 처럼 제일 앞에 `sealed` 추가

#### Option

* 선택적인 값을 표현. 보통 `Map` 등에서 반환
* 값이 있으면 `Some(x)` 없으면 `None`
* 보통 패턴 매치를 써서 처리

#### 변수 정의에서 패턴 사용

```scala
val myTuple = (10, "test")
val (number, string) = myTuple
```

#### case 시퀀스

```scala
{ 
  case x => //... 
  case y => //...
}
```

* 함수 리터럴이 쓰일 수 있는 모든 곳에 사용 가능
* case 시퀀스는 PartialFunction
  * 매칭되지 않는 패러미터를 넘기면 `MatchError` 발생
*   타입이 PartialFuntion이면 컴파일러에 의해서 PartialFuntion으로 변환됨

    ```scala
    new PartialFunction[/* parameter type */, /* return type */] {
      def apply(xs: /* parameter type */) = xs match {
        case x => //...
        case y => //...
      }
      def isDefinedAt(xs: /* parameter type */) = xs match {
        case x => true
        case y => true
        case _ => false
      }
    }
    ```
* 타입이 없거나 Function1이면 isDefinedAt을 빼고 일반적인 함수 리터럴로 변환됨
