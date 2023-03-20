# React

오래 전 웹은 브라우저에 화면을 띄우는 수준에 그쳤다. 점차 요구사항이 많아지고 기능이 추가되면서 웹의 덩치가 커져갔으나 기존의 기술로는 모든걸 감당하기 버거웠다. 웹 개발의 복잡성이 증대함에 따라 다양한 웹 프레임워크(라이브러리)가 탄생했고, 이 중 하나가 React다.

## React

* 사용자 인터페이스를 구축하기 위한 자바스크립트 라이브러리다.
* **SPA**(Single Page Application) 개발이 가능하여 높은 UX 수준을 가진 앱같은 웹을 제공할 수 있다.
* 사용자와 상호작용이 많은 동적 UI를 개발하는 데 용이하다.
* **컴포넌트** 단위를 사용하며, 이는 **UI 재사용**과 밀접한 관련이 있다.
* View 설계에 초점이 맞춰져있다. 따라서 다른 기능을 이용하고 싶다면 Third-party 라이브러리를 사용할 수 있다.

## React의 특징

### Component

React는 함수형에서는 `return` , 클래스형에서는 `render()` 메서드로 화면을 그려낸다. 그 내부에는 화면을 그려내는 부분 뿐만 아니라, 그려낼 화면을 조작하는 여러가지 로직이 포함될 수 있다. 이러한 함수 또는 클래스를 **Component** 라고 하며, React에서는 Component를 조합하여 화면을 구성한다.

즉, 화면의 아주 작은 단위도 Component가 될 수 있고, 필요에 따라 **재사용**이 가능하다.

### 단방향 데이터 흐름과 Props

React 안에서 데이터는 기본적으로 위에서 아래로 흐른다. 부모 컴포넌트에서 자식 컴포넌트에게 데이터를 전달하기 위해서 **Props**를 사용할 수 있다. _단, 전달받은 Props는 읽기 전용이다._

### State

사용자의 행동(이벤트)에 따라 **데이터에 변화를 주기 위해서는 State**를 사용해야 한다. State가 변화하면 리렌더링이 일어난다.

또한 단방향 데이터 흐름 관점에서 봤을 때, React에서 반드시 부모만 자식에게 데이터를 전달할 수 있는 것은 아니다. State를 이용하면 부모의 데이터를 변경할 수 있다.

```javascript
function Parent() {
    const [search, setSearch] = useState('');
    const changeSearch = (value) => {
        setSearch(value);
    };
    return (
        <div>
            <Child changeSearch={changeSearch} />
        </div>
    )
}

function Child({ changeSearch }) {
    return (
        <div>
            <input 
                type="text"
                onChange={(event)=> changeSearch(event.target.value)} /> 
        </div>
    )
}
```

### Virtual DOM

일반적으로 DOM을 조작하게 되면 Render Tree 재생성, Reflow, Repaint를 야기시킨다. 사용자와 상호작용이 빈번하게 일어나는 웹 앱의 경우, 이는 성능 저하의 큰 요인이 된다.

**Virtual DOM**은 이 과정에서 불필요한 부분을 단축하고 **최소한의 범위만을 업데이트** 한다.

1. State가 변경되면 Virtual DOM에 먼저 적용시킨다.
2. 이전 Virtual DOM의 스냅샷과 비교한다. (diffing)
3. 변경된 부분만을 업데이트 시킨다.

{% hint style="info" %}
최소한의 범위만을 업데이트 한다는 것은 Reflow 관점에서의 얘기이다.

화면 업데이트와 관련 없는 하위 컴포넌트라 할지라도 render() 함수는 실행되어 그 내부의 것도 실행된다는 것을 인지하자.
{% endhint %}

### JSX

JSX는 **자바스크립트에서 HTML과 유사하게 사용**할 수 있는 문법이다. JSX를 사용해서 Element를 작성하면 가독성이 좋고, 코드가 간결해진다.

[JSX 페이지](../week2/jsx.md)에서 더 많은 내용을 볼 수 있다.

```javascript
function App() {
    const name = 'Kim';
    return (
    <>
    	<h1>{name}</h1>
    </>
  );
```

### key prop

화면을 구성하다 보면 .map과 같은 순회 메서드를 사용하여 리스트를 작성하기도 한다. 이 때, React는 각 리스트를 명확히 구분하기 어려워진다. 차후 요소를 추가, 수정, 삭제할 때 **각 요소를 식별하기 위해 React는 각 리스트의 고유값을 요구**하며, 이것이 바로 **key**이다.

* key 값으로는 데이터의 ID를 사용하는 것이 무난하다.
* 차후 리스트의 순서가 바뀌게 될 수 있으니 index 값을 사용하는 것은 지양하자.
* 모든 key값에 대해 고유할 필요는 없다. 해당 리스트 내에서 형제 요소에 대해서만 고유하면 된다.

### Inversion of Control

우리는 다른 컴포넌트를 호출할 때 `Component();` 방식으로 직접 호출하지 않고 `<Component />` 형식으로 호출한다. 이 경우 컴포넌트의 호출 제어권이 React에게 넘어간다.

\*\*제어의 역전(Inversion of Control)\*\*은 이처럼 어떤 일을 하기 위해 개발자가 모든 것을 처리하는 것이 아닌 **외부에 제어권을 위임**하는 것이다.

React에게 제어권을 위임하게 되면 React가 컴포넌트의 구성을 인지하고, DOM 조작으로 인한 화면 업데이트 등을 하게 된다. 이로 인해 개발자는 핵심 비즈니스 로직에 더 집중할 수 있다.

## React 렌더링과 리렌더링

렌더링은 DOM 트리 바탕으로 화면을 그리는 것을 의미한다.

사용자의 행동이나 직접 등록한 이벤트 등에 의해 **State가 변화하게 되면** props와 관계없이 자신을 포함한 모든 하위 컴포넌트에 **리렌더링이 발생**한다.

#### 의도대로 리렌더링 되지 않았다면?

* setState 함수, useState hook 으로 state를 업데이트 했는지 확인하자. state를 props로 받고, props를 변화시켜봤자 리렌더링 되지 않는다.
* 동일한 값을 참조한 게 아닌지 확인하자.

### Memoization

모든 하위 컴포넌트가 리렌더링 되는 것이 부담스럽다면 `React.memo` 또는 `useMemo()`를 사용할 수 있다.

이는 이전 렌더링과 현재 렌더링을 비교하여 입력된 인자가 동일하다면 함수를 실행하지 않고 결과값을 그대로 활용할 수 있게 해준다.

```js
function App({ numbers }) {
  const sortNums = () => return numbers.sort();
  const sortedNums = useMemo(sortNums, [numbers]);
  
  return (
    <>
      <ul>
        {sortedNums.map((num, idx) => (
          <li key={num}>{num}</li>
        ))}
      </ul>
    </>
  );
} // 부모 컴포넌트에 리렌더링이 일어나더라도 해당 컴포넌트에서 numbers가 동일하다면 리렌더링 되지 않음
```
