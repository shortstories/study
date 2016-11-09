# Effective Java

## 4장
### Accessor, Mutator
- public 클래스에서 public field를 사용하면 여러 단점 존재
  - 외부에서 직접 접근하므로 캡슐화 X
  - 내부 구조를 변경하기 어려워짐
  - 값이 변하지 않도록 할 수 없음
  - 필드에 접근할 때 부수적인 조치를 취할 수 없음
