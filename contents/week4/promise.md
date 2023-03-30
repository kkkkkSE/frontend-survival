# Promise

## Promise

Promise는 비동기 처리를 위해 ES6에 도입된 패턴이다.

###

### 도입 배경

대부분의 DOM 이벤트나 Timer 함수, API로 데이터를 받아오는 행위 등이 모두 비동기식 처리 모델로 동작한다. 비동기식 처리 모델은 요청을 병렬로 처리하기 때문에 다른 요청이 Blocking 되지 않는 것이 장점이다.

하지만 이렇게 비동기식으로 데이터를 받아오게 되면, 받아온 데이터로 다른 무언가를 하려고 할 때 데이터가 완전히 받아지지 않은 상태에서 먼저 처리되어 원하는 결과를 보장할 수 없다는 것이 문제였다.

그래서 데이터를 받아오고, 그 뒤에 무언가를 처리하는 순서를 보장하기 위해 콜백 함수를 이용했다.

이러한 콜백 패턴도 완전하진 않았다. 데이터를 받아오고 처리해야 할 일이 하나가 아니라 여러 개라면, 거기다 순서까지 보장해야 한다면 **여러 개의 콜백 함수가 중첩되어 복잡도가 높아지는 문제점**이 생겼다. 이는 **가독성도 나쁘고, 유지보수도 어렵게 만든다.** 이를 콜백 지옥이라 한다.

```js
step1(function(val1) {
  step2(val1, function(val2) {
    step3(val2, function(val3) {
      step4(val3, function(val4) {
        // ...
      });
    });
  });
});
```

또한 이 콜백 패턴의 큰 문제점 중 **하나는 에러 처리가 어렵다**는 것이었다.

```js
try {
  setTimeout(() => { throw new Error('Error!'); }, 1000);
} catch (e) {
  console.log('에러를 캐치하지 못한다..');
  console.log(e);
}
```

출처 - [poiemaweb](https://poiemaweb.com/es6-promise)

에러를 캐치하기 위해서는 try 블록 내에 있는 함수가 직접 에러를 발생시켜야 한다. `setTimeout()`의 경우 비동기 함수라 콜백 함수가 실행될 때 까지 기다리지 않고 즉시 종료되며, 정해진 시간이 지나면 tick 이벤트에 의해 콜백 함수가 실행된다. `setTimeOut()`이 콜백 함수의 호출자가 아니게 되는 것이다.

`try-catch`문은 `setTimeout()`를 바라보고 있지만, `setTimeOut()`이 직접적으로 에러를 발생시킨 것이 아니기 때문에 캐치하지 못하는 것이다.

이러한 문제점을 해결하기 위해 Promise가 등장했다.

###

### Promise 객체

promise는 `new Promise`로 생성된다.

```js
const promise = new Promise((resolve, reject) => {
    // ...
});
```

* resolve : 내부의 비동기 작업이 성공했을 경우 실행되는 콜백 함수
* reject : 내부의 비동기 작업이 실패했을 경우 실행되는 콜백 함수

두 콜백함수는 인자 값을 전달하며, 인자 값은 Promise의 결과(반환) 값이다. 인자 값으로 보통 resolve는 value, reject는 error를 전달한다.

또 Promise는 상황에 따라 상태를 가지며, 상태의 종류는 다음과 같다.

* pending : 대기 상태 result(undefined)
* fulfilled : 이행 성공, resolve 함수 실행
* rejected : 이행 실패, reject 함수 실행

###

### Promise 메서드

#### then()

`then()`은 Promise의 후속 처리 메서드다. 이 메서드를 통해 비동기 처리 결과 및 에러 메세지를 전달 받아 처리할 수 있다.

```js
const promise = new Promise((resolve, reject) => {
    resolve('value');
    reject('error');
});

promise.then(
    function(value){ console.log(value) }, // resorve 함수
    function(error){ console.log(error) } // reject 함수
)
```

위와 같이 `then()`은 두 개의 인자를 가지는데, 자동으로 첫번째는 resorve 함수, 두번째는 reject 함수다. promise 내의 비동기 처리 결과에 따라 어떤 함수를 실행시킬지 정해진다.

`then()`문은 체인 형식으로 여러번 사용할 수 있고, 순서대로 이행된다.

```js
promise
    .then((result) => step1(result)) // 성공했을 때 이행되는 함수 1
    .then((result) => step2(result)) // 성공했을 때 이행되는 함수 2
    .then((result) => step3(result)) // 성공했을 때 이행되는 함수 3
    .then(undefined, (error) => console.log(error)); // 실패했을 때 뒷 함수 이행
```

#### catch()

```js
promise.then(
    function(value){ console.log(value) },
    function(error){ console.log(error) }
)

promise.then(
    function(value){ console.log(value) },
).catch(
    function(error){ console.log(error) }
)
```

위 두 코드는 동일한 결과를 가진다. then에서 두가지 함수를 모두 쓰기보다, `catch()`문을 사용하면 가독성이 더 좋다. 또한, `catch()`문을 사용했을 때 앞의 `then()`에서 난 에러도 같이 잡아줄 수 있어 기능적으로도 더 좋다. 웬만하면 `catch()`를 사용하자.

#### finally()

마지막에 `.finally()`문도 추가해줄 수 있는데, 이는 비동기 함수의 성공, 실패의 여부와 관계없이 무조건 마지막에 실행되는 함수다. 로딩화면 같은 것을 없앨 때 용이하다.

### 개선된 Promise

`then()`을 많이 사용하여 가독성이 떨어지는 경우가 있다. 이를 then 지옥 이라고도 한다. 가독성을 개선하기 위해 `asnyc-await`가 등장했다.

#### async

함수 앞에 `async`를 붙여주면 Promise를 반환한다. 즉, 함수 이행 성공과 실패에 따라 resolve, reject 함수를 실행할 수 있다.`then()`도 사용 가능하다.

```js
async function func(){
    // ...
}

func().then((result)=>{
    // ... 
}).catch((err)=>{
    // ...
});
```

#### await

`await`는 Promise가 이행을 성공하거나 실패할 때까지 `async` 함수의 실행을 일시정지하고, 성공하면 다시 일시정지된 곳부터 실행한다. 이는 마치 `then()`과 같다.

`async` 함수 이행 실패 시, 즉시 reject된 값을 던진다. (이후는 실행 안함) 만약 따로 에러 처리를 해주고 싶다면, `try-catch`문을 사용한다.

```js
const func1 = () => {
    return new Promise((res, rej)=>{
        setTimeout(()=> {
            res("func1 실행 완료");
        }, 1000);
    });
};
const func2 = (msg) => {
    console.log(msg);
    return new Promise((res, rej)=>{
        setTimeout(()=> {
            res("func2 실행 완료");
        }, 500);
    });
};

async function run(){
    try{
        const run1 = await func1();
        const run2 = await func2(run1);
        console.log(run2);
    }catch(err){
        console.log(err);
    }
}
```

* `await`는 무조건 `async`와 함께 사용해야 한다.
* 만약 `await` 뒤의 값이 Promise가 아닐 경우, `Promise.resolve(value)`로 변경한다.
