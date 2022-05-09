# Image 개요

![https://github.com/google/go-containerregistry](../.gitbook/assets/image%20%287%29.png)



컨테이너 이미지는 보통 다음과 같이 구성된다.

1. image index
2. manifest
3. config
4. layer

여기서 핵심은 layer과 config라고 볼 수 있다. 결국 이미지라는 것은 컨테이너의 메타데이터와 파일시스템을 모아둔 것이라고 볼 수 있는데 이 layer들이 모여서 하나의 overlayfs를 이루고 config에 각종 메타데이터들이 들어가기 때문이다.

### Image Index

여러 manifest들을 모아 놓은 것이다. 하나의 이미지 이름 + 태그로 여러 architecture에 사용가능한 이미지를 만들 수 있게 해준다.

### Manifest

`docker manifest inspect <image name>` 을 통해서 값을 확인할 수 있다. 이미지의 config와 layer 정보, 그리고 이미지 자체에 대한 메타데이터들을 가지고 있다. 여기에 나오는 layer의 digest와 config의 layer id가 다른 이유가 궁금할 수 있는데 이건 layer의 mediaType을 확인하면 원인을 파악할 수 있다. `application/vnd.docker.image.rootfs.diff.tar.gzip` 이런 식으로 끝 부분이 gzip인 것은 gzip으로 압축되어있단 뜻으로 실제로 이미지를 받아서 gzip 압축해제한 다음 다시 digest를 계산해보면 image config에 들어있는 값과 동일한 값을 확인할 수 있다.

docker 1.10 이전에는 각 layer의 ID가 layer의 내용과 전혀 상관 없는 그냥 랜덤값이었고 하나의 layer가 하나의 이미지라고 볼 수 있었다. 왜냐면 모든 layer는 각자의 config를 가지고 있었기 때문이다. 그에 따라서 registry에서 따로 테이블을 만들어서 layer와 이미지를 관리해야했다. 만약 어떤 이미지를 모두 받아오고자 한다면 그 특정 이미지의 layer들을 보고 모든 parent들을 받아오는 식이었다. 여기에는 또 다른 단점도 있었는데, 완전히 동일한 내용의 layer라도 id가 다르면 중복해서 받아다가 저장하는 일도 있었다. 이런 비효율적인 부분을 개선하기 위해서 새로운 이미지 스펙을 만들어야했고 이것이 manifest이다.  manifest가 생기면서 한번에 필요한 모든 layer들을 참조하는게 가능해졌다. 즉, 이미지와 layer가 같은 의미가 아니게 되었다. manifest의 digest는 모두 내용을 기반으로 해싱한 content-addressable 값이다. 따라서 내용이 같다면 서로 다른 시점에 빌드된 layer라도 사용할 수 있게 되었다. 또한 이제는 이미지의 ID도 이미지의 내용으로 결정되게 바뀌었다.

### Config

이미지의 각종 메타 데이터들을 담고 있는 파일이다. json 포맷으로 되어있으며 `docker inspect <image name>` 명령어를 통해서 그 내용을 확인할 수 있다. 이전에는 각 layer마다 하나의 config가 존재했으며 그에 따라 모든 layer가 각각의 이미지처럼 취급받았었다. 실제로 지금도 로컬에서 직접 `docker build` 를 한 다음 `docker images -a` 를 해보면 중간 과정이 모두 이미지로 나오는걸 확인할 수 있다.  docker 1.10 이후부터는 `docker pull` 을 통해서 이미지를 받을 경우 최종 레이어의 config만 받는다.

### Layer

이미지를 사용해서 새로운 컨테이너를 만들 때 그 컨테이너의 파일시스템을 구성하는 하나의 단위이다. 각각의 layer는 파일시스템의 변경 내역을 담고 있다. 다시 말해 파일 또는 디렉토리의 생성, 삭제, 변경이다. 일반적으로 Dockerfile의 각 커맨드가 하나의 layer가 된다. docker 1.10 이후부터 `ENV` 이나 `LABEL` , `ARG` 같이 메타데이터를 만지는 커맨드들은 layer로 쌓이지 않는다는 점을 제외하면 말이다.

layer는 read-only이다. 컨테이너 안에서 아무리 파일을 생성하고 수정해봤자 기존 layer에는 아무런 영향을 끼치지 못한다. 왜냐하면 overlayfs로 동작하기 때문이다. config에 정의된 모든 layer 들은 overlayfs의 lowerdir로 들어가고 새로운 컨테이너는 overlayfs의 merged dir 위에서 동작한다. 따라서 컨테이너에서 파일을 수정하면 그 변경 내역이 담긴 upper dir을 바탕으로 새로운 layer가 생성될 뿐 기존 layer에 영향을 주지 않는다.

이미지를 빌드할 때 하나의 Dockerfile 커맨드를 실행하는 것은 새로운 컨테이너를 생성하고, 그 안에서 정의된 동작을 수행하고, 결과를 `docker diff` 및 `docker commit` 하는 것과 같다.

docker v1.10 이전에는 layer의 ID가 곧 이미지의 ID였고 이 값은 랜덤이었다. 그러나 v1.10 이후부터는 layer의 내용물을 해싱해서 ID로 사용한다.

