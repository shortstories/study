# Kubernetes namespace의 phase가 Terminating에서 멈춰있을 때

### 원인, 해결책 분석

1. custom finalizer가 정리되지 않는 케이스
   1.  Namespace `.spec.finalizers` 내용을 확인했을 때 배열에 `kubernetes` 를 제외한 다른 element가 남아있다면 어떤 extension controller가 namespace를 초기화는 하고 정리는 못한 케이스.
   2. 이 케이스는 우선 현재 클러스터에 namespace finalizer를 붙이는 controller가 무엇인가를 먼저 파악하고 수정해야한다.
2. `kubernetes` finalizer가 정리되지 않는 케이스
   1. `kubernetes` 는 기본 finalizer로 namespace 내부의 모든 리소스가 정리되면 삭제된다. 그런데 이 finalizer가 삭제되지 않는다는 이야기는 리소스 정리 과정에서 에러가 발생했다는 이야기가 된다.
   2. 따라서 우선 namespace에 진짜 모든 리소스가 삭제된건지 확인해봐야한다. `kubectl get all` 의 경우 category all인 리소스들만 보여주므로 진짜로 모든 리소스를 보여주는게 아니다. `kubectl api-resources --namespaced=true -o name` 명령어로 체크해야될 모든 리소스의 이름을 확인할 수 있다. 확인 과정에서 삭제되지않고 있는 리소스가 있다면 그 리소스가 삭제되지 못하고 있는 이유를 확인해야한다. 보통은 `.metadata.finalizers` 에 원인이 있을 가능성이 높다.
   3.  위에서 확인해본 결과 모든 리소스가 삭제된게 맞다면 이번에는 admission webhook이나 extension api server 에서 에러가 발생했다는 이야기다. namespace가 삭제될 때 각 리소스를 삭제하기 위해서 보내는 요청은 delete가 아니라 delete-collection이므로 delete-collection에 대해서 제대로 처리했는지 다시 한번 확인해야한다.

### 사용한 스크립트

#### 모든 리소스 체크용 스크립트

```bash
#!/usr/bin/env bash
die() {
  echo "$*" 1>&2
  exit 1
}
need() {
  command -v "$1" &>/dev/null || die "Binary '$1' is missing but required"
}

need "kubectl"

echo "------------------------------------------------"
echo "get all resources in namespace..."
echo "------------------------------------------------"
FOUND=""
NOT_FOUND=""
ERROR=""
RESOURCES=$(kubectl api-resources --namespaced=true -o name)
for resource in $RESOURCES; do
  if [[ $resource =~ "metrics" ]]; then
    continue
  fi

  found=$(kubectl get "$resource" 2>&1)
  if [[ "$found" == "No resources found." ]]; then
    echo "$resource: $found"
    NOT_FOUND="$NOT_FOUND\n$resource"
  elif [[ $found =~ "Error" ]]; then
    echo "$resource: $found"
    ERROR="$ERROR\n$resource"
  else
    printf ">> %s:\n%s" "$resource" "$found"
    FOUND="$FOUND\n$resource"
  fi
done

echo "------------------------------------------------"
echo "summary"
echo "------------------------------------------------"
printf ">> found: %b\n" "$FOUND"
echo "------------------------------------------------"
printf ">> not found: %b\n" "$NOT_FOUND"
echo "------------------------------------------------"
printf ">> error: %b\n" "$ERROR"
```

#### namespace finalizers 강제로 비우는 스크립트

```bash
#!/usr/bin/env bash
set -eo pipefail

die() {
  echo "$*" 1>&2
  exit 1
}
need() {
  command -v "$1" &>/dev/null || die "Binary '$1' is missing but required"
}

need "jq"
need "curl"
need "kubectl"

TARGET_NAMEPSACE="$1"
TOKEN="$2"
API_SERVER=$(kubectl cluster-info | grep "Kubernetes master" | awk -F ' ' '{print $6}' | sed -r "s/\x1B\[([0-9]{1,2}(;[0-9]{1,2})?)?[mGK]//g")

test -n "$TARGET_NAMEPSACE" || die "Missing arguments: $0 <namespace> <token>"
test -n "$TOKEN" || die die "Missing arguments: $0 <namespace> <token>"
test -n "$API_SERVER" || die "failed to get kubernets api address"

echo "trying to remove finalizers of namespace '$TARGET_NAMEPSACE'..."
kubectl get namespace "$TARGET_NAMEPSACE" -o json | \
jq 'del(.spec.finalizers[] | select("kubernetes"))' | \
curl -k -X PUT --insecure "$API_SERVER/api/v1/namespaces/$TARGET_NAMEPSACE/finalize" \
  -H "Authorization: Bearer ${TOKEN}" \
  -H "Content-Type: application/json" \
  --data-binary @-
```

