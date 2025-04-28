# 홈서버 백업 및 복구기

현재 집 홈서버의 구성은 아래와 같음

1. 홈 서버 (nvme 4TB)
2. nvme 4bay DAS (nvme 10TB)
3. 백업용 외장 HDD (HDD 16TB) - `/media/shortstories/Elements` mount

모든 파일시스템은 btrfs를 사용하고 있고 RAID는 따로 걸어놓지 않았음. 백업용 HDD가 따로 있으니 최악의 경우에도 HDD에서 복구하면 될 것이라는 생각. 그리고 백업용 외장 HDD를 제외한 모든 storage들은 `/`에다가 `btrfs device add`로 추가해서 사용하는 중이었음.

대신 btrbk를  사용해서 rootfs를 통째로 백업에 넣어두고 있었음

```tsconfig
# Enable transaction log
transaction_log            /var/log/btrbk.log

# Enable stream buffer. Adding a buffer between the sending and
# receiving side is generally a good idea.
# NOTE: If enabled, make sure to install the "mbuffer" package!
stream_buffer              256m
stream_compress zstd


#
# Example retention policy:
#
snapshot_preserve_min   2d
snapshot_preserve       14d

snapshot_create ondemand

target_preserve_min     no
target_preserve         20d 10w *m


#
# Simple setup: Backup root and home to external disk
#
snapshot_dir btrbk_snapshots

volume /mnt/btr_pool
  target /media/shortstories/Elements/btrbk_backups
  subvolume rootfs
```

여기에 맞추기 위해 기본 구조는 btrbk에서 권장하는 방식을 이용함

{% embed url="https://github.com/digint/btrbk/blob/master/doc/FAQ.md#how-should-i-organize-my-btrfs-filesystem" %}

원래 홈서버로 aoostar gem12 8845hs버전을 사용했었는데 전원부가 좀 부실한지 io쪽이 몇번 나갔다 들어오다 하더니 반년만에 마침내 사망해버림. 직구라 AS? 그런거 없음. 공식 구매처에 물어보니 해외에서 AS보내는건  받지않고 주변에 중국인이 있으면 부탁해서 AS 보내고 받으면 된다는데 그런게 어딨음? 사실상 복구 불가능 판정...

다씨는 미니pc를 구매하지 않으리라 깊게 다짐하며 별 수 없이 SFF를 다시 조립해서 홈서버로 사용할 계획을 세움.

당초 계획은 8845hs 대신 8700g와 650i 보드를 사용하고, 대부분의 650i 보드들이 USB4나 썬더볼트, 오큐링크를 지원하지 않으므로 DAS에 사용하던 nvme 4개는 pcie 확장보드로 옮겨서 사용하는 것이었음.

이를 위해 모든 부품을 드래곤볼하고 마침내 조립까지 완료했으나...

8700g를 꽂으면 pcie를 x8밖에 못쓴다는 사실을 뒤늦게 알게되고 패닉이 옴. 내가 사용한 MSI MPG b650i  EDGE 보드에는 usb4, oculink가 없어서 기존에 사용하던 DAS도 사용할 수 없음.

부랴부랴 pcie-oculink 확장보드를 주문했지만 홈서버가 내려감으로 인해 iot가 모두 먹통이 되어 압박이 장난아님.

급한대로 확인해보니 아직 홈서버 스토리지 사용량이 3TB밖에 되지않아 메인보드에 장착한 4TB로도 당분간 어떻게든 운영할 수 있겠다는 계산이 나옴

그에 따라 남은 2개의 스토리지에 백업본을 복구하는 작업에 들어감

제일 좋은 방법은 그냥 2개 스토리지에 바로 백업 subvolume 덮어쓰기 하는거겠지만 내가 못찾은건지 방법이 없는 것으로 보임. ubuntu live cd로 한참 끙끙댔는데 결국 못찾음. btrfs에서 device MISSING이 발생하면 복구가 상당히 어려운 모양임.

그래서 2안인 새 ubuntu 설치하고 subvolume 덮어쓰기로 넘어감

아래는 복구 삽질 내역 기록.

### 복구 절차

1. ubuntu 신규 설치하기
   1. 중간에manual installation 선택하고 btrfs filesystem 선택하기
   2. 기존 ubuntu랑 버전 일치시키면 많은 문제가 해소될 것으로 보임. 나는 아무 생각 없이 최신 버전으로 설치했다가 약간 삽질함.
2. [https://github.com/digint/btrbk/blob/master/doc/FAQ.md#how-should-i-organize-my-btrfs-filesystem](https://github.com/digint/btrbk/blob/master/doc/FAQ.md#how-should-i-organize-my-btrfs-filesystem)
   1. 위 내용대로 btrfs filesystem 구조 변경
   2. device 잘 봐야함. fstab 내용 보는게 제일 확실함. 잘못 덮어씌우면 우분투 다시 설치해야함.
   3. `/mnt/btr_pool` fstab 추가 반드시 하기
   4. 구조 변경 후 reboot, 문제없이 부팅되는지 확인하고 그 다음 `btrfs subvolume show /` , `btrfs subvolume show /mnt/btr_pool` 해서 subvolume id 문제없이 할당되었는지 확인
3. [https://github.com/digint/btrbk?tab=readme-ov-file#restoring-backups](https://github.com/digint/btrbk?tab=readme-ov-file#restoring-backups)
   1. 위 내용대로 btrfs send / receive 실행
   2. 내 경우엔 `/media/shortstories/Elements/btrbk_backups` 아래에 모든 백업 스냅샷이 존재하므로 `btrfs send /media/shortstories/Elements/btrbk_backups/rootfs.{yyyyMMddThhmm} | btrfs receive /mnt/btr_pool/` 실행
   3. 이후 `btrbk ls /` 명령어로 `/mnt/btr_pool/rootfs.{yyyyMMddThhmm}` 정상적으로 생성되었는지 확인
4. rootfs 교체하기
   1. `mv /mnt/btr_pool/rootfs /mnt/btr_pool/rootfs.backup`&#x20;
   2. `btrfs subvolume snapshot /mnt/btr_pool/rootfs.{yyyyMMddThhmm} /mnt/btr_pool/rootfs`&#x20;
   3. fstab 바꿔주기 `cp /mnt/btr_pool/rootfs.backup/fstab /mnt/btr_pool/rootfs/fstab`
   4. boot 바꿔주기 `cp -aRu /mnt/btr_pool/rootfs.backup/boot/* /mnt/btr_pool/rootfs/boot`
   5. modules 바꿔주기 `cp -aRu /mnt/btr_pool/rootfs.backup/lib/modules/* /mnt/btr_pool/rootfs/lib/modules`
   6. `btrfs subvolume set-default "$(btrfs subvolume show /mnt/btr_pool/rootfs | grep 'Subvolume ID' | awk '{printf $3}')" /`&#x20;
   7. 재부팅 후 정상적으로 복구 완료되었는지 확인
5. 정리
   1. 문제 없을 경우 `btrfs subvolume delete /mnt/btr_pool/rootfs.backup` 명령어로 찌꺼기 제거&#x20;

복구 과정에서 `/boot` , `/lib/modules` , `/etc/fstab` 불일치 문제때문에 부팅이 안되거나, 부팅이 되기는 하는데 docker가 동작하지 않거나(overlayfs2 driver를 지원하지 않는다는 에러 발생), iptable이 정상적으로 작동하지 않는 버그 경험

가능하다면 위 영역들은 rootfs subvolume에서 따로 떼어내서 관리하는게 더 깔끔하긴 할듯. 지금 당장은 모든 문제를 해결해서 정상적으로 서버가 돌아가고 있으므로 따로 건드리진 않을 예정.

DAS는 pcie-oculink 확장 카드가 오면 oculink로 연결해서 다시 포맷한 다음 `btrfs device add` 로 추가해서 기존처럼 사용할 예정.
