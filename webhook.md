# Webhook

- 리얼타임 통신을 위한 방법 중 하나
- 이벤트 기반 동작
- Web Callback, HTTP push API 등 다양한 이름으로 불림
- 보통 POST request를 통해서 콜백 데이터 전달

## 개념

### Consuming

- Webhook을 받기 위해서 Webhook provider에게 URL을 제공
- 일반적으로 json, xml 형식의 POST Request가 전달됨

### Debugging

- 기본적으로 비동기 동작이므로 별도의 도구 사용
- ex)
  1. RequestBin으로 리퀘스트 수집
  2. Postman, cURL 등으로 리퀘스트 테스트 전송
  3. ngrok로 로컬에서 테스트
  4. Runscope로 전체적인 플로우 감시

### Security

- webhook의 URL은 기본적으로 공개되어있기 때문에 전송 데이터를 검증할 방법이 필요함
  1. TLS Connections
  2. URL에 auth Token 추가
  3. Basic Auth 제공 https://en.wikipedia.org/wiki/Basic_access_authentication
  4. provider가 Request를 보내기 전에 서명하여 signature를 만들고 public key를 제공하여 받는 쪽에서 적절한 암호화 알고리즘을 통해 검증

### Failure & Retries

#### HTTP Status Code 대응

- 2XX 
  - 성공한 것이므로 종료
- 3XX
  - webhook subscription 정보를 동적으로 수정할 것인지 결정 필요
  - fail으로 처리할 것인지 retry로 처리할 것인지 결정 필요
- 4XX
  - 일반적으로 retry 해야 함
  - 401, 404의 경우 일시적인 문제일 수 있으므로 즉시 처리하지 말아야 함
  - 410이라면 즉시 처리
- 5XX
  - 기본적으로 retry 하되 정해진 횟수나 시간 만큼만 하도록 설정

#### Failure 대응

##### Exponential Backoff

- 같은 webhook에 대해 retry를 할 때마다 간격을 점점 늘려나가며 하는 방법
- 일정 시간 간격 내지는 일정 횟수 제한을 두고 그 제한을 넘어가면 webhook consumer가 비정상인 것으로 간주하여 subscription 자체를 중단하게 설정

##### Claim Check

- webhook에 실패할 경우 별도의 저장소에다가 저장해두고 나중에 확인할 수 있도록 하는 방법
  1. 저장된 webhook을 확인할 수 있게 별도의 URL을 미리 제공하여 webhook consumer가 필요할 때 찾아볼 수 있도록 하는 방법
  2. 정해진 횟수의 retry 이후 저장소에 저장. 이후 만료되기 전까지 정기적으로 저장된 webhook을 확인할 수 있는 URL을 전송하는 방법

#### Ensuring Ordered Delivery

- webhook의 순서를 보장하기 위해선 일종의 sequence ID를 주는 방법이 있음
- 물론 이 경우엔 webhook consumer가 webhook을 받아서 재정렬하는 과정이 필요

### Events

- 보통 name, payload 두 가지로 구성됨

#### Name

- `[Resource Name].[Sub Resource Name].[Event]` 같은 형식으로 명명

#### Payload

- Webhook의 리소스에 해당하는 REST API가 이미 있다면 webhook의 페이로드도 완전히 똑같이 만들어주는 편이 좋음

## 구현 사례

### Slack

- Incoming Webhooks와 Outgoing Webhooks 두 가지를 지원하고 있음
- Rate Limits 존재

#### Incoming Webhooks

- JSON 형식
- 슬랙 내부에 메세지를 보내기 위해서 사용

#### Outgoing Webhooks

- 특정 채널에서 특정 'trigger word'가 발생하였을 때 정해진 URL로 POST request를 보내주는 webhook
- POST body

``` form
token=XXXXXXXXXXXXXXXXXX
team_id=T0001
team_domain=example
channel_id=C2147483705
channel_name=test
timestamp=1355517523.000005
user_id=U2147483697
user_name=Steve
text=googlebot: What is the air-speed velocity of an unladen swallow?
trigger_word=googlebot:
```

### Google Calendar

- webhook 대신에 push notifications 라는 표현 사용
- 