# ScalaTest와 Spring 연동하기
## 예제
``` scala
@RunWith(classOf[JUnitRunner]) //// 1
@SpringBootTest(classes = Array(classOf[/* Your Application Class */])) //// 2
class ActorTest extends UnitSpec {
  val logger = Logger(LoggerFactory.getLogger(this.getClass))
  new TestContextManager(this.getClass()).prepareTestInstance(this) //// 3

  "A Actor" should "receive Session if a message is arrived" in {
    implicit val system = ActorSystem.create(Consts.AKKA_SYSTEM_NAME)
    implicit val timeout = Timeout(1.second)

    val actorRef = TestActorRef(new MyActor)

    val future = actorRef ? "session"
    val Success(result: HttpResponse[String]) = future.value.get

    logger.info(result.toString)
    assert(result.code === 200)
  }
}
```

## 설명
1. Java의 `.class()`와 같은 역할을 하는 것이 `classOf[Type]`, 여기에서 SpringRunner를 쓰면 테스트를 인식하지 못하기 때문에 JUnitRunner을 사용해야 한다.
2. Java와 똑같이 하되 `classOf`를 사용
3. Spring TestContext Framework를 사용하기 위한 부분. ScalaTest 클래스를 자동으로 인식하지 못하기 때문에 수동으로 등록해야함