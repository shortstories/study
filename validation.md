# Spring Validation

손쉽게 annotation을 붙여서 Validation을 수행할 수 있게 도와준다. 또한 validation 로직을 여러 레이어에서 중복되지 않고 하나의 도메인 모델에서 한꺼번에 처리할 수 있도록 도와준다.

## 사전 지식

- JSR-303/JSR-349: http://beanvalidation.org/
- hibernate validator: https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/

## 사용법

1. hibernate validator 의존성 추가
``` xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>5.4.0.Final</version>
</dependency>
```
1. Bean 추가
``` java
@Configuration
public class AppConfig {
    @Bean
    public LocalValidatorFactoryBean validator() {
      return new LocalValidatorFactoryBean();
    }
}
```
1. 도메인 모델에 `@Constraints` 관련 어노테이션 추가
``` java
@Data
public class Message {
    // 이 NotNull처럼 `javax.validation.Constraint` 어노테이션이 붙어있는 어노테이션들
    // `javax.validation.constraints` 패키지를 보면 됨.
    @NotNull
    private String sender;
    
    private String message
}
```
1. Validator `@Autowired` 끌어와서 체크 후 처리. `javax.validation.Validator` 와 스프링 버전이 있는데 특별한 기능이 필요 없으면 그냥 javax 아래에 있는 것을 사용하면 됨
``` java
@RestController
@RequestMapping("/messages")
public class MessageController {
    private final Validator validator;
    
    @Autowired
    public MessageController(Validator validator) {
        this.validator = validator;
    }
    
    @PostMapping
    public Map<String, Object> send(Message message) {
        // 만약에 위의 @Constraints에 위배되는 값이 있다면 이 Set에 들어가게 된다
        Set<ConstraintViolation<Message>> violationSet = validator.validate(message);
        
        if (violationSet.isEmpty() == false) {
            // 만약에 Message.sender가 null이라면 이 Exception의 메세지는 
            // [class: Message, property: sender, message: may not be null]
            throw new IllegalArgumentException(
                violationSet.stream()
                    .map(violation -> String.format("[class: %s, property: %s, message: %s]",
                            violation.getRootBeanClass().getSimpleName(),
                            violation.getPropertyPath().toString(),
                            violation.getMessage()))
                    .collect(Collectors.joining(", "))
            );
        }
        
        // 이하 생략
    }
}
```