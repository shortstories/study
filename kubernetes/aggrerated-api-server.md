---
description: >-
  https://github.com/kubernetes-incubator/apiserver-builder-alpha/tree/master/docs/concepts
---

# Aggrerated API server

## 인증, 권한

delegated authentication and authorization 패턴으로 되어있음

#### 인증서

CA인증서는 신뢰를 위임하기 위해서 사용. CA를 신뢰한다는 것은 이 CA private key로 서명한 어떤 인증서든 신뢰한다는 것을 의미. 이것은 다시 말해 어떤 인증서의 signature를 CA 인증서의 public key로 복호화할 수 있고 그 값이 인증서 내용을 hashing 거친 결과물과 동일하다면 신뢰할 수 있는 인증서라는 의미.

만약에 인증서가 특정한 CA로 서명되지 않았다면 이것을 self-signed 인증서라고 부름. self-signed 인증서는 전적으로 신뢰하던지 신뢰하지 않던지 바로 결정해야 함 \(그 반대는 CA를 통해서 신뢰하게 되는 경우\). 일반적으로 k8s에서 사용하는 client CA 인증서들은 이 신뢰성 체인의 'root'에 위치하기 때문에 self-signed 인증서임. client들은 본질적으로 CA를 신뢰해야 함.

API server는 3개의 서로 다른 CA를 필요로 함

1. serving CA: serving 인증서에 서명하기 위해 필요한 CA. serving 인증서는 HTTPS 통신을 하는데 사용됨. 기본 kube api server랑 같은 CA를 쓸 수도 있고 서로 다른 CA를 사용할 수도 있음. addon API server에 serving 인증서를 전달하지 않으면 기본적으로 self-signed 인증서를 생성해서 사용하기 때문에 CA는 optional함. 그러나 real 환경에서는 믿을 수 있는 CA를 제공하는게 좀 더 좋은 방법.
2. client CA: client 인증서에 서명하기 위해 필요한 CA. 그리고 client가 인증서 방식으로 인증을 시도할 때 검증하기 위해서 사용함. 마찬가지로 기본 kube api server과 같은 CA를 쓸 수도 있고 서로 다른 CA를 쓸 수도 있음. 같은 CA를 쓰는게 두 api server 사이에 동일한 인증 동작을 보증하므로 가급적 같이 쓰는게 좋음.
3. RequestHeader client CA: proxy client 인증서에 서명하기 위해서 필요한 CA. 이 인증서를 사용한 client는 또 다른 identity로 보일 수 있도록 효과적으로 증명된다. API aggregator을 쓰려면 aggregator의 proxy client 인증서와 반드시 같은 CA를 써야만 한다.

#### Serving 인증서

HTTPS 통신을 위해 사용하는 인증서. 기본적으로 addon API server는 self-signed 인증서 한 세트를 생성해서 사용함. 그러나 이 경우 client가 이 인증서를 신뢰할 방법이 없기 때문에 production 환경이나 API aggregator을 쓰는 경우에는 별도의 CA 인증서들을 직접 만든 다음 이것을 통해 서명한 인증서를 사용해야 함.

### Authentication

`--authentication-skip-lookup` flag로 인증 과정을 생략할 수 있음. 그러나 `--client-ca-file` 옵션으로 client CA 인증서가 전달되었다면 client 인증서방식 인증 절차는 수행하게 됨.

#### Client 인증서

api로 요청할 때 인증서를 통해서 인증하는 방식. 이 때 주어진 인증서가 API server의 client CA로 서명되었을 때 인증 통과 시킴.

client CA를 제공하는 방식은 다음 두 가지가 있음

1. `kube-system` namespace에 있는 `extension-apiserver-authentication` configMap의 data를 사용하는 방식. 기본 kube api server가 해당 configMap을 생성하기 때문에 이 방식을 사용하면 기본 kube api server와 동일한 CA로 인증하게 됨.
2. API server를 실행할 때 `--client-ca-file` flag를 지정하는 방식.

#### Delegated Token

api request header의 `Authorization: Bearer $TOKEN` 부분을 보고 인증하는 방식. 이 방식을 사용할 때 addon API server는 토큰만 따로 추출한 다음 다른 API server와 `TokenReview` 를 통해 대조해봄. 보통은 기본 kube api server가 이 역할을 수행. 덕분에 사용자는 addon API server와 기본 kube api server에서 똑같이 인증을 받을 수 있게 됨.

이 인증 방식은 webhook token authentication 방식과 굉장히 유사함. 다만 차이점이 있다면 외부의 webhook server에게 물어보는 대신 이미 존재하는 k8s cluster kube api server에게 물어본다는 점임.

addon API server는 기본 kube api server에 접근하기 위해서 보통은 pod에 주입된 serviceAccount 정보를 사용함. 따라서 addon API server가 클러스터 외부에 있을 경우에는 `--authentication-kubeconfig` flag를 사용해서 접근 정보 및 인증 정보를 제공해야 함.

#### RequestHeader

API server의 proxy에서 오는 연결을 인증하기 위해 사용. 이 때 proxy는 자체적으로 client를 이미 인증 했어야 함. client 인증서 방식이랑 비슷한 부분과 다른 부분이 있음. 우선 비슷한 부분은 proxy의 인증서가 RequestHeader client CA로 서명되었는지를 검사한다는 것임. 다른 부분은 proxy에서 설정한 header들을 읽어서 마치 다른 특정 유저인 것 처럼 동작하게 만들 수 있다는 것. 덕분에 addon API server가 API server aggregator로 동작할 수 있게 됨.

기본적으로 addon API server는 client CA와 마찬가지로 `kube-system` namespace에 있는 `extension-apiserver-authentication` configMap에서 request header CA를 가져와서 사용. 만약에 저기에 없을 경우 직접 `--requestheader-client-ca-file` 옵션으로 인증서를 전달해줘야 함. 그리고 모든 API server의 proxy들은 이 CA로 서명된 client 인증서를 사용해야함.

### Authorization

addon API server는 authorization은 기본 kube api server에 전적으로 위임함. 이 때 `SubjectAccessReview`를 사용. token authentication과 마찬가지로 기본 kube api server에 접근하기 위해서 pod에 주입된 serviceAccount 정보를 사용하거나 `--authorization-kubeconfig` flag로 제공된 파일을 사용.

### RBAC Rules

delegated authentication and authorization을 사용하기 위해서는 두 가지 RBAC세팅이 필요하다.

1. `system:auth-delegator` cluster role을 addon api server에 cluster role binding 할당. 이 cluster role은 `tokenReview`, `subjectAccessReview` 에 대한 create verb 권한을 부여.
2. `kube-system` namespace 아래에 있는 `extension-apiserver-authentication-reader` role을 addon api server에 role binding 할당. 이 role은 `extension-apiserver-authentication` configMap에 대한 get verb 권한을 부여.

## Aggregation

API server를 만들어서 직접 붙이는 방법도 있지만 이 경우 굉장히 불편한 부분들이 많다. 당장 endpoint를 여러 개 각각 관리해야되는 시점에서 골치아파진다. 이 때 k8s의 API server aggregator을 사용하게 되면 이러한 문제가 해결된다. k8s cluster 안에 있는 여러 API를 마치 하나의 API server처럼 활용할 수 있게 도와주는 것이다. 따라서 k8s api를 사용하는 각종 client, 대표적으로 kubectl 등에서 특별히 세팅하지 않고도 새로운 API server의 API들을 자연스럽게 사용할 수 있게 된다.

### Aggregator 활성화

aggregator기능은 k8s 1.7버전부터서 기본 k8s API server에 통합되었다.  그 이전 버전에서는 별도의 pod으로 띄워야만 한다.  1.7버전부터는 API 서버의 secure port에서만 제공된다.

1.7부터는 aggregator를 쓰려면 몇몇 인증서를 만들고 사용해야한다. 이 때 필요한게 위에서 언급한 RequestHeader CA 인증서이다. addon API server는 이 RequestHeader CA로 client 인증서를 서명해서 사용한다. 마찬가지로 기본 k8s API server도 이 RequestHeader CA로 서명된 인증서와 키를 `--proxy-client-cert-file` , `--proxy-client-key-file` 이라는 플래그로 전달받아서 사용한다.

### kubectl and Discovery

`kubectl get <resource name>` 을 하면 어떻게 될까? 사실 바로 그 순간엔 그 리소스가 있는지 없는지 k8s API server도 모른다. 어디까지나 discovery과정을 거쳐서 알게 되는 것이다. 위의 경우 kubectl은 다음과 같은 과정을 거쳐서 리소스를 확인하게 된다.

1. `https://$API_SERVER_DOMAIN/apis` 를 호출하여 모든 API groups, versions에 대한 정보를 획득.
2. 각 groups, versions를 순회하면서 `https://$API_SERVER_DOMAIN/apis/$GROUP/$VERSION` 를 호출하여 리소스 목록을 조회
3. 리소스 상세정보를 통해 명령어에 일치하는 API를 생성하여 호출

각 과정은 kubectl 명령어에 -v=6 또는 -v=9 플래그를 붙여서 stdout에서 확인할 수 있다.

### Registering API

aggregator에 addon API server를 등록하기 위해서는 aggregator가 제공하고 있는 `apiregistration.k8s.io` 의 `APIService` 를 사용하면 된다.

각 `APIService` 는 하나의 k8s group-version에 매핑된다. 따라서 같은 group의 API라도 version이 다르다면 별개의 `APIService` 를 생성해서 등록해야 한다.

제일 중요한 필드는 `caBundle` 인데 aggregator과 addon API server 사이에서 TLS 통신을 하기 위해 필요하다. addon API server는 여기에 등록된 CA로 서명한 인증서를 사용한다.

### Proxying

aggregator는 일종의 proxy처럼 동작한다. aggregator는 근본적으로 discovery 단계에서 사용 가능한 API group과 version들을 넘겨주는 것밖에 안 한다. 그 외의 모든 요청들은 실제 API server에 넘겨서 처리한다. 그 순서는 아래와 같다.

1. 기본 k8s API server에서 인증 및 권한 체크를 수행한다.
2. aggregator가 인증 정보를  추출해서 http header\(`X-Remote-User`\)에다가 기록한다.
3. aggregator가 적절한 addon API server를 찾아 TLS 요청을 보낸다. \(이 과정에서 `APIService` 의 caBundle 사용\)
4. aggregator는 이 과정에서 proxy client certificate를 사용해서 addon API server에 인증한다.
5. addon API server는 request를 받아 proxy client certificate를 인증하고 `X-Remote-User` header의 값을 바탕으로 권한을 체크한다.
6. addon API server가 결과를 돌려주면 aggregator가 그대로 반환한다.

이 때 인증 및 권한 절차는 기본 k8s API server에서 수행한 것을 그대로 사용하므로 사용자는 특별한 추가조치 없이 addon API server를 사용할 수 있게 된다.

 

