# Spring Data

## 개념

### Repository

가장 중요한 인터페이스는 `Repository`. 자신의 모델 클래스와 그 모델 클래스의 ID를 제네릭 타입으로 받음. 이 인터페이스는 기본적으로 어떤 타입들을 사용할 것인지 지정하는 Marker Interface로 동작함. 추가로 이 인터페이스를 상속한 인터페이스들을 찾을 수 있게 도와줌. 예를 들면 `Repository` 인터페이스에다가 기본적인 CRUD 메소드들을 추가로 정의한 `CrudRepository` 인터페이스가 여기에 들어간다고 할 수 있음. 자신이 실제로 사용하는 db에 따라 `JpaRepository`, `MongoRepository` 등 좀 더 상세하게 정의된 인터페이스를 골라서 사용해도 됨.

## 사용 방법

### 일반적인 절차

1. 내 모델 클래스의 Repository 인터페이스를 정의.
2. 필요한 query methods를 인터페이스에 추가
3. Repository Proxy 객체를 만들 수 있도록 설정 추가 \(`@EnableJpaRepositories` , `@EnableMongoRepositories` 등\)
4. `@Autowired` 로 repository 인스턴스를 끌어와서 사용

### 인터페이스 정의하는 방법

`Repository` 인터페이스를 상속하거나, `@RepositoryDefinition` 어노테이션을 인터페이스 위에다가 붙여서 만듬. 물론, `Repository`의 서브 인터페이스를 상속해도 상관없음. 주의해야되는 것은 여러 DataSource들을 동시에 쓸 때인데, 이럴 땐 일반적인 `Repository` 인터페이스를 상속해선 안됨. `JpaRepository`, `MongoRepository` 처럼 어떤 DataSource인지 명확하게 나와있는 인터페이스를 상속하거나 도메인에다가 어노테이션을 `@Entity`, `@Document` 처럼 구분해서 붙여야함.

### 쿼리 작성하는 방법

두 가지 방법으로 쿼리 작성하는 것이 가능. 하나는 메소드 이름으로 표현하는 것이고, 다른 하나는 아예 직접 쿼리를 작성해서 사용하는 방법임. 이 옵션들은 실제 DataSource가 어떤 것이냐에 따라 사용 가능한 종류가 달라짐. 그리고 실제 쿼리가 어떻게 만들어지느냐에 대한 전략은 사용자가 설정할 수 있음.

#### Query lookup strategies

`@Enable...Repositories` 어노테이션의 `queryLookupStrategy`필드 값이나 xml `query-lookup-strategy` 네임스페이스로 설정 가능

1. CREATE : 언제나 메소드 이름으로 새로운 쿼리 생성. 
2. USE\_DECLARED\_QUERY : 이미 정의되어있는 쿼리를 찾고 없으면 Exception을 던짐. 미리 어노테이션이나 기타 다른 방법으로 미리 쿼리를 생성해둬야함.
3. CREATE\_IF\_NOT\_FOUND : 기본값. 이미 정의되어있는 쿼리를 먼저 찾고 없으면 메소드 이름을 바탕으로 새로 생성함.

#### 쿼리 생성

우선 메소드 이름에서 `find...By`, `read...By` 같은 prefix들을 날려버림. 그리고 남은 것들에서 파싱함. `By` 를 기준으로 두 영역으로 나눌 수 있음. `By` 의 앞 부분은 동작, 추가 설정 등을 정의하는 영역으로 `Distinct` 같은 옵션을 추가할 수 있음. `By` 의 뒷 부분은 데이터를 추출할 기준을 정의하는 영역인데 필드를 기준으로 조건문을 넣을 수 있음.

실제 쿼리가 생성되는 결과물은 어떤 DataSource냐에 따라 달라짐. 하지만 공통점은 다음과 같음.

1. `And`, `Or` 같은 옵션으로 조건문들을 조합할 수도 있고, 어떤 db를 쓰느냐에 따라 달라지긴 하지만 `Between`, `LessThan`, `GreaterThan`, `Like` 같은 옵션들을 넣어서 조건문을 만들 수 있음.
2. `IgnoreCase`, `AllIgnoreCase` 같은 옵션을 넣어서 비교할 때 대소문자를 구별하지 않게 만들 수도 있음.
3. `OrderBy` 와 `Asc`, `Desc` 같은 옵션들을 조합하여 결과물이 static하게 정렬되어 나오도록 만들 수 있음. 만약에 그 기준이 매번 달라져서 dynamic하게 정렬하고 싶다면 메소드에 `Sort` 나 `Pageable` 같은 특별한 패러미터를 주는 방법이 있음.

#### 필드 표현 방법

가령 `Parent.child.someProperty` 를 기준으로 가져오고 싶으면 `List<Parent> findByChildSomeProperty(SomeProperty property)` 처럼 만들면 됨. 사이에 특별한 옵션 명령어를 안 넣고 각 필드들을 이어서 한 덩어리로 적으면 카멜 표기법에 따라 쪼개서 사용함.

이 알고리즘이 먼저 제일 큰 덩어리부터 시작해서 일치하는 필드가 없을 경우 쪼개서 비교하는 식으로 동작하기 때문에 예상치 못한 문제가 발생할 수 있음. 위 예제에서 만약에 `Parent.childSome` 이라는 필드가 있다면 정상동작하지 않게 됨. 큰 덩어리부터 보기 때문에 `Parent` 에 `childSome` 이라는 필드가 있는지 체크하고, 그게 있으므로 `Parent.childSome.property` 를 찾으려고 하기 때문.`childSome` 에 `property` 가 없다면 Exception이 발생하게 됨. 이러한 경우엔 `_` 을 사용해서 명시적으로 나눌 수 있음. `... findByParentChild_SomeProperty ...` 이렇게 하면 원하는 대로 동작함.

