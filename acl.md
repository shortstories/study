# ACL

## Design

모든 토큰은 ID, name, type, rule set을 가진다.

ID는 랜덤하게 생성되는 UUID이다.

name은 사람이 읽을 수 있으며 Consul에게 있어 opaque인 이름이다.

type은 client, management 두 가지가 있다. client는 ACL rules를 수정할 수 없는 토큰이라는 것이고, management는 모든 행동이 가능한 말하자면 관리자라는 뜻이다.

이 토큰들은 서버로 통하는 모든 RPC request에 첨부된다. Agent는 `acl_token `프로퍼티를 통해 default token을 지정할 수 있다.

만약에 각 request마다 토큰을 지정하고 싶다면 두 가지 방법이 있는데, 하나는 HTTP Header에다가 `X-Consul-Token: {token}`를 넣는 것이고, 두 번째는 쿼리 스트링 패러미터로 `token={token}` 을 넣는 것이다.

