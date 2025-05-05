# 홈서버 트러블슈팅

### 재부팅이나 종료할 때 꺼지지 않거나 한참 걸리는 이슈

홈서버를 새 기기에 옮겨서 복구한 다음부터 좀 이상한 이슈가 생겼음. 서버를 내리거나 재부팅할 때 엄청 오랜 시간이 걸리는 것. 처음엔 부팅에 오래걸리는줄 알았는데 그게 아니었음. 내려가는데 엄청 오랜 시간이 걸리고 있었음.

이것을 확인하기 위해 우선 다시 재부팅을 한 다음 `journalctl -b -1` 명령어로 확인해봄

확인해보니 현재 홈서버에서 사용 중인 home-assistant docker container가 내려가질 않아 docker daemon도 못 내려가고 있었고 docker daemon이 못 내려가기 때문에 강제종료할 때까지 매우 긴 시간을 기다리고 있었던 것으로 확인되었음.

home-assistant container를 내리기 위해 `docker kill` 과 같은 명령어들을 시도해보았으나 아무 효과가 없음.&#x20;

최후의 수단으로 `sudo ps aux | grep containerd-shim | grep $CONTAINER_ID` 명령어로 containerd-shim 프로세스를 찾아낸 다음 containerd-shim을 kill해서 `docker stop` 이 먹히게 된 것까진 확인함

그 다음 다시 container를 재시작하려 하였으나 port already bind 에러로 실패하는 것을 확인

다시 프로세스를 확인해보니 home-assistant process가 아직 살아있는 상태였음. `kill -9` 역시 당연히 먹히지 않는 상태. 특이한 점이라면 프로세스의 stat이 `Ds` 였다는 것. 확인해보니 process가 D에 빠지는 것은 I/O 대기하고 있는 상태로 다른 어떤 일도 할 수 없는 상태라는 뜻이었음. 그래서 심지어 kill까지 무시하고  있었음. 보통은 빠른 시간 안에 I/O가 완료되어 R(Run)/S(Sleep)으로 돌아가야 정상인데 home-assistant process는 한참동안 D인 것을 보아 뭔가 문제가 있었음.

이러한 경우는 보통 어떤 잘못된 드라이버가 깔려있거나, 이슈가 있는 NFS 또는 원격 파일시스템에 접근 시도하고  있거나, 문제가 발생한 스토리지에 접근 중이거나 하는 경우가 많음. D상태에 빠져있는 프로세스를 종료하려면 이러한 문제가 생긴 파일시스템, 스토리지 등을 정상 상태로 회복시켜 I/O가 완료되게 하거나 아니면 포기하고 시스템을 재시작하는 수 밖에 없음. 다만 내 경우엔 시스템을 재시작하더라도 매우 오랜 시간이 걸린다는 문제가 있었음.

그래서 좀 더 정확한 정보를 알아야했는데 이건 프로세스의 wait channel을 확인하는 것으로 가능했음. `ps -o pid,wchan $PID` 명령어를 실행해보니 `hci_sock_bind` 라고 하는 것에 멈춰있는 것을 확인할 수 있었음. 확인해보니 블루투스에 관련된 함수임. home-assistant에서 bluetooth 연결을 사용하고 있는데 이 부분이 문제가 된 모양. 원래 서버에서는 퀄컴의 QCNFA765 wifi/bluetooth chipset을 사용하고 있었는데 새 서버에선 미디어텍의 칩셋으로 바뀌면서 문제가 발생한 것으로 추측함.&#x20;

`sudo apt purge -y bluez && sudo apt install -y bluez`

위 명령어로 bluez를 삭제하고 다시 설치해준 다음부터는 문제가 해결되었음.
