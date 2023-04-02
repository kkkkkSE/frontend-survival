---
description: 커스텀 훅의 개념과 커스텀 훅 라이브러리
---

# Custom Hook

`useState`, `useEffect` 등의 **React Hook을 사용하여 상태 관련 로직을 재사용할 수 있게 만드는 것**이 Custom Hook이다.

클래스형 컴포넌트를 쓸 때는 고차 컴포넌트나 render props를 이용해 상태 관련 로직을 재사용하곤 했는데, Custom Hook을 이용하면 이 둘처럼 새 컴포넌트를 추가하지 않고도 상태 관련 로직을 재사용할 수 있게 해준다.

## 예시

아래는 Custom Hook의 예시다.

```js
import React, { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```

인자로 `freindID`를 받아 친구의 접속 상태를 구독하여 접속 상태가 바뀔 때 마다 온라인 / 오프라인 상태를 반환해주는 Custom Hook이다.

이는 여러 컴포넌트에서 사용되어 질 수 있다.

```js
// 친구의 정보를 확인하는 화면에서 친구의 상태를 나타내는 컴포넌트
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

```js
// 모든 친구의 접속 상태를 확인할 수 있는 친구 목록 컴포넌트
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

Custom Hook은 완전히 독립되어 있기 때문에, 내부의 state도 독립적이다. 그래서 한 컴포넌트에서 같은 Custom Hook을 사용할 수도 있다.

## 규칙

- Custom Hook의 이름은 `use`로 시작할 것
- 내부에서 다른 Hook을 호출하여 어떠한 로직을 구성할 것

이 두 가지를 만족해야 Custom Hook이라 할 수 있으며, 네이밍을 `useSomething`으로 하면 linter 플러그인이 자동으로 React Hook으로 인식하여 버그를 찾을 수 있게 도와준다.

## Custom Hook 라이브러리

### usehooks-ts

usehooks-ts 타입스크립트로 작성된 유용한 React Custom Hook을 모아놓은 라이브러리다. 실제로 흔하게 사용하는 로직들을 제공하며, Custom Hook에 대한 아이디어를 떠올리는 데도 도움이 된다.

아래에서 소개되지 않은 Hook도 많으니 [usehooks-ts 웹페이지](https://usehooks-ts.com/)를 참고하자. 각 Hook의 내부 코드와 사용 예시를 확인할 수 있다.

#### useBoolean

일반적으로 toggle 기능을 구현할 때 `useState`를 많이 사용하곤 한다.

```js
function App() {
    const [opening, setOpening] = useState(false);
    const handleClick = () => {
        setOpening(opening => !opening);
    }

    return (
    <>
        <p>opening is {opening.toString()}</p>
        <button onClick={handleClick}>toggle</button>
    </>
    )
}
```

이를 간단하게 해주는 Hook이 `useBoolean`이다.

```js
function App() {
    const { value : opening, setTrue, setFalse, toggle : toggleOpening } = useBoolean(false);

    return (
    <>
        <p>opening is {opening.toString()}</p>
        <button onClick={toggleOpening}>toggle</button>
        <button onClick={setTrue}>toggle</button>
        <button onClick={setFalse}>toggle</button>
    </>
    )
}
```

구조 분해 할당으로 `value`와 `toggle`을 기본적으로 받을 수 있다. 나머지는 옵션이다.

- `value` : 토글 시킬 변수
- `toggle` : 이벤트 핸들러로 사용 시 `value`를 토글시킨다.
- `setTrue` : 이벤트 핸들러로 사용 시 `value`를 무조건 true로 만들어준다.
- `setFalse` : 이벤트 핸들러로 사용 시 `value`를 무조건 false로 만들어준다.

모든 프로퍼티는 key-value 형태로 새로운 변수 이름을 사용할 수 있다.

Custom toggle도 가능하니 공식 문서를 참고하자.

#### useEffectOnce

`useEffect`에서 두번째 인자로 빈 배열을 전달한 것과 동일한 결과를 낸다. 렌더링 이후 한 번만 실행된다.

`useEffect`와 다른 점이라고 한다면 가독성이다. 빈 배열이 있는 지 체크할 필요 없이 이름부터 `Once`라는 단어가 들어가 직관적으로 알 수 있게 해준다.

```js
useEffect(() => {
console.log('일반적인 useEffect', { data })
}, [])
```

```js
useEffectOnce(() => {
console.log('useEffectOnce', { data })
})
```

#### useFetch

단순히 데이터를 받아올 때 사용하기 좋다.

```js
export default function fetchProducts() {
  const url = `https://example.com/products`
  const { data, error } = useFetch(url)

  if (error) return error;
  return data;
}
```

하지만 옵션이 별로 없어서 상황에 따라 다른 라이브러리를 사용하는 것이 더 나을 수도 있다.

- [use-http](https://use-http.com/#/) : loading 상태 관리, 네트워크 요청할 때 GET 뿐만 아니라 다른 메서드도 사용 가능 등

데이터를 가져오는 걸로 그치지 않고 캐싱이나 페이지네이션, 네트워크 상태 체크, 중복 쿼리 체크, 재검증 등의 기능을 활용하고 싶을 땐 다음 라이브러리를 사용할 수 있다.

- SWR : 데이터를 요청할 때 먼저 캐시에서 데이터를 불러오고, 검증을 위해 요청(revalidate)을 보낸 뒤 최신 데이터로 업데이트 한다. 자동으로 데이터 업데이트가 이루어져 지속적으로 최신 상태를 유지할 수 있다.
- TanStack Query(React-Query) : 캐싱, 업데이트, 에러 핸들링 등의 비동기 과정을 편리하게 할 수 있도록 제공한다.

#### useInterval

`useEffect`에서 `setInterval`을 사용할 때 버그가 생기기 쉬워 `useInterval`을 사용하는 것을 권장한다.

만약 interval을 등록하는 행위를 단 한번만 해서 count 되도록 하고싶다면 아래와 같이 코드를 짤 수도 있다.

```js
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return <h1>{count}</h1>;
}
```

하지만 React의 입장에서 이는 우리가 거짓말을 했다고 생각한다. 우리가 짠 코드의 내용을 살펴보면, 한 번만 실행해달라고 했으면서 effect 내에서 1초마다 바뀐 count을 참조하여 매번 다른 effect 함수를 실행하도록 만든 것이다.

정리해보자면 effect 함수의 내용은 'count를 참조할거야' 라 해놓고 의존성 배열 안에는 '아무 값에도 의존하지 않을거야'라고 한 셈이 된다.

React는 '아무 것도 의존 안 해? ㅇㅋ! 그럼 count 지금 0이니까 그것만 준다~!' 해서 결과적으론 처음에 count를 하고(0 + 1), 그 이후 1초마다 계속 `setCount(0 + 1)`이 실행되는 것이다.

이를 해결하기 위해서는 effect 함수 내에서 사용할 변수를 자급자족 하도록 만들어야 한다. `setCount(count + 1)`은 반드시 count 변수를 직접적으로 참조해야 하지만, `setCount(c => c + 1)`은 'count 변수를 직접 가져다 줄 필요는 없고, 너는 이미 count 값 알고 있으니 거기서 알아서 1 더해다 저장해 줘' 하도록 하는 것이다.

이러한 이유로 `setInterval`을 조심히 사용해야 하며, 이런 점을 신경쓰지 않고 간단히 쓸 수 있게 해주는 것이 `useInterval`이다.

```js
export default function Component() {
  const [count, setCount] = useState<number>(0)

  useInterval(
    () => {
      setCount(count + 1)
    }, 1000
  )

  return (
    <>
      <h1>{count}</h1>
    </>
  )
}
```

`useInterval`의 두번째 인자인 interval 시간을 `null`로 바꾸면 interval을 일시정지 한다. state로 조정해서 언제든지 일시정지 / 시작 이 가능한 스탑워치 같은 기능도 만들 수 있다.

#### useEventListener

각종 `window`, `document`, `element`, 커스텀 이벤트를 지원한다. 간편하게 다양한 이벤트 리스너를 추가할 수 있다.

```js
import { useRef } from 'react'
import { useEventListener } from 'usehooks-ts'

export default function Component() {
  const buttonRef = useRef<HTMLButtonElement>(null)

  const onScroll = (event: Event) => {
    console.log('window scrolled!', event)
  }
  const onClick = (event: Event) => {
    console.log('button clicked!', event)
  }

  useEventListener('scroll', onScroll)
  useEventListener('click', onClick, buttonRef)

  return (
    <div style={{ minHeight: '200vh' }}>
      <button ref={buttonRef}>Click me</button>
    </div>
  )
}
```

첫번째 인자로 이벤트 종류, 두번째 인자로 이벤트 핸들러, 세번째 인자로 이벤트를 받는 요소를 넣어줄 수 있다.

#### useLocalStorage

로컬 스토리지는 웹 브라우저에서 만료 날짜와 관계없이 데이터를 저장할 수 있는 공간이다. 브라우저가 새로고침 되어도, 껐다 켜도 이 데이터는 유지된다. 사용자의 설정 정보(다크모드를 사용하거나 웹의 언어를 한국어로 설정하는 등)를 저장하기에 좋다.

```js
export default function Component() {
  const [isDarkTheme, setDarkTheme] = useLocalStorage('darkTheme', true)

  const toggleTheme = () => {
    setDarkTheme((prevValue: boolean) => !prevValue)
  }

  return (
    <button onClick={toggleTheme}>
      {`The current theme is ${isDarkTheme ? `dark` : `light`}`}
    </button>
  )
}
```

로컬 스토리지에 데이터가 저장될 때는 key-value 쌍으로 저장된다. 위의 `useLocalStorage()` 괄호 안 첫번째 인자는 key, 두번째 인자는 value 초기 값이다.

이 방식으로 로컬 스토리지에 데이터를 담아 여러 컴포넌트에서 사용했다면, 로컬 스토리지 데이터가 바뀔 때마다 모든 사용처에서 알아서 새 데이터를 반영해준다.

실제로 어떤 값이 담겨있는지 확인하고 싶다면 다음과 같이 console을 찍어볼 수 있다.

```js
console.log(localStorage.getItem('darkTheme'))
```

#### useDarkMode

사용자가 사용 중인 운영체제에서 다크모드 여부를 확인하여 적용되어 있는 상태를 가지고 온다.

```js
import { useDarkMode } from 'usehooks-ts'

export default function Component() {
  const { isDarkMode, toggle, enable, disable } = useDarkMode()

  return (
    <div>
      <p>Current theme: {isDarkMode ? 'dark' : 'light'}</p>
      <button onClick={toggle}>Toggle</button>
      <button onClick={enable}>Enable</button>
      <button onClick={disable}>Disable</button>
    </div>
  )
}
```

- isDarkMode : 다크모드 사용 중일 시 true, 아닐 시 false 반환
- toggle : 현재 상태의 반대 값을 반환
- enable : isDarkMode를 true로 변환
- disable : isDarkMode를 false로 변환
