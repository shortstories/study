# Java8 Lambda expression

## Lambda Translation

이펙티브 자바에서 '익명 클래스를 사용하면 메소드가 호출되어 실행될 때마다 익명 클래스의 새로운 인스턴스가 생성된다. 따라서 만일 이 메소드가 반복해서 실행된다면, private static final 필드에 함수 객체 참조를 저장하고 재사용하는 것을 고려해야 한다.' 라는 항목을 보게 되어 갑자기 불안해졌다. 내 프로젝트에는 `SomeCollection.forEach(Lambda expression)` 이 잔뜩 있고, Lambda는 익명 클래스의 sugar syntax로 실제 실행 시엔 자동으로 적절한 Functional Interface의 구현체로 치환된다고 알고 있었는데, 그럼 Lambda도 같은 문제점이 존재하는게 아닌가 싶었기 때문이다.

뭐, 결론부터 내리자면 걱정할 필요는 없었다. `forEach`는 하나의 callback 객체를 가지고 반복문으로 동작하기 때문이다. 처음부터 불필요한 걱정이긴 했지만 덕분에 Lambda에 대해서 좀 더 찾아볼 수 있는 기회였다.

### 내부 순서

1. 람다식이 컴파일된 결과인 `invokedynamic` instruction 호출됨
2. `LambdaMethodFactory.metafactory()`로 capture argument 및 필요 정보들을 전달하여 `CallSite` 인스턴스 생성
3. `CallSite`가 가지고 있는 `MethodHandle`의 `invoke()`를 호출하여 Functional Interface 생성

### 변환 방법

* Lambda expression의 body를 클래스의 새로운 static method로 변환
  * capture 된 변수들은 모두 이 메소드의 argument
* Functional Interface 중 하나를 구현하여 내부 클래스를 만들고 앞에서 변환한 static method를 연결
  * 만약에 stateful lambda라면 새로운 인스턴스를 생성
  * stateless lambda면 최초 호출에만 인스턴스를 생성하고 캐싱

### Desugaring

#### Stateless

Lambda의 argument와 동일하게 argument를 갖는 static method 생성

#### Stateful

Lambda의 argument에 capture 될 변수들까지 추가하여 argument로 갖는 static method 생성. Functional Interface 인스턴스를 생성할 때 capture 된 변수들을 각각 argument로 넘겨주게 함

#### Stateful - instance member

위와 기본적으론 동일하나 Functional Interface 인스턴스를 생성할 때 capture 변수 그 자체가 아니라 클래스 인스턴스를 통째로 argument로 넘기게 함

### 왜 final 변수만?

> We only allow capture of \(effectively\) final variables. So we can freely copy variables at point of capture
>
> 『From Lambdas to Bytecode』. Brian Goetz, Java Language Architect

## 참고

[https://slipp.net/wiki/pages/viewpage.action?pageId=19530334](https://slipp.net/wiki/pages/viewpage.action?pageId=19530334)

[http://wiki.jvmlangsummit.com/images/1/1e/2011\\_Goetz\\_Lambda.pdf](http://wiki.jvmlangsummit.com/images/1/1e/2011_Goetz_Lambda.pdf)

