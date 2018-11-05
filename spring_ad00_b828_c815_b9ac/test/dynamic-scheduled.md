# Spring Test Dynamic @Scheduled

캐시를 사용하는 클래스에서 테스트를 작성하던 중이었다. 해당 클래스에는 캐시 invalidation을 하는 메소드가 있었고, 이 메소드가 주기적으로 실행되어야 하기 때문에 거기에 `@Scheduled`를 붙여 스프링 스케쥴링을 사용하고 있었다. 그런데 이 `@Scheduled` 의 delay로 3분을 줬었는데, 이렇게 되니까 테스트 하나하나에 걸리는 시간이 너무 길어져서 답답하기 그지 없었다. 그래서 런타임에서 `@Scheduled`의 delay를 동적으로 수정하려고 구글링도 해보고, 여러가지 시도도 해봤다.

처음엔 annotation의 값을 바꿔주면 되지 않을까 싶어 열심히 자바 리플렉션 써가며 바꾸어봤는데, 생각해보니까 어차피 스케쥴링은 처음에 application context 초기화 과정에서 결정되기 때문에 annotation 값이랑은 상관없더라. 그래서 이 방법은 버리고.

두번 째로 `SchedulingConfigurer`를 사용해서 수정하는 방법을 시도해봤다. stack overflow를 뒤져보니까 `ScheduledTaskRegistrar.addTriggerTask()` 메소드를 사용해서 하라는데, 이렇게 하면 물론 런타임에 동적으로 바꿀 수는 있겠지만 테스트 하나 짜겠다고 원본 코드도 수정해야되는데다가 심지어 그 코드가 복잡해지기까지 한다. 도저히 마음에 안 드는 방법이었다. 다른 방법이 있나 생각을 좀 하다가, 혹시 이렇게 하면 되지 않을까 싶어 해보니 생각대로 동작하더라.

## 사용법

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class NpushManagerDAOTest {
  @Test
  public void myTest() {
    // run test logic
  }

  @TestConfiguration
  static class SchedulingTestConfig implements SchedulingConfigurer {
    @Autowired
    private MyService service;

    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
      List<IntervalTask> taskList = taskRegistrar.getFixedDelayTaskList();
      List<IntervalTask> newTaskList = new ArrayList<>();

      for (IntervalTask task : taskList) {
        ScheduledMethodRunnable runnable = (ScheduledMethodRunnable) task.getRunnable();
        if (runnable.getTarget() instanceof MyService) {
           newTaskList.add(new IntervalTask(runnable, CUSTOM_DELAY, CUSTOM_DELAY));
        }
      }

      taskRegistrar.setFixedDelayTasksList(newTaskList);
    }
  }
}
```

보다시피 현재 Task List를 읽어와서 내가 필요한 것들만 골라 `CUSTOM_DELAY`로 새로 객체를 생성하여 리스트로 만들고, 그걸 다시 끼워넣으면 끝이다.

나는 여기서 클래스만 필터링했지만 필요하다면 Method로도 구분하는게 더 안전할지도 모르겠다.

