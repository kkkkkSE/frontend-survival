# React Router

Router란, 사용자 요청(URL)에 따라 적절한 컨텐츠를 보여주거나 페이지를 이동시키기 위한 기술이다.

React Router는 React에서 라우터 기능을 지원하는 라이브러리다. React 자체에서는 라우터 기능을 제공하지 않으므로 별도의 라이브러리를 설치해서 사용해야 한다.

## 설치하기

```bash
npm i react-router-dom
```

## 기본 구성 요소

### Route

Route는 기본적으로 path와 컴포넌트를 연결해준다. `to` 프로퍼티에 path, `element` 프로퍼티에 컴포넌트를 넣어주면 해당 path에 접근 시 해당 컴포넌트가 렌더링된다.

```js
<Route path="/about" element={<About />}>
```

### Routes

Routes는 Route를 그룹화하기 위해 사용한다. 어느 위치에서나 Routes를 사용할 수 있으며, Routes를 사용한 위치(경로)가 베이스가 되어 현재 path에 대한 하위 path Route의 집합이라 할 수 있다.

URL이 변경되면 Routes는 하위 Route를 탐색하여 가장 일치하는 path를 찾아 컴포넌트를 렌더링한다.

Routes 내의 [Route는 서로 중첩](react-router-feature.md#undefined-1)될 수 있으며, 중첩 구조는 하위 경로에 대한 Route를 나타낸다.

```js
<Routes>
  <Route path="/" element={<Dashboard />}>
    <Route path="messages" element={<DashboardMessages />}/>
    <Route path="tasks" element={<DashboardTasks />} />
  </Route>
  <Route path="about" element={<AboutPage />} />
</Routes>
```

### Outlet

라우트를 중첩 구조로 구성했을 경우, `Outlet`이라는 컴포넌트로 하위 경로의 컴포넌트가 어디에 표시될 지를 체크해줘야 한다.

```js
const App = () => {
    return (
        <div>
            <Header />
            <Routes>
                <Route path="/" element={<HomePage />} />
                <Route path="/about" element={<AboutPage />}>
                    <Route path="123" element={<AboutDetailPage />} />
                </Route>
            </Routes>
            <Footer />
        </div>
    );
};
```

위 예시의 경우, `HomePage`, `AboutPage` 컴포넌트는 화면상에서 `Routes`의 위치인 `Header`와 `Footer` 사이에 표시된다.

하지만 `AboutDetailPage`는 부모 라우트인 `AboutPage` 안에서 표시되어야 하므로, `AboutPage` 컴포넌트에서 하위 경로의 컴포넌트를 표시할 위치에 `Outlet` 을 추가해줘야 한다.

```js
const AboutPage = () => {
    return (
        <div>
            <h2>AboutPage</h2>
            <Outlet />
        </div>
    );
};
```

예시 코드대로라면 페이지 제목이 표시된 h2 태그 아래에 `AboutDetailPage` 컴포넌트가 렌더링 된다.

## Router

### BrowserRouter

브라우저 라우터는 브라우저 환경에서 [history API](https://developer.mozilla.org/ko/docs/Web/API/History\_API)를 사용하여 라우팅을 처리하는 역할을 한다. URL을 읽고 매핑되는 컴포넌트를 렌더링 시켜주거나 뒤로가기와 같은 사용자 이벤트를 처리하여 UI를 업데이트 한다.

브라우저 라우터는 (라우트를 사용할) 최상위 컴포넌트를 감싸는 형태로 사용한다.

```js
import { BrowserRouter } from "react-router-dom";

const rootNode = document.getElementById('root');

ReactDOM.createRoot(rootNode).render(
  <BrowserRouter>
    <App />
  </BrowserRouter>
);
```

### MemoryRouter

메모리 라우터는 메모리 상에서 라우팅을 처리하는 역할을 한다. 테스트 환경에서는 브라우저를 사용하지 않으므로 메모리 상에서 라우팅을 테스트한다.

브라우저 라우터가 `App`을 감쌌던 것처럼 메모리 라우터도 동일한 형태를 가져야 한다.

* `initialEntries` : 테스트할 path를 설정해줄 수 있으며, 프로퍼티 값은 배열 형태로 들어가야 한다.

```js
describe('App', () => {
    context('when the current path is “/”', () => {
        it('renders the home page', () => {
            render((
                <MemoryRouter initialEntries={['/']}>
                    <App />
                </MemoryRouter>
            ));
            screen.getByText(/Hello/);
        });
    });
});
```

### createBrowserRouter

React Router 6.4버전부터 지원한 방법으로, **라우터를 객체로 직접 정의**하여 사용할 수 있다.

라우터를 생성하기 위해 **`createBrowserRouter()`** 를 사용한다. 두번째 인자인 [opt](https://reactrouter.com/en/main/routers/create-browser-router#type-declaration)는 옵션으로 넣을 수 있다.

```js
const router = createBrowserRouter(routes[, opt]);
```

첫번째 인자인 **routes**는 배열 구조이며, **route 객체**로 이루어져 있다.

**route 객체**는 `path`, `element`, `children` 등 [다양한 프로퍼티](https://reactrouter.com/en/main/route/route#type-declaration)로 구성된다.

* path : 컴포넌트와 연결할 경로
* element : 경로와 연결할 컴포넌트
* children : 하위 경로에 route를 추가해줄 때 사용(중첩 구조). routes와 동일하게 배열 구조이고, route 객체로 이루어진다.

아래 코드는 기존의 Routes 구성 방식을 routes 배열로 변경한 예시다.

```js
// 기존 방식
<Routes>
  <Route path="/" element={<HomePage />} />
  <Route path="/about" element={<AboutPage />}>
    <Route path=":id" element={<AboutDetailPage />} />
  </Route>
</Routes>
```

```js
// routes 배열로 변경
const routes = [
  { path: '/',
    element: <HomePage />
  },
  {
    path: '/about',
    element: <AboutPage />,
    children: [
      {
        path: ':id',
        element: <AboutDetailPage />
      }
    ]
  }
];
```

### RouterProvider

routes 배열을 만들고 `createBrowserRouter`로 만든 라우터를 사용할 준비가 되었으면, 원래 `BrowserRouter`가 있던 자리에 새롭게 만든 라우터를 적용시키기 위해 **`RouterProvider`** 를 대신 추가한다.

```js
const router = createBrowserRouter(routes);

root.render(
    <RouterProvider router={router} />
);
```

참고로 `RouterProvider`를 사용해서 홈페이지(`'/'`)에 접속하게 되면, 루트 경로와 매핑되는 컴포넌트만 불러오게 된다. 즉, 위 예시로 따지면 **`RouterProvider`가 `App` 컴포넌트를 감싸고 있더라도 `App` 컴포넌트를 거치지 않고 바로 `HomePage` 컴포넌트만 나타낸다**는 의미다.

그렇다면 라우터 객체를 사용했을 때 Header, Footer, Nav 등을 기본 레이아웃으로 잡고싶다면 어떻게 해야할까?

#### Router에 레이아웃 포함시키기

레이아웃을 담당할 컴포넌트를 하나의 라우트라 생각하고, 원래 담겨있던 routes 객체를 children에 담는 방식으로 사용할 수 있다.

```js
import { Outlet } from 'react-router-dom';

function Layout() {
  return (
    <div>
      <Header />
      <Outlet /> {/* 라우트된 컴포넌트 표시할 곳 */}
      <Footer />
    </div>
  );
}
```

```js
const routes = [
  {
    element: <Layout />, // 기본 레이아웃
    children: [ // 기존 Routes 배열
      { path: '/', element: <HomePage /> },
      {
        path: '/about',
        element: <AboutPage />,
        children: [
          {
            path: ':id',
            element: <AboutDetailPage />
          }
        ]
      }
    ],
  },
];    
```

### createMemoryRouter

`createBrowserRouter`로 라우터를 만들어 사용하면 기존에 테스트할 때 썼던 `MemoryRouter`는 사용할 수 없다. 대신 `createMemoryRouter`로 메모리 상에서 라우팅을 테스트 할 수 있다.

1. `createBrowserRouter`로 라우터를 만들어줬던 것처럼 `createMemoryRouter`로 라우터를 만들어줘야한다.
2. `createMemoryRouter`의 **첫번째 인자로 routes 배열**이 필요하고 **두번째 인자에서 테스트 경로**를 잡을 수 있다.
3. `render()` 함수에서는 `createMemoryRouter`로 만든 라우터를 `router` 프로퍼티에 적용한 `RouterProvider`를 렌더해줘야 한다.

```js
describe('App', () => {
    context('when the current path is “/”', () => {
        it('renders the home page', () => {
            const router = createMemoryRouter(routes, { initialEntries: ['/'] });
            render((<RouterProvider router={router} />));
            
            screen.getByText(/Hello/);
        });
    });
});
```

> `createBrowserRouter`를 이용하여 라우터를 만들 경우, routes 배열을 위한 `routes.tsx` 파일을 따로 만들면 관리도 용이하고 테스트하기에도 편하다. `routes.tsx`에 대한 테스트 파일은 `routes.test.tsx`로 만들어쓰자.

## Navigation

일반적인 웹페이지에서 URL의 변경이 일어나면 새로고침이 발생한다. 새로고침에 대한 좋지 않은 현상을 방지하고자 새로고침을 하지 않고도 새로운 페이지를 가져오려는 노력은 예전부터 이어져왔다.

### location.hash

URL의 해시(fragment)를 이용해서 일부 화면을 업데이트할 수 있고, 업데이트한 내용에 대해 뒤로 가기, 앞으로 가기도 가능하다.

```js
window.location.hash = 'about';
```

```js
window.addEventListener('hashchange', () => {
  if (window.location.hash === '#about') {
    // about에 관련된 내용으로 뷰 업데이트 코드
  } else {
    // ...
  }
});
```

### history.pushState

history API의 `pushState` 메서드는 새로고침 없이 URL을 변경해주는 역할을 한다. 이는 브라우저의 세션 기록에 추가되어, 뒤로 가기와 앞으로 가기가 가능하다.

```js
const state = {};
const title = '';
const url = '/about';

history.pushState(state, title, url);
```

* state : 저장할 상태 객체
* title : 페이지의 탭에 표시될 타이틀(대부분의 브라우저에서는 적용되지 않고 무시됨)
* url : 업데이트 할 URL 주소

{% hint style="info" %}
**state**는 key-value 쌍의 상태 객체를 받는다. `pushState`에 의해 상태 객체가 생성되면 history에서 해당 객체의 복사본을 갖게 된다.

이 상태 객체를 받아오려면 **`popstate` 이벤트가 발생**해야하며, `popstate` 이벤트는 뒤로가기나 앞으로가기를 통해서 발생시킬 수 있다. 해당 이벤트 발생 후 `event.state`에서 상태 객체를 받아올 수 있다.

다만, 뒤로 가기 또는 앞으로 가기를 통해 **pushState에서 기입한 `url`에 접근**해야 해당 상태 객체를 받는 것이 가능하다.

```js
const AboutPage = () => {
    useEffect(() => {
        // popstate 발생 시 상태 객체 출력 이벤트 등록
        window.onpopstate = function(event) { 
          console.log(JSON.stringify(event.state));
        };
      }, []);

    const handleClick = () => {
        history.pushState({page: 16}, "title 1", "/about/123");
    }
    // 버튼 클릭 후 뒤로가기 -> 앞으로가기로 '/about/123' 에 접근하면 콘솔에 state가 출력된다.

    return (
        <div>
            AboutPage
            <button onClick={handleClick}>디테일 페이지로 이동</button>
            <Outlet />
        </div>
    );
};
```
{% endhint %}

하지만 React에서 `pushState` 메서드로 URL을 변경하면 URL 주소만 변경되고, 화면은 업데이트 되지 않는다. React가 URL의 변화를 감지하지 못해 DOM을 업데이트하지 못하기 때문이다. 이러한 이슈를 해결하기 위해 사용하는 것이 아래 React Router의 네가지 요소이다.

### Link

리액트 라우터에서 지원하는 컴포넌트로, `a` 태그와 유사한 동작을 한다.

* `a`태그처럼 특정 path에 링크된 버튼처럼 동작한다.
* `a`태그와 다르게 새로고침을 하지 않는다.
* `to` 프로퍼티에서 이동할 path를 받는다.
* 특정 path로 이동하고, 라우터에서 매칭된 컴포넌트로 화면을 업데이트 한다.

```js
return (
  <nav>
    <ul>
      <li><Link to="/">Home</Link></li>
      <li><Link to="/about">About</Link></li>
    </ul>
  </nav>
);  
```

### NavLink

`NavLink`는 `Link`와 거의 같으나, 추가적으로 해당 페이지가 활성된 상태인지 알 수 있는 **Active 상태**와 페이지 이동 중을 나타내는 **Pending 상태**를 제공한다.

참고로 `NavLink`에서 Active 상태에 따른 className은 default로 제공되고 있다.

```js
<NavLink
  style={({ isActive, isPending }) => { // 상태에 따른 스타일 지정 가능
    return {
      color: isActive ? "red" : "inherit",
    };
  }}
  className={({ isActive, isPending }) => { // 상태에 따라 클래스 이름 추가
    return isActive ? "active" : isPending ? "pending" : "";
  }} // 활성 시 class='active'는 기본으로 제공
/>
```

### Navigate

`Navigate`는 렌더링 될 때 현재 위치를 변경한다. 렌더링 영역에서만 사용할 수 있다.

```js
return (
    // ...
    {user && <Navigate to="/user" />}
    // ...
);
```

### useNavigate

`useNavigate`는 `Navigate`처럼 현재 위치를 변경시키지만, 자바스크립트 영역에서만 사용할 수 있다.

```js
export default function Footer() {
    const navigate = useNavigate();
        
    const handleClick = () => {
        navigate('/');
    };
        
    return (
        <div>
            <button type="button" onClick={handleClick}>
                Home
            </button>
        </div>
    );
}
```

### 테스트 시 주의점

`Navigate`는 다른 페이지로 이동하는 과정에서 이동할 페이지에 대한 정보를 가져와 뿌려준다. 브라우저에서는 이 과정이 전혀 문제없지만, 테스트 환경에서는 페이지 정보를 가져오는 기능을 자체적으로 지원하지 않기 때문에 polyfill이 필요하다. 강의에서는 `whatwg-fetch` polyfill을 사용했다.

```bash
npm i -D whatwg-fetch
```

```js
// src/setupTests.ts

import 'whatwg-fetch';
```
