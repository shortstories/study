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

### acl\_default\_policy

별도 policy가 지정되지 않았을 때 어떻게 동작할 건지 선택하는 부분이다. Agent 옵션 중 `acl_default_policy`값을 설정하면 된다.

* allow : 기본적으로 모두 받아들이고 차단할 부분만 차단하여 일종의 블랙 리스트처럼 동작. 기본값
* deny : 기본적으로 모두 거부하고 필요한 부분만 허용하여 일종의 화이트 리스트처럼 동작

### acl\_down\_policy

`acl_data_center` 에 지정된 값이나 리더 노드로부터 토큰에 대한 policy를 읽어올 수 없는 상황에서 어떻게 동작할 것인지 선택하는 부분이다. Agent 옵션 중 

* allow
* deny
* extend-cache





