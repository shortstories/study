# Spring Security

Java EE를 기반으로하는 엔터프라이즈 시스템에서 사용할 수 있는 보안 라이브러리. 꼭 스프링이 아니라도 사용은 가능하다.

보안 적용을 위해 Java EE의 서블릿 스펙이나 EJB의 스펙을 봤을 때 필요한 시나리오에 적합한 기능이 존재하지 않거나 해서 사용하는 경우가 많다. 그 외에 위의 기능들은 WAR이나 EAR 레벨에서 관리하기가 힘들다는 것도 크다. 즉, 새로운 서버에다가 어플리케이션을 배포하기가 아주 힘든 것이다. 이러한 문제들에 대해서 해결책을 제시하는 것이 Spring security이다.

보통 보안이라고 하면 크게 잡아서 "authentication" 과 "authorization"의 두 영역이 존재한다. 이 둘은 Spring security가 목표로 삼는 두 영역이기도 하다. "Authentication"은 행동의 주체가 누구인지 파악하고 정립해나가는 과정이며, "Authorization"은 그 주체에게 어떤 행동들이 허락되었는지 확인하고 결정하는 과정이다. 이 행동의 주체를 spring security에서는 principal이라고 부른다. 당연하지만 "Authorization"은 "Authentication"이 완료되어서 현재 이 행동을 하려고 하는 자가 누구인지 파악된 다음에 가능하다.



## 패키지 구성





