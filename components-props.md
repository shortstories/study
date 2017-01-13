# Components, Props

Component는 UI레벨에서 하나의 재사용 가능하고 독립적인 개체라고 볼 수 있다. 객체지향의 클래스같은 개념이라고 생각해도 무방할 듯.

개념적으로 자바스크립트의 function과 비슷하다. 'props' 라고 하는 input을 받아서 적절하게 처리 후 화면에 그려야하는 React Element를 돌려준다.

props -&gt; Components -&gt; React Element

## 구현 방법

### Function

```js
function MyComponent(props) {
  return <h1>Hello {props.name}!</h1>;
}
```

가장 간단하게는 위와 같은 함수로 component를 만들 수 있다. 하나의 props를 받고, JSX로 표현된 React Element를 돌려주고 있다.

### Class

```js
import React from 'react';
class MyComponent extends React.Component {
  render() {
    return <h1>Hello {props.name}!</h1>;
  }
}
```

es6의 클래스를 이용해서 component를 만들 수도 있다. 위 component는 function으로 구현한 것과 완전히 동일하게 동작한다. 그러나 상속을 통해서 React.Component 아래에 있는 여러 method들을 override할 수도 있는 등 여러모로 편리한 점이 많다. 물론, function을 사용해서도 동일하게 만들 순 있지만 여러모로 복잡해진다.

## 사용법

```js
const ReactDOM = require('react-dom');

function MyComponent(props) {
  return <h1>Hello {props.name}!</h1>;
}

const element = <MyComponent name='OhChang' />
ReactDOM.render(
  element,
  document,getElementById('container')
);
```

component는 JSX에서 마치 HTML tag중 하나인 것 처럼 사용할 수 있다. 이 때, 어떤 특정한 `key = value`를 넣어두면 component를 생성할 때 `props.key = value`로 들어가게 된다. 앞서 JSX가 React Component를 만든다고 했으므로 그걸 `ReactDOM.render()`을 통해서 DOM에 그리면 된다. 여기서 주의할 것은 component의 이름은 첫 글자가 대문자여야 한다는 것이다. 그래야 다른 HTML 태그와 구분하기가 쉽다.





