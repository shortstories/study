# Bean init, destroy 순서

## 순서 확인하는 법
- `org.springframework.beans.factory.support.DefaultListableBeanFactory` log level 'DEBUG' 등록

## Bean init 순서
- 의존성을 기준으로 정렬

## Bean destroy 순서
- Bean init 순서의 역순

## 임의로 순서를 조절하는 방법
http://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html#beans-factory-dependson
> The depends-on attribute in the bean definition can specify both an initialization time dependency and, in the case of singleton beans only, a corresponding > destroy time dependency. Dependent beans that define a depends-on relationship with a given bean are destroyed first, prior to the given bean itself being ?> > destroyed. Thus depends-on can also control shutdown order.

`@DependsOn` annotation을 사용하면 직접적인 의존관계가 아닌 두 Class의 순서를 조절할 수 있음.