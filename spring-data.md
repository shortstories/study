# Spring Data

---

## 개념

### Repository

가장 중요한 인터페이스는 `Repository`.  자신의 모델 클래스와 그 모델 클래스의 ID를 제네릭 타입으로 받음. 이 인터페이스는 기본적으로 어떤 타입들을 사용할 것인지 지정하는 Marker Interface로 동작함. 추가로 이 인터페이스를 상속한 인터페이스들을 찾을 수 있게 도와줌. 예를 들면 `Repository` 인터페이스에다가 기본적인 CRUD 메소드들을 추가로 정의한 `CrudRepository` 인터페이스가 여기에 들어간다고 할 수 있음. 자신이 실제로 사용하는 db에 따라 `JpaRepository`, `MongoRepository` 등 좀 더 상세하게 정의된 인터페이스를 골라서 사용해도 됨.

---

## 사용 방법

### 일반적인 절차

1. 내 모델 클래스의 Repository 인터페이스를 정의.
2. 필요한 query methods를 인터페이스에 추가
3. Repository Proxy 객체를 만들 수 있도록 설정 추가 \(`@EnableJpaRepositories` , `@EnableMongoRepositories` 등\)
4. `@Autowired` 로 repository 인스턴스를 끌어와서 사용

### 인터페이스 정의하는 방법

`Repository` 인터페이스를 상속하거나, `@RepositoryDefinition` 어노테이션을 인터페이스 위에다가 붙여서 만듬. 물론, `Repository`의 서브 인터페이스를 상속해도 상관없음. 주의해야되는 것은 여러 DataSource들을 동시에 쓸 때인데, 이럴 땐 일반적인 `Repository` 인터페이스를 상속해선 안됨. `JpaRepository`, `MongoRepository` 처럼 어떤 DataSource인지 명확하게 나와있는 인터페이스를 상속하거나 도메인에다가 어노테이션을 `@Entity`, `@Document` 처럼 구분해서 붙여야함.

---



