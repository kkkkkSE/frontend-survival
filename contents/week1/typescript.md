---
description: 타입스크립트 기초 지식
---

# TypeScript

### 기본적인 타입 정의

```typescript
// 기본형 데이터
let name: string;
let age: number;
let bool: boolean;

// 참조형 데이터
let student: {
  name: string;
  age: number;
};
let arr: string[];
let tuple: [string, number] = ['string', 123];

// 함수 : 파라미터 타입, 반환값 타입 정의
funcion printInfo(name: string, age: number) : string {
  return `이름 ${name}, ${age}`;
}
```

기본 타입이 아닌 특정 값으로도 타입을 정의할 수 있다.

```typescript
let category: "MOBILE";
```

정의된 타입과 다른 타입의 데이터 할당 시 에러가 발생한다.

### 재사용 가능한 타입 정의

복잡한 타입 재사용을 위해 `Interface`나 `Type`을 사용할 수 있다.

```typescript
interface Person {
  name: string;
  age: number;
};

type Student = {
  name: string;
  age: number;
};

const person1: Person = { name: 'hong', age: 13 };
const person2: Student = { name: 'kang', age: 15 };
```

### Interface VS Type

두가지 모두 새로운 타입을 정의할 때 기존 타입을 가져다 확장하는 용도로 사용할 수 있다.

* `Type`은 **&** 를 사용하여 2개 이상의 타입을 가져다 쓸 수 있다. 이를 **Intersection Type**이라 한다.

```typescript
type Person = {
  name: string;
}
interface Adult{
  car: string;
}

type Student = Person & {
  age: number;
}
type Worker = Person & Adult; // interface를 교차시키는 것도 가능하다.
```

* `Interface`는 **extends** 를 사용하여 확장한다.

```typescript
type Person {
  name: string;
}
interface Student extends Person {
  age: number;
} // type을 상속받는 것도 가능하다.
```

_\*기존에 정의된 타입에서 확장하는 것은 **Interface**만 가능하다._

```typescript
interface Student {
  name : string;
}
interface Student {
  age : number;
}

let person: Student = {name: 'kim', age: 23};
```

### Union Type

`|`기호를 사용하여 2가지 이상의 타입을 정의할 수 있다. Union 타입은 Parameter 제한에 용이하다.

```typescript
let data: number | string;

type Category = "DESKTOP" | "MOBILE";
function fetchProducts (category : Category){
    console.log(category);
}
```

### Optional Property

변수나 객체의 Key와 함께 `?` 기호를 사용하면 값이 무조건 존재하지는 않는다는 의미이다.

데이터가 할당되지 않은 Optional Property 접근 시 `undefined` 값을 얻게 되므로 주의해야 한다.

```typescript
type Student = {
  name: string;
  age?: number;
};
const person1: Student = { name: 'hong' };

function add(a : number, b? : number) : number {
	return a + (b || 0);
}
```

### Default 값 할당

Optional Property처럼 인자 값을 전달하지 않았을 때 undefined 값을 얻는 것이 불편하다면 기본 값을 할당할 수 있다. 이 경우도 동일하게 인자 값을 전달하지 않아도 에러가 나지 않는다.

```typescript
function add2(a : number = 0, b : number = 0) : number {
	return a + b;
}
add2() // return 0
add2(1) // return 1
```

### 타입 추론

변수에 타입을 정의하지 않더라도 타입스크립트는 자동으로 타입을 추론하여 결정한다. 여러 타입의 데이터를 할당하더라도 최적의 타입을 추론한다.

### 간단히 REPL(Read-Eval-Print-Loop) 해보기

* 콘솔 환경에서 TypeScript를 사용하기 위해 `ts-node`를 이용할 수 있다.

```typescript
npx ts-node
```
