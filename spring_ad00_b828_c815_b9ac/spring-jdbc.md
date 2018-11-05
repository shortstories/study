# Spring JDBC

JDBC를 쓸 때 개발자가 해야되는 여러 low-level 작업들을 스프링에서 대신 처리해주는 모듈이다. 주로 connection 관리, SQL문 실행 및 결과 정리, 예외처리, 트랜잭션 처리 등을 도와준다.

## 패키지 구조

### core

`JdbcTemplate` 클래스와 수많은 callback 인터페이스들, 그리고 관련된 각종 클래스들을 포함하고 있다. 이 아래에 또 `simple`, `namedparam` 패키지 등이 존재하며 각각 `JdbcTemplate`를 좀 더 사용하기 쉽게 만든 클래스들이 들어있다.

### datasource

`DataSource` 에 쉽게 접근할 수 있게 도와주는 유틸리티 클래스들을 포함하고 있다. 또한 간단한 `DataSource` 구현체들도 여기에 들어가있어 테스트에 쓸 수 있다. 아래에 `embedded` 패키지가 있어 embedded database를 쓸 수 있게 도와준다.

### object

재사용 가능하고 thread-safe한 자바 객체로 RDBMS의 query, update, stored procedure 등을 사용할 수 있게 도와주는 클래스들을 포함하고 있다.

### support

일부 유틸리티 클래스들과 `SQLException`을 처리하는 기능을 포함하고 있다. 스프링에서는 JDBC 사용 중 발생하는 에러들을 `org.springframework.dao` 패키지 아래에 있는 exception으로 바꿔준다.

## 사용법

### 접근 방식 선택

1. JdbcTemplate : 기본적이며 가장 많이 쓰이는 방식. 또한 가장 "lowest level" 한 접근 방식이기도 하다. 나머지들은 모두 이 `JdbcTemplate`를 wrapping한 것이나 다름 없다.
2. NamedParameterJdbcTemplate : 전통적인 JDBC의 "?" placeholder들에다가 이름을 붙일 수 있게 한 것.
3. SimpleJdbcInsert, SimpleJdbcCall : 데이터베이스의 metadata들을 사용해서 필요한 설정들이 최소로 되도록 최적화한 것들이다. 테이블이나 프로시져의 이름과 column에 알맞는 데이터를 map으로 넣는 것 만으로 사용할 수 있어 간편하다. 다만 적절한 metadata가 있고 또 해당 데이터베이스를 지원하는 경우에만 그렇게 사용할 수 있다는게 흠이다. 그렇지 않다면 자기가 직접 설정을 입력해 넣어야만 한다.
4. RDBMS Objects : `MappingSqlQuery`, `SqlUpdate`, `StoredProcedure` 등이 있다. 재사용 가능하고 thread-safe한 객체들을 생성하여 사용한다. query string을 넣고, 패러미터들을 정의하고, 컴파일하면 사용준비가 완료되며, 이렇게 한번 만들고 나면 패러미터를 바꿔가며 몇번이고 재사용할 수 있게 된다.

### JdbcTemplate

`JdbcTemplate`를 사용하기 위해서는 우선 적절한 `DataSource`를 만드는 것이 우선이다. 일반적인 데이터베이스들은 자신들의 `DataSource`를 이미 지원하고 있으므로 적절히 설정하여 bean으로 등록해두는 것이 좋다. 보통은 `BasicDataSource`로 충분하다.

`JdbcTemplate`는 이미 thread-safe하기 때문에 멀티쓰레딩 환경에서도 문제 없이 동작한다. 그러므로 여러 객체를 만들어 쓸 일이 좀처럼 없다. 다만, 만약에 여러 데이터베이스들에 동시에 접근한다면 그 땐 각 `DataSource`마다 `JdbcTemplate`객체를 새로 생성해야할 것이다.

#### 사용

```java
@Repository
public class MyRepository {
  private final JdbcTemplate jdbcTemplate;
  private final MyObjectMapper myObjectMapper;

  @Autowired
  public MyRepository(DataSource myDataSource) {
    this.jdbcTemplate = new JdbcTemplate(myDataSource);
    this.myObjectMapper = new MyObjectMapper();
  }

  public MyObject read(int id) {
    return jdbcTemplate.queryForObject(
      "select * from my_table where id = ?",
      new Object[]{id},
      myObjectMapper
    );
  }

  public List<MyObject> readAll() {
    return jdbcTemplate.query(
      "select * from my_table",
      myObjectMapper
    );
  }

  private static class MyObjectMapper implements RowMapper<MyObject> {
    @Override
    public MyObject mapRow(ResultSet rs, int rowNum) throws SQLException {
      return new MyObject(rs.getString("param1"), rs.getString("param2"));
    }
  }
}
```

select는 `query()` insert, update, delete는 `update()` 그 외에 임의의 SQL문이나 DDL문들은 `execute()` 메소드를 사용하면 된다.

