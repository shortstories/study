# Effective Java

## 4장
### Accessor, Mutator
- public 클래스에서는 public field 대신에 getter, setter 사용
  - 외부에서 직접 접근하므로 캡슣화 X
  - 필드를 추가하거나 수정하려면 API도 같이 수정해야함
  - 값이 변하지 않도록 할 수 없음
  - 필드에 접근할 때 부수적인 조치를 취할 수 없음

