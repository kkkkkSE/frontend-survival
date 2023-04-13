# usestore-ts

TSyringe에서 했던 예제에서는 상태가 업데이트됨에 따라 리렌더링하기 위해 `forceUpdates`에 `forceUpdate`을 담고(이벤트 리스너) 실제 화면을 업데이트(publish) 하는 과정이 필요했다.

하지만 usestore-ts에서는 이벤트 리스너와 퍼블리싱 없이 간편히 전역 객체를 만들어 사용할 수 있다.

## 설치 및 세팅

TSyringe와 함께 사용하기 위해 [TSyringe도 설치](/contents/week6/tsyringe.md#설치-및-세팅)해놓자.

아래는 usestore-ts 설치 코드다.

```bash
npm install usestore-ts
```

타입스크립트의 데코레이터를 사용하기 위해 `tsconfig.json` 파일에서 데코레이터 관련된 설정을 `true`로 설정해야 한다.

```json
{
  "compilerOptions": {
    // ...
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true, 
    // ...
  }
}
```

## 전역 객체 만들기

### 기본 예제

```js
import { Store, Action, useStore } from 'usestore-ts';

@Store()
class CounterStore {
    count = 0;

    @Action()
    increase(step = 1) {
        this.count += step;
    }

    @Action()
    decrease(step = 1) {
        this.count -= step;
    }
}

const counterStore = new CounterStore();

export default function Counter() {
  const [ { count }, store ] = useStore(counterStore); // 인스턴스 생성

  return (
    <div>
      <p>
        Count:
        {' '}
        {count}
      </p>
      <p>
        <button type="button" onClick={() => store.increase(10)}>
          Increase
        </button>
        <button type="button" onClick={() => store.decrease(10)}>
          Decrease
        </button>
      </p>
    </div>
  )
}
```

- `[ state, store ] = useStore(Store)` 형태로 인스턴스 객체를 만들 수 있다.
- Action 함수 위에 `@Action()`을 붙여줘야 한다.

### 예제 with TSyringe

데코레이터 순서가 바뀌면 제대로 동작하지 않을 수 있으므로 순서에 유의하자.

```js
// Store
import { singleton } from 'tsyringe';
import { Store, Action } from 'usestore-ts';

@singleton()
@Store()
export default class CounterStore {
    count = 0;
        
    @Action()
    increase() {
        this.count += 1;
    }
        
    @Action()
    decrease() {
        this.count -= 1;
    }
}
```

```js
// 인스턴스 생성 커스텀 훅
import { container } from 'tsyringe';
import { useStore } from 'usestore-ts';

import CounterStore from '../stores/CounterStore';

export default function useCounterStore() {
    const store = container.resolve(CounterStore);

    return useStore(store);
}
```

DI container를 사용해서 객체를 만드는 부분을 커스텀 훅으로 만들어두면 깔끔하게 사용할 수 있다.

## 장점

- `@Store`, `@Action` 데코레이터를 사용하면서 가독성이 좋아진다.
- Action을 메서드처럼 사용할 수 있어 사용하기 편리하다. `storeName.`까지 입력하면 어떤 Action을 사용할 수 있는지 자동완성으로 뜬다.
- 일반적인 함수에서 인자를 받아 함수 내 로직에서 사용되듯, Action 함수를 호출할 때 인자로 전달한 값이 Action 함수 내 로직에서 바로 사용될 수 있다.

## usestore-ts 사용 시 주의점

- store 내에서 객체 State내 value가 변경됐을 경우, 변화를 감지시켜 리렌더링 하기 위해 완전 새로운 객체를 할당해야 한다. 새로운 객체를 할당하기 위해 아래같은 방법을 사용하거나, [immer](https://immerjs.github.io/immer/)를 사용할 수 있다.

```js
export default class CounterStore {
    state = {
        x: 1,
    }
        
    @Action()
    increase() {
        this.state = { ...this.state, x : this.state.x + 1 };
    }

}
```

- 비동기 함수를 Action으로 쓰기 위해서는 비동기 함수 자체에 `@Action`을 붙이는 것이 아니라, 비동기 함수 내에 비동기 부분을 제외한 부분을 Action으로 처리한다.

```js
// 잘못된 예
@singleton()
@Store()
class PostStore {
    posts: Post[] = [];
        
    @Action()
    async fetchPosts() {
        this.posts = [];
        const posts = await fetchPosts();
        this.posts = posts;
    }
}
```

```js
// 수정 후
@singleton()
@Store()
class PostStore {
    posts: Post[] = [];
        
    async fetchPosts() {
        this.startLoading();
        const posts = await fetchPosts();
        this.completeLoading(posts);
    }
        
    @Action()
    startLoading() {
        this.posts = [];
    }
        
    @Action()
    completeLoading(posts: Post[]) {
        this.posts = posts;
    }
}
```

## 참고

usestore-ts에는 내부적으로 `useSyncExternalStore`를 사용하고 있다. `useSyncExternalStore`는 외부 스토어를 구독할 수 있는 React Hook이다.

```js
const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)
```

- subscribe : store에 구독하고, 언마운트 시 clean-up 하는 로직이 담긴 함수
- getSnapshot : store의 데이터 스냅샷을 반환하는 함수
- snapshot : getSnapshot으로 가져와진 데이터(상태)

```js
// Store
let nextId = 0;
let todos = [{ id: nextId++, text: 'Todo #1' }];
let listeners = [];

export const todosStore = {
  addTodo() {
    todos = [...todos, { id: nextId++, text: 'Todo #' + nextId }]
    emitChange();
  },
  subscribe(listener) { // 구독 + 구독취소(clean up)
    listeners = [...listeners, listener];
    return () => {
      listeners = listeners.filter(l => l !== listener);
    };
  },
  getSnapshot() { // 데이터 스냅샷 반환
    return todos;
  }
};

function emitChange() {
  for (let listener of listeners) {
    listener();
  }
}

```

```js
// Component
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

export default function TodosApp() {
  const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
  return (
    <>
      <button onClick={() => todosStore.addTodo()}>Add todo</button>
      <hr />
      <ul>
        {todos.map(todo => ( // snapshot 활용
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}
```
