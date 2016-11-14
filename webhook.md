# Webhook
- 리얼타임 통신을 위한 방법 중 하나
- 이벤트 기반 동작
- Web Callback, HTTP push API 등 다양한 이름으로 불림
- 보통 POST request를 통해서 콜백 데이터 전달

## Consuming a Webhook
- Webhook을 받기 위해서 Webhook provider에게 URL을 제공
- 일반적으로 json, xml 형식의 POST Request가 전달됨

## Debugging a Webhook
- 기본적으로 비동기 동작이므로 별도의 도구 사용
1. RequestBin으로 리퀘스트 수집
2. Postman, cURL 등으로 리퀘스트 테스트 전송
3. ngrok로 로컬에서 테스트
4. Runscope로 전체적인 플로우 감시

## Securing a Webhook
- webhook의 URL은 기본적으로 공개되어있기 때문에 가짜 데이터를 전송하는 것을 막을 필요가 있음
1. TLS Connections
2. URL에 auth Token 추가
3. Basic Auth 제공 https://en.wikipedia.org/wiki/Basic_access_authentication
4. 모든 Request마다 서명하여 signature를 만들고, 클라이언트 단에서 검증
