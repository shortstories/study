# Programming in Scala
## 1장
- 스킵

## 2장
- 스킵

## 3장
### List
- list의 제일 앞에 원소 추가하는건 O(1), 뒤에 추가하는건 O(n)
- 그러므로 앞에 추가하는게 좋음. 만약에 뒤에 추가하고 싶다면 우선 뒤집고 앞에 추가한 다음에 다시 뒤집거나 `ListBuffer` 사용

### Tuple
- tuple은 `tuple._i` 같은 형태로 원소 호출
- i는 1부터 시작 (하스켈, ML 등 정적 타입 언어의 영향)
- list와 tuple의 차이점은 여러 자료형을 한번에 담을 수 있느냐의 차이

### Map
- map의 각 원소는 tuple

### 함수형 스타일
- var보단 val을 더 권장
- side effect를 최소화
  - 만약에 Unit을 반환하는 함수가 있다면 주의할 것

