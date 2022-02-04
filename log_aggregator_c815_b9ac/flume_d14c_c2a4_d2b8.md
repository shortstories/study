# Flume 테스트

## Flume

### 설치

다운로드 : [http://apache.tt.co.kr/flume/1.6.0/apache-flume-1.6.0-bin.tar.gz](http://apache.tt.co.kr/flume/1.6.0/apache-flume-1.6.0-bin.tar.gz)

```bash
FLUME_HOME="/your/path/"
cd $FLUME_HOME
wget http://apache.tt.co.kr/flume/1.6.0/apache-flume-1.6.0-bin.tar.gz

tar -xvf apache-flume-1.6.0-bin.tar.gz
ln -s apache-flume-1.6.0-bin flume
rm -rf apache-flume-1.6.0-bin.tar.gz

echo "export FLUME_HOME=${FLUME_HOME}/flume" >> ~/.bashrc
echo 'export PATH=$PATH:$FLUME_HOME/bin' >> ~/.bashrc
source ~/.bashrc

echo 'export JAVA_OPTS="-Xms100m -Xmx2000m"' >> $FLUME_HOME/conf/flume-env.sh
wget -O $FLUME_HOME/lib/zookeeper-3.4.5.jar https://repo1.maven.org/maven2/org/apache/zookeeper/zookeeper/3.4.5/zookeeper-3.4.5.jar

flume-ng version
```

### 실행방법

1. config 파일 `$FLUME_HOME/conf` 아래에 작성
2. listener 실행 `flume-ng agent --conf $FLUME_HOME/conf -f $FLUME_HOME/conf/listener.properties --name listener -Dflume.root.logger=INFO,console`
3. consumer 실행 `flume-ng agent --conf $FLUME_HOME/conf -f $FLUME_HOME/conf/consumer.properties --name consumer -Dflume.root.logger=INFO,console`

### 구성

![](<../.gitbook/assets/flume 구성.png>)

### Agents

#### Consumer Agent

&#x20;**Spooling Directory Sources --- Memory Channel ----> Avro Sink (Load balancing Sink Processor)**&#x20;

* Spooling Directory Sources
  * 특정 directory를 지정하면 그 안에 있는 파일과 생기는 파일들을 일괄 읽어 line 단위 Event로 만드는 Source
  * log 파일이 rotation되어 쌓이는 directory를 지정
  * 완료된 파일은 지우도록 할 수도 있고, 이름을 바꿔서 그대로 놔둘 수도 있음
  * 바로 지우는 것 보다는 이름을 바꿔서 저장해놓고 별도의 스크립트 등으로 지우는 것이 안전할듯
* Memory Channel
  * Memory Channel은 버퍼를 메모리에 두는 방식의 채널
  * 처리 중에 flume 프로세스가 강제종료되면 데이터 유실.
  * 그러나 어차피 완료되지 않은 log 파일 자체는 directory에 남아있으므로 굳이 별도의 파일로 저장할 필요가 없고, 그런 전제하에 Memory Channel이 성능적으로 우세하므로 선택
* Avro Sink
  * Flume에서 다층형 구조를 만들때 주로 사용하는 Sink. netty 기반 avro RPC 사용.
  * 필요하다면 deflate 형식으로 데이터를 압축해서 전송하도록 설정할 수 있음. 1-9 사이의 압축 레벨 설정 가능.
* Load balancing Sink Processor
  * 여러 Sink를 묶어서 load balancing 해주는 역할
  * Round Robin 사용
* $FLUME\_HOME/conf/flume-env.sh 에서 Heap Size 꼭 수정할 것

#### Listener Agent

&#x20;**Avro Source --- File Channel ----> HDFS Sink (|| File Roll Sink)**&#x20;

* Avro Sources
  * Consumer Agent에서 Avro Sink를 목표로 보내는 event들을 source로 이용
* File Channel
  * 이 단계에서 메세지가 유실되면 찾을 수 없으므로 성능에서 약간 손해를 보더라도 디스크에 버퍼를 쓰는 File Channel 사용
* HDFS Sink
  * 전달받은 event를 바탕으로 text, sequence 파일을 생성해서 HDFS에 sync.
  * 혹시라도 이 Sink에서 문제가 생기면 그냥 File Roll Sink를 이용하여 파일로 쓰고 기존 Listener 서버의 스크립트를 이용하는 방법도 고려

### 테스트 1 (물리 서버 -> 가상 서버)

> xdevnnidb02.npush CPU v12, Mem 16GB 256MB File x10, 약 2490000 lines (line : 약 1k)

#### Consumer \[xdevnnidb02.npush] -> Listener \[dev-ohchang.ncl]

&#x20;**Batch size**&#x20;

| batch size | avg CPU (%) | time (s) | TPS  | MBps      |
| ---------- | ----------- | -------- | ---- | --------- |
| 100        | 1.43%       | 599s     | 4156 | 4.27 MB/s |
| 500        | 1.72%       | 447s     | 5570 | 5.72 MB/s |
| 1000       | 2.10%       | 345s     | 7217 | 7.42 MB/s |
| 3000       | 2.87%       | 310s     | 8032 | 8.26 MB/s |
| 10000      | 3.77%       | 259s     | 9613 | 9.88 MB/s |

### 테스트 2 (물리 서버 -> 물리 서버)

#### Consumer \[xdevnnidb02.npush] -> Listener \[xdevnnidb01.npush]

&#x20;**Batch size**&#x20;

| batch size | avg CPU (%) | time (s) | TPS   | MBps       | 비고                                                    |
| ---------- | ----------- | -------- | ----- | ---------- | ----------------------------------------------------- |
| 10         | 5.01%       | 193s     | 12901 | 13.26 MB/s | 메모리 사용량 약 10배 이상, 파일을 채널에 쓰는 과정에서 심하면 CPU 50%까지 튀기도 함 |
| 100        | 4.83%       | 121s     | 20578 | 21.2 MB/s  |                                                       |
| 1000       | 4.81%       | 119s     | 20924 | 21.5 MB/s  |                                                       |
| 3000       | 4.93%       | 119s     | 20924 | 21.5 MB/s  |                                                       |
| 10000      | 4.90%       | 123s     | 20243 | 20.8 MB/s  |                                                       |
| 20000      | 5.18%       | 123s     | 20244 | 20.8 MB/s  |                                                       |

#### 참고

[http://appsintheopen.com/posts/42-experimenting-with-flume-performance](http://appsintheopen.com/posts/42-experimenting-with-flume-performance)

### 요구 조건

#### TPS 최대 20000

* 20000 TPS 소화 가능

#### CPU 5% \~ 10%

* 20000 TPS에서 5% 수준

#### 전송 데이터 압축

* 압축 가능

#### 네트워크 대역폭 컨트롤

* Document에서는 batch size를 통해서 throughput을 조절하게 유도하고 있으며, 네트워크 대역폭을 직접적으로 제어하는 기능은 찾을 수 없었음.

### 장점

#### 자바

* jar 파일로 배포되며 API를 제공하기 때문에 상대적으로 고쳐 쓰기 편함. 기본적으론 독자 프로그램으로 동작하지만 필요하다면 API를 끌어다 자신의 프로그램에 붙여 쓰는 것도 가능함.

#### 자동 load balancing 및 failover

* 옵션에서 몇 줄 지정해주는 것으로 동작함.

#### HDFS Sink 제공

* 파일로 떨어뜨린 다음 별도의 스크립트로 동작시키는 과정 없이 바로 HDFS로 넣을 수 있음.

### 단점

#### Document 부실

* 로컬에 비해서 서버에서 성능이 훨씬 떨어지는데다가 CPU 사용량이 훨씬 높아 며칠동안 원인을 찾아 분석하느라 고생했었는데 JVM Heap size로 인한 문제였음. 이 문제를 해결하기위해선 conf폴더 아래에 flume-env.sh 파일을 생성하고 환경 변수를 설정해야하는데 Document에 관련 언급이 없었음.
* 마찬가지로 load balancing을 위해 서버를 세팅해줬을 때, 별다른 에러메세지 없이 한 서버로만 가는 문제가 있어 찾아보니 안 가는 서버쪽 heap size 세팅이 안 되어있어서 생기는 문제.

#### 네트워크 대역폭 제한 불가

* 네트워크 대역폭 사용량을 조절하기 위해서는 batch size를 줄이는 수밖에 없음. 그런데 File Read와 Network의 속도 차이로 인하여 Channel에 Event가 쌓이게 됨. 만약에 TPS가 높게 지속된다면 당연히 Channel의 용량을 초과하게 됨. 그런 증상을 최대한 줄이려면 Channel의 최대 크기를 늘이는 수 밖에 없는데 이렇게 되면 메모리를 대량으로 사용할 수도 있음. 또한 근본적으로 대역폭을 조절하는 방법도 아니며, 각 트랜잭션이 발생하게 되는 시점을 조절할 수 없어서 실질적으로 초당 전송량을 제어하는 것이 불가능함.

#### 서버 세팅 곤란

* 서버를 추가하고 제거해야 할 때마다 config를 수정하고 인스턴스를 재시작해야되는 방식이라 단 하나의 서버를 추가하더라도 배포되어있는 모든 서버의 config를 수정하고 재시작해야되는 문제가 있음. Zookeeper을 사용하는 방법도 있긴 하지만 아직 실험적 기능이라서 적용시키기 난감함

#### 불필요한 오버헤드 발생

* 불필요하게 각 파일을 라인단위로 분리하여 전송하는 문제가 있음.

## Zookeeper 연동

### Zookeeper 설정

```
/
├── {flumeNode}
   ├── {agentName}
   ├── {agentName2}
   └── {agentName3}
```

* 각 agentName node는 그 agent의 설정을 data로 가지고 있어야함.

### 실행 방법

* 실행할 때 -z, -p 옵션을 주어서 실행.
  * &#x20;**-z** : Zookeeper 쿼럼의 주소:포트. 여러 개 등록 가능.
  *   &#x20;**-p** : Flume Config node가 올라갈 Zookeeper 내부 path

      `flume-ng agent --conf {ConfPath} -z {host:port}[,{host:port}] -p {flumeNode} --name {agentName} -Dflume.root.logger=INFO,console`

### 비고

* 각 agentName 노드의 data에 변경이 생기면 모든 agent가 다시 reload.
  * 새로운 agent를 추가할 때 자동 갱신 가능
* agent에 장애가 발생했을 때 Zookeeper로 관리해주지 않음. 무한 재시도.
