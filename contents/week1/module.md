---
description: CommonJS VS ES modules
---

# Module

## 모듈 시스템의 역사

과거에는 자바스크립트가 단순히 웹페이지를 동적으로 만들기 위해 보조하는 역할로서 사용되어졌기 때문에 모듈 기능이 없었다. 그 당시 자바스크립트 파일은 아래와 같이 로드 되었다.

```js
<html>
    // ...
    <script src="index.js"></script>
    <script src="calc.js"></script>
    // ...
</html>
```

이 방식의 가장 큰 문제점은 각 파일 간의 스코프가 분리되지 않는다는 점이었다. 즉, 전역 객체를 공유하게 되니 모든 스크립트가 하나의 파일로 병합된다고 볼 수 있다.

스코프가 분리되지 않았을 때 생기는 문제로 :

- 식별자의 이름을 동일하게 사용하면 겹치게 되어 제대로 동작하지 않거나,
- 순서를 신경써서 사용해야 했으며,
- 어떤 script에 의존성을 가지고 있는지 파악하기 어려웠다.

이러한 이유로 각 파일을 완전히 분리하여 서로 간섭할 수 없도록 **모듈화**하기 위해 여러가지 시스템이 생겼다. 이렇게 독립되어 자신만의 스코프를 가지게 된 각 파일을 **모듈**이라고 한다.

모듈화를 위해 생긴 초창기 시스템은 비동기를 지원하는지 여부에 따라 *CommonJS*와 *AMD*로 나눠졌다. 이 두 가지의 호환을 돕거나 더욱 간편하게 사용하기 위한 노력이 계속되며 여러가지 시도가 있었고, 결국 자바스크립트 자체에서 ES6 사양부터 모듈 시스템을 지원하게된 것이 *ES modules*다.



## CommonJS

### 특징

- 브라우저 외에도 사용할 수 있는 자바스크립트 모듈 시스템이다.
- 동기적으로 `require`를 실행하여 주 스레드를 차단한다.
- 런타임에 파싱되기 때문에 `require`문을 어디서든 사용할 수 있다.
- 순환 참조 관리가 어렵다.
- **Node.js의 기본 모듈 시스템으로 채택됐다.**

### 사용하기

#### exports / require

한 모듈 안에 여러 함수가 있고, 다른 파일에서 재사용하려면 **`exports` 객체의 메서드**로 정의할 수 있다.

```js
// calc.js
const add = (a,b) => a + b;
exports.add = add;

exports.minus = (a,b) => a - b;
```

```js
const calc = require('./calc.js');

calc.add(10, 20); // 30
calc.minus(20, 10); // 10
```

#### module.exports / require

`exports` 객체에는 여러 메서드가 정의될 수 있었지만, `module.exports`는 문자열, 함수, 객체 등 상관없이 **하나의 값만 할당**할 수 있다.

```js
// devide.js
module.exports = function(a, b) {
  return a / b;
};
```

```js
const devide = require('./devide.js');
const myDevide = devide(4, 2);

console.log(myDevide) // 2
```

`exports` 객체처럼 여러 메서드를 사용하고 싶다면 아래와 같이 함수를 할당하여 여러 메서드를 반환할 수 있다.

```js
// calc.js
module.exports = function(a, b) {
  return {
    add() { return a + b },
    minus() { return a - b }
  };
}
```

```js
const calc = require('./calc.js');
const myCalc = calc(5, 2);

myCalc.add(); // 7
myCalc.minus(); // 3
```

## ES modules

### 특징

- `import`를 읽고 의존성 파일을 불러오는 동안 주 스레드를 차단하지 않는다.
- `import`한 변수들은 *readonly*다.
- 순환 참조 시 *Error*를 반환한다.
- 트리 쉐이킹이 쉽다.
- 문법이 간단하다.
- ES6를 지원하지 않는 브라우저에선 이전 버전으로 변환하여 사용해야 한다.

### 사용하기

#### export / import

한 파일 안에서 여러 개의 값을 `export` 할 수 있다.

```js
// calc.js

export const add = (a, b) => a + b;

const minus = (a, b) => a - b;
export { minus };
```

```js
import { add, minus } from './calc.js';

add(10, 20);
minus(20, 10);
```

모든 `export`를 가져와 메서드처럼 사용할 땐 다음과 같이 작성한다.
```js
import * as calc from './calc.js';

calc.add(10, 20);
calc.minus(20, 10);
```

#### export default / import

한 파일에서 `export default`는 하나만 사용할 수 있다.

```js
export default const calc = {
    add: function(a, b) {
        return a + b;
    },
    minus: function(a, b) {
        return a - b;
    }
}
```

```js
import calc from './calc.js'

calc.add(10, 20);
calc.minus(20, 10);
```



{% hint style="info" %}
TODO : 모듈 시스템 특징 보충 / 모듈 구성, 인스턴스화, 평가 단계 차이 자세히 알아보기
{% endhint %}