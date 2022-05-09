---
description: 쿠버네티스 기반 워크플로우 툴
---

# Argo workflow

## 설치법

```bash
curl -sSL -o /usr/local/bin/argo https://github.com/argoproj/argo/releases/download/v3.3.2/argo-linux-amd64
chmod +x /usr/local/bin/argo
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-cd/master/manifests/namespace-install.yaml
```



## 기본 용어

* &#x20;Workflow: 하나 또는 그 이상의 template을 실행한 것. 별개로 이름을 가짐.
* Template: 하나의 step 또는 여러 step들의 집합
*
