# Etcd

- 일관성을 유지하는 분산 Key-Value 저장소
- Go로 작성됨
- CoreOS 용으로 작성됨

## 설치

### 릴리즈 바이너리 받기

- https://github.com/coreos/etcd/releases/ 링크에서 필요한 버전을 찾아 다운로드

### 직접 빌드

1. go 1.6 이상 설치
2. $GOPATH 환경변수 제대로 지정되어있는지 확인
3. https://github.com/coreos/etcd.git master 브랜치 clone 후 들어가서 build 스크립트 실행

## 실행

### 서버 실행

``` shell
etcd
```

- 기본적으로 2379 포트에서 클라이언트 통신을 받고 2380 포트에서 서버간 통신을 진행

### 클라이언트 접속

``` shell
etcdctl set test "hello world"
etcdctl get test
```

## 클러스터링

### Static

- `--inital-cluster` 옵션에 모든 클러스터 노드의 이름과 peer-url을 넣어서 클러스터를 시작하는 방법
- `--initial-cluster-token` 옵션을 통해 같은 설정을 가진 고유한 클러스터들을 구분 가능하게 생성할 수 있음

### etcd Discovery

- `--discovery` 옵션에 특정 etcd 클러스터나 노드의 URL을 넣어서 그 URL에서 얻은 정보를 바탕으로 클러스터를 시작하는 방법
- 처음에 시작할 때 지정한 노드 갯수를 충족하지 않으면 전체 클러스터가 동작하지 않고, 노드 갯수를 넘어가게 추가하면 추가된 노드는 proxy 노드로 동작
- reconfiguration을 통해서 클러스터에 새로운 노드를 추가하거나 제거했을 경우엔 discovery에 적용되지 않음

#### 로컬 테스트 클러스터 구축

``` bash
# Discovery 생성
./etcd --name discovery >discovery.log 2>&1 &

## 노드 갯수 지정
curl -X PUT http://localhost:2379/v2/keys/_etcd/registry/test/_config/size -d value=5

## discovery 정보 확인
curl -X GET http://localhost:2379/v2/keys/_etcd/registry/test

# 새로운 클러스터 생성
# 노드 갯수를 충족하지 않으면 동작하지 않음
# 노드 갯수를 넘어가면 proxy 노드로 추가됨

## 1번 노드
./etcd --name test1 --initial-advertise-peer-urls http://localhost:12380 \
--listen-peer-urls http://localhost:12380 \
--listen-client-urls http://localhost:12379 \
--advertise-client-urls http://localhost:12379 \
--discovery http://localhost:2379/v2/keys/_etcd/registry/test >test1.log 2>&1 &

## 2번 노드
./etcd --name test2 --initial-advertise-peer-urls http://localhost:22380 \
--listen-peer-urls http://localhost:22380 \
--listen-client-urls http://localhost:22379 \
--advertise-client-urls http://localhost:22379 \
--discovery http://localhost:2379/v2/keys/_etcd/registry/test >test2.log 2>&1 &

## 3번 노드
./etcd --name test3 --initial-advertise-peer-urls http://localhost:32380 \
--listen-peer-urls http://localhost:32380 \
--listen-client-urls http://localhost:32379 \
--advertise-client-urls http://localhost:32379 \
--discovery http://localhost:2379/v2/keys/_etcd/registry/test >test3.log 2>&1 &

## 4번 노드
./etcd --name test4 --initial-advertise-peer-urls http://localhost:42380 \
--listen-peer-urls http://localhost:42380 \
--listen-client-urls http://localhost:42379 \
--advertise-client-urls http://localhost:42379 \
--discovery http://localhost:2379/v2/keys/_etcd/registry/test >test4.log 2>&1 &

## 5번 노드
./etcd --name test5 --initial-advertise-peer-urls http://localhost:52380 \
--listen-peer-urls http://localhost:52380 \
--listen-client-urls http://localhost:52379 \
--advertise-client-urls http://localhost:52379 \
--discovery http://localhost:2379/v2/keys/_etcd/registry/test >test5.log 2>&1 &

## 현재 클러스터 확인
./etcdctl --endpoint http://localhost:12379 member list
```

### DNS Discovery

- DNS SRV record를 사용하여 discovery
- `-discovery-srv` 옵션에 URL 등록

### Runtime Reconfiguration

- 클라이언트 명령어를 사용해서 클러스터에 노드의 추가 및 삭제 가능
- 이 방법을 통해 추가하거나 삭제한 노드는 discovery 사용 불가

``` bash
# 노드 추가
## discovery에 반영되지 않음
## etcdctl member add 이후 가능
./etcdctl --endpoint http://localhost:12379 member add test6 http://localhost:62380

## discovery를 쓸 수 없고 대신에 --initial-cluster 필요
## 6번 노드
./etcd --name test6 --initial-advertise-peer-urls http://localhost:62380 \
--listen-peer-urls http://localhost:62380 \
--listen-client-urls http://localhost:62379 \
--advertise-client-urls http://localhost:62379 \
--initial-cluster-token test-cluster \
--initial-cluster test6=http://localhost:62380,test1=http://localhost:12380,test5=http://localhost:52380,test3=http://localhost:32380,test4=http://localhost:42380,test2=http://localhost:22380 \
--initial-cluster-state existing >test6.log 2>&1 &
```