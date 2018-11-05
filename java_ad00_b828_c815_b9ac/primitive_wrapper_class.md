# Java Primitive Wrapper class

보통 Null을 받을 필요성 있을 때 primitive대신에 사용하곤 하였다. 그런데 과연 이게 괜찮은 버릇인걸까?

## 빠른 결론

되도록 지양하자

## 이유

[http://paulgrenyer.blogspot.kr/2009/10/item-49-prefer-primitive-types-to-boxed.html](http://paulgrenyer.blogspot.kr/2009/10/item-49-prefer-primitive-types-to-boxed.html)

* 불필요하게 heap 공간 소요
* primitive에 비해서 더 큰 공간 차지
* boxing, unboxing 오버헤드
* 사용이 끝난 뒤 GC 오버헤드

## 대안

1. Exception으로 처리
   * 비용이 더 비쌈 \(Stack trace penalty\) [http://java-performance.info/throwing-an-exception-in-java-is-very-slow/](http://java-performance.info/throwing-an-exception-in-java-is-very-slow/)

