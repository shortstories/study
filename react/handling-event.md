# Handling Event

기본적으로 DOM에서 이벤트 처리하는 것이랑 크게 다를 바가 없고, 몇몇 차이점들만 알고 있으면 된다.

## 차이점

1. react의 이벤트는 camel case로 이름 붙여져있다. \(onclick 대신에 onClick인 식\)
2. JSX는 이벤트 핸들러로 string을 받는게 아니라 function 그 자체를 받는다.
3. 핸들러에서 `return false`로 default 동작을 막을 수 없어서 `event.preventDefault()`를 직접 호출해줘야 한다.

## 사용법

```javascript
import React from 'react';

class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      text: "this is my component"
    };
    this.onClickButton = this.onClickButton.bind(this); // 유의할 부분
  }

  onClickButton(e) {
    alert(this.state.text);
  }

  render() {
    return (
      <button onClick={this.onClickButton}>
        {this.props.buttonValue}
      </button>
    )
  }
}
```

여기서 아마 `this.onClickButton.bind(this)` 부분이 왜 있는지 의아할 수 있는데, 이건 자바스크립트의 이벤트 처리 방식에 따라 필요한 부분이다. 자바스크립트에서는 이벤트가 발생했을 때 이벤트가 발생한 DOM 객체를 `this`로 바인딩해서 이벤트 핸들러를 호출한다. 그런데 위 코드에서는 component의 state 데이터를 참조하고 있고 그에 따라 component를 `this`로 미리 바인딩 해놓은 것이다. `bind()`의 특성 상 몇번 호출되더라도 제일 첫번째 `bind()`가 적용되기 때문에 `onClickButton`가 호출될 때 항상 `this`는 `MyComponent`이다. 그러나 만약에 저 `bind()`가 없다면 `this`는 button 객체가 될거고 그에 따라 state가 없다는 에러를 뿜을 것이다.

