# AOP (Aspect-Oriented Programming)
- OOP가 Class를 기반으로 모듈화 한다면 AOP는 Aspect를 기반으로 모듈화 함

## 용어
- **Aspect** : transaction 관리처럼 여러 type, object들이 공통적으로 갖는 관심. Spring AOP에서는 특정 interface를 구현하거나 `@Aspect` annotation을 붙여서 사용 가능
- **Join point** : 특정 Method를 실행하거나 Exception을 처리할 때 처럼 프로그램이 실행되고 있는 어느 한 시점. Spring AOP 에서는 언제나 Method 실행 시점을 의미함
- **Advice** : 특정 join point나 aspect에 행할 행동을 의미. `around`, `before`, `after` 등의 종류가 존재. Spring AOP를 포함해서 많은 AOP 프레임워크들은 advice를 `Interceptor` 로 간주하고 join point around의 interceptor 체인을 관리함
- **Pointcut** : join points의 한 종류. advice는 pointcut에 일치하는 join point에 실행됨 (예를 들면 어떤 특정 이름의 메소드 실행). Spring AOP는 AspectJ 의 pointcut 표현 방식을 기본으로 함.
- **Introduction** : 이익에 따라 추가적인 method나 field를 정의하는 것. Spring AOP는 advised object에 새로운 interface와 그 구현체를 introduce하도록 허가함. 
- **Target object** : 하나 또는 그 이상의 aspect에 의해 advised 되는 객체. advised object라고도 함. Spring AOP에서는 언제나 runtime proxies를 사용하므로 proxied object라고도 할 수 있음.
- **AOP proxy** : aspect contracts를 구현하기 위해 AOP 프레임워크에 의해서 만들어진 object. Spring Framework에서는 JDK dynamic proxy 또는 CGLIB proxy를 사용.
- **Weaving** : aspects와 어플리케이션의 object 또는 type들을 연결하여 advised object들을 만들어내는 과정. compile time, load time, runtime 모두에 실행될 수 있음. 다른 순수 자바 AOP 프레임워크들과 마찬가지로 Spring AOP도 runtime에 weaving을 수행.


## Advice 종류
- **Before advice** : join point가 실행되기 이전 시점에 실행되는 advice. Exception을 던지지 않는 이상에야 join point의 실행을 막을 수는 없음
- **After returning advice** :  join point가 완료되고 리턴한 다음에 실행되는 advice
- **After throwing advice** : join point가 Exception을 던지며 종료된 다음에 실행되는 advice
- **After (finally) advice** : join point가 정상적으로 종료된 여부에 관계 없이 항상 실행되는 advice
- **Around advice** : method 호출처럼 join point를 둘러싸는 advice. 가장 강력한 종류의 advice로 method의 호출의 이전과 이후에 특정한 행동을 수행하도록 하는 것도 가능. join point를 실행할 것인가 아니면 자체적인 값을 리턴하거나 Exception을 던져서 생략할 것인가 결정할 수도 있음

