# Systemd

## 참고

[http://0pointer.de/blog/projects/systemd.html](http://0pointer.de/blog/projects/systemd.html)

## 개요

init system은 리눅스 시스템이라는 하나의 건물을 쌓아올릴 때 가장 먼저 놓이는 초석이다. 다른 모든 프로세스에 우선해서 커널이 실행하기 때문에 init system의 pid는 1이 된다. 이러한 이유에 따라 다른 프로세스들이 할 수 없는 일을 할 수 있지만, 그에 따라 특별한 책임을 짊어지게 된다. 가령 예를 들면 userspace를 가져오고 관리하는 일이 있겠다.

기존의 리눅스 시스템은 init system으로 Upstart라는 프로그램을 사용했었다. 분명 안정적이고 훌륭한 프로그램이지만 속도가 빠르다고는 말하기가 힘들다. 좋은 init system의 요건 중에 빠른 부팅 속도가 있음에도 불구하고 말이다.

빠르고 효율적인 부팅을 위해서 염두에 둘 두 가지 중요한 요소가 있다.

1. 좀 더 적게 시작하기
2. 가능한한 많이 병렬 실행하기

좀 더 적게 시작하란 말은 lazy를 생각하면 딱일듯 하다. 즉, 정말로 필요해지기 전까진 굳이 시작하지 않아도 된단 것이다. 이런 경우를 생각해보자. syslog, D-Bus system같은 것들은 즉시 필요한 경우가 많다. 그러나 bluetoothd의 경우엔 어떠한가? 내 리눅스 기기에 블루투스 장치를 설치하기 전에는 굳이 실행할 필요가 없는 프로세스다.

가능한한 많이 병렬 실행하란 말은 말 그대로 이해하면 된다. 기존의 sysvinit은 하나하나 직렬로 수행했기 때문에 상대적으로 불리하다. CPU나 IO 대역폭이 버텨주는 만큼 되는대로 동시 실행하는게 전체 부팅 시간을 줄이는데 도움이 된다.

## Socket services 병렬화

지금 쓰고있는 다른 시스템들도 병렬화를 시도하지 않은 것은 아니다. 그런데 현재 방식에 약간 단점이 있다. 어떤 데몬들이 의존관계에 있을 때 동기식으로 동작하는 것이다. 즉 어떤 클라이언트 데몬이 다른 서비스 데몬을 필요로 할 때, 서비스 데몬이 완전히 실행될 때까지 기다리게 되는 것이다. 그에 따라 부팅에 걸리는 시간이 꽤 늘어나게 된다.

그럼 어떻게 필요한 서비스 데몬이 완전히 실행되었는지 판단하는 걸까? 전통적인 Unix 데몬들은 여기에 AF\_UNIX Socket을 사용한다. 즉, 각 데몬들은 자신의 서비스가 준비 완료되면 그에 따라 socket connection을 가능하게 만드는 것으로 표현하는 것이다. 예를 들어, D-Bus를 필요로 하는 데몬들은 `/var/run/dbus/system_bus_socket`에 연결이 가능해지기를 기다린다. 또, syslog를 필요로 하는 데몬들은 `/dev/log`의 연결을 기다린다.

데몬이 완전히 실행되기까지 기다리는 시간을 없애고 싶은 것이므로, 다른 방법을 사용해야 한다. 그 방법은 모든 데몬의 socket을 미리 만들어서 연결 가능한 상태로 바꿔놓고, 실제로 실행하는 `exec()`동안 넘겨주는 것이다. 이렇게 하면 단 두 번의 단계로 모든 데몬을 실행하는 것이 가능하다. 다만 이렇게 되면 위에서 말한 것 처럼 데몬 사이에 의존관계가 있을 땐 어떻게 되는지 궁금할 것이다. 미리 대답부터 하자면 문제 없다. 서비스 데몬이 아직 준비가 덜 된 상황이라면 그저 connection은 서비스 데몬의 socket buffer queue에 들어가고, 클라이언트 데몬은 그 요청에 대해서 잠시 blocking 될 뿐이다. 잠시 후 서비스 데몬이 준비 완료되면 queue의 connection들을 하나씩 처리하여 정상적으로 동작할 것이다. 이 때 클라이언트 데몬의 요청이 동기가 아니라 비동기라면 잠시 blocking 되는 과정조차 없을 것이다.

이렇게 되면 부팅 시간이 짧아질 뿐만 아니라 여러가지 장점이 더 생긴다. 우선, 병렬화 과정에서 의존성 관리를 위해 실행 순서같은 별도 설정을 할 필요가 없어진다. 그리고 두 번째로 장애상황에 대해 좀 더 유연해진다. 소켓은 실제 데몬이 살아있던 죽어있던 언제나 정상적으로 떠있다. 덕분에 서비스 데몬에 예기치못한 문제가 생겨서 종료되어도 클라이언트 데몬의 시점에서는 아무런 문제가 되지 않는다. 종료된 사이에 들어온 요청은 모두 socket buffer에 저장되어있을 것이므로 데몬을 되살려서 처리하면 되기 때문이다.

사실은 이 방식들은 이미 애플의 MacOS의 launchd에서 쓰는 방식이다. 뿐만 아니라 launchd보다 더 오래된 inetd도 이와 비슷한 방식을 사용했다.

## Bus Services 병렬화

현대적인 리눅스 데몬들은 위에서 말한 AF\_UNIX socket보다는 D-Bus를 이용한다. 여기에도 위에서 말한 해결법이 통용된다. 왜냐하면 D-Bus에서 이미 관련 기능들을 제공하기 때문이다. 데몬들은 access가 발생하면 그제서야 실행된다. 그리고 socket 방식과 마찬가지로 D-Bus도 request 단위의 최소화된 동기화를 제공한다. 의존 관계에 있는 두 데몬이 있고 각각 실행되는 시간이 다를 때 request들을 queue에 넣어두도록 한다.

따라서, socket과 bus 두 가지를 동시에 사용하면 추가적인 동기화 없이 모든 데몬을 병렬적으로 실행할 수 있다는 뜻이 된다. 또한 데몬들로 하여금 부팅될 때 일괄적으로 실행되는 것이 아니라 그것이 정말로 필요할 때 실행되도록 만들어준다.

## File System jobs 병렬화

사실 데몬만 실행한다고 해서 끝나는게 아니다. 오히려 파일 시스템과 관련된 작업이 엄청 큰 시간을 잡아먹는다. 파일 시스템 작업에는 mounting, fscking, quota 등이 있다. `/etc/fstab` 아래에 있는 device 하나하나마다 그러한 작업을 체크하고 수행하는 동안 대기하다가, 모두 완료된 이후에야 데몬들을 실행하게 된다.

