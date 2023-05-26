# 로그인

관리자 웹에서는 모든 페이지에 접근하기 위해서는 로그인을 반드시 거쳐야한다. 첫 화면이 로그인 화면이고, 로그인 후에 모든 페이지에 접근할 권한이 생긴다.

이를 반영한 routes 파일은 다음과 같이 구성된다.

```js
const routes = [
  { path: '/login', element: <LoginPage /> },
  {
    element: <Layout />,
    children: [
      { path: '/', element: <HomePage /> },
      // ...
    ],
  },
];
```

로그인 Form이나 AccessToken을 로컬스토리지에서 관리하고, 사용자 웹과 동일하게 사용하는 ApiService는 복붙해서 가져오자.

## 로그인 화면

- 일반 사용자가 이 페이지에 접근하게 됐을 경우를 고려하여 관리자 페이지인지 구분하지 못하게 하기 위해 로그인에 필요한 구성 외에 모든 것은 제외한다.
- `Layout`이 적용되지 않기 때문에 따로 레이아웃을 잡아줘야 한다.

```js
// Layout.tsx
import { Outlet } from 'react-router-dom';

import styled from 'styled-components';

import Header from './Header';

import useCheckAccessToken from '../hooks/useCheckAccessToken';

const Container = styled.div`
  margin-inline: auto;
  width: 90%;
`;

export default function Layout() {
  const ready = useCheckAccessToken();

  if (!ready) {
    return null;
  }

  return (
    <Container>
      <Header />
      <Outlet />
    </Container>
  );
}
```

## 로그인 상태 체크(`useCheckAccessToken`)

- 로그인한 회원 정보를 얻는 과정에 성공하면 정상적으로 로그인 되어있단 의미이므로 로그인 성공 신호를 반환하고, 반대의 경우엔 로컬 스토리지의 AccessToken을 비워준다.
- useEffect를 이용해서 accessToken이 없으면 무조건 로그인 화면으로 이동하도록 한다.(페이지 이동 때마다 체크)

```js
import { useEffect, useState } from 'react';

import { useNavigate } from 'react-router-dom';

import useAccessToken from './useAccessToken';

import { apiService } from '../services/ApiService';

export default function useCheckAccessToken(): boolean {
  const [ready, setReady] = useState(false);

  const { accessToken, setAccessToken } = useAccessToken();

  const navigate = useNavigate();

  useEffect(() => {
    const fetchCurrentUser = async () => {
      try {
        await apiService.fetchCurrentUser();
        setReady(true); // 정상적인 로그인 상태 확인
      } catch (e) {
        setAccessToken('');
      }
    };

    fetchCurrentUser();
  }, [accessToken, setAccessToken]);

  useEffect(() => {
    if (!accessToken) {
      navigate('/login');
    }
  }, [accessToken, navigate]);

  return ready;
}
```

## 페이지 내에서 로그인 상태 체크

- 로그인 페이지를 제외한 모든 페이지가 `Layout`과 함께 렌더링되기 때문에, `Layout` 컴포넌트에서 로그인 상태를 체크한다.
- 로그인 상태 값을 반환받고, 정상적인 로그인 상태가 아니라면 화면을 아예 비운다. (어차피 로그인 화면으로 이동되긴 함)

```js
import { Outlet } from 'react-router-dom';

import styled from 'styled-components';

import Header from './Header';

import useCheckAccessToken from '../hooks/useCheckAccessToken';

const Container = styled.div`
  margin-inline: auto;
  width: 90%;
`;

export default function Layout() {
  const ready = useCheckAccessToken();

  if (!ready) {
    return null;
  }

  return (
    <Container>
      <Header />
      <Outlet />
    </Container>
  );
}
```

## Header

사용자가 이용하는 웹의 헤더보다 관리자 웹의 헤더는 더 심플하게 사용할 수 있다. 특히 로그인 버튼같은 경우는, 어차피 로그인 해야만 컨텐츠에 접근이 가능하기 때문에 필요하지 않다.

```js
import { Link, useNavigate } from 'react-router-dom';

import useAccessToken from '../hooks/useAccessToken';

import { apiService } from '../services/ApiService';

import Button from './ui/Button';

const Container = styled.header`
  h1{
    font-size: 2rem;
  }
`;

export default function Header() {
  const navigate = useNavigate();

  const { setAccessToken } = useAccessToken();

  const handleClickLogout = async () => {
    await apiService.logout();
    setAccessToken('');
    navigate('/');
  };

  return (
    <div>
      <h1>Shop Administrator</h1>
      <nav>
        <ul>
          <li>
            <Link to="/">Home</Link>
          </li>
          <li>
            <Link to="/users">Users</Link>
          </li>
          <li>
            <Link to="/categories">Categories</Link>
          </li>
          <li>
            <Link to="/products">Products</Link>
          </li>
          <li>
            <Link to="/orders">Orders</Link>
          </li>
          <li>
            <Button onClick={handleClickLogout}>
              Logout
            </Button>
          </li>
        </ul>
      </nav>
    </div>
  );
}
```
