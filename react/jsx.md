# JSX

자바스크립트의 확장 문법으로, React에서 UI를 묘사하는데 사용한다. 일종의 템플릿 언어라고 생각해도 괜찮을듯.

babel같은 transpiler를 사용해야만 정상적으로 동작한다.

HTML과 유사하게 생겨 알아보기가 쉽다. 보통 Javascript에서 HTML을 표현하려면 무조건 String으로 해야해서 코드를 작성하거나 수정할 때, 복사하거나 붙여넣을 때 상당히 귀찮은 부분이 많은데 그 부분을 해소해준다.

```javascript
// javascript
function MyObject() {
  // object generate
}

const domElement = 
  "<div>" +
    MyObject() +
  "</div>";


// jsx
const jsxElement = (
  <div>
    <MyObject />
  </div>
)
```

일반적인 자바스크립트와 섞어 사용할 수 있어 if, for등의 로직을 사용하기도 편하다.

```javascript
function exampleIf(flag) {
  if (flag) {
    return <h1>TRUE!!</h1>;
  } else {
    return <h1>FALSE!!</h1>;
  }
}
```

뿐만 아니라 자바스크립트의 코드를 {/\* javascript code \*/} 같은 형식으로 내부에 끼워넣을 수도 있다.

```javascript
function jsx(myString) {
  const prefix = "MY_PREFIX";

  return (
    <h1>{prefix + myString}</h1>
  );
}
```

React DOM에서 JSX를 rendering 하기 전에 한번 escape 조치를 취하기 때문에 XSS 공격과 같은 Injection 공격들을 막아준다.

참고로, JSX의 모든 태그는 하나의 React Element로 취급된다. 그래서 보통 HTML 태그를 JSX에서 사용하더라도 실제로는 React Element를 생성하게 된다.

```javascript
// jsx
const obj = <h1 className="object">Object</h1>;

// real
const obj = React.createElement(
  "h1",
  {className: "object"},
  "Object"
);
```

