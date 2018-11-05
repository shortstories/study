# DDOS

## 유형

* CPU, Memory 등 서버의 자원이 고갈되게끔 하는 공격
* 네트워크 트래픽을 늘려서 대역폭이 고갈되게끔 하는 공격
* 프로그래밍 취약점에 대한 공격

### SYN Flooding

* unreachable host를 source address로 갖는 패킷을 계속 보내서 Backlog Queue에 overflow를 발생하게 만듬
* Queue가 꽉 차면 다시 빌 때까지 request를 거부
* TCP/IP는 ACK가 돌아올 때까지 일정 시간 대기하므로 가능한 공격인듯

### UDP Flooding

* source address와 port를 공격 대상의 것으로 바꾼 UDP 패킷을 대량으로 보냄
* 패킷을 받은 서버는 공격 대상에게 그만큼의 Response를 하려고 시도할 것이고 그 사이에 대량의 트래픽이 발생하여 과부화 초래
* 여기서 Response는 ICMP 패킷이 될 수 있음

### ICMP Flooding

* 특정 port에 listen 하고 있는 서비스가 필요 없음
* 대량의 ICMP 패킷을 보내서 네트워크 과부하 유도
* worm 등으로 좀비 pc를 만들고 대량의 패킷을 한번에 쏴버리는 방식 유행

