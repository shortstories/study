# Effective Java

## 4장
### 필드 직접 노출 < Accessor, Mutator 제공
- public 클래스에서 public field를 사용하면 여러 단점 존재
  - 외부에서 직접 접근하므로 캡슐화 X
  - 내부 구조를 변경하기 어려워짐
  - 값이 변하지 않도록 할 수 없음
  - 필드에 접근할 때 부수적인 조치를 취할 수 없음

### 가변 객체 < 불변 객체
#### 불변 클래스
- 불변 클래스는 생성할 때 이외엔 인스턴스의 값을 변경할 수 없음
  - 설계와 구현 및 사용이 더 쉬움
  - 에러 발생이 더 적음
  - 보안 및 사용 측면에서 안전
- 불변 클래스는 객체를 보다 더 많이 재활용하도록 유도하면 좋음
- static 팩토리를 제공하면 유연한 캐싱 가능
- clone 또는 복사 생성자가 필요 없음
- 객체 그 자체 뿐만 아니라 내부 구조의 공유 가능
- 값마다 새로운 객체가 필요한 것이 단점
  - 최종 결과만 필요한 다단계 연산이 있다면 그 사이에는 가변 클래스를 쓰는게 효율적
  - 패키지 전용의 가변 클래스 사용 추천
  - ex) `String`, `StringBuilder`
- 인스턴스가 가변적이어야 할 타당한 이유가 없다면 클래스는 불변 클래스로 만들어야 함
  - 그게 불가능하더라도 가능한한 필드는 `final`로
  - 생성자나 static 팩토리 메소드에서 완벽하게 초기화된 객체를 반환
  - 객체 재사용을 위한 재 초기화 메소드 제공 X
    - 기대하는 만큼 성능 향상이 없음

##### 생성 규칙
1. Mutator 제공 X
2. 상속 막기 (보통은 final class)
3. 모든 필드를 final 지정
4. 모든 필드를 private 지정
5. 가변 객체 필드는 외부에서 참조할 수 없게 막기
  - Constructor, Accessor, readObject에서 객체의 방어 복사본을 만들어 사용

### 상속 < 컴포지션
#### 상속
- 상속은 캡슐화를 위배. 부모 클래스의 변화에 지나치게 영향을 받음
  - 부모 클래스의 변경이 자식 객체를 망가뜨릴 수 있음
  - 개발하고 있는 순간에도 부모 클래스의 문서에 나와있지 않는 부분 때문에 버그가 발생할 수 있음
  - 부모 클래스에서 적용하는 여러 validation들이 미처 적용되지 않아 보안 취약점 발생 가능
  - private 필드, 메소드에 접근해야만 구현 가능한 케이스도 많음
  - 꼭 상속을 해야 된다면 되도록 `override` 대신에 새로운 메소드로 확장해나가는 것이 안전함
    - 물론 이 경우에도 부모 클래스가 같은 이름, 패러미터 타입을 가진 메소드를 추가한다면 의도치않게 오버라이딩 해버리거나 최악의 경우엔 컴파일 자체가 불가능해짐
- 부모 클래스와 자식클래스가 `is-a` 관계일 때만 상속 사용

#### 컴포지션
- 확장을 원하는 클래스를 상속하는 대신에 그 인스턴스를 필드로 갖는 새로운 클래스를 만들어 쓰는 방법

##### 포워딩
- 새로운 클래스의 각 메소드를 기존 클래스의 메소드와 매핑하여 외부에 노출시키는 것
- 매핑된 메소드는 포워딩 메소드
- 포워딩 메소드로만 이루어진 클래스는 포워딩 클래스
- 래퍼를 만들 땐 바로 상속하는 것 보다 포워딩 클래스를 만들어 상속하는 것이 좋음

``` java
public class ForwardingList<T> implements List<T> {
  private final List<T> instance;

  public ForwardingList(List<T> instance) {
    this.instance = instance;
  }

  @Override
  public int size() {
    return instance.size();
  }

  @Override
  public boolean isEmpty() {
    return instance.isEmpty();
  }

  @Override
  public boolean contains(Object o) {
    return instance.contains(o);
  }
  
  //... 이후 생략
}
```

``` java
public class PrintWrapperList<T> extends ForwardingList<T> {
  public PrintWrapperList(List<T> instance) {
    super(instance);
  }

  @Override
  public boolean add(T t) {
    System.out.println("element added: " + t.toString());
    return super.add(t);
  }

  @Override
  public boolean remove(Object o) {
    System.out.println("element deleted: " + t.toString());
    return super.remove(o);
  }
  
  //... 이후 생략
}

```

##### 데코레이터 패턴
- 기존 클래스의 인스턴스를 갖고 거기에 새로운 기능을 덧붙인 래퍼 클래스를 만들어 쓰는 것
- 래퍼 클래스는 콜백 프레임워크에서는 사용 불가
  - 콜백을 할 때 래퍼 객체 자체가 아니라 일부분에 불과한 자기 자신 `this` 를 전달함 => SELF 문제
##### 위임
- 컴포지션과 포워딩을 결합하고, 래퍼 객체 참조가 자신이 포함한 객체에 전달되는 것

### 상속을 위한 설계, 문서화
