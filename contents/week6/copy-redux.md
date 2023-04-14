# Redux 따라하기

TSyringe를 이용해서 Redux가 동작하는 방식을 따라해보는 시간을 가졌다. 우선은 Redux가 뭔지, Redux에서 사용하는 함수는 어떤 것이 있는지부터 알아보자.

## Redux

Redux는 [Flux 아키텍처](/contents/week6/soc.md#flux-architecture)에 영감을 받아 만들어졌으며, 전역 상태 관리에 사용되는 라이브러리다.

### 구성 요소

- Store : 전역에서 사용되는 상태를 담고 있는 객체다. 스토어를 만들 때 리듀서 함수를 등록한다. Redux는 단일 스토어를 사용한다.
  - useSelector : 스토어에 있는 상태를 받아올 때 쓰는 hook이다. 인자로 콜백함수를 받으며, 이 콜백함수는 스토어의 상태 객체를 받아서 원하는 값을 반환할 수 있다. 다만 이 hook을 사용하면 반환된 상태가 업데이트 될 때마다 컴포넌트를 자동으로 리렌더링 시킨다. 만약 반환된 상태가 여러 데이터의 집합이고, 해당 컴포넌트에서 그 중 몇 개만 사용 중이라면 받아온 상태 중 사용중이지 않은 상태가 업데이트 되더라도 재렌더링이 일어나니 유의해야 한다.

```js
const count = useSelector(state => state.count);
```

- Action : 상태를 변경하기 위한 객체다. 일반적으로 `type` 필드를 가지고 있으며, 필요 시 `payload` 필드를 추가할 수 있다. Dispatcher를 이용해서 Action을 넘겨주면, Reducer에서 이 Action의 type을 받아 어떻게, 어떤 상태를 업데이트 할 지 구분한다. `payload`는 해당 `type`의 상태 변경 로직에서 사용할 값을 나타낸다.
- Reducer : 이전 상태와 액션 객체를 받아 새로운 상태를 반환해주는 함수다. 새로운 상태로 변환하기 위한 로직이 들어간다.
- Dispatcher : 액션을 전달해주기 위해 사용하는 메서드다. 인자로 액션 객체를 받으면 리듀서 함수가 실행되어 상태를 변경한다.

### 기본 예제

```js
// store
import { createStore } from 'redux';

const initialState = {
  count: 0,
};

function reducer(state = initialState, action) { // 이전 상태 + action 객체 받음
  switch (action.type) {
    case 'INCREMENT':
      return {
        ...state,
        count: state.count + (action.payload || 1), // payload로 증가하는 값을 조정할 수 있음, 기본값은 +1
      };
    case 'DECREMENT': // payload 받지 않고 무조건 -1
      return {
        ...state,
        count: state.count - 1,
      };
    default:
      return state;
  }
}

const store = createStore(reducer);

export default store;
```

```js
// actions
export const increaseCount = (count) => { 
  return {
    type: 'INCREASE_COUNT',
    payload: count,
  };
};

export const decreaseCount = () => { 
  return {
    type: 'DECREASE_COUNT',
  };
};
```

```js
// Counter
import { useSelector, useDispatch } from 'react-redux';
import { increase, decrease } from './actions';

function Counter() {
  const count = useSelector(state => state.count); // 상태 count 가 변경되면 자동으로 업데이트 + 리렌더링
  const dispatch = useDispatch();

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => dispatch(increase(3))}>Increase</button> {/* 스토어에 액션 전달 */}
      <button onClick={() => dispatch(decrease())}>Decrease</button> {/* 스토어에 액션 전달 */}
    </div>
  );
}
```

### 특징

- 모든 컴포넌트에 공유되는 데이터를 하나의 저장소에서 관리하기 때문에 컴포넌트 간 데이터 전달이나 상태 관리 코드를 최소화 할 수 있다.
- 변경된 상태는 별다른 이벤트를 발생시키지 않아도 다른 컴포넌트에서 쉽게 상태를 업데이트 할 수 있다.
- 어플리케이션 아무 곳에서나 상태를 사용하고 업데이트 할 수 있다.
- 상태를 변경하는 방법이 통일되어 있다.
- Redux는 다양한 미들웨어와 함께 사용할 수 있어서 비동기 작업과 같은 복잡한 상태 관리를 더 쉽게 처리할 수 있다.

## Redux with TSyringe

위에서 말한 Redux 구성 요소를 만들기 위해 다음을 고려했다.

- 상태가 업데이트되면 화면도 업데이트 하기
- 상태 관리를 위해 필요한 hook 만들기

### Store 만들기

다음 항목은 스토어에 담겨야 하는 요소다.

- State
- Reducer
- useDispatch 사용 시 실행될 로직
- dispatcher를 통해 받은 Action으로 Reducer 함수를 실행할 로직
- 상태 변경 시 forceUpdate 하기 위해 필요한 이벤트 리스너(+ 클린업 함수)

강의에서는 개발자가 직접 건드려줘야 하는 부분(State, Reducer)과 한 번 만들어놓고 바꿀일 없는 나머지 부분을 나눠서 작업했다.

```js
// src/stores/BaseStore.ts
// 타입 정의
export type Action<Payload> = {
    type: string;
    payload?: Payload;
}
type Reducers<State> = Record<string, Reducer<State, any>>;
type Reducer<State, Payload> = (state: State, action: Action<Payload>) => State;
type Listener = () => void;

export default class BaseStore<State> {
    // 타입 정의
    state: State;
    reducer: Reducer<State, any>;
    listeners = new Set<Listener>();
    
    constructor(initialState: State, reducers: Reducers<State>) {
        this.state = initialState;
        this.reducer = (state: State, action: Action<any>): State => {
            // 아래 dispatch에서 this.reducer를 호출하면
            // action 객체에 있는 type에 해당하는 reducer를 실행하여 상태를 변경한다.
            // reducers에 없는 action.type일 경우 Reflect.get()은 undefined를 반환한다.
            const reducer = Reflect.get(reducers, action.type);
            if (!reducer) return state;
            return reducer(state, action);
        };
    }
    
    dispatch<T>(action: Action<T>) { // store.dispatch 로직
        this.state = this.reducer(this.state, action); // reducer 실행
        this.listeners.forEach((listener) => listener()); // forceUpdate 실행
    }
    
    // 이벤트 리스너 및 클린업으로 쓸 함수
    addListener(listener: Listener) {
        this.listeners.add(listener);
    }
    removeListener(listener: Listener) {
        this.listeners.delete(listener);
    }
}
```

```js
// src/stores/Store.ts
import { singleton } from 'tsyringe';

import BaseStore, { Action } from './BaseStore';

const initialState = {
    count: 0,
};

export type State = typeof initialState;

function increase(state: State, action: Action<number>) {
    return {
        ...state,
        count: state.count + (action.payload ?? 1),
    };
}

function decrease(state: State) {
    return {
        ...state,
        count: state.count - 1,
    };
}

const reducers = {
    increase,
    decrease,
};

// State 초기값과 reducers를 BaseStore의 constructor에 넘겨주고,
// Store 인스턴스를 생성하면 BaseStore에 정의된 모든 것을 상속받아 사용할 수 있게된다.
@singleton()
export default class Store extends BaseStore<State> {
    constructor() {
        super(initialState, reducers);
    }
}
```

{% hint style="info" %}

위 예제에서 `action.type`에 맞는 reducer 함수를 찾는 과정에서 `Reflect.get()`이 사용됐다. **Reflect는 뭐고, 왜 썼을까?**

우리는 자바스크립트의 배열, 객체, 맵 등이 포함된 내장 객체를 사용하면서 내장 메서드를 흔하게 사용한다. Reflect는 이런 내장 메서드를 사용하는 것에 대해 인터셉트하여 동작을 변경하거나, 정보를 확인하는 등의 메서드를 제공한다.

```js
const reducer = Reflect.get(reducers, action.type);
if (!reducer) return state;
```

`Reflect.get()`의 동작만 단순히 따지면 `const reducer = reducers[action.type]`과 동일하다. 굳이 `Reflect.get()`을 사용한 이유는 다음과 같다.

- 런타임 오류 방지 :

만약 `action.type`로 넘어온 값이 `reducers`에 없는 프로퍼티라면 어떻게 될까? `object[prop]` 형태로 사용했다면 에러가 발생한다. 반면에 같은 상황에서 `Reflect.get(target, prop)` 형태를 사용하면 Reflect는 `undefined`를 반환한다. 이로써 런타임 오류를 방지할 수 있게 된다.

- 타입 안정성 :

`object[prop]` 형태를 사용하면 코드 입장에서 `prop`이 실제로 객체에 있는지 없는지 알 수 없고, 타입 체크도 할 수 없다. 물론 `keyof`를 사용해서 타입 체크를 해줄수도 있지만 가독성 면에서 `Reflect.get()`을 사용하는 것이 훨씬 좋다.

{% endhint %}

### useDispatch 만들기

- Store의 인스턴스를 생성한다.
- `const dispatch = useDispatch()` -> `dispatch(Action)` 형태로 써주기 위해서 `useDispatch()`가 함수를 반환하도록 만든다.
  - 반환되는 함수는 Action을 인자로 받아야하며, store에 있는 dispatch 함수가 실행되어야 한다.

```js
import { container } from 'tsyringe';

import { Action } from '../stores/BaseStore';
import Store from '../stores/Store';

export default function useDispatch<Payload>() {
    const store = container.resolve(Store);
    return (action: Action<Payload>) => store.dispatch(action);
}
```

### useSelector 만들기

- Store 인스턴스를 생성한다.
- 컴포넌트가 요청한 상태가 변경될 때 자동으로 업데이트를 해줘야한다.
  - 상태가 변경되면 리렌더링 될 수 있도록 `상태 변경 체크 + 상태 변경 시 forceUpdate` 하는 함수를 listener에 추가한다. 이 listener는 dispatch 시마다 자동으로 실행된다.
  - 상태가 2번 이상 변경되면 그 전에 등록된 listener는 삭제할 수 있도록 클린업 함수도 작성한다.
- 상태 변경 체크를 위해서 이전 state를 유지하기 위해 `useRef`를 사용한다.

```js
import { container } from 'tsyringe';
import { useEffect, useRef } from 'react';

import Store, { State } from '../stores/Store';
import useForceUpdate from './useForceUpdate';

type Selector<T> = (state: State) => T;

export default function useSelector<T>(selector: Selector<T>): T {
    const store = container.resolve(Store);
        
    const state = useRef(selector(store.state));
        
    const forceUpdate = useForceUpdate();
        
    useEffect(() => {
        const update = () => {
            const newState = selector(store.state);
            const curState = state.current;
            if ( // 상태 변경 체크, 객체나 배열을 위해 두가지 조건 사용
                (curState === Object(curState) && !Object.is(curState, newState))
                || newState !== curState
               ){
                forceUpdate();
                state.current = newState;
            }
        };
        store.addListener(update);
        return () => store.removeListener(update);
    }, [store, forceUpdate]);

    return state;
}
```

### 수제 Redux 사용하기

```js
const dispatch = useDispatch();
const count = useSelector((state) => state.count);

dispatch({ type: 'increase' });
dispatch({ type: 'decrease' });

dispatch({ type: 'increase', payload: 10 });
```

## Action을 메서드처럼 사용하기

이 부분은 아직 더 이해가 필요할 것 같아 차후에 정리해봐야겠다.
