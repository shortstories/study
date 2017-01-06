# Lambda expression

## Lambda Translation

이펙티브 자바에서 '익명 클래스를 사용하면 메소드가 호출되어 실행될 때마다 익명 클래스의 새로운 인스턴스가 생성된다. 따라서 만일 이 메소드가 반복해서 실행된다면, private static final 필드에 함수 객체 참조를 저장하고 재사용하는 것을 고려해야 한다.' 라는 항목을 보게 되어 갑자기 불안해졌다. 내 프로젝트에는 `SomeCollection.forEach(Lambda expression)` 이 잔뜩 있고, Lambda는 익명 클래스의 sugar syntax로 실제 실행 시엔 자동으로 적절한 Functional Interface의 구현체로 치환된다고 알고 있었는데, 그럼 Lambda도 같은 문제점이 존재하는게 아닌가 싶었기 때문이다.

뭐, 결론부터 내리자면 걱정할 필요는 없었다. forEach는 하나의 callback 객체를 가지고 반복문으로 동작하기 때문이다. 처음부터 불필요한 걱정이긴 했지만 덕분에 Lambda에 대해서 좀 더 찾아볼 수 있는 기회였다.

### 내부 순서

1. invokedynamic instruction 호출됨
2. LambdaMethodFactory.metafactory\(\)로 capture argument 및 필요 정보들을 전달하여 CallSite 인스턴스 생성

3. CallSite가 가지고 있는 MethodHandle의 invoke\(\)를 호출하여 Functional Interface 생성

### 변환 방법

* Lambda expression의 body를 클래스의 새로운 static method로 변환
  * capture 된 변수들은 모두 이 메소드의 argument
* Functional Interface 중 하나를 구현하여 내부 클래스를 만들고 앞에서 변환한 static method를 연결
  * 만약에 stateful lambda라면 새로운 인스턴스를 생성
  * stateless lambda면 최초 호출에만 인스턴스를 생성하고 캐싱



### Desugaring

#### Stateless



#### Stateful

#### Stateful - instance member



