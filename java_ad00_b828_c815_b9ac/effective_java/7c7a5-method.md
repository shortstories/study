# 7장 - Method

## validation을 꼭 하기

1. public 메소드라면 검사하여 적절한 `Exception`을 던지도록 구현한다. 그리고 문서에다가 관련된 사항들을 기입해야한다.
2. public이 아닌 메소드라면 `assert` 를 사용해서 검사하는 것이 권장된다.
3. 검사 자체의 비용이 아주 비싸거나 메소드 실행 가운데 \(메소드 실행 앞 부분이나 다 끝난 다음이 아니라\) 묵시적으로 검사를 실행하고 있다면 생략할 수 있다.

## 필요하다면 방어복사본 만들기

의도는 불변객체를 만드는 것이지만 그 객체의 필드 중에 가변 객체가 있다면 의도치않게 수정이 가능해지는 경우가 있다. 이러한 경우를 막기 위해서 필요하다면 불변객체를 생성할 때 그 필드에다가 원본 객체 대신에 그 객체의 값을 복사한 새로운 객체를 만들어서 할당해야한다.

### 주의사항

1. validation을 하기 전에 미리 복사본을 만들고, 그 복사본을 바탕으로 validation을 진행해야한다. 왜냐하면 값을 검사하고 나서 복제를 하게 되면 검사가 완료되고 복제를 하려고하는 그 사이 시간에 값을 변경하는 것이 가능하기 때문이다. 
2. `clone` 메소드가 악의적으로 설계된 서브 클래스의 인스턴스를 반환할 수 있기 때문에 `clone` 메소드를 믿으면 안 된다.
3. getter에서도 필드 인스턴스의 복사본을 만들어서 반환하도록 해야 한다. 여기서 차이점은 `clone` 를 써도 된다는 점이다. 왜냐하면 이미 생성 시점에서 검사 완료하고 안전한 인스턴스를 만들어서 넣어두었기 때문이다.
4. 방어복사본을 만드는 것이 성능적으로 불리하므로 내부적으로 안전한 환경이라면 문서화를 적절히 하고 복사본을 쓰는 것을 생략할 수도 있다.

## API 설계 힌트

1. 메소드 이름을 신중하게 짓기. 1차 목표는 일관성 2차 목표는 누가 봐도 이해가 가도록.
2. 너무 많은 메소드를 만들지 않기. 메소드의 갯수는 클래스의 학습 난이도, 문서화 난이도, 테스트 난이도 등을 모두 높인다. 올지도 모르는 미래에 조금 더 편리하게 만들겠다고 필요없는 메소드까지 추가하는 일을 막아야 한다. 필요하다면 약식 메소드를 고려하자.
3. 너무 많은 패러미터 만들지 않기. 가급적 패러미터는 4개 이하를 유지하도록.
   1. 메소드 쪼개기. 가급적 직교성을 고려하여 쪼개도록 해야 너무 많은 메소드가 생기는 것을 막을 수 있다.
   2. 여러 메소드에 중복적으로 보이는 패러미터들을 모아서 하나의 helper 클래스로 만들기. 당연하지만 inner static class으로 만드는게 좋다.
   3. 빌더 패턴 적용하기. 특히 패러미터 일부가 선택적으로 필요할 때 유용
4. 패러미터의 타입은 가급적 인터페이스로 지정
5. `boolean` 타입의 패러미터를 넣는 대신에 두 개의 요소를 갖는 enum 사용하기

## 오버로딩 적절하게 쓰기

오버로딩은 오버라이딩이랑 다르게 호출될 메소드가 컴파일 시점에 결정된다. 따라서 오버로딩을 마치 `switch`문 쓰듯이 쓰게 되면은 예상한 동작과 다른 결과물을 볼 수 있다. 실제 인스턴스가 어떤 것이냐에 상관없이 동작하기 때문이다. 그러므로 이 경우에는 오버로딩은 쓸 수 없고 직접 `instanceof` 를 사용해서 일일히 구분해줘야하게 된다.

1. 같은 수의 패러미터를 가지는 2개 이상의 오버로딩 메소드를 만들지 않는다.
2. 어떤 메소드가 가변인자를 사용한다면 오버로딩하지 않는다.

=&gt; 위와 같은 경우에는 그냥 메소드 이름을 다르게 하는게 더 알아보기 쉽다.

생성자의 경우에는 메소드 이름을 다르게 할 수 없으므로 같은 패러미터 갯수를 가지는 생성자가 생길 수 밖에 없는데, 이 경우 서로 캐스팅이 안 되는 근본적으로 다른\(관계가 없는\) 타입들을 기준으로 해야된다.

그러나 하위호환같은 이유로 별 수 없이 서로 상속관계에 있는 두 클래스를 오버로딩 메소드의 패러미터로 각각 가질 수 있는데, 이 경우 같은 패러미터라면 항상 같은 동작을 하도록 보장해줘야한다. 좀 더 상세한 타입이 좀 더 추상적인 타입을 패러미터로 갖는 메소드를 재호출하도록 구현하면 언제 어떤 패러미터가 들어오더라도 동일한 동작을 보장할 수 있다.

## 가변인자 적절하게 쓰기

1. 가변인자를 쓰되 '최소 x개 이상' 이런 조건이 있다면, 메소드 내부에서 직접 array를 가지고 validation을 하는 대신에 패러미터를 `someMethod(Object first, Object... remain)` 이렇게 하는 것이 낫다. 컴파일 시점에서 판단이 가능하고 validation 로직을 별도로 추가할 필요가 적어진다.
2. 모든 final array type parameter들을 가변인자로 바꾸면 안된다. 이것은 type validation을 망쳐버릴 수 있다. 패러미터의 갯수가 유동적인 경우에만 적용하는게 유리하다
3. 가변 인자 메소드가 호출될 때마다 배열 생성, 초기화가 이루어지므로 성능적으로 불리하다. 성능도 중요하고 패러미터의 갯수도 유동적이라면 `method1()` `method1(int var1)` `method1(int var1, int var2)` `method1(int var1, int var2, int... remains)` 처럼 여러 개를 만들자. 물론 자신의 상황에 따라 몇개나 만들지는 알아서 결정해야할 것이다. 이렇게 하면 가변 인자 메소드가 호출되는 것을 최소화할 수 있다.

## 문서화하기

외부에 제공하는 모든 클래스, 인터페이스, 생성자, 메소드, 필드의 선언부 앞에 문서화 주석을 넣어야 한다. 특히 메소드 앞에다가 문서화 주석을 붙일 때는 메소드와 클라이언트 사이의 연관관계를 간명하게 설명해야 한다. 그리고 메소드가 어떻게 일을 처리하는지 보다는 무슨 일을 처리하는지에 대해 중점적으로 설명할 필요성이 있다. 또한 메소드의 모든 사전조건과 사후조건을 기록해야 한다. 일반적으로 사전조건은 패러미터와 관련된 요소이며 `@throws` , `@param` 등 태그에서 자연스럽게 설명되기 마련이다. 또한 메소드의 사이드 이펙트에 대해서도 설명해야되며, 쓰레드 세이프한지에 대해서도 설명해야 한다.

`@param` , `@return` 태그에는 매개 변수의 이름과 설명, 반환값에 대한 설명이 바로 뒤따라 나와야된다.

`@throws` 태그 다음에는 if와 함께 그 exception이 발생하게 되는 조건을 설명해야한다. 수식을 사용하는 경우도 많다.

원칙적으로 위의 텍스트들은 마침표를 찍지 않는다.

HTML 태그를 사용할 수 있다. 만약에 여러 줄로 이루어진 코드를 넣고 싶다면 &lt;pre&gt; 태그와 함께 사용하면 된다. &lt;나 &gt; 처럼 HTML 메타 문자가 포함된 코드를 작성하고 싶다면 `@literal`을 사용하면 된다.

문서화 주석의 첫번째 문장은 전체 문서의 요약이어야 하며 클래스나 인터페이스의 다른 어떤 멤버든 생성자든 같은 요약설명을 가지지 않도록 해야한다. 그리고 이 문장은 의도치않게 끊기지 않도록 마침표를 넣지 않아야한다. 그리고 대상이 메소드나 생성자라면 이 설명이 동사구이고, 그 외 클래스, 인터페이스, 필드 등은 명사구가 되어야 한다.

제네릭을 쓰고 있다면 제네릭의 타입 매개 변수도 문서화하도록 해야한다.

enum은 타입과 public 메소드 뿐만 아니라 각각의 상수도 문서화해야한다.

어노테이션 타입의 경우 타입 자신과 모든 멤버를 문서화해야한다. 멤버는 마치 필드인 것 처럼 명사구로 문서화하면 되고 타입 자체의 요약 설명은 동사구를 사용하면 된다.

