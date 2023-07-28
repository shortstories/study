# Buildkit

1.  buildkit

    moby 프로젝트의 일환으로 2017년부터 분리하여 개발 시작. 2018년 도커 18.09버전부터 도커엔진에 정식으로 탑재됨. 모비 엔진 빌드 기능의 성능, 스토리지 관리, 확장성 개선을 목표로 함. 기존의 `docker build`를 완전히 대체하는 것을 목표로 개발되고 있음 ([https://github.com/moby/moby/issues/40379](https://github.com/moby/moby/issues/40379))

    `DOCKER_BUILDKIT=1 docker build .`

    `docker buildx build`


2.  LLB (Low-Level Builder)

    ```bash
    {
      "Op": {
        "Op": {
          "source": {
            "identifier": "docker-image://docker.io/library/alpine:latest"
          }
        },
        "platform": {
          "Architecture": "amd64",
          "OS": "linux"
        },
        "constraints": {}
      },
      "Digest": "sha256:665ba8b2cdc0cb0200e2a42a6b3c0f8f684089f4cd1b81494fbb9805879120f7",
      "OpMetadata": {
        "caps": {
          "source.image": true
        }
      }
    }
    ```

    LLB는 저수준 바이너리 포맷으로, 빌드에 필요한 의존성 그래프를 그리는 것에 사용됨. LLB와 Dockerfile의 관계가 LLVM IR과 C언어의 관계와 동일하다고 보면 됨. LLB의 경우 특정 플랫폼에 구애받지 않아 다양한 문법으로 구현할 수 있음. Dockerfile 외에도 Mockerfile이라던가 Gockerfile, HLB와 같은 LLB를 지원하는 여러 문법들이 존재하고 있고, 아예 코드를 짜서 이미지를 빌드하는것도 가능함.
3. 개선점
   1.  자동화된 GC

       주어진 gcpolicy에 따라 저장된 layer들을 자동으로 정리하는 기능이 추가됨

       ```bash
       [worker.oci]
         [[worker.oci.gcpolicy]]
           keepBytes = 512000000
           keepDuration = 172800
           filters = [ "type==source.local", "type==exec.cachemount", "type==source.git.checkout"]
         [[worker.oci.gcpolicy]]
           all = true
           keepBytes = 1024000000
       ```
   2.  손쉬운 Multi-Platform 빌드 지원

       `platform=linux/amd64,linux/arm64`와 같이 platform을 지정해서 이미지를 빌드할 수 있게 추가되었음. 단, 사용자의 Dockerfile에 이미 관련 내용이 추가되어있어야 함. `FROM **--platform=$BUILDPLATFORM** golang:1.17-alpine AS build`
   3.  병렬 빌드 지원

       multi-staged 빌드의 경우 각 stage가 모두 병렬로 실행되며 만약 서로 다른 두 \*\*stage 사이에 의존 관계가 있는 경우에만 완료를 기다림.
   4.  개선된 캐시 기능

       LLB 각 operation의 checksum과 마운트된 데이터를 비교해서 사용하는 새로운 캐싱 시스템이 적용됨. 이에 따라 좀 더 빠르고 정확한 캐싱이 가능함. 또한 캐시의 export/import를 지원함. 캐시를 로컬 저장소 뿐만 아니라 registry에 저장하여 여러 buildkit daemon들이 cache를 공유하게 하는 것도 가능해짐.
   5.  새로운 빌드 기능 제공

       이미지를 빌드할 때 캐시 전용 공간이나 secret 등을 mount 할 수 있는 기능이 추가되었고, network mode나 security context를 설정하는 기능등 다양한 기능들을 새로운 Dockerfile 문법과 함께 제공 중.
   6. rootless로 사용 가능
4. buildkit에 최적화된 Dockerfile 작성방법
   1.  최대한 stage를 잘게 나눠서 사용하기

       stage를 나눠서 구성했을 때 buildkit의 병렬 빌드 기능을 최대한 활용할 수 있음. 뿐만 아니라 완성된 이미지의 크기를 좀 더 작게 관리할 수 있음.

       1.  base stage를 만들어 여러 stage에서 공유하기

           ```docker
           FROM ubuntu AS base
           RUN apt-get update && apt-get install git

           FROM base AS src1
           RUN git clone …

           FROM base AS src2
           RUN git clone …
           ```
       2.  불필요한 stage는 건너뛸 수 있게 구성하기

           ```docker
           ARG BUILD_VERSION=1

           FROM alpine AS base
           RUN …

           FROM base AS branch-version-1
           RUN touch version1

           FROM base AS branch-version-2
           RUN touch version2

           FROM branch-version-${BUILD_VERSION} AS after-condition

           FROM after-condition
           RUN …
           ```
   2.  dependency를 저장할 때는 `--mount=cache`를 활용하기

       ```docker
       # syntax=docker/dockerfile:1.2
       # Build Stage
       FROM maven as builder

       COPY pom.xml ~/pom.xml
       RUN --mount=type=cache,target=~/.m2,id=java-sample-cache,uid=500,gid=500 \
           mvn -Dmaven.repo.local=/home1/irteam/.m2/repository dependency:go-offline

       COPY src ~/src
       RUN --mount=type=cache,target=~/.m2,id=java-sample-cache,uid=500,gid=500 \
           mvn -Dmaven.repo.local=~/.m2/repository install

       # Main Stage
       ARG DEPENDENCY=~/build/dependency
       COPY --from=builder ${DEPENDENCY}/BOOT-INF/lib ~/apps/spring-boot-app/lib
       COPY --from=builder ${DEPENDENCY}/META-INF ~/apps/spring-boot-app/META-INF

       COPY entrypoint.sh ~/entrypoint.sh
       ENTRYPOINT ["~/entrypoint.sh"]
       ```
