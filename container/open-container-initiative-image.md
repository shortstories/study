# Open Container Initiative Image

{% embed url="https://github.com/opencontainers/image-spec" %}

 oci image = manifest + image index \(optional\) + fs layers + configuration

### OCI Image media types

* application/vnd.oci.descriptor.v1+json: Content Descriptor 
* application/vnd.oci.layout.header.v1+json: OCI Layout 
* application/vnd.oci.image.index.v1+json: Image Index 
* application/vnd.oci.image.manifest.v1+json: Image manifest
* application/vnd.oci.image.config.v1+json: Image config 
* application/vnd.oci.image.layer.v1.tar: "Layer", as a tar archive 
* application/vnd.oci.image.layer.v1.tar+gzip: "Layer", as a tar archive compressed with gzip 
* application/vnd.oci.image.layer.v1.tar+zstd: "Layer", as a tar archive compressed with zstd 
* application/vnd.oci.image.layer.nondistributable.v1.tar: "Layer", as a tar archive with distribution restrictions 
* application/vnd.oci.image.layer.nondistributable.v1.tar+gzip: "Layer", as a tar archive with distribution restrictions compressed with gzip 
* application/vnd.oci.image.layer.nondistributable.v1.tar+zstd: "Layer", as a tar archive with distribution restrictions compressed with zstd

image index는 여러개의 image manifest를 가질 수 있음

image manifest는 하나의 config와 여러 layer를 가질 수 있음

manifest를 통해 달성하려 하는 목표는 크게 봐서 3가지이다. 하나는 content-addressable image를 만드는 것이다. 이는 image의 configuration을 해싱했을 때 그 image 자신과 구성 요소들을 unique하게 식별할 수 있게 만들면 된다. 두번째 목표는 multi-architecture image를 가능케 하는 것이다. 이것은 image index로 가능해진다. image index는 일종의 "fat manifest" 이고 여러 특정 플랫폼용 image를 한번에 관리하는 것이 가능하게 만든다. 마지막 목표는 OCI Runtime Specification으로 변환이 가능케 하는 것이다.

### OCI Image Manifest

image manifest는 image configuration과 fs layers에 대한 정보들을 가지고 있다. 이 때 이 정보는 특정 architecture, os에 국한된 것이다. 

#### Media Type

`application/vnd.oci.image.manifest.v1+json`

#### Example

```javascript
{
  "schemaVersion": 2,
  "config": {
    "mediaType": "application/vnd.oci.image.config.v1+json",
    "size": 7023,
    "digest": "sha256:b5b2b2c507a0944348e0303114d8d93aaaa081732b86451d9bce1f432a537bc7"
  },
  "layers": [
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "size": 32654,
      "digest": "sha256:9834876dcfb05cb167a5c24953eba58c4ac89b1adf57f28f2f9d09af107ee8f0"
    },
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "size": 16724,
      "digest": "sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b"
    },
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "size": 73109,
      "digest": "sha256:ec4b8955958665577945c89419d1af06b5f7636b4ac3da7f12184802ad867736"
    }
  ],
  "annotations": {
    "com.example.key1": "value1",
    "com.example.key2": "value2"
  }
}
```

### OCI Image Index

image index는 상위 레벨의 manifest라고 생각하면 된다. 여러 개의 manifest를 가지고 있고 그 각각의 manifest가 어떤 플랫폼에 가장 적합한지 가르키는 정보를 포함하고 있다.

#### Media Type

`application/vnd.oci.image.index.v1+json`

#### Example

```javascript
{
  "schemaVersion": 2,
  "manifests": [
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "size": 7143,
      "digest": "sha256:e692418e4cbaf90ca69d05a66403747baa33ee08806650b51fab815ad7fc331f",
      "platform": {
        "architecture": "ppc64le",
        "os": "linux"
      }
    },
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "size": 7682,
      "digest": "sha256:5b0bcabd1ed22e9fb1310cf6c2dec7cdef19f0ad69efa1f392e94a4333501270",
      "platform": {
        "architecture": "amd64",
        "os": "linux"
      }
    }
  ],
  "annotations": {
    "com.example.key1": "value1",
    "com.example.key2": "value2"
  }
}
```

### Layer

정확히는 "Image Layer Filesystem Changeset" 말 그대로 파일 시스템 변경 내역이 바로 이미지의 레이어이다. 이 레이어들이 차곡차곡 쌓여서 하나의 완전한 파일시스템을 이룬다. 

#### Media Type

* `application/vnd.oci.image.layer.v1.tar`
* `application/vnd.oci.image.layer.v1.tar+gzip`
* `application/vnd.oci.image.layer.v1.tar+zstd`
* `application/vnd.oci.image.layer.nondistributable.v1.tar`
* `application/vnd.oci.image.layer.nondistributable.v1.tar+gzip`
* `application/vnd.oci.image.layer.nondistributable.v1.tar+zstd`

#### +gzip Media Type

layer의 payload가 gzip으로 압축됨

#### +zstd Media Type

layer의 payload가 [zstd](https://github.com/facebook/zstd)으로 압축

#### Distributable Format

레이어는 반드시 tar로 묶어야하고 완성된 tar archive에는 중복되는 엔트리가 존재하지 않아야 함.

#### Change Types

* Additions
* Modifications
* Removals

이 중 Additions와 Modifications는 tar archive 시점에서는 동일하게 취급되고 Removals만 whiteout file entries로 적용된다.

#### File Types

* regular files
* directories
* sockets
* symbolic links
* block devices
* character devices
* FIFOs

#### File Attributes

* Modification Time \(`mtime`\)
* User ID \(`uid`\)
  * User Name \(`uname`\) _secondary to `uid`_
* Group ID \(`gid` \)
  * Group Name \(`gname`\) _secondary to `gid`_
* Mode \(`mode`\)
* Extended Attributes \(`xattrs`\)
* Symlink reference \(`linkname` + symbolic link type\)
* [Hardlink](https://github.com/opencontainers/image-spec/blob/master/layer.md#hardlinks) reference \(`linkname`\)

#### Creating

layer를 생성하는 과정은 다음과 같다.

1. root 파일 시스템 초기화: root 파일 시스템은 base 또는 이전 레이어
2. 비교를 위한 파일시스템 생성: 단순히 파일 시스템을 복사하거나 이전 root 파일 시스템의 snapshot 생성
3. 변경내역 결정: 파일시스템에 추가, 변경, 삭제된 것들을 비교
4. 변경된 것들만 모아서 새로운 tar archive 생성: Added, Modified는 변경된 파일을 엔트리에 등록. Deleted는 whiteout 파일을 엔트리에 등록

#### Applying Changesets

whiteout 파일을 처리하는 것을 제외하고는 일반적인 tar archive extract랑 동일하게 동작.

tar 엔트리를 처리하는 과정에서 이미 존재하는 파일이 있을 때 그 파일이 directory이면 file attributes만 엔트리 기준으로 적용하고 나머지 타입의 파일들은 삭제하고 엔트리 파일로 새로 생성해넣는다.

#### Whiteouts

whiteout 파일은 이름이 `.wh.` 으로 시작하는 빈 파일로 부모 레이어에서 이 이름을 가진 파일이 삭제되었음을 표시. whiteout 파일의 순서에 상관없이 가장 먼저 처리되어야 한.

#### Opaque Whiteout

`.wh..wh..opq` 이름을 가진 특별한 whiteout 파일로 이 파일이 존재하는 path의 모든 child 가 삭제되었음을 표시한. 만약에 이 파일과 같은 path에 entry가 존재한다면 삭제된 다음 다시 생성되었다는 의미이.

#### Non-Distributable Layers

법이나 기타등등 여러가지 사정으로 인하여 일반적으로 사용하는 것 처럼 배포할 수 없는 레이어들이다. 예를 들면 마이크로소프트의 윈도우 베이스 이미지 같은게 있다. 이러한 "non-distributable" 레이어들은 인증받은 특정 url에서 직접 다운로드 받아야하고 레지스트리에 업로드되면 안된다. 따라서 non-distributable 레이어의 Descriptor에는 보통 이러한 레이어를 직접적으로 받을 수 있는 url이 `urls` 필드에 기록되어있는 경우가 많다.

