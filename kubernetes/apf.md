# APF

### APF

k8s 1.18 버전부터 추가됨. 1.20버전부터 default로 활성화되기 시작했으며 1.29부터 stable 기능으로 포함됨.\
기존에는 kube api-server의 과부하를 막기 위해 아래와 같은 두 가지 설정을 통해 제어했음.

* `--max-requests-inflight`: query-type(get,list,watch...) 요청의 최대 동시 처리 수. 기본값 400
* `--max-mutating-requests-inflight`: mutating-type(create,update,delete...) 요청의 최대 동시 처리 수. 기본값 200

위 설정들에는 치명적인 단점이 있었으니 요청의 우선순위를 구분할 수 없다는 점.\
버그나 오류로 인해 특정 operator나 controller가 과도한 요청을 보내는 경우 꼭 필요한 요청들까지 거부될 수 있어 전체 시스템에 장애를 일으킬 수 있음.

APF는 이러한 요구사항을 해결하기 위해 만들어졌으며 아래와 같은 기능들을 제공하고 있음.

1. kube api-server에 들어오는 요청들을 api, user 등 원하는 기준에 따라 분류하고 우선순위를 부여
2. fair-queuing을 통해 최악의 경우에도 전체 kube api-server가 마비되는 것을 방지

log tailing과 같은 long-running 요청들은 APF의 대상이 아님. 이는 기존 max-inflight 설정에서도 동일했음.\
하지만 APF는 watch 요청은 관리한다는 차이점도 가지고 있음.

### 동작 방식

![image](https://media.oss.navercorp.com/user/1709/files/0af2e4b2-a106-4c45-b831-5a2c2bff5056)

1. kube-apiserver로 들어오는 요청들을 `FlowSchema`를 통해 분류해서 하나의 `Flow`로 묶음.
2. 각 `Flow`는 `FlowSchema`에 정의된 `Priority Level`에 매핑됨.
3. `Priority Level`에 정의된 seat가 충분할 경우 요청은 즉시 처리됨
4. 만약 seat 제한을 초과할 경우 요청은 `Priority Level`의 설정에 따라 queue에 들어가거나 즉시 거부됨

APF에서는 `flowcontrol.apiserver.k8s.io` api group에 정의된 리소스들을 사용. 1.29부터는 `v1`, 그 이전 버전에서는 `v1beta3` version을 사용하면 됨.

#### Seat

![image](https://media.oss.navercorp.com/user/1709/files/1b859a5c-cf95-4153-8e13-10dd9909271f)

* APF에서는 열차나 비행기의 좌석에 빗대서 k8s-apiserver에서 요청을 처리할 수 있는 갯수를 `seat`라고 표현
* seat의 최대값은 `max-requests-inflight`와 `max-mutating-requests-inflight`의 값을 더해서 사용
* 하나의 요청은 하나의 seat를 차지하는게 보통이지만 list, watch와 같이 무거운 요청들은 1개 이상의 seat를 차지할 수 있음

#### FlowSchema

```yaml
apiVersion: flowcontrol.apiserver.k8s.io/v1beta3
kind: FlowSchema
metadata:
  annotations:
    apf.kubernetes.io/autoupdate-spec: "true"
  name: service-accounts
spec:
  distinguisherMethod:
    type: ByUser
  matchingPrecedence: 9000
  priorityLevelConfiguration:
    name: workload-low
  rules:
  - nonResourceRules:
    - nonResourceURLs:
      - '*'
      verbs:
      - '*'
    resourceRules:
    - apiGroups:
      - '*'
      clusterScope: true
      namespaces:
      - '*'
      resources:
      - '*'
      verbs:
      - '*'
    subjects:
    - group:
        name: system:serviceaccounts
      kind: Group
```

* api 요청을 분류해서 원하는 prioity level에 할당하기 위해 사용하는 object
* 각 요청이 들어오면 `matchingPrecedence`가 작은 것부터 순서대로 rule을 비교해서 가장 먼저 통과하는 쪽을 사용
* `distinguisherMethod`는 flow를 어떤 식으로 구성할지 결정하며 `ByUser`, `ByNamespace` 중 하나를 사용. flow 단위로 fair-queuing이 적용 되기 때문에 적절한 것을 골라 사용해야함
  * 예를 들어 `n3r-system` namespace에만 적용되는 flow schema의 경우 `ByUser`로 설정해야 정상적으로 fair-queuing이 작동함. `ByNamespace`로 설정할 경우 모든 요청이 하나의 flow로 간주되기 때문

#### PriorityLevelConfiguration

```yaml
apiVersion: flowcontrol.apiserver.k8s.io/v1beta3
kind: PriorityLevelConfiguration
metadata:
  name: example
spec:
  type: Limited
  limited:
    nominalConcurrencyShares: 40 # default: 30
    lendablePercent: 50 # default 0, [0~100]
    borrowingLimitPercent: 120 # default: nil
    limitResponse:
      type: Queue
      queuing:
        handSize: 6
        queueLengthLimit: 50
        queues: 128
```

* priority level을 정의하는데 사용하는 object. seat의 갯수와 queue에 관련된 설정들이 존재함
* type은 `Limited`, `Exempt` 설정 가능. `Exempt`는 APF에서 완전히 제외하고 싶을 때 사용.
* `limitResponse`은 요청이 주어진 seat보다 더 많이 들어올 때 대응하는 방법을 정의함. type에 `Queue`와 `Reject`가 올 수 있고 `Queue`는 초과된 요청들을 queue에 넣고 `Reject`는 즉시 429 에러를 리턴함.
  * `queuing` 설정을 통해 [shuffle sharding, fair queuing 알고리즘](https://github.com/kubernetes/enhancements/tree/master/keps/sig-api-machinery/1040-priority-and-fairness)을 튜닝 가능.
* 현재 n3r에서는 1.23버전이기 때문에 `flowcontrol.apiserver.k8s.io/v1beta2`만 사용 가능
  * `nominalConcurrencyShares` 대신 `assuredConcurrencyShares` 이고 `borrowingLimitPercent`, `lendablePercent`가 존재하지 않음.

![image](https://media.oss.navercorp.com/user/1709/files/7e7e6530-c252-4561-b4b0-81cc8e30b591)

* 각 priority level의 seat는 `nominalConcurrencyShares`의 비율만큼 정해짐
* 요청이 적을 경우 `lendablePercent`만큼 다른 priority level에 대여해줄 수 있음
* 요청이 많이 들어오는 경우 `borrowingLimitPercent`만큼 다른 priority level에서 빌려올 수 있음.
* 각 priority level의 seat 갯수는 10초마다 재조정됨

```
❯ kubectl get prioritylevelconfigurations
NAME              TYPE      ASSUREDCONCURRENCYSHARES   QUEUES   HANDSIZE   QUEUELENGTHLIMIT   AGE
catch-all         Limited   5                          <none>   <none>     <none>             3y57d
exempt            Exempt    <none>                     <none>   <none>     <none>             3y57d
global-default    Limited   20                         128      6          50                 3y57d
leader-election   Limited   10                         16       4          50                 3y57d
node-high         Limited   40                         64       6          50                 3y57d
system            Limited   30                         64       6          50                 3y57d
workload-high     Limited   40                         128      6          50                 3y57d
workload-low      Limited   100                        128      6          50                 3y57d
```

* 기본적으로 위와 같은 priority level들이 자동으로 생성됨.
* `catch-all`, `exempt`는 Mandatory Objects로 수정 불가능.
* 그 외 나머지는 Suggested Objects로 수정이 가능하지만 `apf.kubernetes.io/autoupdate-spec`이라는 annotation의 값을 `"false"`로 설정해야 원래 값으로 되돌아가는 것을 방지할 수 있음

### 모니터링

#### metrics

https://kubernetes.io/docs/concepts/cluster-administration/flow-control/#metrics\


주요 메트릭들

* `apiserver_flowcontrol_current_inqueue_requests`
  * 현재 queue에 들어있는 요청의 갯수
* `apiserver_flowcontrol_current_executing_requests`
  * 현재 실행 중인 요청의 갯수
* `apiserver_flowcontrol_rejected_requests_total`
  * 429 에러가 발생한 요청의 갯수
  * `reason`에서 원인 확인 가능
    * `concurrency-limit`: priority level의 limitResponse.type이 `Rejected`이고 최대 seat 갯수를 초과함
    * `queue-full`: 특정 flow의 queue에 들어간 요청 갯수가 `handSize * queueLengthLimit`를 초과함
    * `time-out`: 요청이 queue에 들어간 상태에서 제한 시간을 초과함. 제한 시간은 kube-apiserver `--request-timeout`의 1/4이며 기본값은 15초
    * `cancelled`: 요청이 보호 상태가 아니었고 큐로부터 제거됨
* `apiserver_flowcontrol_request_wait_duration_seconds`
  * 요청이 처리될 때까지 queue에서 기다리는 시간

이 중 `apiserver_flowcontrol_current_inqueue_requests`, `apiserver_flowcontrol_request_wait_duration_seconds` 두 메트릭을 통해 현재 apf 설정에 문제가 있는지 감지할 수 있음. queue가 지속적으로 차있거나 request wait duration이 늘어나기 시작하면 priority level의 seat를 늘리거나 문제되는 요청을 찾아 격리해야한다는 신호로 이해하면 됨.

`apiserver_flowcontrol_rejected_requests_total`가 발생하면 request dropped가 발생하고 있다는 의미이므로 reason을 보고 대응해야함. `queue-full`의 경우 priority level의 queueLengthLimit이나 handSize를 늘려주는 것으로 대응할 수 있음. 하지만 queue 사이즈를 계속 늘리다간 결국 `time-out`이 발생할 수 밖에 없으므로 근본적인 문제를 찾아 수정해줘야함.

#### debug endpoint

* `kubectl get --raw /debug/api_priority_and_fairness/dump_priority_levels`
* `kubectl get --raw /debug/api_priority_and_fairness/dump_queues`
* `kubectl get --raw /debug/api_priority_and_fairness/dump_requests`

dump\_priority\_levels, dump\_queues의 경우에는 metrics로도 대부분 확인 가능한 부분으로 크게 필요성이 느껴지지 않음\
dump\_requests는 현재 특정 priority level이 어떤 요청때문에 밀리고있는지 확인하기 위해서 필요

```
PriorityLevelName, FlowSchemaName,               QueueIndex, RequestIndexInQueue, FlowDistingsher,                                          ArriveTime,                     InitialSeats, FinalSeats, AdditionalLatency, UserName,                                                 Verb,   APIPath,                                             Namespace, Name,           APIVersion,               Resource,                         SubResource
workload-high,     kube-scheduler,               0,          0,                   ,                                                         2025-05-07T05:29:17.776292385Z, 1,            0,          0s,                system:kube-scheduler,                                    watch,  /apis/storage.k8s.io/v1/csidrivers,                  ,          ,               storage.k8s.io/v1,        csidrivers,                       
workload-high,     kube-controller-manager,      20,         0,                   ,                                                         2025-05-07T05:29:14.588073957Z, 1,            0,          0s,                system:kube-controller-manager,                           create, /apis/authentication.k8s.io/v1/tokenreviews,         ,          ,               authentication.k8s.io/v1, tokenreviews,                     
workload-high,     kube-system-service-accounts, 34,         0,                   system:serviceaccount:kube-system:cilium,                 2025-05-07T05:29:04.758197878Z, 1,            0,          0s,                system:serviceaccount:kube-system:cilium,                 watch,  /apis/cilium.io/v2alpha1/ciliumegressnatpolicies,    ,          ,               cilium.io/v2alpha1,       ciliumegressnatpolicies,          
workload-high,     kube-system-service-accounts, 34,         1,                   system:serviceaccount:kube-system:cilium,                 2025-05-07T05:29:06.870180444Z, 1,            0,          0s,                system:serviceaccount:kube-system:cilium,                 watch,  /apis/discovery.k8s.io/v1/endpointslices,            ,          ,               discovery.k8s.io/v1,      endpointslices,                   
workload-high,     kube-system-service-accounts, 34,         2,                   system:serviceaccount:kube-system:cilium,                 2025-05-07T05:29:07.784092776Z, 1,            0,          0s,                system:serviceaccount:kube-system:cilium,                 watch,  /apis/cilium.io/v2/ciliumegressgatewaypolicies,      ,          ,               cilium.io/v2,             ciliumegressgatewaypolicies,      
workload-high,     kube-system-service-accounts, 34,         3,                   system:serviceaccount:kube-system:cilium,                 2025-05-07T05:29:08.57969627Z,  1,            0,          0s,                system:serviceaccount:kube-system:cilium,                 get,    /version,                                            ,          ,               ,                         ,                                 
workload-high,     kube-system-service-accounts, 34,         4,                   system:serviceaccount:kube-system:cilium,                 2025-05-07T05:29:08.591909362Z, 1,            0,          0s,                system:serviceaccount:kube-system:cilium,                 get,    /version,                                            ,          ,               ,                         ,                                 
workload-high,     kube-system-service-accounts, 34,         5,                   system:serviceaccount:kube-system:cilium,                 2025-05-07T05:29:08.677503566Z, 1,            0,          0s,                system:serviceaccount:kube-system:cilium,                 get,    /version,                                            ,          ,               ,                         ,                                 
workload-high,     kube-system-service-accounts, 34,         6,                   system:serviceaccount:kube-system:cilium,                 2025-05-07T05:29:08.73207926Z,  1,            0,          0s,                system:serviceaccount:kube-system:cilium,                 get,    /version,                                            ,          ,               ,                         ,                                 
workload-high,     kube-system-service-accounts, 34,         7,                   system:serviceaccount:kube-system:cilium,                 2025-05-07T05:29:08.891268256Z, 1,            0,          0s,                system:serviceaccount:kube-system:cilium,                 get,    /version,                                            ,          ,               ,                         ,                                 
...
```

### 429에러 대응 방안

1. 제일 좋은 방법은 workload 자체를 개선하는 것. APF는 차선책
2. 중요하거나 문제가 되는 요청들을 격리시키기
   * 새로운 FlowSchema, PriorityLevelConfiguration을 생성해서 문제가 되는 요청들을 다른 요청에게서 격리
3. 특정 요청에 더 많은 seat 부여하기
   * PriorityLevelConfiguration에 ConcurrencyShare를 더 부여하거나 더 많은 seat를 가진 priority level에 할당하기

새 PriorityLevelConfiguration을 생성하거나 수정할 때 추가된 ConcurrencyShare만큼 다른 곳에서 줄여줘야 전체 priority level의 seat 갯수가 줄어드는 것을 방지할 수 있음

### N3R

#### mrse9 적용 케이스

* 사용자가 배포를 시도하면 순간적으로 요청이 몰려 429 에러가 다수 발생하고 있던 상황
* 429 에러가 발생하는 원인이 모두 `queue-full`이므로 queue를 늘리는 조치 적용
* `queue-full`은 더이상 발생하지 않았지만 이번에는 `time-out`으로 429 에러가 발생. request wait duration이 1분 넘게 올라가는 증상 발생.
* audit log에서 제일 요청이 많은 것으로 나오는 `pod patch`, `deployments, statefulsets update`를 별도의 priorityLevelConfiguration, flowSchema로 분리했으나 여전히 429 에러 발생.
* 동일한 상황에서 `kubectl get --raw "/debug/api_priority_and_fairness/dump_requests"`를 사용하여 APF requests dump를 수집
* dump 분석 결과 cilium watch, ciliumendpoints list 요청이 가장 큰 병목인 것으로 확인되어 exempt에 넣어 해결

#### pgrma1 적용 케이스

* 5분 주기로 pod list/patch 요청이 대량으로 들어오며 request drop이 발생하던 상황
* 각 namespace에 배포되어있는 postgresql pod이 status 갱신을 하며 발생하는 요청들로 확인됨
* postgresql의 설정을 고쳐 status 갱신 주기를 늘리는 등 조치
* 그 후에도 지속적으로 request drop이 발생하여 n3r.registry와 관련된 요청들을 격리하여 새 FlowSchema와 PriorityLevelConfiguration 추가
* pod list에서 지속적으로 request drop이 발생하여 list 요청들을 격리하기 위한 FlowSchema, PriorityLevelConfiguration 추가
* `service-account` priority level에서 지속적으로 request drop이 발생하여 queue 용량을 늘리는 조치 이후 해결

#### 공통 설정?

* cluster scope list, watch 요청들은 별도의 priority level로 격리가 필요
  * `time-out`으로 429 에러가 발생한 대부분이 이 케이스
  * fair queuing이 동작한다 하더라도 한건 한건마다 초단위로 걸리는 요청들이 대량으로 쌓이면 순식간에 timeout이 발생
* `n3r-system` namespace에 전용 `FlowSchema`, 필요하다면 `PriorityLevelConfiguration` 까지 추가
  * kube-system과 동일한 방식으로 관리 필요
  * n3r-system에는 leader-election을 위한 FlowSchema 설정이 없다보니 부하 상황에서 lease 관련 요청이 밀리게 되면 leadership을 잃어 operator container가 계속 재시작되는 현상 발견
* cilium이나 masterdns처럼 클러스터 전체에 영향을 줄 수 있는 컴포넌트들은 exempt에 넣어서 APF를 타지 않게 관리
  * mrse9의 적용 과정에서 cilium의 경우 exempt에 넣어 APF를 타지 않게 설정하니 오히려 kube apiserver의 cpu 사용량이 낮아짐

#### 추가 검토

* `idle` priority level 추가?
  * 전체 seat 갯수를 늘리려면 `--max-mutating-requests-inflight`, `--max-requests-inflight`의 값을 바꿔야하는데 이 값을 바꾸려면 kube apiserver를 재시작해야하니 꽤 부담스러운 작업
  * 처음부터 `--max-mutating-requests-inflight`, `--max-requests-inflight` 값을 크게 주고 늘어난 만큼 `idle`이라는 priority level을 추가하는 방법
  * `idle` priority level에는 아무 flowSchema도 연결되어있지 않아 사용되지 않음. 오로지 seat의 갯수를 조절하기 위한 용도. `--max-mutating-requests-inflight`, `--max-requests-inflight` 값을 기존의 2배로 주더라도 기존 priority level의 concurrencyShares 총합과 동일한 용량을 가진 `idle` priority level를 만든다면 실제 각 priority level에 부여되는 seat 수는 기존과 동일
  * kube apiserver의 설정을 건드리지 않고 `idle`의 concurrency share를 줄이거나 다른데 부여하는 식으로 seat의 갯수를 조절할 수 있음
* k8s 버전 업그레이드?
  * 현재 버전 (v1.23) 에서는 APF를 사용할 때 여러가지 문제점들이 있음.
  * `borrowingLimitPercent`, `lendablePercent` 스펙이 존재하지 않아 pgrma1처럼 순간적으로만 요청이 대량으로 들어오는 priority level들의 경우 seat 갯수가 낭비되고 있음
  * `/debug/api_priority_and_fairness/dump_requests` api의 결과값이 정상적으로 나오지 않음
  * APF 관련 metric들의 이름, 값이 최신 버전과 서로 다름
* trace 연동?
  * 현재 n3r audit 및 request dump에서는 요청이 실제로 얼마나 걸렸는지에 대한 정보가 없음
  * 요청의 갯수 및 metric을 전후로 검토하여 추정하고 있는 관계로 trace 정보가 제대로 나오면 큰 도움이 될 것으로 보임
