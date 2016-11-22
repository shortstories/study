# Service Discovery Tools

- 네트워크에 분산되어 유기적으로 구조가 변경되는 시스템이 있을 때, 각 노드 정보 및 설정 정보를 저장, 탐색하는데 사용하는 도구

## 조건

- 노드가 정지하거나 네트워크 장애가 발생했을 때 적절하게 제거되어야 함
- 새로운 노드를 추가했을 때 적절하게 전파되어야 함
- 데이터의 Consistency가 보장되어야 함
- 데이터 변경에 대한 watch가 가능해야 함

## 구조

- 일반적으로 `Provider` - `Service Discovery Tool` - `Proxy` - `Consumer` 구조

## 목록
- zookeeper
- etcd
- eureka
- consul