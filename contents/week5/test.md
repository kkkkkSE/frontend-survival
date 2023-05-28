# Jest로 테스트 코드 작성하기

## Jest란?

[Jest 관련 페이지 이동](../week1/testing-library.md#jest)

## test 함수 사용하기

test 함수는 Jest를 포함한 다양한 테스트 도구에서 일반적으로 사용되는 함수다.

* 첫번째 인자는 테스트 케이스를 설명할 수 있는 문자열이 들어간다.
* 두번째 인자는 테스트할 함수를 정의한다.

```js
test('add', () => {
  expect(add(1, 2)).toBe(3);
});
```

단위 테스트를 수행할 때 유용하며, 좀 더 구체적으로 테스트를 나누고 싶다면 Describe-Context-It 패턴을 사용할 수 있다.

### expect 함수 메서드

expect 함수는 테스트 함수 내에서 사용되어지며, 테스트할 값을 지정하고 이 값이 '기대한 값'과 일치하는지 확인하는 함수다.

'기대한 값'에는 여러가지 형태가 올 수 있으며, 이 형태는 expect 함수의 메서드로 정할 수 있다. vscode 내에서 expect함수의 메서드 부분을 `Command + Click` 하면 다양한 메서드를 확인할 수 있다. 이 메서드를 Matcher라고 한다.

{% hint style="info" %}
_유용한 Matcher 함수_ 검색 시 다양한 Matcher 함수를 확인할 수 있다. 참고하자.
{% endhint %}

* `expect().toEqual(참조형 데이터)` / `expect().toBe(기본형 데이터)` : 데이터 일치성 확인

## BDD 스타일 패턴 사용하여 작성하기

BDD(Behavior Driven Development)는 비즈니스 요구사항에 초점을 맞춰 소프트웨어를 개발하는 방식이다.

유명한 BDD 스타일의 테스트 코드 작성 패턴은 Describe-Context-It 패턴과 Given-When-Then 패턴이 있다.

### Describe-Context-It

Describe-Context-It은 테스트 코드를 그룹화하고, 테스트 케이스를 구분하기 위해 사용되는 패턴이다. 이 패턴은 테스트 코드를 가독성 있고 일관성 있게 작성할 수 있게 돕는다. 각 항목은 테스트 함수처럼 첫번째 인자인 문자열 인자에 설명을 적을 수 있다.

* Describe : 테스트의 대상을 명시하거나, 하위 테스트들을 대표할 수 있는 큰 테스트 주제를 설명하기도 한다.
* Context : Describe에서 정한 테스트 대상 또는 환경 내에서의 특정 상황들을 설명한다.
* It : Context에서 설명된 특정 상황에 대한 결과를 명시하고, 테스트 케이스를 정의한다.

```js
const context = describe;
describe('add 함수에 대해', () => {
  it("두 숫자의 합을 돌려준다", () => {
    expect("검증 대상").toXX("기대 결과");
  });
    
  context('두 숫자가 다 음수라면', () => {
    it("0을 리턴한다", () => {
      expect("검증 대상").toXX("기대 결과");
    });
  });
  
  context('양수, 음수가 각각 하나씩 주어지면', () => {
    it("항상 더 작은 값을 돌려준다", () => {
      expect("검증 대상").toXX("기대 결과");
    });
  })
});
```

참고로 Jest에서는 Context를 따로 지원하지 않아 `describe`를 중첩하여 사용한다. 읽을 때 헷갈리는 것을 방지하여 `context` 변수에 `describe`를 저장하여 사용하곤 한다.

아래의 Given-When-Then 예제에서도 Describe-Context-It 패턴을 확인할 수 있다.

### Given-When-Then

Given-When-Then는 테스트 케이스를 작성하는 방식 중 하나다.

기대 효과로는 요구사항을 명확하게 표현하고, 테스트 케이스의 목적을 분명히 할 수 있다. 또한 가독성과 유지보수성이 높아진다.

* Given : 테스트 케이스에서 필요한 상태를 설정한다. (입력값이나 시스템 초기 상태 등)
* When : 테스트할 행동에 대한 코드를 작성한다. (한 번에 한가지 메서드만 사용 추천)
* Then : 테스트 행동에 대한 검증을 실시하여 예상 결과가 나왔는지 확인한다. (어떠한 로직(행동)도 담아선 안된다.)

```js
function calculator(a, b, operator) {
  switch (operator) {
    case '+':
      return a + b;
    case '-':
      return a - b;
    default:
      throw new Error('Invalid operator');
  }
}

const context = describe;
describe('calculator', () => {
  context('when operator is +', () => {
    it('should add two numbers', () => {
      // given
      const a = 2;
      const b = 3;
      const operator = '+';

      // when
      const result = calculator(a, b, operator);

      // then
      expect(result).toBe(5);
    });
  });

  context('when operator is -', () => {
    it('should subtract two numbers', () => {
      // given
      const a = 5;
      const b = 3;
      const operator = '-';

      // when
      const result = calculator(a, b, operator);

      // then
      expect(result).toBe(2);
    });
  });
});
```



### before과 after

Jest에도 Test Fixture를 위한 `beforeEach`와 `beforeAll`, `afterEach`와 `afterAll` 함수를 지원하고 있다.

* before : 테스트 케이스 실행 전에 필요한 초기화 작업을 진행한다.
* after : 테스트 케이스 실행 후에 정리 작업을 진행한다. after 함수를 사용함으로써 테스트 환경을 깨끗하게 유지할 수 있다.

`beforeEach`와 `afterEach`는 각각의 테스트 케이스(It문) 마다 직전, 직후에 한번씩 실행되고, `beforeAll`과 `afterAll`은 선언된 곳 전역에서 딱 한번씩만 실행되기 때문에 모든 테스트 케이스에 영향을 미칠 수 있어 사용 시 주의해야 한다.

참고로 \~Each, \~All이 선언되는 위치에 따라 적용되는 범위는 달라질 수 있으므로 주의해야 한다.

```js
describe('counter', () => {
  let counter;

  beforeEach(() => {
    counter = 0;
  });

  afterEach(() => {
    counter = null;
  });

  it('increments the counter', () => {
    counter++;
    expect(counter).toEqual(1);
  });

  it('decrements the counter', () => {
    counter--;
    expect(counter).toEqual(-1);
  });
});
```

## Test Fixture 활용하기

테스트를 위해 변경되지 않는 상태의 객체나 데이터를 만들어 둔 것을 Fixture라고 한다. 테스트를 진행할 때 외부 요인의 간섭을 제거하고, 테스트에만 집중할 수 있도록 Fixture를 활용하는 것이다.

강의에서는 루트 폴더에 `fixtures` 폴더를 만들어 fixture 데이터들을 관리했다.

```typescript
// fixtures/data1.ts
const data1 = 'data1';

// fixtures/data2.ts
const data1 = 'data1';
// fixtures/index.ts
import data1 from './data1';
import data2 from './data2';

export default {
  data1,
  data2
}
import fixtures from '../fixtures'; // 자동으로 index.ts를 불러옴

jest.mock('./hooks/useFetchData1', () => () => fixtures.data1);
```

## React Testing Library

### React Testing Library란?

[React Testing Library 관련 페이지 이동](../week1/testing-library.md#react-testing-library)

### React Testing Library를 이용한 테스트

먼저, 테스트에 사용할 React Testing Library 함수를 불러와야 한다.

```js
import { render, screen } from '@testing-library/react';
```

React Testing Library에서 기본적으로 사용되는 함수는 다음과 같다.

* `render()` : 사용자와의 상호작용을 시뮬레이션할 수 있는 함수다. 괄호 안의 인자로 렌더링 될 UI 컴포넌트 등을 넣을 수 있다.
* `screen()` : `render()`함수를 통해 렌더링 된 결과를 검증하는 코드가 들어간다. 단, screen을 사용하기 위해서는 **먼저 render 함수를 통해서 컴포넌트를 호출해야 한다.**

```js
test("테스트 설명", () => {
  render(<컴포넌트 />);

  screen.getByText('문자열1'); // 문자열1이 존재하는지
  screen.getByText(/문자열2/); // 문자열2가 결과물에 포함 되었는지
  expect(screen.queryByText(/문자열3/)).toBeFalsy(); // 문자열3이 존재하는가에 대한 결과가 falsy 인지
  expect(screen.queryByText(/문자열4/)).not.toBeInTheDocument(); // 문자열4을 찾아봤는데 없는지
});
```

위의 예제와 같이 `screen` 함수는 `expect` 함수와 같이 사용될 수 있다.

참고로 각 메서드에 대해 요소를 찾지 못한 경우에 무조건 에러가 발생하므로, 요소가 존재하지 않을 수도 있다면 예외 처리를 꼭 해주어야 한다.

#### screen 함수 메서드

렌더링 된 결과를 검증하기 위해 사용할 수 있는 다양한 screen 메서드가 존재한다. 아래는 가장 많이 사용되는 screen 메서드다.

* `getByText()` : 특정 텍스트를 포함하는 요소를 찾는다. 예를 들어, `getByText('Apple')`은 'Apple'이라는 텍스트를 포함하는 요소를 찾는다.
* `getByLabelText()` : 라벨(label)과 연결된 요소를 찾는다. 예를 들어, `getByLabelText('User Name')`은 'User Name'이라는 라벨과 연결된 요소를 찾는다.

문자열을 찾을 때, 일반 문자열을 입력하면 반드시 그 문자열이어야 통과된다. 만약 찾는 문자열이 포함되어 있길 바란다면 정규식을 이용하면 된다.

이외에도 많은 메서드가 존재하니 필요 시 [공식 문서](https://testing-library.com/docs/queries/about)를 참고하자.

#### fireEvent

사용자 인터렉션을 시뮬레이션 하기 위해 사용되는 함수다. 버튼 클릭이나 input에 값 넣기 등이 사용자 인터렉션에 포함된다.

사용자 이벤트를 시뮬레이션 하기 위해 click, change, submit 등의 다양한 이벤트를 지원한다. 이를 이용하여 DOM을 조작하고, 변경사항을 테스트 할 수 있다.

```js
import { render, screen, fireEvent } from '@testing-library/react';

test('input 요소의 값 변경 테스트', () => {
  render(<input />);
  const input = screen.getByRole('textbox');

  fireEvent.change(input, { target: { value: 'Hello, World!' } });
  expect(input.value).toBe('Hello, World!');
});
```

위 코드의 fireEvent는 두번째 인자에 `target`을 전달해야만 하는 change 이벤트 사용 예제이다. 이렇게 이벤트 종류마다, 대상에 따라 두번째 인자의 값이 바뀔 수 있으니 사용 전 반드시 확인해야 한다.
