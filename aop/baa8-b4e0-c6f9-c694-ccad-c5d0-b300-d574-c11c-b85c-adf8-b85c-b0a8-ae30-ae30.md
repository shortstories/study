# Spring AOP로 모든 Request 로그 남기기

처음에는 Controller 메소드 하나하나에 Logger로 로그 찍다가 언제까지 이 짓을 반복할건지 너무 귀찮아졌다. 그래서 그 다음으로 Interceptor 써가지고 로그 찍다가, 이래저래 관리하기 귀찮기도 하고 무엇보다 하나하나의 요청이 얼마나 시간을 소요하는지 찍어볼까했더니 좀 불편하더라. 뭐 더 좋은게 없을까 싶어 잠깐 고민하다가 전에 문서만 봐두고 써보진 않은 Spring AOP가 번뜩 떠올라서 써보게 되었다.

설정도 간편하고 원래 로직과 불필요한 로깅 관련 코드를 완전히 추출해낼 수 있었으며, 원하는 대로 정확히 동작하는 부분이 마음에 들었다. 물론 request 로그를 남기는건 interceptor으로도 충분하지만 웹 요청이 아니라 특정 서비스 등에 대한 로그를 남기는건 AOP로밖에 안 되기도 할 것이고..

## 사용법

전혀 어렵지 않다. 내 프로젝트는 새로 만든 프로젝트라 스프링 부트 버전 `1.5.3.RELEASE` 를 사용하고 있다.

### 1. Dependency 추가

두 가지 방법이 있다. starter을 쓰는 방법과 일일히 하나하나 추가해주는 방법. 스프링 부트를 쓰고 있다면 당연히 starter을 쓰는게 편하다.

#### groovy

```groovy
dependencies {
  // 생략
  compile('org.springframework.boot:spring-boot-starter-aop')
}

// 내지는 하나하나 추가해도 된다.
dependencies {
  // 생략
  compile("org.springframework:spring-aop:${springAopVersion}")
  compile("org.aspectj:aspectjweaver:${aspectjweaverVersion}")
}
```

### 2. Annotation 붙이기

정확히는 `@EnabledAspectJAutoProxy` 라는 어노테이션을 앱의 어느 `@Configuration` 클래스에 붙여야된다. 나는 스프링 부트를 쓸 때 보통 이런 설정 어노테이션들은 나중에 찾기 쉽게 메인에다가 몰아넣는 편이다.

```java
package com.navercorp.npush2;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@SpringBootApplication
@EnableConfigurationProperties
@EnableAspectJAutoProxy
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

### 3. @Aspect 클래스 만들기

AOP를 적용하기 위해서는 `@Aspect` 어노테이션을 붙인 클래스를 하나 만들고 Bean으로 추가해주면 된다. 나는 다음과 같은 코드를 작성하였다.

```java
package com.navercorp.npush2.aop;

import java.util.Map;
import java.util.stream.Collectors;

import javax.servlet.http.HttpServletRequest;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import com.google.common.base.Joiner;

@Component // 1
@Aspect // 2
public class RequestLoggingAspect {
  private static final Logger logger = LoggerFactory.getLogger(RequestLoggingAspect.class);

  private String paramMapToString(Map<String, String[]> paramMap) {
    return paramMap.entrySet().stream()
        .map(entry -> String.format("%s -> (%s)",
            entry.getKey(), Joiner.on(",").join(entry.getValue())))
        .collect(Collectors.joining(", "));
  }

  @Pointcut("within(com.navercorp.npush2.controller..*)") // 3
  public void onRequest() {}

  @Around("com.navercorp.npush2.aop.RequestLoggingAspect.onRequest()") // 4
  public Object doLogging(ProceedingJoinPoint pjp) throws Throwable {
    HttpServletRequest request = // 5
        ((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()).getRequest();

    Map<String, String[]> paramMap = request.getParameterMap();
    String params = "";
    if (paramMap.isEmpty() == false) {
      params = " [" + paramMapToString(paramMap) + "]";
    }

    long start = System.currentTimeMillis();
    try {
      return pjp.proceed(pjp.getArgs()); // 6
    } finally {
      long end = System.currentTimeMillis();
      logger.debug("Request: {} {}{} < {} ({}ms)", request.getMethod(), request.getRequestURI(),
          params, request.getRemoteHost(), end - start);
    }
  }
}
```

1. Bean으로 등록하기위해 `@Configuration` 클래스에서 직접 등록할 수도 있겠지만 나는 그냥 `@Component`로 등록하는 것을 선호한다.
2. 이 어노테이션을 붙여야 동작한다.
3. Pointcut을 등록하는 부분. 사용할 수 있는 Pointcut 타입들은 [https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html\#aop-pointcuts-designators](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html#aop-pointcuts-designators) 여기를 참고하시라. 현재 이 코드에서는 `com.navercorp.npush2.controller` 패키지 아래에 있는 모든 public 메소드들을 포함하도록 만들었다. \(Spring AOP도 프록시 기반으로 동작하다보니까 public이 아닌 다른 메소드들은 인식하지 않는다.\) 또 여기서 중요한 것은 `@Pointcut` 은 어디까지나 적용 범위를 잡는 개념이므로 method body에 뭘 넣든 의미가 없다는 것이다. 빈 채로 두도록 하자.
4. Advice를 등록하는 부분. 사용할 수 있는 Advice 타입들은 역시 [https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html\#aop-advice](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html#aop-advice) 문서를 참고하자. advice에다가 Pointcut을 등록하는 방법엔 이미 정의된 Pointcut 메소드를 가르키게 하는 방법과 inline으로 바로 pointcut을 작성해서 하는 방법 두가지가 있는데 뭐 입맛대로 골라쓰면 될 듯 하다. 나는 따로 정의해서 등록했다.
5. 혹시 어떻게 `HttpServletRequest` 를 받아와야할지 모르겠다면 이 부분을 그대로 사용하면 된다.
6. Around advice는 메소드의 실행 전, 후 접근 및 리턴 값까지 바꿀 수 있는 가장 강력한 advice인데, 이 코드는 실질적으로 Controller의 메소드가 실행되는 부분이다.



