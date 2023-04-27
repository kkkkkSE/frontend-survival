# Global Style & Theme

## Reset CSS

각종 HTML 요소에는 브라우저가 제공하는 기본 스타일이 있다. 하지만 브라우저마다 기본으로 제공하는 스타일이 일부 다르고, 우리가 화면을 스타일링 하는 데 방해가 되기도 한다. 이러한 방해 요소를 제거하기 위해 CSS를 초기화하는데, 이것이 바로 Reset CSS다.

CSS를 초기화하는 요소와 방법은 개인마다, 회사마다 다 다른 방법을 채택하고 있다. 강의에서는 npm에 등록되어 있는 **styled-reset**를 추천했다. styled-components를 위한 Reset CSS 도구다.

배포에 포함시키려면 dependency로 설치해야 한다.

```bash
npm i styled-reset
```

최상위 컴포넌트에서 `Reset` 컴포넌트를 불러온다.

```js
import { Reset } from 'styled-reset';

export default function App() {
    return (
        <>
            <Reset />
            <Main />
        </>
    );
}
```

{% hint style="info" %}
Reset CSS와 Normalize는 비슷하지만 다르다.

- Reset CSS : HTML 요소의 기본 스타일을 제거(초기화)하여 브라우저 간 동일한 스타일을 적용할 수 있게 도와줌.
- Normalize CSS : 브라우저 간 스타일의 일관성을 제공하기 위해 기본 스타일을 재정의 하는 것.
{% endhint %}

## Global Style

모든 컴포넌트에서 공동적으로 사용할 스타일에 대해 정의한 것을 Global Style이라 한다.

보통 Global Style에서는 `box-sizing`, `word-break` 설정과 루트 폰트 사이즈를 설정한다.

{% hint style="info" %}
웹의 기본 폰트 사이즈(1em)가 보통 16px인데, `html` 태그에서 `font-size: 62.5%`로 설정하면 루트 사이즈 (1rem)이 10px로 설정된다.

이렇게 해뒀을 때의 장점으로는, 1rem을 10px로 설정했을 때 우리가 rem 단위로 폰트 사이즈를 설정하더라도 픽셀로 몇인지 쉽게 알 수 있다.

또한, rem 단위를 사용하면 만약 화면의 크기나 해상도가 달라지면서 기본 폰트 사이즈가 달라지게 되면 그에 맞게 폰트 사이즈의 비율이 달라지기 때문에 반응형 웹에 맞는 방식이라 할 수 있다.
{% endhint %}

styled-components에서는 `createGlobalStyle`를 사용해서 Global Style을 정의할 수 있다. Reset CSS와 마찬가지로 최상위 컴포넌트에서 불러오면 된다.

```js
import { createGlobalStyle } from 'styled-components';

const GlobalStyle = createGlobalStyle`
    html {
        box-sizing: border-box;
    }
        
    *,
    *::before,
    *::after {
        box-sizing: inherit;
    }
        
    html {
        font-size: 62.5%;
    }
        
    body {
        font-size: 1.6rem;
    }
        
    :lang(ko) {
        h1, h2, h3 {
            word-break: keep-all;
        }
    }
`;

export default GlobalStyle;
```

```js
import { Reset } from 'styled-reset';
import GlobalStyle from './styles/GlobalStyle';

export default function App() {
    return (
        <>
            <Reset />
            <GlobalStyle />
            <Main />
        </>
    );
}
```

Reset 컴포넌트와 GlobalStyle 컴포넌트의 순서에 유의하자.

## Theme

Theme은 웹의 전반적인 디자인에 적용되는 스타일을 일관성있게 관리하는 방법으로서, 디자인 시스템의 근간을 마련하는데 활용한다.

보통은 Color, Font, Spacing 등의 작은 스타일 정보를 변수에 저장해놓고 사용한다. 이 방식을 사용하면 나중에 테마 스타일이 바뀌었을 때, 관리하기 매우 편해진다.

> 스타일 관련 파일은 `styled/` 폴더를 따로 만들어 관리하자.

```js
const defaultTheme = {
    colors: {
        background: '#FFF',
        text: '#000',
        primary: '#F00',
        secondary: '#00F',
    },
};

export default defaultTheme;
```

Theme 또한 Reset CSS, GlobalStyle 처럼 최상위 컴포넌트에서 불러오면 되지만, Theme은 `ThemeProvider`로 전체를 감싸는 형태로 사용한다. `ThemeProvider` 속성으로 위에서 만든 Theme 객체를 넘겨주면 된다.

```js
import { ThemeProvider } from 'styled-components';

import { Reset } from 'styled-reset';

import defaultTheme from './styles/defaultTheme';

import GlobalStyle from './styles/GlobalStyle';

export default function App() {
    return (
        <ThemeProvider theme={defaultTheme}>
            <Reset />
            <GlobalStyle />
            <Greeting />
        </ThemeProvider>
    );
}
```

### Theme 타입 정의

**ThemeProvider가 제공하는 Theme을 import 없이 자유롭게 사용하려면 `DefaultTheme` 타입을 정의해야 한다.**

스타일 관련 파일을 모아놓은 폴더 안에 `styled.d.ts` 파일에 해당 타입을 정의하면 된다.

```js
import 'styled-components';

declare module 'styled-components' {
    export interface DefaultTheme {
        colors: {
            background: string;
            text: string;
            primary: string;
            secondary: string;
        }
    }
}
```

하지만 이렇게 작성했을 경우, `defaultTheme`에서 스타일이 추가/제거 되었을 때 매번 변경해줘야 하니 번거로울 수 있다. 그래서 `defaultTheme`에 대한 타입을 직접 적어주는게 아니라, `defaultTheme`의 타입을 가져와서 적용시켜버리자.

```js
import defaultTheme from './defaultTheme';

type Theme = typeof defaultTheme;

export default Theme;
```

```js
import 'styled-components';

import Theme from './Theme';

declare module 'styled-components' {
    export interface DefaultTheme extends Theme {}
}
```

이렇게 `defaultTheme`의 타입을 추출한 `Theme`을 `DefaultTheme` 에 상속시켜 아무것도 직접 적어줄 필요 없이 `DefaultTheme`에 대한 타입 작성을 완료할 수 있다.

이후에는 모든 컴포넌트에서 Theme에 있는 스타일을 import없이 자유롭게 가져다 쓸 수 있다.

```js
import { createGlobalStyle } from 'styled-components';

const GlobalStyle = createGlobalStyle`
    body {
        background: ${(props) => props.theme.colors.background};
        color: ${(props) => props.theme.colors.text};
    }

    input,
    textarea {
        background: ${(props) => props.theme.colors.background};
        color: ${(props) => props.theme.colors.text};
    }
`;

export default GlobalStyle;
```

### Theme 활용하여 다크모드 만들기

`defaultTheme`처럼 파일을 작성하면 되는데, 이 때 앞에서 만든 `Theme` 타입을 가져다 쓴다.

```js
import Theme from './Theme';

const darkTheme: Theme = {
    colors: {
        background: '#000',
        text: '#FFF',
        primary: '#F00',
        secondary: '#00F',
    },
};

export default darkTheme;
```

이제 상황에 맞게 해당 Theme을 사용하기만 하면 끝!

```js
import { useDarkMode } from 'usehooks-ts';

import { ThemeProvider } from 'styled-components';

import { Reset } from 'styled-reset';

import defaultTheme from './styles/defaultTheme';
import darkTheme from './styles/darkTheme';

import GlobalStyle from './styles/GlobalStyle';

import Greeting from './components/Greeting';
import Button from './components/Button';

export default function App() {
    const { isDarkMode, toggle } = useDarkMode();

    const theme = isDarkMode ? darkTheme : defaultTheme;

    return (
        <ThemeProvider theme={theme}>
            <Reset />
            <GlobalStyle />
            <Greeting />
            <Button onClick={toggle}>
                Dark Theme Toggle
            </Button>
        </ThemeProvider>
    );
}
```

### Theme 테스트 문제

위 코드처럼 다른 Theme를 사용하면 테스트 시 Jest에서 `window.matchMedia` 에러를 발생시킨다.

`window.matchMedia`는 미디어 쿼리를 사용하여 Viewport의 너비, 높이를 검사 후 스타일을 변경하는 등의 작업에 사용되는 속성이다.

테스트 환경에서는 `window` 객체가 없기 때문에 발생하는 에러라서, 해당 부분을 Mocking하여 해결할 수 있다.([관련 문서](https://jestjs.io/docs/manual-mocks#mocking-methods-which-are-not-implemented-in-jsdom))

`window.matchMedia` Mocking한 코드를 `src/setupTests.ts` 파일에 넣어주기만 하면 정상적으로 동작한다.

```js
Object.defineProperty(window, 'matchMedia', {
    writable: true,
    value: jest.fn().mockImplementation((query) => ({
        matches: false,
        media: query,
        onchange: null,
        addListener: jest.fn(), // deprecated
        removeListener: jest.fn(), // deprecated
        addEventListener: jest.fn(),
        removeEventListener: jest.fn(),
        dispatchEvent: jest.fn(),
    })),
});
```