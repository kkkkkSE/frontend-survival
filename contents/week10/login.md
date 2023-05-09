# 로그인 기능 구현

웹사이트에서 사용자는 이메일, 아이디와 같은 사용자의 고유 username과 password를 입력하여 로그인을 한다.

이를 프론트엔드 개발 입장에서 바라본다면, 사용자의 username과 password를 입력받으면 이를 백엔드로 전송하고, 사용자가 입증되면 웹에서의 다양한 요청에 필요한 AccessToken을 얻는 것이 로그인이라 할 수 있겠다.

강의에서는 JWT를 사용하고, 이를 관리하기 위해 로컬 스토리지를 이용했다.

{% hint style="info" %}
TODO : 사용자 인증 방식 종류 정리하기 (쿠키, 세션, 토큰)
{% endhint %}

이번 목표는 다음과 같다.

1. 로그인 페이지 만들기 : `LoginForm` + `LoginFormStore`
2. 로그인 화면 구현 : `LoginForm`
3. 로그인 기능 구현 + 상태 관리 : `LoginFormStore`

로그인 기능이 들어갈거라 변경된 API base URL을 사용했다.

## 구현할 페이지 준비

로그인 페이지에는 usename, password를 입력하고 submit 할 수 있는 `LoginForm` 컴포넌트가 포함되고, 로그인 상태를 관리할 `LoginFormStore` 스토어도 사용한다.

로그인 시 서버에서 AccessToken을 받으면 자동으로 홈(`/`)으로 이동시킨다.

```js
import { useEffect } from 'react';

import { useNavigate } from 'react-router-dom';

import LoginForm from '../components/login/LoginForm';

import useLoginFormStore from '../hooks/useLoginFormStore';

export default function LoginPage() {
  const navigate = useNavigate();

  const [{ accessToken }, store] = useLoginFormStore();

  useEffect(() => {
    store.reset();
  }, []);

  useEffect(() => {
    if (accessToken) {
      store.reset();
      navigate('/');
    }
  }, [accessToken]);

  return (
    <LoginForm />
  );
}
```

### 추가 세팅 - UI 요소

또한, 로그인 화면에서는 필연적으로 `input` 요소가 필요하다. 강의 자료에서 만들어준 `TextBox`를 사용하자.

```js
import React, { useRef } from 'react';

import styled from 'styled-components';

const Container = styled.div`
  margin-block: .5rem;

  label {
    display: inline-block;
    width: 10rem;
    margin-right: .5rem;
    text-align: right;
    vertical-align: middle;
  }

  input {
    width: 20rem;
  }
`;

type TextBoxProps = {
  label: string;
  placeholder?: string;
  type?: 'text' | 'number' | 'password'; // ← 계속해서 지원할 타입을 쭉 써주자.
  value: string;
  onChange: (value: string) => void;
}

export default function TextBox({
  label, placeholder = undefined, type = 'text', value, onChange,
}: TextBoxProps) {
  const id = useRef(`textbox-${Math.random().toString().slice(2)}`);

  const handleChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    onChange(event.target.value);
  };

  return (
    <Container>
      <label htmlFor={id.current}>
        {label}
      </label>
      <input
        id={id.current}
        type={type}
        placeholder={placeholder}
        value={value}
        onChange={handleChange}
      />
    </Container>
  );
}
```

텍스트 박스의 형태와 스타일이 지정되어 있으며, 다양한 Props를 받아 텍스트 박스를 구성하게 된다.

여기서, 필수가 아닌 Props에 대해 기본값을 설정해주지 않았다고 eslint가 경고를 보낸다. 원래라면 Props의 각 타입을 정해준 것처럼 필수가 아닌 Props에 대해 따로 기본 값을 정해줘야 한다.

```js
const HelloWorld = ({ name }) => (
  // ...
);

HelloWorld.propTypes = {
  name: PropTypes.string
};

HelloWorld.defaultProps = {
  name: 'john'
};
```

하지만 실제로는 다음과 같은 방식을 흔하게 사용한다.

```js
const HelloWorld = ({ name = 'john' }: {
    name? : string
}) => (
  // ...
);
```

해당 방식을 사용해도 eslint가 경고를 보내지 않도록 하기 위해 규칙을 추가하도록 하자.

```js
module.exports = {
  // ...
  rules: {
    // ...
    'react/require-default-props': [2, { functions: 'defaultArguments' }],
  }
}
```

그리고 로그인을 위해 type이 submit인 버튼을 사용해야 하는데, 이전에 만들어 뒀던 `Button`에는 type을 `button`으로 고정해뒀다. type을 받아서 사용할 수 있도록 변경해주자. 추가로 비활성 상태에서의 버튼 스타일도 추가해주자.

```js
import styled from 'styled-components';

interface ButtonProps {
  type : string;
}

const Button = styled.button.attrs<ButtonProps>((props) => ({
  type: props.type ?? 'button',
}))`
  border: .1rem solid #888;
  background: transparent;
  color: ${(props) => props.theme.colors.primary};
  cursor: pointer;

  :disabled {
    filter: grayscale(80%);
    cursor: not-allowed;
    color: lightgrey;
    border: .1rem solid lightgrey;
  }
`;

export default Button;

```

## 로그인 화면 구현

로그인 기능의 핵심은 AccessToken 발급과 그에 따른 로그인 상태 와 AccessToken 관리이다.

AccessToken을 클라이언트 쪽에서 어떻게 관리하는지 숨기기 위해 AccessToken 관리 과정을 `useAccessToken` 커스텀 훅으로 뺐다.

그리고 로그인 처리와 상태 관리를 위한 `LoginFormStore` 객체도 커스텀 훅을 통해 불러왔다.

로그인 페이지에 접근하면 스토에에 있는 모든 상태가 초기화된다. 다른 페이지에 방문했다가 다시 돌아오면 텍스트가 그대로 남아있는 것을 방지하기 위함이다. 또한 로그인에 성공해도 초기화를 진행한다. 로그인에 필요했던 Form일 뿐이니 불필요한 정보를 남아있지 않게 한다.

```js
import React, { useEffect } from 'react';

import styled from 'styled-components';

import TextBox from '../ui/TextBox';
import Button from '../ui/Button';

import useAccessToken from '../../hooks/useAccessToken';
import useLoginFormStore from '../../hooks/useLoginFormStore';

const Container = styled.div`
  h2 {
    margin-bottom: 1rem;
    font-size: 4rem;
  }

  button {
    margin-left: 10.5rem;
  }

  p {
    margin-block: 1rem;
  }
`;

export default function LoginForm() {
  const { setAccessToken } = useAccessToken();

  const [{
    email, password, valid, error, accessToken,
  }, store] = useLoginFormStore();

  useEffect(() => {
    if (accessToken) {
      setAccessToken(accessToken);
    }
  }, [accessToken]);

  const handleChangeEmail = (value: string) => {
    store.changeEmail(value);
  };

  const handleChangePassword = (value: string) => {
    store.changePassword(value);
  };

  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    store.login();
  };

  return (
    <Container>
      <h2>로그인</h2>
      <form onSubmit={handleSubmit}>
        <TextBox
          label="E-mail"
          placeholder="tester@example.com"
          value={email}
          onChange={handleChangeEmail}
        />
        <TextBox
          label="Password"
          type="password"
          value={password}
          onChange={handleChangePassword}
        />
        <Button type="submit" disabled={!valid}>
          로그인
        </Button>
        {error && (
          <p>로그인 실패</p>
        )}
      </form>
    </Container>
  );
}
```

로그인에 실패하면 페이지 이동 없이(AccessToken 발급 못함) 로그인 실패 화면이 뜨게 된다.

## 로그인 상태 관리와 로그인 로직 구현

`LoginFormStore`에는 로그인 실패 여부(error)를 체크하거나 유효성 검사를 실시하고, 로그인 로직이 실행되는 등의 로그인 관련 비즈니스 로직이 담겨있다.

강의에서는 어떤 이유로 로그인 할 수 없는지에 대해 알려주는 대신 로그인 버튼이 비활성화 되도록 설정했다. 유효성 검사도 '@' 여부만 확인하고, 비밀번호가 틀린지 정도만 체크한다.

그리고 `accessToken`을 갱신하거나 `error` 상태를 변경하는 부분은 외부에서 접근하지 못하게 private으로 설정했다.

```js
import { singleton } from 'tsyringe';
import { Action, Store } from 'usestore-ts';

import { apiService } from '../services/ApiService';

@singleton()
@Store()
export default class LoginFormStore {
  email = '';

  password = '';

  error = false;

  accessToken = '';

  get valid() {
    return this.email.includes('@') && !!this.password;
  }

  @Action()
  changeEmail(email: string) {
    this.email = email;
  }

  @Action()
  changePassword(password: string) {
    this.password = password;
  }

  @Action()
  private setAccessToken(accessToken: string) {
    this.accessToken = accessToken;
  }

  @Action()
  private setError() {
    this.error = true;
  }

  @Action()
  reset() {
    this.email = '';
    this.password = '';
    this.error = false;
    this.accessToken = '';
  }

  async login() {
    try {
      const accessToken = await apiService.login({
        email: this.email,
        password: this.password,
      });

      this.setAccessToken(accessToken);
    } catch (e) {
      this.setError();
    }
  }
}
```

```js
export default class ApiService {
  // ...

  async login({ email, password }: {
    email: string;
    password: string;
  }): Promise<string> {
    const { data } = await this.instance.post('/session', { email, password });
    const { accessToken } = data;
    return accessToken;
  }
}

export const apiService = new ApiService();
```

로그인에 성공하면 `accessToken`을 갱신하며 로컬 스토리지의 `accessToken`
도 갱신시킨다.

## AccessToken 관리

AccessToken을 관리하기 위해 로컬 스토리지를 편리하게 이용할 수 있게 도와주는 usehooks-ts의 `useLocalStorage`를 사용했다.

```js
// useAccessToken.ts
import { useLocalStorage } from 'usehooks-ts';

export default function useAccessToken() {
  const [accessToken, setAccessToken] = useLocalStorage('accessToken', '');

  return { accessToken, setAccessToken };
}
```

`LoginForm` 컴포넌트의 useEffect로 인해 `LoginFormStore`의 `accessToken`이 바뀔 때마다 로컬 스토리지의 `accessToken`이 새로운 AccessToken으로 갱신된다.

## 로그인 여부에 따른 화면

로그인에 성공하면 기존의 화면과 달리 주문하기나 로그아웃 버튼 등이 Header에 생성된다. AccessToken의 여부에 따라 이를 컨트롤 해보자.

```js
// Header.ts
export default function Header() {
  const { accessToken } = useAccessToken();

  const { categories } = useFetchCategories();

  const handleClickLogout = () => {
    // DELETE /session
    // reset accesstoken
    // navigate ('/')
  };

  return (
    <Container>
      <h1>Shop</h1>
      <nav>
        <ul>
          {/* ... */}
          {accessToken ? (
            <>
              <li>
                <Link to="/cart">Cart</Link>
              </li>
              <li>
                <Button onClick={handleClickLogout}>
                  Logout
                </Button>
              </li>
            </>
          ) : (
            <li>
              <Link to="/login">Login</Link>
            </li>
          )}
        </ul>
      </nav>
    </Container>
  );
}
```

```js
// AddToCartForm.tsx
import { Link } from 'react-router-dom';

import Options from './Options';
import Quantity from './Quantity';
import Price from './Price';
import SubmitButton from './SubmitButton';

import useAccessToken from '../../../hooks/useAccessToken';

export default function AddToCartForm() {
  const { accessToken } = useAccessToken();

  if (!accessToken) {
    return (
      <div>
        주문하려면
        {' '}
        <Link to="/login">로그인</Link>
        하세요
      </div>
    );
  }

  return (
    <div>
      <Options />
      <Quantity />
      <Price />
      <SubmitButton />
    </div>
  );
}
```

상품 상세 페이지 내에서 로그인 화면에 접근했을 때, 다시 해당 페이지로 돌아올 수 있도록 파라미터를 같이 보내줄 수도 있다.

## API 호출 시 사용자 인증하기

로그인 기능이 추가되면서 장바구니와 같은 일부 기능 사용 시 AccessToken을 전달하여 사용자 인증을 해야만 사용할 수 있도록 변경되었다.

HTTP 요청 시 AccessToken을 전달하기 위해 요청의 헤더에서 Authorization 속성 값으로 토큰을 담아 보내야한다.

```js
export default class ApiService {
  private instance = axios.create({
    baseURL: API_BASE_URL,
    headers: { Authorization: `Bearer ${TOKEN}` },
  });

  // ...
}
```

사용자가 로그인 했는지, 하지 않았는지를 AccessToken을 통해 알 수 있고, AccessToken의 상태에 따라 Authorization 추가 여부가 달라진다. 따라서 이런식으로 코드를 짤 수 있다. undefined를 넣으면 Authorization가 전달되지 않는다.

```js
export default class ApiService {
  private instance = axios.create({
    baseURL: API_BASE_URL,
  });

  private accessToken = '';

  setAccessToken(accessToken: string) {
    if (accessToken === this.accessToken) {
      return;
    }

    const authorization = accessToken ? `Bearer ${accessToken}` : undefined;

    this.instance = axios.create({
      baseURL: API_BASE_URL,
      headers: { Authorization: authorization },
    });

    this.accessToken = accessToken;
  }

  // ...
}
```

AccessToken 상태가 바뀔 때마다 `setAccessToken`이 실행되어야 하므로, AccessToken 상태가 바뀔 때마다 실행되고 있던 `useAccessToken` 안에서 `setAccessToken`를 실행시키자.

```js
export default function useAccessToken() {
  const [accessToken, setAccessToken] = useLocalStorage('accessToken', '');

  useEffect(() => {
    apiService.setAccessToken(accessToken);
  }, [accessToken]);

  return { accessToken, setAccessToken };
}
```

이렇게 해주면 로그인 상태일 때 사용자 인증이 필요한 요청인 장바구니 목록 가져오기나 장바구니 담기 등의 기능을 사용할 수 있다.

참고로 개발자 도구의 Network-Headers-Request Headers 에서 요청에 Authorization가 담겨있는지 확인할 수 있다.

## Test

### 테스트 항목

- 로그인 화면에서 Header에 Nav가 잘 나오는지
  - API로 카테고리 목록을 받아와서 Nav를 구성하기 때문에 waitFor로 확인
- 이메일, 비밀번호 입력 후 로그인 해보기
  - 올바른 형태로 입력 후 시도
  - 올바르지 않은 형태 또는 맞지 않는 비밀번호 입력 후 시도
  - 로그인 성공 시 홈으로 이동 (테스트를 위한 폴리필 설치 잊지 않기)
- 로그인 이후에 이용할 수 있는 페이지 + 기능이 존재하는지
  - AccessToken Mocking하여 사용
