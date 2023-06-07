# React Hook

React Hook은 클래스형 컴포넌트에서만 가능했던 state 관리와 생명주기 메서드를 대체하여 함수형 컴포넌트에서도 편하게 할 수 있게 도와주고, 여러 React의 기능을 편하게 쓸 수 있도록 도와주는 함수들이다.

## 등장 배경

Hook이 등장하기 전 수년동안 React 팀과 React 사용자들은 아래 3가지와 같은 여러가지 문제점을 인지해왔고, 이를 해결하기 위해 Hook이 등장했다.

### 고차 컴포넌트 : Wrapper Hell

컴포넌트 로직을 재사용하기 위한 방법으로 고차 컴포넌트(High Order Component)를 사용했다.

고차 컴포넌트는 중첩된 여러 역할을 가진 컴포넌트들이 횡단 관심사(Cross-Cutting Concerns, 공통된 관심사)를 가지면서 이를 어떻게하면 재사용할 수 있을지에 대해 고민한 결과물이다. 같은 기능의 코드가 다수의 컴포넌트에 포함되어야 한다면, 하드 코딩보다는 하나를 만들어놓고 재사용하는 편이 훨씬 편하기 때문이다.

재사용하기 위한 컴포넌트는 UI 컴포넌트를 감싸는 형태로 사용할 수 있으며, 리액트 개발자 도구에서 본 구조는 다음과 같아진다.

```js
<App>
  <WithClickEvent>
    <Header>
  <WithClickEvent>
    <Button>
```

재사용되는 컴포넌트와 UI 요소가 별로 없다면 구조를 파악하기 쉽지만, 여러 개의 고차 컴포넌트를 사용해야 하는 경우도 있을 것이다.

```js
  <App>
    <WithMouseEvent>
        <WithWindowEnvet>
            <WithScrollEvent>
                <UIComponent> 
```

이 경우 불필요하게 depth가 커지게 되어 복잡하고, 디버깅할 때 불편함을 느낄 수 밖에 없다.

고차 컴포넌트를 Hook으로 대체하면 다음과 같아진다.

```js
function App(){
  const mouseEvent = useMouseEvent()
  const windowEvent = useWindowEvent()
  const scrollEvent = useUserEvent()
  
  return (
    <UIComponent
      mouseEvent={mouseEvent}
      windowEvent={windowEvent}
      scrollEvent={scrollEvent}
    />
  )
}
```

사용하기 편리하고 가독성도 훨씬 좋아진다.

### 이해하기 어려운 복잡한 컴포넌트

생명주기 메서드는 각각 맡은 역할이 있지만 관련 없는 로직이 섞여들어갈 때도 있다. 원래 데이터를 가져오는 데 사용되어야 하지만 이벤트 리스너를 설정하거나, 엉뚱한 곳에서 cleanup 로직을 수행하기도 한다. 이로 인해 버그가 발생할 가능성이 높고, 쉽게 무결성을 해칠 수 있다.

위와 같은 상황에서 상태 관련 로직 때문에 컴포넌트를 작게 분리하는 것도 힘들고, 테스트하기도 어렵다.

### 클래스 자체의 불편함

클래스를 사용함으로써 많은 불편함이 따라왔다.

- 항상 `extends`를 사용해 `React.Component`를 상속받아야 해서 번거롭다.
- state를 사용하기 위해선 `constructor` 내부 상단에 `super(props)`를 매번 작성해야 해서 번거롭다.
- 자바스크립트의 `this`에 대한 이해가 필수다. 대부분의 다른 언어와 다르게 작동했기에 혼란을 주기도 한다.
- 이벤트 핸들러의 등록 방법을 정확히 알아야한다.
- 코드의 최소화를 힘들게 한다.

## Hook 사용 규칙

_React Hook 사용 규칙을 지키기 위해 ESLint Plugin을 제공하고 있다._

- React 함수 내부 또는 Custom Hook 내부에서만 Hook을 호출할 것
- 최상위 레벨에서만 Hook을 호출할 것
  - 반복문, 조건문 등의 블록 내에서 Hook을 사용하지 말 것 -> 반대로 Hook 내부에서 사용하면 OK

React는 Hook이 호출되는 순서에 의존한다. 특정 state가 어떤 `useState` 호출에 해당하는지를 Hook 호출 순서에 의해 알 수 있다는 의미이다.

렌더링이 반복될 때마다 같은 순서로 Hook이 호출되면 state를 각 Hook에 연동할 수 있다. 조건문이나 함수에 의해 상황에 따라 호출되지 않을 수도 있다면 React는 자신이 기억하고 있는 순서와 달라져 버그를 발생시킬 수 있다는 것이다.

이러한 이유로 Hook은 컴포넌트 내의 최상위 레벨에서 호출되어야만 한다.

## useState

함수형 컴포넌트에서도 state를 사용할 수 있게 해주는 Hook이다.

### 선언하기

```js
import React, { useState } from 'react';

function Example() {
  const [count, setCount] = useState(0);
  // ...
}
```

1. `import { useState } from 'react'`로 useState를 호출하여 state 변수를 선언할 수 있게 된다.
2. `useState()`는 **state 변수**(count)와 **이 변수를 갱신할 수 있는 함수**(setCount)를 반환한다. (구조 분해 할당*)
3. `useState()` 괄호 안에는 **state의 초기 값**을 넣어줄 수 있다. state 값은 다양한 타입을 사용할 수 있다.

state를 갱신하기 위한 함수는 관례적으로 네이밍할 때 `set` + `state 변수명`으로 짓는다.

{% hint style="info" %}
구조 분해 할당을 이용하면 `useState()`가 반환하는 아이템을 바로 새로운 변수에 저장할 수 있고, 코드를 좀 더 직관적으로 파악할 수 있다. 아래 두 코드는 동일한 코드다.

```js
var countStateVariable = useState(0);
var count = countStateVariable[0];
var setCount = countStateVariable[1];
```

```js
const [count, setCount] = useState(0);
```

{% endhint %}

여러 개의 state는 다음과 같이 선언할 수 있다.

```js
function ExampleWithManyStates() {
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
  // ...
}
```

state의 초기값이 간단한 값이 아니라면, 함수를 사용하여 반환해줄 수도 있다.

```js
const [state, setState] = useState(() => {
  const initialState = someExpensiveComputation(props);
  return initialState;
});
```

### 갱신하기

useState가 state를 갱신하기 위해 반환한 함수(setCount)를 호출하여 state를 갱신한다.

```js
import React, { useState } from 'react';

function Example() {
    const [count, setCount] = useState(0);

    return(
        <div>
            <button onClick={() => setCount(count + 1)}>
                Click : add 1
            </button>
        </div>
    );
}
```

이전 state를 사용하여 새로운 state를 계산하는 것도 가능하다.

```js
function Counter({initialCount}) {
  const [count, setCount] = useState(initialCount);
  return (
    <>
      Count: {count}
      <button onClick={() => setCount(initialCount)}>Reset</button>
      <button onClick={() => setCount(prevCount => prevCount - 1)}>-</button>
      <button onClick={() => setCount(prevCount => prevCount + 1)}>+</button>
    </>
  );
}
```

만약 갱신된 state가 이전 state와 동일하다면(`Object.is`로 비교하는 로직) React는 하위 컴포넌트를 렌더링하거나 어떤 것을 실행하지 않고 처리를 종료한다.

## useEffect

함수형 컴포넌트에서 사이드 이팩트를 수행할 수 있게 해주는 Hook이다.

_선언형 프로그래밍에선 사이드 이팩트가 실행 상태를 예측하기 어렵게 한다 하여 이를 지양하고 있다._

{% hint style="info" %}
사이드 이팩트 : 컴포넌트가 렌더링 된 이후 **컴포넌트 외부**에 존재하는 값이나 상태를 변경시키는 행위 등 비동기로 처리되어야 하는 부수적인 효과들을 사이드 이팩트라고 한다.
{% endhint %}

effect 함수가 실행되고 `useEffect`는 추가로 코드를 실행해야 하는 경우가 있는데, 이를 Clean-up이라고 한다. React 컴포넌트에는 Clean-up을 사용하는 사이드 이팩트와 사용하지 않는 이팩트 두 종류로 나눠져있다.

네트워크 데이터 송신, 수동으로 컴포넌트의 DOM 수정, 로깅 등의 작업은 작업 이후 정리(Clean-up)가 필요하지 않지만, 이벤트 리스너를 등록하거나 외부 데이터에 구독을 설정하는 경우에는 Clean-up 함수가 필요하다.

Clean-up 함수를 사용하면 다음 렌더링 이후 effect가 다시 발생했을 때 이 전에 파생됐던 effect를 정리하여 메모리 누수와 버그를 방지할 수 있다.

### 특징

- `useEffect`는 컴포넌트의 렌더링 이후에 어떤 일을 수행해야 하는지 React에게 알려준다. React는 우리가 넘겨준 함수를 기억했다가 DOM 업데이트가 끝나면 함수를 호출한다.
- `useEffect`는 컴포넌트 안에서 사용한다. 컴포넌트 내에서 실행함으로써 컴포넌트 내 변수에도 접근할 수 있게 된다.
- `useEffect`는 매 렌더링마다 실행된다. effect가 수행될 시점에는 DOM의 업데이트가 완료되어 있다.
- Clean-up이 필요한 경우, `useEffect`에서 함수를 반환할 수 있다. Clean-up은 React 컴포넌트가 언마운트 될 때 실행된다.

### 사용해보기

#### 기본 사용법

```js
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

1. `import { useEffect } from 'react'`로 `useEffect`를 호출한다.
2. `useEffect()`의 괄호 안에 함수를 넣으면 매 렌더링 이후마다 실행된다.

위의 코드에서 `useEffect`는 문서의 타이틀을 변경하고 있는데, 같은 컴포넌트 내에 있는 state를 바로 얻어서 활용할 수 있다.

state 값이 변경되면 렌더링이 일어나고, 렌더링 이후에는 effect 함수가 실행된다. effect 함수에서 state값을 활용하고 있다면 함수 실행 결과는 매번 다를 것이다. 이로써 `useEffect`에 전달된 함수가 매 렌더링마다 다르다는 것을 알 수 있다. state 값이 제대로 반영되는지에 대한 걱정을 할 필요가 없다.

#### Clean-up 사용법

```js
useEffect(()=> {
  function handleScroll() {
    console.log(window.scrollY);
  }
  document.addEventListener("scroll", handleScroll); // 구독

  return () => { // Clean-up 함수
    document.removeEventListener("scroll",handleScroll)
  }
}, []);
```

1. 외부 이벤트를 구독한다.
2. Effect 함수에서 Clean-up 함수를 반환시킨다.
3. 컴포넌트가 언마운트 될 때 Clean-up 함수로 인해 구독을 제거한다.

Clean-up 함수를 `useEffect`에서 반환함으로써 구독의 추가와 제거 로직을 가까이 묶어둘 수 있다.(로직의 연관성)

#### Effect 건너뛰기

모든 렌더링마다 실행되는 `useEffect`가 성능저하의 원인이 되어 곤란한 상황이 생길 수도 있다. 이 때, `useEffect`의 두번째 인자를 이용해서 effect를 건너뛸 수 있다.

```js
useEffect(() => {
  document.title = `Success Change`;
}, []);
```

두번째 인자로 빈 배열을 넘겨주면, 최초 렌더링 이후에만 effect 함수를 실행한다. 그 뒤에 렌더링이 이뤄져도 effect 함수는 실행되지 않는다.

{% hint style="info" %}
두번째 인자에 빈 배열을 넣었을 때, 간혹 effect 함수가 두번씩 실행되는 것을 볼 수 있다. 이는 아래와 같은 이유로 두번 실행된다.

1. 해당 컴포넌트가 페이지 내에서 두번 사용했을 경우
2. 상위 컴포넌트에서 해당 컴포넌트가 언마운트 또는 리마운트 됐을 경우. 상위 컴포넌트에서 key가 변경되거나 했을 때 등이 이 경우에 속한다.
3. React StrictMode를 사용했을 경우. ScrictMode는 개발 단계에서 사전에 발생할 수 있는 에러를 파악하여 예기치 못한 오류를 찾아내기 위해 일부 메서드가 2번씩 호출된다고 한다.

{% endhint %}

```js
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]);
```

두번째 인자의 배열 안에 변수가 들어가면, 최초 렌더링 + 해당 변수가 변경된 뒤에 렌더링이 일어났을 경우에만 effect 함수가 실행된다.

배열 안의 변수가 변경되고 렌더링이 일어나면 React는 이전에 렌더링 된 값과 그 다음 렌더링 시의 값을 비교하고, 그 두 값이 같지 않으면 effect를 실행하게 되는 것이다.

이 배열 안에는 여러 개의 변수가 들어갈 수 있으며, 그 중 하나라도 이전 렌더링과 비교했을 때 변경되었다면 effect 함수가 실행된다.

이렇게 의존성 배열을 이용하여 데이터를 받아올 때는 [주의사항](https://react.dev/learn/synchronizing-with-effects#fetching-data)이 있다. 참고하자.

#### Multiple Effect

관심사의 구분을 위해 여러 개의 `useEffect`를 사용할 수 있다.

```js
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }
  });
  // ...
}
```

## useLayoutEffect

전체적인 내용은 `useEffect`와 동일하나, 다른 점이 하나 있다.

- `useEffect` 함수는 컴포넌트의 렌더링 이후 화면이 다 그려지고 나서(paint) 비동기적으로 실행된다. 만약 DOM에 영향을 주는 코드가 있다면, 화면에 깜빡임이 있을 수 있다.

화면의 깜빡임 때문에 `useEffect`에서는 보통 데이터를 가져오거나, 타이머, 구독 등의 사이드 이팩트 작업이 권장된다.

- 반면에 `useLayoutEffect`는 컴포넌트의 렌더링 이후, paint 이전에 동기적으로 실행된다. `useLayoutEffect` 함수 내 코드를 모두 수행하고 화면을 그리므로 화면이 나올 때까지 약간의 로딩이 있을 수 있다.

DOM 요소의 변화로 화면의 깜빡임으로 사용자가 불편한 경우가 생길 수 있다면, `useLayoutEffect`를 사용하는 것이 더 나을 수 있다.

## useContext

### context

일반적으로 React에서는 데이터가 위에서 아래로 흐르며 데이터가 이동하기 위해선 props로 전달한다. 하지만 어떤 특정 값을 여러 컴포넌트에 전달해야 할 경우 props만으로 전달하기에 아주 번거로울 수 있다. 이 때, 명시적으로 차례차례 props를 넘겨주지 않고도(props drilling) 한 번에 여러 컴포넌트가 특정 값을 전달받을 수 있게 하는 것이 context다.

context는 전역적으로 사용할 수 있도록 고안된 방식이다. 이런 방식에 맞는 데이터로는 로그인한 유저 관련 데이터나 애플리케이션 테마, 선호하는 언어 등이 있다.

다만 context를 사용하면 컴포넌트를 재사용하기 어려울 수 있어 꼭 필요할 때 쓰라고 강조하고 있다.

또한, props drilling 이슈로 context를 고려중이라면 [컴포넌트 합성](https://ko.reactjs.org/docs/context.html#before-you-use-context)을 이용하는 것이 더 좋을 수 있다.

### context 사용법

1. `React.createContext` 를 사용하여 context 생성(초기값 세팅 가능)
2. `<ContextName.Provider>`로 context를 전달할 컴포넌트들의 최상위 컴포넌트 감싸기
3. `value` 프로퍼티를 사용하여 context로 전달해줄 값을 입력
4. `<UserContext.Consumer>`를 통해 전달받을 값을 사용. context의 변화를 구독한다. 단, `<UserContext.Consumer>`의 자식은 무조건 함수여야 한다.

```js
const themes = {
  light: {
    foreground: "#000000",
    background: "#eeeeee"
  },
  dark: {
    foreground: "#ffffff",
    background: "#222222"
  }
};
const ThemeContext = React.createContext(themes.light); // 1. context 생성, 괄호 안에 초기값 세팅


function App() {
  return ( // 2. Provider로 전달할 최상위 컴포넌트 감싸기
    <ThemeContext.Provider value={themes.dark}> {/* 3. 전달할 값 세팅 */}
      <ThemedButton />
    </ThemeContext.Provider>
  )
}

function ThemedButton() {
  return ( // 4. Consumer로 값 전달 받아 사용하기
    <ThemeContext.Consumer>
      {value => (
        <button style={{ background: value.background, color: value.foreground }}>
          I am styled by theme context!
        </button>
      )} 
    </ThemeContext.Consumer>
  )
}
```

동일한 context의 Provider가 여러개 존재한다면 Consumer는 가장 가까운 Provider에게서 value를 받아오며, 상위 컴포넌트들 중에 Provider가 없다면 컨텍스트를 만들 때 정해줬던 초기값을 사용한다.

### 기존 사용법 보완 : useContext

`useContext`를 사용하면 Consumer 없이 context를 사용할 수 있다.

```js
const themes = {
  light: {
    foreground: "#000000",
    background: "#eeeeee"
  },
  dark: {
    foreground: "#ffffff",
    background: "#222222"
  }
};

const ThemeContext = React.createContext(themes.light);

function App() {
  return (
    <ThemeContext.Provider value={themes.dark}>
      <ThemedButton />
    </ThemeContext.Provider>
  );
}

// 기존 방법과 다른 부분
function ThemedButton() {
  const theme = useContext(ThemeContext);
  return (
    <button style={{ background: theme.background, color: theme.foreground }}>
      I am styled by theme context!
    </button>
  );
}
```

- `useContext()`의 괄호 안 인자는 **context 객체 그 자체**여야 한다.
- `useContext`를 호출한 컴포넌트는 context 값이 변경되면 항상 리렌더링 된다.

### 주의

context가 내려주는 값을 업데이트 하는 것은 권장되지 않는다. context가 변경되면 그 context를 사용하고 있는 모든 컴포넌트에서 재렌더링이 일어난다. 애초에 자주 변경되지 않는 테마로 context를 사용했다면 모를까, 그렇지 않은 경우는 성능상의 문제로 이어질 것이다.

## useRef

### Ref

HTML에서 특정 DOM 요소를 선택하여 어떤 행위를 하려면 `id`를 사용했다.

```js
// html
<div id='my-element'></div> 

// javascript
document.getElementById('my-element');
```

리액트에서는 `id` 대신 `ref`를 사용한다. `id`를 사용하면 안 되는 건 아니지만, 컴포넌트의 재사용 측면에서 `id`가 중복될 가능성이 있어 사용하지 않는 편이 좋다.

`ref`는 `id`처럼 DOM 노드나 React Element에 접근할 수 있게 해준다. `id`는 `documetn.getElementById`로 엘리먼트에 접근할 수 있지만, `ref`가 엘리먼트에 접근하는 방법은 다음과 같다. 아래는 클래스형 컴포넌트의 `ref` 사용 예시다.

```js
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    this.textInput = React.createRef(); // 1. ref 생성
    this.focusTextInput = this.focusTextInput.bind(this);
  }

  focusTextInput() {
    this.textInput.current.focus(); // 3. ref에 접근
  }

  render() {
    return (
      <div>
        <input
          type="text"
          ref={this.textInput} /> {/* 2. 생성한 ref를 엘리먼트에 부착 */}
        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```

`ref`가 HTML 엘리먼트에 쓰였다면, 엘리먼트 자체를 `.current`로 받는다. 클래스형 컴포넌트에서는 엘리먼트 외 다른 곳에서도 사용하지만, 지금은 클래스형 컴포넌트를 거의 사용하지 않으므로 생략한다.

함수형 컴포넌트에서 이 `ref`는 작동하지 않는다. 대신 React Hook 중 하나인 `useRef`를 사용한다.

### 함수형 컴포넌트의 ref : useRef

`useRef`를 사용하는 예시는 다음과 같다.

```js
function TextInputWithFocusButton() {
  const inputEl = useRef(null); // 1. ref 객체 생성, 괄호 내 인자는 .current 초기값
  const onButtonClick = () => {
    inputEl.current.focus(); // 3. useRef에 부착된 엘리먼트 조작
  };
  return (
    <>
      <input ref={inputEl} type="text" /> {/* 2. useRef를 엘리먼트에 부착  */} 
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```

`useRef()`는 아래와 같은 객체를 반환한다.

```js
{current: ...}
```

`ref`와 마찬가지로 HTML 엘리먼트에서 `useRef`를 사용하면 엘리먼트 자체를 `.current`로 받는다. React는 노드가 변경될 때마다 `.current`에 변경된 DOM 노드를 저장시킬 것이다.

### ref보다 유용한 useRef

`useRef()`는 가변값을 유지하는 데에 편리하다.

1. `useRef()`는 **매 렌더링마다 동일한 ref 객체를 제공**한다. `{current: ...}` 모양의 객체 자체를 만드는 것과의 차이점이다.
2. state와는 반대로 **값이 변경되었다 하더라도 리렌더링을 일으키지 않는다.** 실제 값이 바뀔 순 있어도 그것을 직접 알려주지는 않는 것이다.

### 그래서, 어디서 쓰지?

DOM 엘리먼트에 직접 관여하기 위해서 사용한다. state나 기타 다른 방식으로 하지 못하는 기능을 useRef를 통하면 가능한 경우가 있다. 예시로는 특정 input에 포커스 주기, 스크롤 박스 조작하기, canvas 요소에 그림 그리기 등이 있다.

## useLocation

`useLocation()`은 `window.location` 객체를 반환한다. location 객체는 URL에 포함된 각종 정보를 포함하고 있다.

- pathname : 도메인을 제외한 현재 위치의 경로를 표기한다.
- search : 쿼리 스트링을 표기한다.

```js
import { useLocation } from 'react-router';

export default function App() {
  const location = useLocation();
  const queryString = location.search;

  // location.pathname = /pathname/id
  // location.search = ?query=value

  return (
    <div>
      {queryString}
    </div>
  );
}

```

## useSearchParams

쿼리 스트링에 담겨있는 값만 사용하고 싶을 때 사용하는 hook이다. `useLocation`의 search를 사용해서 쿼리 값을 가져오려면 가공 과정이 필요하지만, `useSearchParams`를 사용하면 객체 Key 값으로 Value를 가져오는 것처럼 값만 가져올 수 있다.

```js
import { useSearchParams } from 'react-router-dom';

export default function OrderCompletePage() {
  const [searchParams] = useSearchParams();
  const orderId = searchParams.get('orderId');

  return (
    <div>
      {orderId} {/* 값만 표기 됨 */}
    </div>
  );
}
```
