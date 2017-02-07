# AMQP

**AMQ**는 서버 안에서 동작하는 동적 라우팅과 queuing 모델을 의미하고, **AMQP**는 클라이언트 어플리케이션들이 이러한 기능을 갖춘 서버와 통신하는데 쓰는 프로토콜을 말한다.

AMQP 어플리케이션에서 **exchanges**은 라우팅을 수행하고, **queues** 들은 queuing을 수행한다. 데이터가 publisher에서 시작하여 queues를 거쳐 consumers에게 전달되는 흐름은 보통 다음과 같다. 먼저, queues를 exchanges에 **bind**하고 consumers들로 하여금 그 queues로부터 **consume** 해오도록 만든다. 이윽고 어떤 publisher가 exchanges로 **publish**를 수행하게 되면 queues를 통해 consumers로 데이터가 흘러가게 된다.

**publisher** -`publish`-> **exchanges** -`bind`-> **queues** -`consume` -> **consumers**

## Centralised Messaging Servers

각 서버마다 연결을 맺게 될 경우 규모가 작을 때는 상관 없지만 규모가 커지면 커질 수록 복잡도가 기하급수적으로 증가함에 따라 여러 문제가 발생한다. 이러한 문제를 해결하기 위해 등장한 것이 바로 messaging 서버이다.

모든 서버가 서로서로 연결을 맺게 되면 `n(n - 1)/2` 개의 커넥션이 생성되는 반면, 한 곳으로 집중하여 연결을 맺게 되면 오직 `n`개의 커넥션만 필요하다.

p2p 통신을 통해 해결하려 할 수도 있겠지만 p2p 통신은 아직 검증되지 않았고, 무엇보다도 데이터의 손실을 보장할 수 있느냐는 부분의 신뢰성이 떨어져서 엔터프라이즈 환경에서는 사용하기 힘들다.

일반적으로 messaging 서버들은 여러 어플리케이션들이 동시에 쓰거나 읽을 수 있는 FIFO 자료구조의 저장소를 제공하는 편이다.   
