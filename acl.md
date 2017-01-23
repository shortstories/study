# ACL

## Design

### 토큰 구조

모든 토큰은 ID, name, type, rule set을 가진다.

```
token = {
  ID: "",
  name: "",
  type: ["client", "management"],
  rule_set: {
    // ...
  }
}
```

ID는 랜덤하게 생성되는 UUID이다.

name은 사람이 읽을 수 있으며 Consul에게 있어 opaque인 이름이다.

type은 client, management 두 가지가 있다. client는 ACL rules를 수정할 수 없는 토큰이라는 것이고, management는 모든 행동이 가능한 말하자면 관리자라는 뜻이다.

이 토큰은 서버로 통하는 모든 RPC request에 첨부된다.

### 토큰 설정

토큰을 지정하는 방법에는 총 세 가지가 있다.

1. Agent 옵션의 `acl_token`프로퍼티를 통해 default token 설정
2. 원하는 Request의 HTTP Header에다가 `X-Consul-Token: {token}`설정
3. 원하는 Reqeust의 쿼리 스트링 패러미터에 `token={token}` 설정

이 세 가지 방법의 우선 순위는 다음과 같다.

> default token &lt; X-Consul-Token header &lt; token query string parameter

별도 policy가 지정되지 않았을 때 어떻게 동작할 건지 선택할 수도 있다. Agent 옵션 중 `acl_default_policy`값을 설정하면 된다.

* allow : 기본적으로 모든 요청을 받아들이고 차단할 부분만 차단하여 일종의 블랙 리스트처럼 동작. 기본값
* deny : 기본적으로 모든 요청을 거부하고 필요한 부분만 허용하여 일종의 화이트 리스트처럼 동작

### ACL Datacenter

요청을 받아들일지 거부할지 판단하는 것은 언제나 서버 노드에서 실행된다. 모든 서버는 `acl_datacenter` 옵션에 특정 값을 줘야한다. 이 값을 통해서 ACL 적용 여부가 결정되며, 동시에 인증 datacenter가 어떤 것인지 지정된다.Consul은 multi datacenter을 지원하기 위해서 RPC forwarding에 의지하고 있다. 이렇게 되면 요청이 각 datacenter의 범위를 넘나들며 오갈 수 있기 때문에 ACL 토큰은 글로벌하게 동작해야 한다. 이 때, 동시성 문제를 피하기 위해 하나의 인증 받은 datacenter를 별도로 운영하고 거기에 적절한 형태의 토큰들을 저장하는 것을 고려해야한다.

만약에 요청이 인증받지 않은 datacenter의 서버로부터 왔다면 이것은 반드시 적절한 policy에 의해서 검증되어야 한다. 이러한 절차는 인증받은 서버로부터 토큰을 읽어와서 그 결과를 캐싱하는 것으로 이루어진다. 이 때, 이 캐시는 `acl_ttl` 옵션을 통해 설정할 수 있다. 저 값의 의미는 policy를 얼마마다 새로운 policy로 교체할 것인가를 결정한다. ttl 값을 0으로 줄 수도 있다. 그러나 만약에 인증 datacenter가 다른 datacenter이라면 모든 요청에 WAN RPC call을 날릴 수도 있어 성능에 큰 영향을 줄 수도 있으니 주의해야 한다.

### 장애 대비, Replication

`acl_data_center` 에 지정된 값이나 리더 노드로부터 토큰에 대한 policy를 읽어올 수 없는 상황에서 어떻게 동작할 것인지 선택할 수 있다. Agent 옵션 중 `acl_down_policy`값을 설정하면 된다.

* allow : 모든 요청을 받아들임
* deny : 모든 요청을 거부함
* extend-cache : 만약에 캐싱된 policy가 있다면 TTL을 무시하고 그 policy대로 동작. 캐싱하지 않는 상황이라면 deny와 동일하게 동작. 기본값

그 외에 다른 선택지로 인증 datacenter 서버가 아닌 곳에서도 전체 토큰을 복제해서 유지하도록 하는 방법이 있다. 캐싱과는 전혀 관계없다. 해당 서버의 옵션에 `acl_replication_token`을 주게 되면 대략 30초에 한번씩 백그라운드 프로세스로 복제를 진행하게 되며, 이 작업의 업데이트는 초당 100개정도로 쓰로틀링이 걸려있기 때문에 토큰의 양이 많다면 몇분씩 걸릴 수도 있다. 이 때, 복제용 token은 적절한 권한을 가지고 있어야하며 master ACL 토큰과 같을 수도 있다. 인증 datacenter에 장애가 발생하면 `acl_down_policy`값에 의해 동일하게 동작한다. 복제된 ACL로 인증된 경우에도 `acl_ttl`세팅에 따라 캐싱이 이루어진다. 이 경우 인증 datacenter가 다시 복구되더라도 TTL이 만료되기 전 까지는 갱신되지 않는다.

복제는 인증 datacenter을 옮길 때 사용할 수도 있다.

1. 서비스가 중단되지 않도록 모든 datacenter에서 ACL replication을 활성화시킨다. 옮길 datacenter에서 API를 사용하여 replication이 적절하게 이루어졌는지 확인한다.
2. 이전 인증 datacenter을 종료한다.
3. 옮길 datacenter의 서버들을 재시작하면서 `acl_datacenter`을 자기 자신으로 바꾼다. 이렇게 하면 자동으로 replication은 꺼지고 자기 자신을 인증 datacenter으로 동작하게 만든다. 이 때, 복제된 토큰들을 사용한다.
4. 새롭게 옮겨진 datacenter을 제외한 다른 모든 datacenter들을 재시작하면서 `acl_datacenter`을 새로운 datacenter으로 변경한다.

### Bootstrapping ACLs

`acl_master_token`을 넣어서 서버를 시작하는 것으로 ACL을 만들 수 있음. 이 경우 토큰은 "management" 타입으로 생성됨. `acl_master_token`은 서버가 리더일 때만 설정되므로 만약에 다른 토큰으로 새로 설정하고 싶다면 현재 리더를 제외한 다른 모든 서버에 새로운 토큰을 `acl_master_token`으로 지정하고 현재 리더를 재시작해서 다른 서버로 리더를 넘기면 됨.

### Rule 설정

- 기본적으로 HCL 사용 https://github.com/hashicorp/hcl/
- policy : read, write, deny
  - write는 read를 포함하며 write만 허용하는 방법은 없음
- 기본 값을 지정하고 싶으면 비어있는 String을 주면 됨. (ex) key "" { })
- rule이 지정되지 않으면 `acl_default_policy`를 따름
- 만약 일부 겹치는 부분이 생길 땐 가장 길고 상세한 prefix를 가지는 policy를 따름

#### Key Policy

- key "prefix" { policy }

#### Service Policy

- service "service name" { policy }
- read는 service prefix의 discovery에 대한 접근을 제한할 수 있음

#### User event Policy

- event "event name" { policy }
- 현재로선 항상 write 레벨이므로 언제나 read 가능

#### Prepared query Policy

- query "query name" { policy }