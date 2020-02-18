# Knative

knative는 주로 빌드, 배포에 관련된 쿠버네티스 미들웨어 확장 컴포넌트. 

## 컴포넌트 종류

1. Build: source에서 컨테이너까지 각종 빌드에 관련된 컴포넌트
2. Eventing: 이벤트를 관리하고 전달해주는 컴포넌트
3. Serving: 스케일링과 관련된 컴포넌트

## Build

Knative Build는 operator pattern으로 동작. Build라는 커스텀 리소스를 생성하면 그 리소스의 spec을 가지고 Pod을 생성하여 build작업을 수행. 

하나의 Build는 여러 steps를 가지며 각 steps는 하나의 Builder를 가짐. 여기서 각 step 하나 하나는 실제로 pod이 생성될 때 하나의 init-container로 생성됨.

Builder는 컨테이너 이미지. 하나의 step 전용으로 만들어도 되고 범용으로 만들어도 되고 원하는대로 자유롭게 작성 가능.

BuildTemplate, ClusterBuildTemplate 쓰면 반복적으로 사용되는 부분을 추출해서 사용 가능

source는 깃 저장소, 구글 클라우스 스토리지, 임의의 컨테이너 이미지 등이 될 수 있음.

Secret, ServiceAccount를 사용해서 repository 등에 대한 authentication 수행

nop, build-step-credential-initializer, build-step-git-soruce 등 초기화작업을 위한 init-container를 먼저 모두 수행한 다음 사용자의 step을 init-container로 만들어서 수행

### 장점

* jenkins같은 기존 CI 툴에 비해서 상대적으로 가벼움
* kubernetes native
* credential, scm checkout과 같은 필수 기능 자체 탑재
* 모든 init-container가 `/builder/home` , `/workspace` 를 공유하기 때문에 Builder 작성 용이
* 사용하는 컨테이너 이미지들을 자체적으로 caching

### 단점

* argo workflow에 비해서 상대적으로 부족한 편의 기능
* 하나의 `Build` 에 source 한개만 허용
* 하나의 `Build` 에 template 한개만 허용

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

### 사용 예시

#### 1. secret, service account, cluster role binding 생성

private registry를 쓴다면 annotation에 host를 넣고 필요한 인증정보를 secret으로 생성

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: knative-secrets-oss
  namespace: ncc-bd  
  annotations:
    build.knative.dev/git-0: oss.navercorp.com
type: kubernetes.io/ssh-auth
data:
  ssh-privatekey: <deploy key>
  known_hosts: <ssh known hosts>

---

apiVersion: v1
kind: Secret
metadata:
  name: knative-secrets-registry
  namespace: ncc-bd  
  annotations:
    build.knative.dev/docker-0: registry.navercorp.com
type: kubernetes.io/basic-auth
stringData:
  username: <user name>
  password: <user password>

---

apiVersion: v1
kind: ServiceAccount
metadata: 
  name: knative-service-account
  namespace: ncc-bd
secrets:
  - name: knative-secrets-oss
  - name: knative-secrets-registry

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: knative-service-account
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: knative-service-account
  namespace: ncc-bd
```

#### 2. BuildTemplate 생성

바로 Build를 만드는 것 보다 BuildTemplate을 만들고 Build에서는 필요한 패러미터만 채워주는 형식으로 만드는 편이 유리한듯.

아래 yaml은 전체 클러스터에서 사용할 BuildTemplate이므로 ClusterBuildTemplate으로 생성

```yaml
apiVersion: build.knative.dev/v1alpha1
kind: ClusterBuildTemplate
metadata:
  name: init-namespace
spec:
  parameters:
  - name: NAMESPACE
    description: The name of target namespace
  - name: PASTA_SERVICE_ID
    description: The id of target pasta service
  - name: CEPH_CLASSNAME
    description: rbd-ssd | rbd-hdd
    default: rbd-hdd
  - name: CEPH_STORAGE
    default: 300Gi
  - name: PROMETHEUS_REPLICA_NUM
    description: number of pods
    default: "1"
  - name: OPENSTACK_ENVIRONMENT
    default: dev
  - name: GRAFANA_API_URL
  - name: GRAFANA_AUTH_TOKEN
  - name: GRAFANA_REQUEST_BODY

  steps:
    - name: pre-install
      image: base.registry.navercorp.com/ubuntu:18.04
      command: ["/bin/bash", "-c"]
      args: 
        - "curl -X POST -d '{\"openstack_environment\":\"${OPENSTACK_ENVIRONMENT}\", \"projectid\":\"${PASTA_SERVICE_ID}\", \"gigabytes\":\"+1000\",\"volumes\":\"+10\"}' -H 'Content-Type: application/json' -H 'Authorization: Basic MTM1ZjMwNTktOWI2ZC00ZmM3LWIwZGEtYmU1YTY2NzY5OWVmOjl1MmlZWXNHdVZJamExNUZhYjdvZ3REZWZEcWhESHRsaWdPelNHZGl4NjdGOVp6MFYwdE1RUGtVaHlreXU4VVg=' 'https://lambda-real.navercorp.com/api/v1/namespaces/operations/actions/operations/set_quota?blocking=true&result=true';"
    - name: install-prometheus
      image: registry.navercorp.com/ncc/ncc:v0.1.1
      workingDir: "/workspace/incubator/ncc-monitoring"
      command: ["/bin/bash", "-c"]
      args: 
        - ncc cluster use-in-cluster ${NAMESPACE};
          kubectl delete configmap -n ${NAMESPACE} container-dashboards --ignore-not-found=true;
          kubectl create configmap -n ${NAMESPACE} --from-file=grafana-templates/Containers_Detail.json --from-file=grafana-templates/Containers.json --from-file=grafana-templates/Deployments.json --from-file=grafana-templates/L4_LoadBalancer.json --from-file=grafana-templates/L7_Traefik.json --from-file=grafana-templates/Pod_Autoscaler.json container-dashboards;
          ncc helm list --namespace=${NAMESPACE};
          ncc helm install --namespace=${NAMESPACE} . --name=ncc-monitoring --replace --set server.namespace=${NAMESPACE} --set pasta.serviceid=${PASTA_SERVICE_ID} --set ceph.storage=${CEPH_STORAGE} --set server.replicas=${PROMETHEUS_REPLICA_NUM} --set ceph.classname=${CEPH_CLASSNAME}
    - name: install-grafana
      image: registry.navercorp.com/ncc/builds/grafana-builder:v0.0.3
      args: ["grafana-builder", "-u", "${GRAFANA_API_URL}", "-t", "${GRAFANA_AUTH_TOKEN}", "-b", "${GRAFANA_REQUEST_BODY}"]
    - name: post-install-grafana
      image: registry.navercorp.com/ncc/ncc:v0.1.1
      command: ["/bin/bash", "-c"]
      args:
        - ls -al;
          kubectl annotate --overwrite=true namespace ${NAMESPACE} ncc.navercorp.com/grafana-id=$(cat /workspace/grafana-id);
          kubectl annotate --overwrite=true namespace ${NAMESPACE} ncc.navercorp.com/grafana-url=$(cat /workspace/grafana-url);

```

#### 3. Build 생성

```yaml
apiVersion: build.knative.dev/v1alpha1
kind: Build
metadata:
  name: namespace-initialization
  namespace: ncc-bd
spec:
  serviceAccountName: knative-service-account
  source:
    git:
      revision: master
      url: <git url>
  template:
    name: init-namespace
    kind: ClusterBuildTemplate # ClusterBuildTemplate인 경우에만 필요
    arguments:
      - name: NAMESPACE
        value: <value>
      - name: PASTA_SERVICE_ID
        value: <value>
      - name: CEPH_CLASSNAME
        value: <value>
      - name: GRAFANA_API_URL
        value: <value>
      - name: GRAFANA_AUTH_TOKEN
        value: <value>
      - name: GRAFANA_REQUEST_BODY
        value: <value>
  timeout: 10m0s
```



