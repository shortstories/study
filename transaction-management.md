# Transaction Management

- JTA, JDBC, Hibernate, JPA, JDO 등의 transaction API를 지원
- declarative transaction management 제공
- programmatic transaction management 제공
- Spring data 관련 모듈들과 잘 호환됨

## 기존 Java EE transaction management와 비교

요약하면, 기존 자바 EE 개발자들은 Global, Local transaction 모델 중 하나를 골라서 써야했는데 그 둘다 어떠한 제한점을 가지고 있었고, 스프링의 transaction management는 이러한 제한점에 대한 지원을 제공한다는 것이다.

### Java EE Global transactions

Global transaction은 데이터 베이스, 메세지 큐 등 여러 transactional resources들과 동시에 작업할 수 있게 도와준다. 어플리케이션 서버는 JTA를 통해서 global transaction을 관리하게 되는데 이 JTA라는 물건이 Exception같은 이유로 좀 성가시게 군다. 게다가 JTA의 `UserTransaction`이 보통 JNDI를 통해서 얻어지기 때문에 JTA를 쓰기 위해서는 JNDI까지 써야 한다. 그 결과, global transaction 코드를 재활용하는데 큰 애로사항이 따르게 된다.

그 외에 global transaction을 위해 EJB CMT(Container Managed Transaction)을 사용할 수 있다. CMT는 declarative transaction management의 한 형태이고, JNDI를 사용할 필요가 없게 된다. 그러나 transaction을 사용하기 위해 자바 코드를 작성해야할 필요가 있다. 뿐만 아니라 CMT는 JTA와 서버 환경에 강하게 영향을 받으며 비지니스 로직을 EJB로 구현할 때만 사용할 수 있는 단점이 있다.

### Java EE Local transactions

