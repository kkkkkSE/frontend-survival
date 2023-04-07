# Mocking

Mocking은 테스트 중 외부 의존성을 가짜로 대체하는 것을 의미한다.

프론트엔드 애플리케이션을 개발할 때, 외부 API 호출이나 데이터베이스 연동 등은 흔한 일이다. 이러한 작업이 필요한 모듈을 테스트할 때, 실제 API나 데이터베이스와 연동하여 테스트를 할 경우 여러 문제가 발생할 수 있다.

이러한 문제가 발생했을 경우, 코드의 문제가 아닌 API나 데이터베이스의 문제로 테스트에 실패하게 되면 어떤 것이 문제인지 원인을 파악하기 쉽지 않다. 그래서 진짜 데이터가 아닌 가짜 데이터를 이용하는 것이다.

## Jest에서의 Mocking

Jest에서도 Mocking을 지원하기 위해 제공하고 있는 함수들이 있다.

### jest.fn()

변수에 할당하여 가짜 함수를 생성하는 함수다. 인자로 가짜 함수가 실행할 함수를 넣을 수 있다. 아무것도 넣지 않으면 undefined를 반환한다.

```js
const mockFn = jest.fn(); // mockFn은 빈 함수가 된다.
mockFn(1); // undefined 

const mockFn = jest.fn((name) => `My name is ${name}`);
console.log(mockFn("John")); // 'My name is John'
```

jest.fn()로 만들어진 가짜 함수의 반환값이나 함수 자체 등을 조작할 수 있는 여러 메서드들이 있으니 필요시 찾아보자.

### jest.mock()

모듈에 있는 함수들을 한꺼번에 Mocking할 때 사용할 수 있다. 가져온 함수들은 이름만 빌려와 가짜 함수를 만드는거고 기능은 복제해오지 않는다.

첫번째 인수는 모듈의 경로, 두번째 인수는 해당 모듈을 대체할 가짜 모듈을 생성하는 함수를 전달한다.

```js
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}
```

```js
import { add, subtract } from './math';

jest.mock('./math', () => ({
  add: jest.fn(),
  subtract: jest.fn(),
}));

test('add function is called', () => {
  add(1, 2);
  expect(add).toHaveBeenCalledWith(1, 2);
});
```

위 코드의 경우 모듈에 있는 특정 함수만 골라 Mocking할 때 유용하며, 한 모듈에 Mocking 해야할 함수가 너무 많을 경우 다음과 같이 사용하는 것도 가능하다.

```js
import { add, subtract } from './math';

jest.mock('./math');

test('add function is called', () => {
  add(1, 2);
  expect(add).toHaveBeenCalledWith(1, 2);
});

test('subtract function is called', () => {
  subtract(2, 1);
  expect(subtract).toHaveBeenCalledWith(2, 1);
});
```

이렇게 사용하면 모듈 내에 있는 모든 함수를 가짜 함수로 사용할 수 있다.

### Mock 초기화하기

현재 테스트 케이스에서 Mock을 사용하면 다음 테스트 케이스에 영향을 줄 수도 있기 때문에 그 사이에 Mock을 정리해주는 것이 좋다.

Mock을 정리하는 메서드는 `가짜함수명.mockClear()` 와 `jest.clearAllMocks()` 등이 있다. 해당 함수들은 보통 `before` 또는 `after` 함수 내에서 쓰인다.
