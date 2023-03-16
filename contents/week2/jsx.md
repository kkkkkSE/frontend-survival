# JSX

JSX는 자바스크립트를 HTML과 유사하게 사용할 수 있는 확장 문법이다.

## React에서 왜 JSX를 사용할까?

React는 본질적으로 UI를 조작하는 여러가지 로직과 렌더링 로직이 긴밀하게 연결되어 있다. 이 두가지 로직을 억지로 분리하지 않고, [컴포넌트](https://sienna-organization.gitbook.io/dev-road/contents/week1/react#component)라는 단위 안에 두 로직을 모두 포함하여 사용하고 있다.

이런 점에서 JSX가 React와 잘 맞는다. **자바스크립트 안에서 UI 작업**을 함께 할 수 있기 때문에 **직관적으로 코드를 짤 수 있고 가독성도 좋다.**

또한, JSX는 React Element를 쉽게 생성할 수 있도록 **Syntactic Sugar**를 제공한다. 아래의 두 코드는 동일한 결과를 나타내지만, JSX로 표현한 코드가 훨씬 이해하기 쉽다.

```js
// React.createElement
import App from './App.js';

React.createElement(
    App, 
    {className: "app", color: "red"}, 
    React.createElement("h1", null, "Title"), 
    React.createElement("p", null, "Description")
);
```

```js
// JSX
import App from './App.js';

<App className="app" color="red">
  <h1>Title</h1>
  <p>Description</p>
</App>
```

{% hint style="info" %}
Syntactic Sugar : 코드의 기능은 유지하면서 읽는 사람이 쉽게 이해할 수 있도록 코드를 간결하게 표현하는 것을 말한다. 우리가 늘 사용하는 삼항 연산자나 화살표 함수 등도 이에 속한다.
{% endhint %}

이러한 이유로 대부분 React와 JSX를 함께 사용하지만, React에서 반드시 JSX를 사용해야 되는 것은 아니라는 것만 참고하자.

## JSX 문법

### 기본

- JSX은 React Element 타입, Property, Children으로 구성된다.

```js
import App from './App.js';

const element1 = <h1 className="greeting">Hello World!</h1>;
const element2 = <App title={data.title} />;

/*
React Element 타입 : h1, App
Property : className, title
Children : 'Hello World!'
*/
```

- JSX 내에서 **자바스크립트 표현식을 사용할 수 있다.** 표현식을 중괄호로 감싸주면 된다.
- JSX 자체도 표현식이기 때문에 변수에 할당하거나, 함수에서 반환하거나, 반복문 사용 등 모두 가능하다.

```js
const userData = {name : "kim", level : "basic"};

function greeting(name){
    return <p>'환영합니다, ' + name + '님'</p>;
}

function App(){
  return(
    <div level={userData.level}>
        {greeting(userData.name)}!
    </div>
  );
};
```

- JSX는 JS + XML으로, XML처럼 **닫는 태그가 필수**이다.

```js
function App(){
  return(
    <div>
        <h1>닫는 태그는 필수다. Children이 없다면 아래처럼 해주자.</h1>
        <SubPage />
    </div>
  );
};
```

### React Element 타입

- 타입을 컴포넌트로 지정하려면 무조건 대문자로 시작해야 한다. 대문자로 시작하지 않으면 HTML 태그로 인식한다.
  - 만약 컴포넌트명의 첫 글자를 대문자로 하기 어렵다면, 대문자로 시작하는 변수에다 컴포넌트를 할당하여 타입으로 사용한다.

- 타입이 컴포넌트일 때 점 표기법을 사용할 수 있다.

```js
const Components = {
  Greeting: function Greeting() {
    return <div>Hello World!</div>;
  }
}

function BlueDatePicker() {
  return <Components.Greeting />;
}
```

- 타입으로 컴포넌트를 사용하기 위해 일반적인 표현식은 사용할 수 없다. 표현식을 사용하려면 대문자로 시작하는 변수에 할당하여 사용해야 한다.

```js
import { Phone, Tablet } from './products.js';

const components = {
  phone : PhoneList,
  tablet : TabletList
};

function Story(props) {
  const ProductList = components[props.productType];
  return <ProductList />;
}
```

### Property

- 프로퍼티 명은 lowerCamelCase로 작성해야 한다.
- 프로퍼티 값을 할당하지 않으면 기본적으로 `true`가 할당된다.

### Children

- `&&`를 사용하여 조건부 렌더링을 할 수 있다.
  - 조건을 줄 `&&` 앞의 표현식은 무조건 진리값(false, null, undefined, true)을 출력해야 한다. 진리값은 렌더링되지 않는다.

```js
<div>
  {props.products.length &&
    <ProductList products={props.products} />
  }
</div>
```

위의 경우 length가 0이면 실제론 falsy한 값이지만, 이 경우는 0이 렌더링 될 가능성이 있다. 원하는대로 동작하려면 아래처럼 사용해야 한다.

```js
<div>
  {props.products.length > 0 &&
    <ProductList products={props.products} />
  }
</div>
```

## JSX 컴파일

JSX는 브라우저가 해석할 수 없다. 따라서 Babel과 같은 컴파일러로 React 코드인 `React.createElement()`로 컴파일해준다.

React는 `React.createElement()`가 문제 없이 작성됐는지 체크 후 우리가 알고있는 일반 객체로 변환한다. 이를 위해 React 라이브러리도 같은 스코프 내에 존재해야 한다.

```js
// JSX
import React from 'react';

const element = <h1 className="greeting">Hello World</h1>
```

```js
// React.createElement
const element = React.createElement(
    "h1", 
    {className : "greeting"}, 
    "Hello World"
);
```

```js
const element = {
  type: "h1",
  props: {
    className: "greeting",
    children: "Hello World"
  }
};
```


{% hint style="info" %}
이러한 객체를 **React Element**라고 한다. React는 이 객체를 읽어서 DOM을 생성하고 최신 상태로 유지하는 데 사용한다. 어떻게 DOM을 최신 상태로 유지하는지는 Virtual DOM 페이지를 참고하자.
{% endhint %}



React 17 이후 도입된 새로운 JSX transform 환경에서는 JSX를 `React.createElement()`가 아닌 `react/jsx-runtime`의 `jsx` 함수로 변환한다. 이 방식은 개발자가 `react/jsx-runtime`을 직접 참조할 필요가 없다.

```js
function App() {
  return <h1>Hello World</h1>;
}
```

```js
import {jsx as _jsx} from 'react/jsx-runtime';

function App() {
  return _jsx('h1', { children: 'Hello world' });
}
```

이 방식을 사용하려면 `@babel/preset-react`의 `runtime`을 `automatic`으로 변경해야 한다.



{% hint style="info" %}
TODO : React StrictMode에 관한 내용 정리하기, Virtual DOM 페이지 따로 만들기
{% endhint %}
