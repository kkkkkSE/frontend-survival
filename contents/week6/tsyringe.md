# TSyringe

## 들어가기 전에

하위 컴포넌트에 props를 내려주는 것 또한 의존성 주입에 속한다. props를 이용하여 바로 하위에 있는 컴포넌트에 값을 내려주는 건 어렵지 않지만, 여러 컴포넌트를 거쳐서 저멀리 하위에 있는 컴포넌트에 값을 전달하는 건 매우 번거롭고 비효율적이다. 이를 Props Drilling이라 한다.

이 문제를 해결할 수 있는 방안으로 context가 거론됐다. Provider로 감싸진 컴포넌트를 포함해 하위에 있는 컴포넌트들은 손쉽게 context에 저장된 데이터를 전달받을 수 있다.

하지만 context를 사용하게 되면, 불필요하게 context를 사용하지 않는 컴포넌트까지 렌더링 되어버리는 부작용이 있다. 또한 context를 사용한 컴포넌트는 재활용이 어려워진다는 단점도 있다.

이러한 부작용을 해결할 수 있는 방안으로 External Store를 '전역 상태 관리'처럼 이용하여 의존성 주입을 하는 방법이 있다. 이 External Store를 관리하는데 활용할 수 있는 것이 TSyringe다.

_명심해야 할 점은, 그렇다고 둘 중 어떤 한 가지가 낫기 때문에 하나만 지향하라는 아니라는 것이다. 상황에 따라 맞는 방법을 사용하자._

## TSyringe란?

자바스크립트 및 타입스크립트용 의존성 주입을 위한 도구이다.

### 설치 및 세팅

설치 커맨드는 다음과 같다.

```bash
npm i tsyringe reflect-metadata
```

TSyringe 내부에서 Reflect API를 사용중인데, 이를 지원하지 않는 환경에서 접속할 수도 있으니 Reflect polyfill을 함께 설치해야 한다. 여기서는 `reflect-metadata`를 설치했다.

그 다음은 `index.html` 파일과 가장 가까운 자바스크립트 또는 타입스트립트 파일에서 `reflect-metadata`를 import 해야한다.

그리고 테스트 코드 실행을 위해 `jest.config.js` 파일에서 `setupFilesAfterEnv`에 `<rootDir>/src/setupTests.ts`를 추가하고, 이 파일 안에서도 `reflect-metadata`를 import 해준다.

```js
import 'reflect-metadata';
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

### 전역 객체 만들기

`@singleton()` 데코레이터가 지정된 클래스는 전역에서 단 하나의 인스턴스만 생성되고 관리된다.

```js
import { singleton } from 'tsyringe';

@singleton()
class CounterStore {
    count = 0;

    forceUpdate : () => void = () => {};

    update(){
        this.forceUpdate();
    }
}
```

DI Container를 사용해서 singletone 클래스의 인스턴스를 생성할 수 있고, 여러 컴포넌트에서 생성된 인스턴스는 동일한 인스턴스로 관리된다.

```js
import { container } from 'tsyringe';
import useForceUpdate from '../hooks/useForceUpdate';
import Store from '../stores/Store';

function Counter{
    const store = container.resolve(CounterStore); // 객체 생성
    const forceUpdate = useForceUpdate();
    store.forceUpdate = forceUpdate; // store에 있는 forceUpdate가 업데이트 됨

    return(
        <p>{store.count}</p>
    )
}
```

```js
import { container } from 'tsyringe';
import Store from '../stores/Store';

function CountControl() {
    const store = container.resolve(CounterStore);

    const handleClick = () => {
        store.count += 1;
        store.update();
    }
}
```

`Counter` 컴포넌트에서 store에 있는 `forceUpdate`를 업데이트 시키면, 다른 컴포넌트에서도 store의 새롭게 할당된 `forceUpdate`를 사용할 수 있다.

### 개선하기

#### 여러 컴포넌트의 forceUpdate 사용하기

위에서 만든 카운터 컴포넌트에는 잠재적 문제가 있다. 만약 카운터 두개를 나란히 사용하게 되면, `forceUpdate` 함수가 제대로 동작하지 않는다. `forceUpdate`에 대해 할당을 두번하게 되므로 마지막에 할당된 `forceUpdate`에 대해서만 제대로 동작하는 것이다.

이를 보완하기 위해서는 store에 있는 프로퍼티에 바로 할당을 하는 게 아니라, 사용 되어야 할 `forceUpdate`들을 모두 저장할 수 있는 공간을 만들어두고 저장된 함수를 모두 실행하는 프로퍼티를 하나 더 사용하는 것이다.

```js
import { singleton } from 'tsyringe';

type ForceUpdate = () => void;

@singleton()
class CounterStore {
    count = 0;

    forceUpdates = new Set<ForceUpdate>();

    update(){
        this.forceUpdats.forEach((forceUpdate) => forceUpdate());
    }
}
```

```js
import { container } from 'tsyringe';
import useForceUpdate from '../hooks/useForceUpdate';
import Store from '../stores/Store';

function Counter{
    const store = container.resolve(CounterStore); // 객체 생성
    const forceUpdate = useForceUpdate();
    store.forceUpdates.add(forceUpdate);

    return(
        <p>{store.count}</p>
    )
}
```

#### 무지성 forceUpdate 등록 막기

리액트 자체에서는 렌더링 될 때마다 새로운 함수를 반환시킨다. `{} !== {}`인 것처럼 기능이 똑같고, 코드가 동일해도 새로운 함수로 인식한다. 이 `forceUpdate`가 `forceUpdates`에 계속 추가되어 쌓이게 된다면 메모리 낭비가 발생한다.

이를 방지하기 위한 방안은 아래와 같다. 여기서 `forceUpdates`와 같이 '실행시킬 함수'를 저장해놓는 프로퍼티를 `listeners`, 실행시킬 함수를 `listener`라 칭했다.

- store에서 저장될 함수에 useCallback 사용하기 :
useCallback은 의존성 배열에 있는 요소가 변하기 전까지는 항상 같은 값을 유지한다. 의존성 배열을 빈 배열로 두고 useCallback을 사용했을 때 `listener`는 리렌더링 돼도 새로운 함수가 아닌 기존과 동일한 함수를 유지할 수 있다.
- store에서 함수들을 저장시킬 그릇으로 Set 사용하기 :
useCallback을 사용하여 계속 같은 `listener`가 들어온다 하더라도 `listeners`에 계속해서 쌓이면 메모리가 낭비된다. 같은 함수가 들어왔을 때 쌓이지 않고 함수 리스트를 유지할 수 있도록 Set을 사용하자.
- useEffect에서 의존성 요소 설정하기 :
store가 변경되거나, `listener`가 변경될 혹시 모를 상황에 대비하여 의존성 배열에 store와 `listener`를 추가한다. 그리고 useEffect 내에서 store에 `listener`를 저장시킨다.
- useEffect에서 Clean-up 해주기 :
만약 store가 변경되거나 `listener`가 변경되어 store에 새롭게 추가되어야 한다면, 기존에 있던 안 쓰게 된 `listener`는 삭제시켜야 한다. 예상치 못한 오류를 발생시킬 수 있다.

```js
useEffect(()=>{
    store.listeners.add(forceUpdate);
    
    return () => {
        store.listeners.delete(forceUpdate);
    }
}, [store, listener]);
```
