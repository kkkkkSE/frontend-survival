# MSW

프론트엔드 개발에서 백엔드와 소통은 꽤 큰 비중을 차지한다. 테스트 코드를 구현하다보면 백엔드에 의존해야 하는 부분이 많고, 이 부분을 모두 가짜로 구현하여 사용하기에는 꽤 번거로울 수 있다. 이 때 사용할 수 있는 대안이 MSW와 같은 라이브러리다.

## Service Worker란?

Service Worker는 웹 애플리케이션에서 주 스레드와 관련없이 동작되는 자바스크립트 코드로, 브라우저와 서버 사이의 프록시 서버처럼 통신을 중개하는 역할을 한다. 웹 애플리케이션에서 네트워크 요청과 응답을 가로채 캐싱하며, 이를 통해 앱처럼 오프라인에서도 작동할 수 있게 된다.

## MSW란?

Service Worker를 대신하여 로컬에서 HTTP 요청을 가로채 가짜 응답을 반환하는 라이브러리다. 즉, 코드에서 Mocking을 하는 것이 아닌 네트워크 단계에서 Mocking을 하는 것이다.

MSW를 통해 백엔드 API가 구현되지 않았거나, 로컬 환경에서 테스트를 하고자 할 때 백엔드 서버 없이 개발 또는 테스트를 할 수 있다.

MSW는 REST API, GraphQL 모두 지원하고, Node와 웹 브라우저도 모두 지원한다.

### 설치

npm을 통해 MSW를 설치한다.

```bash
npm i -D msw
```

### 사전 준비

Mock 서버를 세팅하기 위해 몇가지 생성해야 하는 파일이 있다. 우리는 Node 환경에서 이용할거라 [이 페이지](https://mswjs.io/docs/getting-started/integrate/node)를 참고하여 필요한 파일을 작성하도록 하자.

```js
// src/mocks/server.js
import { setupServer } from 'msw/node'
import { handlers } from './handlers'

const server = setupServer(...handlers) // handlers를 통해 서버를 구성

export default server;
```

```js
// src/setupTests.js
import { server } from './mocks/server.js'

beforeAll(() => server.listen()) // 모든 테스트에 Mock 서버 설정
afterEach(() => server.resetHandlers()) // 테스트 도중 handlers가 변경될 수 있으므로 매 테스트 후 초기화 실행
afterAll(() => server.close())
```

Jest에서 MSW에서 세팅한 Mock 서버를 사용하기 위해 `jest.config.js` 파일의 `setupFilesAfterEnv` 속성에 `setupTests.ts` 파일을 추가한다.

```js
module.exports = {
    testEnvironment: 'jsdom',
    setupFilesAfterEnv: [
        '@testing-library/jest-dom/extend-expect',
        '<rootDir>/src/setupTests.ts',
    ],
    // ...
};
```

### 서버 작성하기

우리는 REST API를 사용할거라 rest를 import 해줘야 한다.

참고로 API를 만들때 사용하는 문법은 Express 문법과 매우 비슷하다. (궁금하면 [Express 노트](../week4/api.md#express)에서 문법 확인)

`handlers`는 배열로 되어있고, 배열의 요소로는 각 요청에 대한 응답이 담겨있다.

Express와 다른점은 다음과 같다.

- `const app = express()` 대신 import한 `rest` 사용
- 핸들러 함수에 전달되는 세번째 인자는 `next` 대신 `ctx`로, 요청과 응답을 다루기 위한 메서드를 담고 있는 context 객체를 의미
- `res.send()` 대신 `return res( 응답내용 )`로 응답

```js
import { rest } from 'msw';

const BASE_URL = 'http://localhost:3000';

const handlers = [
    rest.get(`${BASE_URL}/products`, (req, res, ctx) => {
        const products = [
            {
                category: 'Fruits', price: '$1', stocked: true, name: 'Apple',
            },
        ];

        return res(
            ctx.status(200),
            ctx.json({ products }),
        );
    }),
];

export default handlers;
```

이렇게 세팅하고 테스트를 하면 더이상 직접 만든 가짜 함수를 하지 않고, 실제 백엔드와 통신하는 것처럼 사용할 수 있다.

단순히 임시 서버를 쓸거라면 Express를 사용하는게 더 낫지만 MSW는 테스트 코드와 웹 브라우저 모두 지원되고 백엔드 서버가 따로 필요하지 않으니 상황에 따라 더 나은 선택일 수 있다.

### Mock 서버로 테스트 시 주의점

서버에서 데이터를 가지고 올 때, 비동기로 받아오기 때문에 테스트를 할 때도 비동기 코드를 테스트해야 될 경우도 생긴다. 이럴 때 사용할 수 있는게 `waitFor()` 함수다. 이 함수는 Promise를 반환하기 때문에 반드시 await와 같이 써야한다.

```js
import { render, screen, waitFor } from '@testing-library/react';
import MyComponent from './MyComponent';

test('renders data after loading', async () => {
  render(<MyComponent />);

  await waitFor(() => screen.getByText('Loading'));
  await waitFor(() => screen.getByText('Complete'));
  expect(screen.getByText('Data is loaded.')).toBeInTheDocument();
});
```

이 코드에서 처음에 'Loading' 텍스트를 얻을 때까지 기다렸다가, 얻으면 다음으로 넘어가 'Complete' 텍스트를 얻을 때까지 기다린다. 그러고 맨 아래 expect 문이 실행된다.

## Fetch Polyfill

Polyfill은 브라우저나 기타 환경에서 지원하지 않는 기능을 자바스크립트 코드를 통해 구현한 라이브러리다.

요즘의 Fetch API는 대부분의 브라우저에서 지원되지만, 특정 환경이나 일부 브라우저에서는 지원하지 않는 경우도 있다. 이때를 대비하여 사용하는 것이 Fetch Polyfill이다.

우리가 지금까지 한 테스트는 Node 환경에서 이뤄지는데, 과거의 Node 또한 Fetch API를 지원하지 않았다. 이런 경우 사용할 수 있는 Fetch Polyfill 중 하나가 [GitHub에서 만든 Fetch Polyfill](https://github.com/github/fetch)이다.

MSW에서 GitHub에서 만든 Fetch Polyfill을 사용하려면 다음과 같은 절차를 거쳐야한다.

### 설치하기

```bash
npm install whatwg-fetch --save
```

### 사용하기

MSW를 사용하기 위해 만든 파일 중 루트 파일이라고 할 수 있는 `src/setupTests.js` 파일에 import 해준다.

```js
import 'whatwg-fetch'
```
