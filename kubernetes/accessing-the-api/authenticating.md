# Authenticating

## Users

1. 쿠버네티스에서 관리하는 Service account
   1. 쿠버네티스 API를 사용해서 관리
   2. 특정 namespace에 제한됨
   3. API 서버에 의해 자동으로 생성될 수도 있고 직접 호출해서 생성할 수도 있음
   4. `Secrets` 와 연결되어있음. Secrets는 Pod에 마운트되서 클러스터 내부의 프로세스들이 쿠버네티스 API와 소통하는 것을 가능케 함
2. 일반적인 유저: 외부에서 관리되는 유저. 쿠버네티스는 일반 유저에 대해서 별도로 제공하는게 없음.

## Authentication strategies

인증 종류는 다음과 같음

1. client 인증서
2. 토큰
3. 인증 프록시
4. HTTP basic auth

거기에 필요한 것들은 보통 다음과 같음

1. Username
2. UID
3. Groups
4. Extra fields

여러가지 방법이 있는데 동시에 활성화시켜놓는 것도 가능. 그런데 보통 적어도 두가지는 활성화시켜놓는 편.

1. service account token
2. user authentication 중 하나

여러 모듈이 활성화되어있는 경우 가장 먼저 성공하는대로 통과함. 각 모듈의 순서는 보장하지 않음.

authencation에 성공한 모든 유저는 `system:authenticated` 그룹에 속하게 됨

다른 인증 프로토콜하고 연동하려면 authenticating proxy나 authentication webhook을 쓰면 됨

