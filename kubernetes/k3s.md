# k3s 설치 및 삽질

## 설치

설치는 옵션을 env로 세팅하고 스크립트 실행하면 됨.

우리 환경에서는 k8s가 1.16이므로 k3s도 1.16버전을 골라서 설치

```bash
INSTALL_K3S_VERSION='v1.16.15+k3s1'
**curl -sfL <https://get.k3s.io> | sh -**
```

## 삽질

`kubectl get nodes` 를 실행했을 때 k8s api에서는 문제없이 리턴이 돌아오는 반면 node가 하나도 나오지 않는 문제가 발생함.

확인해보니 k3s server는 정상적으로 떴으나 k3s agent가 올라오지 못한 경우 이렇게 됨.

로그를 확인해보니 아래와 같은 에러로그가 지속적으로 발생하고 있었음.

```bash
Waiting for containerd startup: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.RuntimeService
```

처음엔 containerd의 버전과 k3s의 버전이 맞지 않아서 그런 것 같았지만 확인해보니 k3s에서는 containerd를 내장하고 있고 이걸 새로 띄워서 사용하므로 불가능한 시나리오임.

좀 더 찾아보니 로컬에 설치된 docker의 containerd를 쓰는 옵션이 있어 k3s에 `—docker`옵션을 넣어 마침내 node가 Ready상태로 올라오게 만들 수 있었음.

```bash
$ sudo cat /etc/systemd/system/k3s.service
# ...
ExecStart=/usr/local/bin/k3s \\
    server --docker \\

$ sudo systemctl daemon-reload && sudo systemctl restart k3s
# ...

$ kubectl get node
NAME                  STATUS   ROLES    AGE   VERSION
dev-ohchang-k8s-ncl   Ready    master   46m   v1.16.15+k3s1
```

해피엔딩인줄 알았는데.. pod돌리다보니까 갑자기 buildkit pod에서 아래와 같은 에러가 발생함.

```bash
buildkitd: /builder/home/.local/share/buildkit/runc-overlayfs/snapshots does not support d_type. If the backing filesystem is xfs, please reformat with ftype=1 to enable d_type support
```

확인해보니까 내 서버의 파일시스템이 xfs인데.. 파일시스템이 xfs이면 ftype이라는 옵션이 1이어야한다는 것. 근데 centos 구버전에서는 이 옵션의 기본값이 0으로 설정됨.

```bash
[irteamsu@dev-ohchang-k8s-ncl ~]$ sudo /sbin/xfs_info /
meta-data=/dev/xvda1             isize=256    agcount=4, agsize=655360 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=0        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=2621440, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=0 <<<<<<<<< 이 부분
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

해결 방법은 ftype을 1로 가지는 새로운 파티션을 생성하고 fypte=1 옵션을 주고 포맷(`mkfs.xfs -ftype=1 $DEVICE_NAME`), mount 한 다음 관련 path들을 모두 새로운 파티션쪽으로 옮기는 것.

또는 xfs가 아닌 다른 파일시스템을 mount하고 사용해도 됨.

나의 경우 다행히도 ext4로 포맷된 디바이스가 있어서 그쪽으로 모두 옮기기로 결정함.

containerd의 경우 `/etc/containerd/config.toml` 파일에 `root` 옵션을 추가

docker의 경우 `/etc/docker/daemon.json` 파일에 `"graph"` 옵션을 추가

k3s의 경우 k3s의 data 와 kubelet root를 모두 설정해야하는데 아래와 같이 args로 설정.

```bash
 $ sudo cat /etc/systemd/system/k3s.service
# ...
ExecStart=/usr/local/bin/k3s \\
    server --docker --data-dir=/home1/irteamsu/k3s-home --kubelet-arg=root-dir=/home1/irteamsu/kubelet-home \\
```

이후 systemctl로 서비스 재시작하면 적용 완료

이 경우 `/etc/rancher/k3s/k3s.yaml` 파일의 내용이 변경되므로 다시 설정해야함.
