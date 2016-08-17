# Double colon operator

## 형태
`Reference::method`

## 설명
- 특정 메소드를 호출하는 람다식을 간략화한 연산자
- 패러미터와 반환 여부, 반환 타입에 따라 `@FunctionalInterface` 중 하나와 대응됨

## 유의점
- 패러미터가 없고, 반환 값이 있을 때 Double colon operator은 `Supplier`, `Callable` 두 가지 모두에 대응됨