---
description: 쿠버네티스 기반 워크플로우 툴
---

# Argo workflow

## 설치법

```bash
$ curl -sSL -o /usr/local/bin/argo <https://github.com/argoproj/argo/releases/download/v3.3.2/argo-linux-amd64>
$ chmod +x /usr/local/bin/argo
$ kubectl apply -f <https://raw.githubusercontent.com/argoproj/argo-cd/master/manifests/namespace-install.yaml>
```

## 기본 용어

* Workflow: 쿠버네티스 리소스. 하나 또는 그 이상의 template을 실행한 것. 별개로 이름을 가짐.
* Template: 하나의 step 또는 여러 step들의 집합 또는 DAG
* Step: workflow의 하나의 동작 단계. 보통 input을 넣어 컨테이너를 실행한 다음 output을 획득하는 방식으로 동작
* Entrypoint: workflow의 첫번째 step
* Node: step과 동의어
* Directed Acyclic Graph (DAG): 여러 step(==node)과 dependency(==edge)의 그래프.
* Workflow Template: 쿠버네티스 리소스. namespace에 재사용 가능한 workflow spec을 정의해둔 것.
* Cluster Workflow Template: 쿠버네티스 리소스. workflow template과 기본적으로 동일하지만 cluster 전역에서 사용가능.
* Inputs: step을 실행할 때 전달되는 parameters, artifacts
* Outputs: step의 실행 결과로 얻을 수 있는 parameters, artifacts
* Parameters: object, string, boolean, array
* Artifacts: 컨테이너에 저장된 파일
* Artifact Repository: artifact들이 저장되는 장소
* Executor: 컨테이너를 실행하기 위한 수단. Docker 또는 PNS와 같은 것들.
* Workflow Service Account: workflow를 실행할 때 사용되는 service account

## 기본 개념

1. Workflow
   1. 이미 실행된 workflow를 정의하고 있음
   2. workflow의 상태에 대한 정보를 가지고 있음
   3. 따라서, workflow는 static이 아닌 “live” object로 간주해야하고 어떤 정의에 대한 instance로 간주해야함.
2. Workflow.spec
   1. templates:
      1. container: 가장 일반적. 주어진 이미지로 컨테이너 만들어서 명령어 실행
      2. script: container와 대부분 동일하지만 `source` 필드를 사용할 수 있음. source 필드는 즉시 작성해서 쓸 수 있는 script라고 보면 됨. source 필드의 결과물은 `{{tasks.<NAME>.outputs.result}}` 또는 `{{steps.<NAME>.outputs.result}}` 중 하나에 저장됨.
      3. resource: 클러스터 리소스에 접근하는데 사용됨. get, create, apply, delete, replace, patch 등 일반적인 kubectl와 동일하게 사용 가능
      4. suspend: `argo resume` 을 실행할 때까지 또는 주어진 시간만큼 대기
   2. entrypoint: 가장 먼저 실행되는 template
   3. template invocators
      1.  steps:

          1. template의 리스트. 이중 배열이라고 보면 되는데 바깥 배열의 경우 순차로 하나하나 실행하는 반면 내부 배열은 병렬로 실행함. 따라서 아래 예제의 경우 step1 → step2a, step2b 와 같은 순서로 실행됨

          ```yaml
          - name: hello-hello-hello
              steps:
              - - name: step1
                  template: prepare-data
              - - name: step2a
                  template: run-data-first-half
                - name: step2b
                  template: run-data-second-half
          ```

          1. 필요하다면 `when` 과 같은 필드를 사용해서 step의 실행조건을 넣을 수도 있음.
      2.  dag:

          1. template들을 dependency graph로 구성할 수 있음. 각각의 template는 task가 됨. dependencies가 존재하지 않는 task는 즉시 실행됨. dependencies에는 하나 또는 여러개의 task를 넣을 수 있는데 이 dependency task가 모두 완료되어야만 실행됨. 따라서 아래 예제의 경우 A → B,C → D 순서로 실행됨.

          ```yaml
          - name: diamond
              dag:
                tasks:
                - name: A
                  template: echo
                - name: B
                  dependencies: [A]
                  template: echo
                - name: C
                  dependencies: [A]
                  template: echo
                - name: D
                  dependencies: [B, C]
                  template: echo
          ```

## Variable

workflow spec의 어떤 필드들은 미리 정의된 variable을 사용할 수 있음.

사용법은 simple, expression 두가지가 있음.

### Template Tag Kinds

#### Simple

`{{ workflow.name }}`

해당 이름을 가진 tag의 값으로 대치됨.

#### Expression

`{{= workflow.name }}`

v3.1버전부터 사용 가능

tag의 내용을 연산한 다음 결과물로 대치함.

만약 parameter 이름이나 step 이름에 하이픈이 존재할 경우 에러가 발생하므로 index를 사용하는 방식으로 피해야함 ex) `inputs.parameters['my-param']` `steps['my-step'].outputs.result`

### Inputs

#### Parameter

workflow의 argument는 entrypoint template에 전달됨

template의 inputs는 template caller (steps, dag 등)에 의해 전달됨

DAG의 경우 dependency task에서 output을 받아 input으로 사용하고 싶을 수 있음. 이런 경우에는 artifact, parameter 두가지 방식이 존재. 차이점은 parameters는 `value` 필드를 쓰고 artifact는 `from` 필드를 쓴다는 것.
