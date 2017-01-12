# React Element

Element는 React 어플리케이션을 작성할 때 사용하는 가장 작은 단위로, 건물로 치면 하나의 벽돌이라고 봐도 좋다.

일반적으로 이 Element는 immutable하다. 따라서 한번 생성된 element를 수정할 방법은 없다고 봐도 된다.

ReactDOM.render\(\)을 사용하여 실제 HTML DOM에 파싱, 끼워넣게 된다.

만약에 어떤 데이터가 변경되면 DOM에 존재하는 기존 element와 비교한 뒤, 거기에 영향받는 부분만 새롭게 만들어 바꿔넣게 된다.



