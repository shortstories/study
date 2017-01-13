# State, Lifecycle

## State

앞서 React Element는 immutable 개체라고 했다. 그러나 component는 stateless 개체가 아니다. component는 어떤 자신의 state를 가질 수 있다. 물론 앞서 설명에는 모든 react component를 pure function처럼 취급하라고 했는데 바로 다음 페이지에서 이렇게 말을 뒤집으니 좀 난감하긴 하다. 뭐, 정말로 stateless한 pure function으로 할 때 여러모로 불편한 점이 생기니 만든 대안이긴 할 것이다.

### 사용법

우선, 만약에 component를 function으로 작성했다면 es6 class 형태로 바꾸는게 여러모로 유리하다.

```js
import React from 'react';

// 1. use prop
class MyComponent extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

// 2. use state
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {name: props.name}
  }

  render() {
    return (
      <h1>Hello, {this.state.name}</h1>
    );
  }
}
```

2번 예제처럼 `this.state` 에 객체로 저장해두고 자유롭게 사용할 수 있다. 그런데 여기서 궁금해지는게 있다. 그냥 class field로 두고 사용해도 될 것 같은데 왜 하필 꼭 저렇게 써야되나? 이유는 다음과 같다. 그냥 class field로 어떤 state를 저장하고 변경이 발생했을 땐 실제 DOM까지 그 변경이 전파되지 않기 때문이다. 그래서 `this.state`에다가 저장하고 이 state를 변경할 때도 특정 메소드를 사용해서 변경해야만 한다.

#### 유의사항

1. state를 직접 수정하지 않아야 함

   1. 이유는 위에서 말한 것 처럼, DOM까지 state의 변경을 전파하기 위해서. 대신에 `setState()`를 사용

   2. ```js
      this.state.name = "other'; //false  
      this.setState\({name: 'other'}\); // true

      constructor\(props\) {  
        super\(props\);  
        this.state = {name: 'other'}; // 생성자에서는 허용됨  
      }
      ```

2. state의 변경은 비동기적으로 적용됨

   1. react에서 성능을 향상시키기위해 여러 `setState()`를 몰아서 한번에 적용하기 때문에 발생. 따라서 이전 state에 의존해서 작동하는 코드는 고장날 수 있음. 이럴 땐 `setState()`의 argument에 callback function을 넘겨서 처리 가능

   2. ```js
      // 이렇게 여러번 호출하면 호출한 횟수만큼 this.state.counter의 값이 증가해있길 바라지만 
      // 순서가 보장되지 않기 때문에 원하는 값이 나오지 않을 수 있음
      this.setState({
        counter: this.state.counter + this.props.increment
      });

      // 고로 이렇게 function을 넣으면 원하는 대로 동작
      this.setState((prevState, props) => ({
        counter: prevState.counter + props.increment
      }));
      ```

3. state의 변경은 엄밀히 따져서 set이 아니라 merge에 가까움

   1. state 내부에 여러 key가 존재할 때, `setState()`에서 하나씩 수정해도 문제 없음

   2. ```js
      import React from 'react';

      class Point extends React.Component {
        constructor(props) {
          super(props);
          this.state = {
            x: 0,
            y: 0
          }
        }

        componentDidMount() {
          calculateX().then(newX => {
            this.setState({x: newX}); // 기존의 y값은 놔두고 x값만 바꿈
          });

          calculateY().then(newY => {
            this.setState({y: newY}); // 기존의 x값은 놔두고 y값만 바꿈
          });
        }
      }
      ```

### 탑 다운 데이터 플로우

component의 state는 local, 내지는 encapsulated된 데이터라고 볼 수 있다. 즉, 다른 component에서 접근할 수 없다는 뜻이다. 이 사실은 parent - child 관계에 놓인 component들 끼리도 마찬가지다. 그러나 parent -&gt; child 간의 데이터 전달은 가능하며, 이 경우 parent에서 어떤 데이터를 넘겨주게되면 child에서는 그걸 props로 간주하여 사용한다. 이 때, child는 parent가 전달하는 데이터가 state이든, props든, 상수이든 전혀 신경쓰지 않는다.

```js
import React from 'react';

class Parent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {data: props.obj.data};
  }

  render() {
    // 모두 문제 없음
    return (
      <Child data={this.props.obj.data} />
      <Child data={this.state.data} />
      <Child data='my Data' />
    )
  }
}

class Child extends React.Component {
  render() {
    return (
      <h1>{this.props.data}</h1>
    );
  }
}
```

## Lifecycle

component들은 모두 각자의 라이프 사이클을 가지고 있으며, 이 각각의 라이프 사이클에 대해 원하는 동작을 써 넣을 수 있다. 보통, 메소드 이름에 will이 붙으면 특정 동작을 하기 전에 수행하고, did가 붙으면 동작을 완료한 다음 수행한다.

### 종류

#### Mounting

component가 생성되고 DOM에 끼워넣어질 때

1. constructor\(\)
2. componentWillMount\(\)
3. render\(\)
4. componentDidMount\(\)

#### Updating

props나 state의 변경에 의해서 component가 다시 렌더링 될 때

1. componentWillReceiveProps\(\)
2. shouldComponentUpdate\(\)
3. componentWillUpdate\(\)
4. render\(\)
5. componentDidUpdate\(\)

#### Unmounting

component가 DOM에서 제거될 때

1. componentWillUnmount\(\)



