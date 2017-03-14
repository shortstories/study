# Transaction Management

- JTA, JDBC, Hibernate, JPA, JDO 등의 transaction API를 지원
- declarative transaction management 제공
- programmatic transaction management 제공
- Spring data 관련 모듈들과 잘 호환됨

## 기존 Java EE transaction management와 비교

요약하면, 기존 자바 EE 개발자들은 Global, Local transaction 모델 중 하나를 골라서 써야했는데 그 둘다 각각 나름의 한계를 가지고 있었고, 스프링의 transaction management는 이러한 한계에 대한 보완책을 제공한다는 것이다.

### Java EE Global transactions

Global transaction은 데이터 베이스, 메세지 큐 등 여러 transactional resources들과 동시에 작업할 수 있게 도와준다. 어플리케이션 서버는 JTA를 통해서 global transaction을 관리하게 되는데 이 JTA라는 물건이 Exception같은 이유로 좀 성가시게 군다. 게다가 JTA의 `UserTransaction`이 보통 JNDI를 통해서 얻어지기 때문에 JTA를 쓰기 위해서는 JNDI까지 써야 한다. 그 결과, global transaction 코드를 재활용하는데 큰 애로사항이 따르게 된다.

그 외에 global transaction을 위해 EJB CMT(Container Managed Transaction)을 사용할 수 있다. CMT는 declarative transaction management의 한 형태이고, JNDI를 사용할 필요가 없게 된다. 그러나 transaction을 사용하기 위해 자바 코드를 작성해야할 필요가 있다. 뿐만 아니라 CMT는 JTA와 서버 환경에 강하게 영향을 받으며 비지니스 로직을 EJB로 구현할 때만 사용할 수 있는 단점이 있다.

### Java EE Local transactions

통상 쓰는 transaction이 바로 local transaction이다. 특정 리소스마다 각각 정의되어있다. 예를 들면 JDBC Connection transaction 등이 있다. 배우기 쉽고 쓰기 쉬운 장점이 있는데 반대로 여러 리소스를 동시에 써야되는 케이스는 적용할 수 없다는 단점이 있다. 현재 내 경우에도 RabbitMQ와 MySQL과 로컬 스토리지 세개를 동시에 써야되므로 적합하지 않다고 볼 수 있다. 그 외에 내 코드가 특정 리소스에 종속되는 문제점도 발생한다.

### Spring Framework's consistent programming model

스프링은 위에서 말한 두 transaction management의 단점들을 보완하고 어떤 환경에서든 똑같은 코드로 동작시킬 수 있게 해준다. 스프링에서는 declarative와 programmatic transaction management 두가지 방법을 모두 제공한다. 일반적인 경우에는 declarative transaction management가 더 추천되는 방법이긴 하다.

programmatic transaction management 방법을 쓸 땐 스프링의 transaction abstraction을 써서 어떤 transaction 환경에서든 동작할 수 있게 한 단계 더 추상화를 한다.

declarative transaction management 방법을 쓸 땐 아주 조금의 코드, 내지는 전혀 코드를 작성하지 않아도 되기 때문에 transaction API에 전혀 구애받지 않는다. 심지어는 스프링의 transaction API까지도 말이다.

## Spring framework transaction abstraction

``` java
public interface PlatformTransactionManager {

    TransactionStatus getTransaction(
            TransactionDefinition definition) throws TransactionException;

    void commit(TransactionStatus status) throws TransactionException;

    void rollback(TransactionStatus status) throws TransactionException;
}
```

Spring transaction abstraction은 transaction strategy를 표현하기 위해서 사용한다. 위의 `PlatformTransactionManager` 인터페이스를 implement해서 사용할 수 있다.

이 인터페이스는 주로 service provider interface (SPI, 서드파티에서 구현하여 제공할 수 있게 만든 인터페이스) 이며, 필요하다면 직접 코드를 짜서 사용하는 것도 가능하다. 

이 인터페이스를 구현한 클래스는 일반적인 스프링 bean처럼 생성해서 사용하게 된다. 덕분에 만약에 JTA를 사용하더라도 스프링의 abstraction을 통해서 쓰는 것이 더 나은데, 그 이유는 직접 JTA를 쓰는 것보다 테스트같은 부분이 훨씬 쉽기 때문이다.

만약 위 인터페이스 메소드 중에 문제가 발생하면 `TransactionException`이 던져진다. 이 exception은 unchecked exception이다. 왜냐면 대부분 문제가 발생하는 transaction은 항상 실패할 가능성이 높기 때문이다. 그런데 만약에 transaction 실패를 복구할 수 있는 가능성이 있다면 직접 exception을 catch해서 처리하는 것도 가능하다.

`getTransaction()` 메소드에 `TransactionDefinition` 객체를 패러미터로 담아 호출하면 `TransactionStatus` 객체를 얻을 수 있다. `TransactionStatus` 객체는 하나의 새로운 transaction 이거나, 현재 콜 스택에 일치하는 transaction이 있다면 그 transaction이라고 할 수 있다.

`TransactionDefinition` 인터페이스는 말 그대로 transaction을 정의하기위해 사용하는 인터페이스이다. isolation, propagation, timeout, read-only 등의 설정값들을 가질 수 있다.

`TransactionStatus` 인터페이스는 transaction을 실행하고 컨트롤하는 방법, 현재 상태를 쿼리하기 위한 방법들을 제공한다. 

``` java
public interface TransactionStatus extends SavepointManager {

    boolean isNewTransaction();

    boolean hasSavepoint();

    void setRollbackOnly();

    boolean isRollbackOnly();

    void flush();

    boolean isCompleted();

}
```

스프링의 declarative, programmatic transaction management에 관계없이 제일 중요한건 적절한 `PlatformTransactionManager` 구현체를 정의하는 것이다.

### JDBC Example

``` java
// JdbcProperties.java
@ConfigurationProperties
@Data
public class JdbcProperties {
    private String driverClassName;
    private String url;
    private String userName;
    private String password;
}

// JdbcConfig.java
@Configuration
public class JdbcConfig {
    @Autowired
    private JdbcProperties properties;
    
    @Bean(destroyMethod = "close")
    public DataSource dataSource() {
        BasicDataSource dataSource = new BasicDataSource();
        
        dataSource.setDriverClassName(properties.getDriverClassName());
        dataSource.setUrl(properties.getUrl());
        dataSource.setUserName(properties.getUserName());
        dataSource.setPassword(properties.getPassword());
        
        return dataSource;
    }
    
    @Bean
    public PlatformTransactionManager txManager() {
        return new DataSourceTransactionManager(dataSource());
    }
}
```

## Transaction과 resource의 동기화

### High-level

`JdbcTemplate`처럼 스프링에서 제공하는 API나 다른 transaction 친화적인 ORM API, 내지는 native resource factory에 대한 프록시를 통해서 동기화하는 방법이다. 이러한 솔루션들은 내부적으로 알아서 생성, 재사용 등 transaction을 관리하고, exception을 매핑한다. 덕분에 사용자들은 코드를 짤 때 자신의 문제를 해결하는 데만 집중할 수 있게 된다.

### Low-level

JDBC라면 `DataSourceUtils`, JPA라면 `EntityManagerFactoryUtils` 등의 유틸리티 클래스를 사용하여 직접 low-level의 작업들을 처리하는 방법이다. 최소한 exception정도는 스프링에서 매핑해준다.

물론, 되도록 스프링에서 제공하는 기능들을 활용하는게 좋다. 바퀴를 재발명할 필요는 없지 않은가.

## 