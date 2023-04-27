# Style 정의하기

리액트에서 HTML 요소에 스타일을 정의하는 방법은 여러가지가 있다.

## HTML 파일 안에서 정의

HTML 파일에서 스타일을 정의해주려면 두가지가 필요하다.

- `<style>` 태그
- 스타일을 정의할 요소의 class 또는 id

`<style>` 태그는 보통 HTML 파일 내 `<head>` 태그 안에서 사용한다. 이 `<style>` 태그에 HTML 요소 또는 특정 class, id를 가진 요소의 스타일을 정의할 수 있으며, 모든 페이지에 공유된다.

```html
<head>
    <title>Document</title>
    <style>
        h1 {
            font-size: 2.8rem;
        }

        .greeting {
            color: #00F;
        }
    </style>
</head>
```

리액트에서 class를 지정하려면 `className` 프로퍼티를, id를 지정하려면 `id` 프로퍼티를 사용해야 한다. class나 id를 기준으로 스타일을 정의할 경우, 여러 요소에 같은 스타일을 정의하기 쉽다. _(사실 리액트에서는 이미 컴포넌트가 재활용되고 있기 때문에 의미없는 부분일 수 있다.)_

> 단, HTML 표준상 id는 고유 식별자로 사용되기 때문에 document 내에서 유일해야 한다. id를 사용했을 경우 리액트의 특성상 컴포넌트를 재활용하면서 2번 이상 사용될 우려가 있으므로 id 사용을 지양하는 것이 좋다.

```js
function Greeting() {
    return (
        <h1 className="greeting">
            Hello, world!
        </h1>
    );
}
```

Bootstrap이나 Tailwind CSS 등 해당 방식을 효율적으로 사용하기 위해 만들어진 도구들도 있다. 작은 스타일 그룹을 클래스로 정해놓고, 요소에 클래스만 써주면 스타일이 적용되는 방식이다. 대게 클래스 이름에서 어떤 스타일이 적용될지 바로 드러난다.

```js
function App() {
  return (
    <div className="bg-blue-500">
      <h1 className="text-white font-bold text-3xl text-center">Hello, Tailwind CSS!</h1>
    </div>
  );
}
```

## 인라인 스타일 정의

스타일을 컴포넌트 외부의 다른 곳에 정의하는게 아니라, 인라인 방식으로 요소에 직접 스타일을 줄 수 있다.

리액트에서는 스타일을 정의할 때 style 프로퍼티에 객체 형태로 정의하며, 스타일 이름은 lower Camel Case로 표기해야 한다.

자바스크립트 객체 형태로 스타일을 정의하기 때문에 변수나 함수를 활용하기 쉽다. 타입스크립트를 사용하면 자동 완성이나 타입 검사 등의 도움을 받을 수 있다.

아래 예제는 같은 결과를 나타내며, 객체를 직접 넣어줄수도, 미리 정의해서 변수로 넣어줄수도 있다.

```js
<p
    style={{
        color: '#00F',
    }}
>
    Hello, world!
</p>
```

```js
const style = {
    color: '#00F',
};

return (
    <p style={style}>
        Hello, world!
    </p>
);
```

하지만 인라인 방식을 사용하게 되면, 아래와 같은 단점이 있다.

- 스타일 시트와 달리 우선순위 규칙이 적용되지 않아 유연성이 떨어진다.
- 가독성이 나빠져 코드의 구조를 파악하는 데 방해가 된다.
- 클래스 네이밍에서 구체적인 스타일이 명시되어 있을 경우, 리뉴얼 등으로 인해 스타일이 바뀌면 유지보수가 어렵다.

## CSS in JS

CSS in JS는 자바스크립트를 사용해서 컴포넌트의 스타일을 지정하는 스타일링 기법이다. 자바스크립트가 parse되면 CSS가 생성되고, 이 CSS가 DOM에 주입되는 형식이다. 이 개념을 돕기 위한 다양한 도구가 있으며, [npm trends](https://npmtrends.com/aphrodite-vs-emotion-vs-glamorous-vs-jss-vs-radium-vs-styled-components-vs-styletron)에서 라이브러리 종류와 트랜드를 확인할 수 있다.

### CSS in JS 이전의 문제점

다음은 대규모 프로젝트를 진행할 때 기존의 CSS에서 발생할 수 있는 문제에 대해 지적된 사항이다.

- CSS는 전역 공간을 사용하므로 클래스 이름이 충돌할 수 있다.
- CSS는 HTML 요소가 다른 요소에 감싸지면서 종속성을 가질 수 있는데, 이 때 불필요한 스타일 규칙이 추가될 수 있다.
- 사용하지 않는 코드가 남아있을 수 있다.
- CSS에서 상수를 공유하려면 전역 변수를 사용해야 한다.
- CSS에서 스타일 우선순위를 결정하는 방법은 명시적이지 않다.
- CSS는 스타일 규칙의 범위를 지정하는 방법이 없다.

### 장점

CSS in JS는 대규모 프로젝트에서 CSS의 문제점에 대해 다음과 같은 해결점을 제시한다.

- 스타일 규칙을 컴포넌트 범위로 한정할 수 있어 스타일 충돌을 방지하고, 특정 컴포넌트에만 스타일을 적용할 수 있다.
- 해당 페이지에서 필요한 스타일만 적용하도록 할 수 있어 페이지 로딩 속도를 개선한다.
- 사용하지 않는 스타일 규칙을 제거하고, 중복 코드를 최소화하는 등의 최적화를 제공한다.

또한, 다른 장점도 있다.

- CSS를 문서 단위가 아닌 컴포넌트 단위로 관리할 수 있다.
  - 동일 스타일 재사용
  - 각 컴포넌트 간 스타일 규칙 격리
- 스타일 컴포넌트 간 스타일을 공유할 수 있다.
- 자바스크립트의 모든 기능을 활용할 수 있다.
- 자동으로 vendor prefix를 고려하기 때문에 코드의 중복을 줄일 수 있다.
- 단위 테스트로 스타일을 검증할 수 있다.

### 단점

CSS in JS는 말 그대로 자바스크립트 안에 CSS가 포함되어 있다. 자바스크립트가 파싱되어 화면이 완전히 표현되기 전까지 아무런 액션도 취할 수 없는데, 그 과정에서 CSS도 동적으로 생성되어 주입하는 시간이 포함되어야 한다.

반면 CSS가 따로 관리된다면 그만큼 자바스크립트를 로드하는 시간을 줄일 수 있다. 따라서 CSS in JS와 CSS를 별도로 분리하는 것은 성능에서 차이를 보일 수 있다. (상황에 따라 유의미할 수도, 아닐수도 있음)

이를 보완하기 위해 Linaria, vanilla-extract를 사용할 수 있다. 이 도구들은 빌드 단계에서 CSS를 따로 추출해주어 런타임에 생성되는 CSS가 없으므로 해당 문제에 대한 성능 보완이 가능하다.

### 인라인 VS CSS in JS

인라인 방식에서 자바스크립트 객체로 스타일을 정의한다고 했었는데, 인라인 스타일 정의와 CSS in JS는 같은 것 아닐까? 라고 할 수도 있다.

인라인 스타일 정의 방식과 CSS in JS는 다음과 같은 차이점이 있다.

#### 인라인 방식

```js
// 인라인 방식 스타일 정의
const style = {
    color: '#00F',
};

return (
    <p style={style}>
        Hello, world!
    </p>
);
```

```html
// DOM 
<p style="color: white; backgrond-color: black;">inline style!</p>
```

인라인 방식은 DOM 노드에 프로퍼티로 스타일이 부여된다.

#### CSS in JS 방식

```js
// CSS in JS (styled-components)
const Title = styled.p`
  color: '#00F'
`

<Title>Hello, World!</Title>
```

```html
// DOM
<style>
.dsz2fd4s {
  background-color: black;
  color: white;
}
</style>

<p class="dsz2fd4s">Hello CSS-in-JS</p>
```

CSS in JS에서는 DOM 상단에 style 태그를 추가한다.

#### 차이점의 의미

**인라인에서는 모든 CSS 기능을 지원하지 않는다.** `::after`, `:nth-child` 등이 포함된 가상 요소, 가상 클래스 사용할 수 없고, html, body 태그도 지원하지 않는다.

반면 CSS in JS에서는 모든 기능을 사용할 수 있다. 앞에서 언급한 가상 요소, 가상 클래스를 사용할 수 있고, 미디어 쿼리도 사용 가능하다.
