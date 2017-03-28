# RabbitMQ 삽질

MEX2 Gateway & Agent를 개발하면서 RabbitMQ를 사용하였다. 그 과정에서 느꼈던 점들을 간략히 정리한다.

## 1. 구조 선택

RabbitMQ를 사용하는 코드를 작성할 때, 가장 중요한 부분이 아닌가싶다.
공식 문서에서 살펴볼 수 있는 구조는 주로 6가지인데, 제일 첫번째 구조는 사실상 이걸 쓰느니 그냥 서버 사이에 HTTP로 통신하는거랑 뭐가 다를까 싶다. 메세징 큐를 사이에 끼워넣는 장점이 거의 없다시피 한 것 같다.

### Work Queues

![](https://www.rabbitmq.com/img/tutorials/python-two.png)

제일 간단한 방식인 듯 하다. 그야말로 메세징 큐 그 자체를 보여준다.

이 구조에서 우리는 하나의 큐를 만들고 그 큐에다가 메세지를 보내는 publisher와 그 메세지를 받아다가 처리하는 consumer들을 죄다 갖다붙인다. Exchange와 routingKey에는 신경을 쓰지 않아도 좋다. 어차피 이 단계에서는 전혀 필요가 없다. consumer들을 여러개 붙이게 되면 Round-Robin 방식으로 메세지들을 알아서 분산 처리하게 된다.

다만 몇 가지 신경써야하는 설정이 있는데 다음과 같다.

1. ack 처리
  - consumer쪽 설정이다. RabbitMQ에서는 consumer로 메세지를 보내고, consumer가 메세지를 정상적으로 받았다는 ack를 보내야 큐에서 메세지를 지우게 된다. 그에 따라 이 ack를 어떻게 처리할건지 고를 수 있는데, `autoAck` 옵션을 활성화시키면 따로 신경쓸 필요 없이 자동으로 ack 처리가 이루어지게 되고, 비활성화시키면 사용자가 직접 ack 처리를 해줘야되는 대신에 예상치못한 데이터 소실을 막을 수 있다. 다만 주의해야되는 점은 사용자가 `autoAck`를 끄고 ack 처리도 잊으면 RabbitMQ 서버가 메모리를 점점 끊임없이 먹다가 터질 수도 있다는 것이다. 
1. 메세지 보관 방식
  - 큐를 최초 생성할 때, 그리고 메세지를 보낼 때 만지는 설정이다. 큐를 만들 때 `durable` 옵션을 활성화시키고, 메세지를 보낼 때 `MessageProperties.PERSISTENT_TEXT_PLAIN` 속성을 부여해서 보내면 된다. 이렇게 되면 서버에서 메세지를 memory가 아니라 storage에 저장하기 때문에 RabbitMQ 서버가 급작스럽게 사망해도 메세지를 잃지 않을 수 있다. 다만 성능적으로 손해를 볼 수 밖에 없긴 하다.
1. 메세지 분산 배치 갯수
  - consumer의 channel에서 설정한다. 이 값은 숫자 값이며, 설정한 값에 따라서 한번에 consumer에게 주어진 갯수 만큼의 메세지만 준다. 다시 말해서 한 consumer 당 값이 1일 때, 이미 consumer에게 하나의 메세지가 들어가있다면 그 consumer가 ack를 하기 전 까지는 추가로 메세지를 주지 않고 다른 consumer를 찾는다는 뜻이다. 특정 consumer에게 메세지가 몰리는 것을 방지하기 위해서 사용할 수 있다.

### Pub/Sub

![](https://www.rabbitmq.com/img/tutorials/exchanges.png)

여기서부터는 Exchange를 사용한다. 특히 Publisher-Subscriber의 경우 Exchange Type `fanout`을 사용한다. `fanout`은 publisher가 어떤 메세지를 Exchange로 보내면 묻지도 따지지도 않고 binding된 모든 큐로 메세지를 보내주는 방식이다. `fanout`을 쓰게 되면 routingKey가 아무런 의미가 없으니 대충 설정해도 된다.

### Routing

![](https://www.rabbitmq.com/img/tutorials/direct-exchange.png)
![](https://www.rabbitmq.com/img/tutorials/direct-exchange-multiple.png)

Exchange type `direct`를 사용하는 방식이다. routingKey에 따라 특정 메세지는 특정 큐에만 들어갔으면 할 때 사용한다. 
위의 그림처럼 하나의 큐당 하나의 routingKey를 binding할 수도 있고, 동일한 routingKey를 여러 큐에다가 binding할 수도 있다. 위에서 두번째 그림처럼 설정하게 되면 사실 `fanout`이랑 다를 것 없이 동작한다.

### Topics

![](https://www.rabbitmq.com/img/tutorials/python-five.png)

Exchange type `topic`을 사용하는 방식이다. 거의 대부분 Routing 구조와 다를게 없는데, 유일하게 다른 부분은 routingKey가 좀 특별하다는 것이다.