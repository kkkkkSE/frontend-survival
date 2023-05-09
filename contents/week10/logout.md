# 로그아웃 기능 구현

## 사용자 정보 확인

사이트에 접속하거나 새로고침을 하는 등의 작업에선 AccessToken을 확인하고 현재 접속한 사용자(본인) 정보를 얻는 작업이 필요하다.

만약 사용자 정보를 얻는 API가 실패하면 AccessToken에 문제가 있는 상황이므로 접근금지(로그아웃) 시켜야 한다.

사용자 본인의 정보를 얻기 위해 API 호출 정보를 작성하자.

```js
export default class ApiService {
  // ...

  async fetchCurrentUser(): Promise<{
    id: string;
    name: string;
  }> {
    const { data } = await this.instance.get('/users/me');
    const { id, name } = data;
    return { id, name };
  }
}
```

AccessToken이 잘못됐다면 내 정보를 가져오는 데 실패할거고, 실패하면 현재 AccessToken을 초기화 시켜준다.

```js
import { useEffect } from 'react';

import { apiService } from '../services/ApiService';

import useAccessToken from './useAccessToken';

export default function useCheckAccessToken(): void {
  const { accessToken, setAccessToken } = useAccessToken();

  useEffect(() => {
    const fetchCurrentUser = async () => {
      try {
        await apiService.fetchCurrentUser();
      } catch (e) {
        setAccessToken('');
      }
    };

    fetchCurrentUser();
  }, [accessToken, setAccessToken]);
}
```

내 정보를 가져오는 과정은 사이트 어디에 있든 무조건 실행되어야 하므로, 최상위 컴포넌트에다 실행되게 해놓는다.

```js
export default function Layout() {
  useCheckAccessToken();

  return (
    <Container>
      <Header />
      <Outlet />
    </Container>
  );
}
```

## 로그아웃 구현

이전에 Header에 만들어놓은 로그아웃 버튼 핸들러에 로그아웃 로직을 넣어주면 된다.

```js
export default function Header() {
  const { accessToken, setAccessToken } = useAccessToken();

  const navigate = useNavigate();

  const handleClickLogout = async () => {
    await apiService.logout();
    setAccessToken('');
    navigate('/');
  };

  // ...
}
```

```js
export default class ApiService {
  // ...

  async logout(): Promise<void> {
    await this.instance.delete('/session');
  }
}
```

참고로 서버에서 로그아웃 처리가 되면, 원래 사용하던 AccessToken은 무효화되어 다시 사용할 수 없다.

그리고 강의에서는 로그아웃 실패에 대한 예외처리를 하지 않았지만, 로그아웃 처리가 잘못되어 `handleClickLogout`이 제대로 실행되지 않아도 `useCheckAccessToken`에서 매번 유효한 AccessToken인지 확인하고 있기 때문에 딱히 고민하지 않아도 된다.
