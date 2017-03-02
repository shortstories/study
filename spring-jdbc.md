# Spring JDBC

JDBC를 쓸 때 개발자가 해야되는 여러 low-level 작업들을 스프링에서 대신 처리해주는 모듈이다. 주로 connection 관리, SQL문 실행 및 결과 정리, 예외처리, 트랜잭션 처리 등을 도와준다.

## 사용법

### 접근 방식 선택

1. JdbcTemplate : 기본적이며 가장 많이 쓰이는 방식. 또한 가장 "lowest level" 한 접근 방식이기도 하다. 나머지들은 모두 이 `JdbcTemplate`를 wrapping한 것이나 다름 없다.
1. NamedParameterJdbcTemplate : 전통적인 JDBC의 "?" placeholder들에다가 이름을 붙일 수 있게 한 것.
1. SimpleJdbcInsert, SimpleJdbcCall : 데이터베이스의 metadata들을 사용해서 필요한 설정들이 최소로 되도록 최적화한 것들이다. 테이블이나 프로시져의 이름과 column에 알맞는 데이터를 map으로 넣는 것 만으로 사용할 수 있어 간편하다. 다만 적절한 metadata가 있고 또 해당 데이터베이스를 지원하는 경우에만 그렇게 사용할 수 있다는게 흠이다. 그렇지 않다면 자기가 직접 설정을 입력해 넣어야만 한다.
1. RDBMS Objects : `MappingSqlQuery`, `SqlUpdate`, `StoredProcedure` 등이 있다. 재사용 가능하고 thread-safe한 객체들을 생성하여 사용한다. query string을 넣고, 패러미터들을 정의하고, 컴파일하면 사용준비가 완료되며, 이렇게 한번 만들고 나면 패러미터를 바꿔가며 몇번이고 재사용할 수 있게 된다.

