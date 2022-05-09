# Buildkit

### LLB

LLB = low-level builder LLB는 빌드 중간과정의 바이너리 포맷. LLVM이랑 비슷한 개념으로 생각하면 될듯.

![](../.gitbook/assets/image%20%2811%29.png)

LLB는 위와 같이 content-addressable dependency graph 로 정의된다고 볼 수 있음. 덕분에 캐싱 모델도 기존 Dockerfile builder와 완전히 달라짐. 관습적으로 이미지를 비교하는 대신에 LLB에 정의된 build graph 각 node의 checksum과 mount된 content들을 바탕으로 비교하게 됨.

또한 LLB는 Dockerfile에서 지원하지 않는 추가기능들을 제공함. 예를들면 direct data mounting이나 nested invocation등이 있음.

 LLB는 golang client로 생성할 수 있음. 근데 보통 사용자들은 LLB를 직접 생성하기보다는 frontend를 사용하게 됨.

 frontend는 human-readable build format을 받아서 이걸 LLB로 만들어주는 역할을 수행하는 컴포넌트임. frontend들은 image로 배포되기 때문에 사용자들이 원하는 버전과 종류를 골라서 사용할 수 있음.

LLB의 특징은 다음과 같음.

1. Protobuf message로 marshaled 됨
2. 동시 실행이 가능
3. 효율적인 캐싱 가능
4. 벤더에 중립적임. 즉 Dockerfile이 아닌 다른 언어로도 만드는게 가능해짐







