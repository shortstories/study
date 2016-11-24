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

### DNS Discovery