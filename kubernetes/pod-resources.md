# pod resources

pod의 resources에는 node의 리소스자원을 할당받기위한 설정이 들어감. 이 때, resources에 하부 설정에는 requests, limits 두가지가 존재함. 각 리소스 종류마다 실질적으로 동작하는 방식은 다르지만 대체적으로 이 두 하부 설정은 아래와 같은 개념임.

1. requests: pod container가 성공적으로 실행되기 위해서 반드시 확보해야하는 리소스 값. pod이 처음에 스케쥴링될 때 스케쥴러가 이 값을 사용해서 node를 선택함. request 값의 총합은 node에서 사용가능한 리소스의 값보다 커질 수 없음
2. limits: pod container가 차지할 수 있는 리소스의 최대값. 이 값을 넘어가면 pod이 evict되거나 컨테이너가 OOM kill 당할 수 있음. limits는 무제한으로 늘어날 수 있음.

### CPU resources

`Compressible Resource Guarantees` : 어떤 상황에서도 pod이나 container 정리되지는 않음.



### Memory resources

`Incompressible Resource Guarantees` : 특정 상황에서는 pod이나 container를 일부 정리하는 방향으로 동작

메모리의 requests는 kubelet과 스케쥴러에 의해 동작하고 limits는 cgroup에 의해 동작하게 됨. memory resources의 설정에 따라 pod container가 OOMKilled 상태에 빠질 수 있음.

#### OOMKilled

1. node 메모리 부족
   1. 위에서 언급했듯이 memory limits의 총합은 node의 실제 메모리 양보다 커질 수 있음. 따라서 node에 메모리가 부족한 상황이 발생할 수 있음. 이 경우, 컨테이너의 메모리 사용량이 `requests < 메모리 사용량 < limits` 인 임의의 컨테이너가 OOM으로 종료될 수 있음. 
   2. 종료 순서는 컨테이너 pod의 `status.qosClass` 에 따라 결정됨.  `Best-Effort` &gt; `Burstable` &gt; `Guaranteed`  순서로  정리됨.  프로세스를 정리하는 것은 global OOMKiller에 의해 정리되는데 qoaClass에 따라 각각  `oom_score_adj` 값을 다르게 주는 식으로 구현되어있음.
      1. 모든 컨테이너, 모든 resource에 대해서 limits가 존재하고, requests가 아예 없거나 limits와 동일할 경우 `Guaranteed` : `oom_score_adj: -998`
      2. 일부 컨테이너, 일부 resource에 대해서만 limits와 requests가 존재하거나 limits와 requests가 다를 경우 `Burstable`: `oom_score_adj: min(999, max(2, 1000 - 10*(node의 전체 메모리 양에서 pod container의 request가 차지하는 비율 %)))`
      3. resources 설정이 아예 존재하지 않을 경우 `Best-Effort`: `oom_score_adj: 1000`  
2. memory limits 초과
   1. 노드에 얼마나 많은 메모리가 남아있던지 거기에 상관없이 pod container의 메모리 사용량이 limits를 초과하면 container의 프로세스가 OOMKilled로 종료될 수 있음.
   2. cgroup OOMKiller에 의해서 종료됨. PID 1이 아닌 프로세스가 너무 많은 메모리를 차지하는 경우  container가 OOMKilled 상태지만 exit code 0으로 정상종료되는 일이 발생할 수도 있음.
   3. docker run의 `--memory` 옵션과와 같은 동작
   4. cgroup의  `memory.limit_in_bytes` 값을 설정



