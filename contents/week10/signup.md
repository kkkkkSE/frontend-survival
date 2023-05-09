# 회원가입 기능 구현

## 회원가입 API 호출 작성

회원가입 직후에 바로 로그인되어 있도록 하기위해 AccessToken을 반환했다. (API 자체가 회원가입 성공 시 토큰을 주도록 설계되어 있음)

```js
export default class ApiService {
  async signup({ email, name, password }: {
    email: string;
    name: string;
    password: string;
  }): Promise<string> {
    const { data } = await this.instance.post('/users', {
      email, name, password,
    });
    const { accessToken } = data;
    return accessToken;
  }
}
```

## 회원가입 기능 및 상태 관리 구현

스토어를 생성하여 회원가입 처리와 회원가입에 필요한 상태 관리, 유효성 검사 등을 할 수 있도록 했다. (구현된걸 보니 로그인 스토어랑 크게 다르지 않게 생겼다.)

비밀번호 확인은 클라이언트 쪽에서 비밀번호를 확인하라는 용도로 사용할 뿐 서버로는 보내지 않는다.

```js
import { singleton } from 'tsyringe';
import { Action, Store } from 'usestore-ts';

import { apiService } from '../services/ApiService';

@singleton()
@Store()
export default class SignupFormStore {
  email = '';

  name = '';

  password = '';

  passwordConfirmation = '';

  error = false;

  accessToken = '';

  get valid() {
    return this.email.includes('@')
      && !!this.name
      && this.password.length >= 4
      && this.password === this.passwordConfirmation;
  }

  @Action()
  changeEmail(email: string) {
    this.email = email;
  }

  @Action()
  changeName(name: string) {
    this.name = name;
  }

  @Action()
  changePassword(password: string) {
    this.password = password;
  }

  @Action()
  changePasswordConfirmation(password: string) {
    this.passwordConfirmation = password;
  }

  @Action()
  setAccessToken(accessToken: string) {
    this.accessToken = accessToken;
  }

  @Action()
  setError() {
    this.error = true;
  }

  @Action()
  reset() {
    this.email = '';
    this.name = '';
    this.password = '';
    this.passwordConfirmation = '';
    this.error = false;
    this.accessToken = '';
  }

  async signup() {
    try {
      const accessToken = await apiService.signup({
        email: this.email,
        name: this.name,
        password: this.password,
      });

      this.setAccessToken(accessToken);
    } catch (e) {
      this.setError();
    }
  }
}
```

스토어 객체를 가져다 쓸 수 있도록 커스텀 훅 만들기.

```js
import { container } from 'tsyringe';

import { useStore } from 'usestore-ts';

import SignupFormStore from '../stores/SignupFormStore';

export default function useSignupFormStore() {
  const store = container.resolve(SignupFormStore);
  return useStore(store);
}
```

## 회원가입 화면 구현

```js
import { useEffect } from 'react';

import styled from 'styled-components';

import useAccessToken from '../../hooks/useAccessToken';
import useSignupFormStore from '../../hooks/useSignupFormStore';

import Button from '../ui/Button';
import TextBox from '../ui/TextBox';

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

export default function SignupForm() {
  const { setAccessToken } = useAccessToken();

  const [{
    email, name, password, passwordConfirmation, valid, error, accessToken,
  }, store] = useSignupFormStore();

  useEffect(() => {
    if (accessToken) {
      setAccessToken(accessToken);
    }
  }, [accessToken]);

  const handleChangeEmail = (value: string) => {
    store.changeEmail(value);
  };

  const handleChangeName = (value: string) => {
    store.changeName(value);
  };

  const handleChangePassword = (value: string) => {
    store.changePassword(value);
  };

  const handleChangePasswordConfirmation = (value: string) => {
    store.changePasswordConfirmation(value);
  };

  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    store.signup();
  };

  return (
    <Container>
      <h2>회원 가입</h2>
      <form onSubmit={handleSubmit}>
        <TextBox
          label="E-mail"
          placeholder="tester@example.com"
          value={email}
          onChange={handleChangeEmail}
        />
        <TextBox
          label="Name"
          value={name}
          onChange={handleChangeName}
        />
        <TextBox
          label="Password"
          type="password"
          value={password}
          onChange={handleChangePassword}
        />
        <TextBox
          label="Password Confirmation"
          type="password"
          value={passwordConfirmation}
          onChange={handleChangePasswordConfirmation}
        />
        <Button type="submit" disabled={!valid}>
          회원 가입
        </Button>
        {error && (
          <p>회원 가입 실패</p>
        )}
      </form>
    </Container>
  );
}
```

## 회원가입 페이지 구현

페이지에 처음 접근하면 모든 텍스트 박스(회원가입 스토어의 상태들)를 모두 초기화시키고, 회원가입이 완료되었을 때도 초기화한다. 다시 회원가입 페이지에 접근했을 때 데이터가 남아있지 않도록 하는 과정이다.

회원가입에 성공했을 때, 웰컴 페이지로 이동시키도록 했다.

```js
import { useEffect } from 'react';
import { useNavigate } from 'react-router-dom';

import useSignupFormStore from '../hooks/useSignupFormStore';

import SignupForm from '../components/signup/SignupForm';

export default function SignupPage() {
  const navigate = useNavigate();

  const [{ accessToken }, store] = useSignupFormStore();

  useEffect(() => {
    store.reset();
  }, []);

  useEffect(() => {
    if (accessToken) {
      store.reset();
      navigate('/signup/complete');
    }
  }, [accessToken]);

  return (
    <SignupForm />
  );
}
```

```js
export default function SignupCompletePage() {
  return (
    <p>
      회원 가입이 완료되었습니다.
    </p>
  );
}
```

마지막으로 로그인 창에 회원가입 페이지로 이동할 버튼을 만들어주면 된다.

```js
export default function LoginForm() {
  // ...

  return (
    <Container>
      <h2>로그인</h2>
      <form onSubmit={handleSubmit}>
        {/* ... */}
      </form>
      <p>
        <Link to="/signup">
          회원 가입
        </Link>
      </p>
    </Container>
  );
}
```

## Test

### 테스트 항목

- 입력한 텍스트가 input의 value로 잘 들어가는지
- 모든 input 초기화 잘 되는지
- 유효성 검사
- 회원가입이 성공적으로 이뤄져서 AccessToken이 반환되는지
- 회원가입 실패 시 액세스 토큰이 제거 되었는지, error 상태가 true 인지
- 이미 존재하는 이메일로 가입
- 비밀번호와 비밀번호 확인 일치하지 않음
