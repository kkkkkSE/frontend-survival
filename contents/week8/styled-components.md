# styled-components

styled-components는 현재 시점(2023.04)에서 가장 인기있는 CSS in JS 도구다. 이 도구를 이용하면 스타일이 적용된 컴포넌트를 손쉽게 만들 수 있다.

## 특징

styled-components 팀에서 말하는 styled-components의 이점에는 아래와 같은 것들이 있다.

- 어떤 컴포넌트가 렌더링 되었는지 추적하여 해당 스타일만 주입하여 최소한의 코드만 로드한다.
- 스타일 컴포넌트에 고유한 클래스 네임을 자동으로 부여한다. 클래스 네임 간 충돌이나 오타의 걱정이 없다.
- 스타일이 컴포넌트로 정의되어 있기 때문에 어디서 사용되었는지 명확히 알 수 있고, 사용되지 않았거나 삭제되었다면 해당 스타일도 모두 제거된다.
- props나 global theme를 통해 자바스크립트로 스타일을 조정하는 것이 가능하며, 이로 인해 많은 클래스를 관리할 필요가 없고 직관적으로 스타일을 적용할 수 있다.
- 컴포넌트와 연결된 스타일을 찾기 매우 쉬워 유지보수가 편리하다.
- 자동으로 vender prefix를 추가해준다.

## 사용하기 전에

vscode의 확장 도구 중 **vscode-styled-components**를 반드시 설치하자.

스타일드 컴포넌트에서는 백틱 기호 안에서 스타일을 정의하기 때문에 vscode에서 이를 일반 문자열처럼 받아들인다.

해당 확장 도구를 사용하면 vscode가 스타일을 일반 문자열이 아닌 CSS 코드로 받아들여 스타일 이름에 대한 자동 완성 기능을 제공해주고, 스타일 이름과 설정값을 색상으로 구분해준다.

## 설치 및 세팅

스타일드 컴포넌트를 사용하려면 컴파일러 또는 트랜스파일러(바벨, swc 등)를 위한 플러그인을 설치해주고, 타입 정의 파일도 같이 설치해주자.

```bash
npm i styled-components
npm i -D @types/styled-components @swc/plugin-styled-components
```

그리고 `.swcrc` 파일에서 해당 플러그인에 대한 설정을 기재해주자.

```json
{
    "jsc": {
        "experimental": {
            "plugins": [
                [
                    "@swc/plugin-styled-components",
                    {
                        "displayName": true,
                        "ssr": true
                    }
                ]
            ]
        }
    }
}
```

## 사용법

### 기본 형태

```js
const Title = styled.h1`
    color: #00F;
`;

export default function Greeting() {
    return (
        <Title>Hello, world!</Title>
    );
}
```

- 스타일 컴포넌트의 이름은 대문자로 시작한다.
- 스타일 컴포넌트는 `styled.`로 시작하며, 바로 이후에는 HTML 태그명이 들어가고 해당 요소에 고유한 클래스명을 부여하게 된다.
- 컴포넌트가 재사용됐을 때, 해당 고유 클래스명을 사용한다.
- HTML 태그명 이후에는 백틱으로 감싸진 공간에서 스타일을 정의한다.

### 중첩 스타일 규칙

컴포넌트 안에 포함되어 있는 요소들의 스타일을 중첩문으로 작성할 수 있으며, 구조를 파악하기 편리하고 스타일을 한 곳에서 관리하기 때문에 편리하다.

```js
const Title = styled.h1`
    font-size: 3.2rem;
    color: #00F;

    span {
        color: #F00;
    }
`;

export default function Greeting() {
    return (
        <Title>
        Hello, 
        <span>World!</span>
        </Title>
    );
}
```

예제에서 span 태그의 부모가 h1 태그이므로, [CSS 규칙](/contents/week8/css.md#css-규칙)에 따라 부모 태그의 스타일을 span이 물려받고, 나중에 작성된 스타일이 우선으로 적용된다.

### 스타일 컴포넌트 상속

다른 컴포넌트를 가져와 새로운 스타일을 추가할 수 있다.

`styled.element` 대신 `styled.(Component)`형태로 사용한다.

```js
const Paragraph = styled.p`
    color: #00F;
`;

const BigParagraph = styled(Paragraph)`
    font-size: 5rem;
`;

function Greeting() {
    return (
        <BigParagraph>
            Hello, world!
        </BigParagraph>
    );
}
```

위처럼 스타일 컴포넌트에 새로운 스타일을 추가하는 것도 가능하지만, UI 컴포넌트에도 스타일을 추가할 수 있다.

다만, UI 컴포넌트에 스타일을 추가할 땐 `styled`가 생성한 고유 className를 직접 넘겨줘야 한다. className을 넘겨줄 때 타입 명시도 주의하자.

```js
function HelloWorld({ className }: React.HTMLAttributes<HTMLElement>) {
    return (
        <p className={className}>
            Hello, world!
        </p>
    );
}

const Greeting = styled(HelloWorld)`
    color: #00F;
`;
```

### props

props는 활성화 여부에 따른 스타일 변화와 같은 동적 스타일을 주고 싶을 때 사용한다.

하위 컴포넌트로 props 객체를 넘겨주는 것처럼, 스타일 컴포넌트로 props 객체를 넘겨주는 것과 비슷한 형태다.

- 타입스크립트 사용 시, props에 대한 타입을 명시해야 한다.
- 동적인 값을 받으려면 `${props => props.property}` 형태로 프로퍼티에 접근해야 한다.
- 특정 props의 상태에 따라 새롭게 스타일을 추가하는 경우에는 `${props => props.property && css``}` 형태로 사용할 수 있다. 백틱 앞에 `css`를 붙이면, 백틱 내의 문자열을 CSS 코드로 받아들인다.

```js
import { useBoolean } from 'usehooks-ts';

import styled, { css } from 'styled-components';

type ParagraphProps = {
    active?: boolean;
}

const Paragraph = styled.p<ParagraphProps>`
    color: ${(props) => (props.active ? '#F00' : '#888')};
    ${(props) => props.active && css`
        font-weight: bold;
    `}
`;

export default function Greeting() {
    const { value: active, toggle } = useBoolean(false);
        
    return (
        <div>
            <Paragraph>
                Inactive
            </Paragraph>
            <Paragraph active>
                Active
            </Paragraph>
            <Paragraph active={active}>
                Hello, world
                {' '}
                <button type="button" onClick={toggle}>
                    Toggle
                </button>
            </Paragraph>
        </div>
    );
}
```

### attrs

HTML 요소의 기본 속성도 스타일 컴포넌트에서 부여할 수 있다. 형태는 `styled.button.attr(속성객체)` 로 사용할 수 있다.

button이나 input의 type 속성처럼 한 가지 속성을 많이 사용한다면 이 기능을 유용하게 사용할 수 있다.

```js
const Button = styled.button.attrs({
    type: 'button',
})`
    background: #FFF;
    font-size: 1.8rem;
`;
```

이후에 해당 스타일 컴포넌트를 사용하면, type을 직접 명시하지 않아도 자동으로 `type='button'`이 입력된다.

만약 속성의 기본 값을 정해놓고, 다른 속성을 사용하고 싶을때만 직접 입력하고 싶을 때는 이렇게도 사용할 수 있다.

```js
type ButtonProps = {
    type?: 'button' | 'submit' | 'reset';
    active?: boolean;
}

// 주의 : 속성에 대한 타입, props에 대한 타입을 따로 잡아줘야 함!
const Button = styled.button.attrs<ButtonProps>((props) => ({
    type: props.type ?? 'button',
}))<ButtonProps>` // action에 대한 타입명시
    background: ${(props) => (props.active ? '#FFF' : '#888')};
    font-size: 1.8rem;
`;

function Greeting() {
    const { value: active, toggle } = useBoolean(false);
        
    return (
        <div>
            <Button type="submit" onClick={toggle} action={action}>
                Submit
            </Button>
        </div>
    );
}
```
