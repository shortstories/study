# NIO
## 용도
- Direct memory access
- Block mediated bulk data transfers

## 장점
- select() 시스템 콜과 같은 기능 사용 가능
- 디스크에서 읽어온 데이터를 Kernel mode buffer 에서 User mode buffer로 복사하지 않고 바로 Network로 전송하는 것이 가능
- Non-blocking data access 가능

## 이슈
- 그러나 network 전송을 nQueue Provider에서 처리하고 있는 상황에서 과연 NIO를 써서 얻을 수 있는 이득이 있을까?


## 결론
- 지금 내 용도인 단순 파일 읽기에서는 오히려 자바의 기본적인 I/O보다도 성능이 좋지 않다. 사용하지 않아야 할 것 같다.

> In some cases NIO actually performs worse than basic Java I/O. For simple sequential reads and writes of small files, for instance, a straightforward streams implementation might be two or three times faster than the corresponding event-oriented channel-based coding. Also, non-multiplexed channels -- that is, channels in separate threads -- can be much slower than channels that register their selectors in a single thread.
http://www.javaworld.com/article/2078654/java-se/java-se-five-ways-to-maximize-java-nio-and-nio-2.html

