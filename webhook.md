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
- RequestBin으로 리퀘스트 수집
- Postman, cURL 등으로 리퀘스트 테스트 전송
- ngrok로 로컬에서 테스트
- Runscope로 전체적인 플로우 감시

