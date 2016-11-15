# Webhook

- 리얼타임 통신을 위한 방법 중 하나
- 이벤트 기반 동작
- Web Callback, HTTP push API 등 다양한 이름으로 불림
- RESTful Webhooks라고 하여 RESTful interface를 제공하는 경우가 많음
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
- HTTPS
- 5XX에 대해서 Retry를 시도하며 Exponential backoff 사용
  - 그 외에는 모두 버림

#### Watch

- `https://www.googleapis.com/apiName/apiVersion/resourcePath/watch`
- webhook 등록

##### Request

``` json
POST https://www.googleapis.com/calendar/v3/calendars/my_calendar@gmail.com/events/watch
Authorization: Bearer auth_token_for_current_user
Content-Type: application/json

{
  "id": "01234567-89ab-cdef-0123456789ab", // Your channel ID.
  "type": "web_hook",
  "address": "https://mydomain.com/notifications", // Your receiving URL.
  // ...
  "token": "target=myApp-myCalendarChannelDest", // (Optional) Your channel token.
  "expiration": 1426325213000 // (Optional) Your requested channel expiration time.
  }
}
```

##### response
``` json
{
  "kind": "api#channel",
  "id": "01234567-89ab-cdef-0123456789ab"", // ID you specified for this channel.
  "resourceId": "o3hgv1538sdjfh", // ID of the watched resource.
  "resourceUri": "https://www.googleapis.com/calendar/v3/calendars/my_calendar@gmail.com/events", // Version-specific ID of the watched resource.
  "token": "target=myApp-myCalendarChannelDest", // Present only if one was provided.
  "expiration": 1426325213000, // Actual expiration time as Unix timestamp (in ms), if applicable.
}
```

#### Sync

- webhook이 시작되었음을 알림

##### Request

``` form
POST https://mydomain.com/notifications // Your receiving URL.
X-Goog-Channel-ID: channel-ID-value
X-Goog-Channel-Token: channel-token-value
X-Goog-Channel-Expiration: expiration-date-and-time // In human-readable format; present only if channel expires.
X-Goog-Resource-ID: identifier-for-the-watched-resource
X-Goog-Resource-URI: version-specific-URI-of-the-watched-resource
X-Goog-Resource-State: sync
X-Goog-Message-Number: 1
```

#### Notifications

- 실질적인 webhook
- HTTPS POST

##### Request
``` form
POST https://mydomain.com/notifications // Your receiving URL.
Content-Type: application/json; utf-8
Content-Length: 0
X-Goog-Channel-ID: 4ba78bf0-6a47-11e2-bcfd-0800200c9a66
X-Goog-Channel-Token: 398348u3tu83ut8uu38
X-Goog-Channel-Expiration: Tue, 19 Nov 2013 01:13:52 GMT
X-Goog-Resource-ID:  ret08u3rv24htgh289g
X-Goog-Resource-URI: https://www.googleapis.com/calendar/v3/calendars/my_calendar@gmail.com/events
X-Goog-Resource-State:  exists
X-Goog-Message-Number: 10
```

#### Stop

- `https://www.googleapis.com/calendar/v3/channels/stop`
- 만료되기 전에 직접 종료하는 방법

``` json
POST https://www.googleapis.com/calendar/v3/channels/stop
Authorization: Bearer {auth_token_for_current_user}
Content-Type: application/json

{
  "id": "4ba78bf0-6a47-11e2-bcfd-0800200c9a66",
  "resourceId": "ret08u3rv24htgh289g"
}
```

### Jira

- webhook을 얻을 수 있는 쿼리 메소드 제공

#### 등록

- `<JIRA_URL>/rest/webhooks/1.0/webhook`

``` json
{
  "name": "my first webhook via rest",
  "url": "http://www.example.com/webhooks",
  "events": [
    "jira:issue_created",
    "jira:issue_updated"
  ],
  "jqlFilter": "Project = JRA AND resolution = Fixed",
  "excludeIssueDetails" : false
}
```

#### 해지

- `<JIRA_URL>/rest/webhooks/1.0/webhook/{id of the webhook}`

#### 쿼리

- `<JIRA_URL>/rest/webhooks/1.0/webhook`
- `<JIRA_URL>/rest/webhooks/1.0/webhook/<webhook ID>`

#### Webhook

```json
 {
	"id": 2,
 	"timestamp": "2009-09-09T00:08:36.796-0500",
	"issue": { 
		"expand":"renderedFields,names,schema,transitions,operations,editmeta,changelog",
		"id":"99291",
		"self":"https://jira.atlassian.com/rest/api/2/issue/99291",
		"key":"JRA-20002",
		"fields":{
			"summary":"I feel the need for speed",
			"created":"2009-12-16T23:46:10.612-0600",
			"description":"Make the issue nav load 10x faster",
			"labels":["UI", "dialogue", "move"],
			"priority": "Minor"
		}
	},
	"user": {
		"self":"https://jira.atlassian.com/rest/api/2/user?username=brollins",
		"name":"brollins",
		"emailAddress":"bryansemail at atlassian dot com",
		"avatarUrls":{
			"16x16":"https://jira.atlassian.com/secure/useravatar?size=small&avatarId=10605",
			"48x48":"https://jira.atlassian.com/secure/useravatar?avatarId=10605"
		},
		"displayName":"Bryan Rollins [Atlassian]",
		"active" : "true"
	},
  	"changelog": {
        "items": [
            {
                "toString": "A new summary.",
                "to": null,
                "fromString": "What is going on here?????",
                "from": null,
                "fieldtype": "jira",
                "field": "summary"
            },
            {
                "toString": "New Feature",
                "to": "2",
                "fromString": "Improvement",
                "from": "4",
                "fieldtype": "jira",
                "field": "issuetype"
            }
        ],
		"id": 10124
	},
	"comment" : {
		"self":"https://jira.atlassian.com/rest/api/2/issue/10148/comment/252789",
		"id":"252789",
		"author":{
			"self":"https://jira.atlassian.com/rest/api/2/user?username=brollins",
			"name":"brollins",
			"emailAddress":"bryansemail@atlassian.com",
			"avatarUrls":{
				"16x16":"https://jira.atlassian.com/secure/useravatar?size=small&avatarId=10605",
				"48x48":"https://jira.atlassian.com/secure/useravatar?avatarId=10605"
			},
			"displayName":"Bryan Rollins [Atlassian]",
			"active":true
		},
		"body":"Just in time for AtlasCamp!",
		"updateAuthor":{
			"self":"https://jira.atlassian.com/rest/api/2/user?username=brollins",
			"name":"brollins",
			"emailAddress":"brollins@atlassian.com",
			"avatarUrls":{
				"16x16":"https://jira.atlassian.com/secure/useravatar?size=small&avatarId=10605",
				"48x48":"https://jira.atlassian.com/secure/useravatar?avatarId=10605"
			},
			"displayName":"Bryan Rollins [Atlassian]",
			"active":true
		},
		"created":"2011-06-07T10:31:26.805-0500",
		"updated":"2011-06-07T10:31:26.805-0500"
	},  
	"timestamp": "2011-06-07T10:31:26.805-0500",
    "webhookEvent": "jira:issue_updated"
}
```