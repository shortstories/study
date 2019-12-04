# Go dependency injection

자바를 쓰다가 go를 쓰면서 여러모로 불편한 점들이 많다. 가장 아쉬웠던건 maven, gradle. go dep이랑 make를 섞어서 어찌어찌 해결보긴 했지만 전반적으로 느린데다가 고장도 너무 잘 나서 뭐 하나 추가하기만 하면 매번 꼬인 dependency 푸느라 고생이 이만저만이 아니다. 

그러다가 두번째로 괴로움을 겪은 것이 바로 dependency 관리 문제다. constructor를 수정할 일이 생기면 사용한 곳을 검색해서 나오는 곳마다 고치기를 몇번 반복하고나니까 진절머리가 났다. 초기화 과정을 관리하는 것도 힘들고, 처음에는 `func init()` 을 많이 활용했더니 갖가지 타이밍 이슈가 발생했다. 자연히 spring처럼 별도의 container를 두고 한꺼번에 관리하고싶다는 생각이 들더라. 그래서 관련된 라이브러리, 문서들을 찾아보게 되었다.

### Dependency injection 라이브러리

일반적으로 DI 라이브러리는 두 가지 기능을 제공한다.

1. 'Providing' 기능: 새로운 component를 어떻게 생성할 것인지 정의할 수 있게 해준다. 이를테면 설계도를 집어넣는 기능인데 golang에서는 보통 `func NewObject() Object` 같은 형태를 취하는 constructor를 말한다고 봐도 될 것이다.
2. 'Retrieving' 기능: 생성된 component를 검색할 수 있게 해준다.

DI 라이브러리는 이 'provider' 들을 제공받아서 일종의 graph를 생성하고, 필요로 하는 위치에 적절하게 집어넣어서 하나의 dependency tree를 구성해준다.

먼저 golang 라이브러리에서 di 관련된 것들을 찾아봤다. 관리가 되고 있지 않는 프로젝트는 제외하고 문서가 부실한 것도 제외하고 github star가 많은 것부터 아래로 차근차근 내려오면서 찾아보니 제일 쓸만해보이는건 두가지였다. 하나는 uber의 dig \([https://github.com/uber-go/dig](https://github.com/uber-go/dig)\) 이고, 나머지 하나는 google의 wire \([https://github.com/google/wire](https://github.com/google/wire)\) 이다. 

