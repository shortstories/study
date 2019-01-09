# Knative

knative는 주로 빌드, 배포에 관련된 쿠버네티스 미들웨어 확장 컴포넌트. 

## 컴포넌트 종류

1. Build: source에서 컨테이너까지 각종 빌드에 관련된 컴포넌트
2. Eventing: 이벤트를 관리하고 전달해주는 컴포넌트
3. Serving: 스케일링과 관련된 컴포넌트

## Build

Knative Build는 operator pattern으로 동작. Build라는 커스텀 리소스를 생성하면 그 리소스의 spec을 가지고 Pod을 생성하여 build작업을 수행. 

하나의 Build는 여러 steps를 가지며 각 steps는 자신의 Builder을 가짐

Builder은 컨테이너 이미지인데 하나의 step 전용으로 만들어도 되고 범용으로 만들어도 되고 아무튼 원하는대로 작성하면 됨

step에서 repository로 push하는 것도 가능

BuildTemplate를 쓰면 두루 쓰이는 부분을 미리 만들어두는게 가능

source는 깃 저장소, 구글 클라우스 스토리지, 임의의 컨테이너 이미지 등이 될 수 있음.

Secret, ServiceAccount를 사용해서 repository 등에 대한 authentication 수행

### 설치법

```bash
$ kubectl apply --filename https://raw.githubusercontent.com/knative/serving/v0.2.2/third_party/config/build/release.yaml
```

위 명령어를 실행하면 `knative-build` namespace를 생성하고 거기에 아래와 같은 리소스를 생성. 

1. **ETC**: cluster role, cluster role binding, serviceaccount, service, configmaps 등
2. **CRDs**: Build, BuildTemplate, ClusterBuildTemplate, Image
3. **Deployments and Services**: build-controller, build-webhook

### 컴포넌트

#### Build

* spec: template이나 steps가 적어도 하나는 있어야함
  * steps : 하나의 빌드 단계. 빌드용 이미지와 argument로 정의.
    * array
      * name
      * image
      * args
  * template: 빌드 템플릿을 사용하는 단계. 빌드 템플릿 이름과 빌드 템플릿에 정의된 argument 전달
    * name
    * arguments
  * source:  steps에 데이터를 전달하기 위해서 사용. source에 정의한 데이터는 `/workspace` 아래로 마운트됨. 
    * git
      * url
      * revision
    * gcs
    * custom
  * serviceAccountName: 인증을 위해서 사용. 만약에 별도로 정의하지 않으면 네임스페이스의 default service account 사용
  * volumes
  * timeout

#### BuildTemplate

* spec
  * parameters: step에서 사용할 수 있는 parameter 정의
    * array
      * name
      * description
      * default
  * steps: `Build` 의 steps와 동일. 차이점은 parameters에서 정의한 값을 `${NAME}` 으로 사용 가능

#### Builder

`Build` 또는 `BuildTemplate` 의 steps의 image에 들어가는 값

* Typical builders: 특정한 도구를 쓸 목적으로 만드는 빌더 이미지. entrypoint 또는 command에 해당 도구를 실행하는 명령어를 등록하고 args만 받아서 step 실행. 성공적으로 실행하면 zero status를 반환. 
* Atypical builders: 임의의 이미지를 빌더로 쓰는 케이스. command랑 args를 오버라이딩해서 사용.

빌더는 `/workspace` 를 기본 working directory로 가져야 함. 이 디렉토리는 knative build의 `source` 에 의해서 채워지고 `steps` 사이에서 공유됨

`/builder/home` 은 `$HOME` 으로 노출되어야 함.

service account에 붙어있는 credentials를 git 또는 docker credentials로 바꿔서 노출해야됨.

### 인증

kubernetes secret type 두 가지 지원

* kubernetes.io/basic-auth 
* kubernetes.io/ssh-auth

이 두 가지 type의 secret를 service account에 붙이고 그 service account name으로 인증 절차 수행

secret에 annotation `build.knative.dev/<git or docker>-<index>: <host name>` 을 붙여서 어디 언제 사용할지 결정. 

ssh-auth를 쓸 경우 주의할 부분이 몇가지 있는데 문서에서는

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ssh-key
  annotations:
    build.knative.dev/git-0: https://github.com # Described below
type: kubernetes.io/ssh-auth
data:
  ssh-privatekey: <base64 encoded>
  # This is non-standard, but its use is encouraged to make this more secure.
  known_hosts: <base64 encoded>
```

위와 같이 어노테이션의 호스트에 https가 붙어있는데 이걸 붙이면 정상작동하지 않음.

그리고 `Build` 의 source 필드를 넣을 때 url도 https가 아니라 `git@<registry url>:<organization>/<path>.git` 형식으로 넣어야 함.

