# 배송 정보 입력

## 주문 페이지 구현

주문자의 정보와 결제를 위한 주문 페이지를 먼저 만든다. 장바구니의 형태와 비슷하게 주문 상품 목록을 보여주고, 아래에서 배송 정보를 입력할 수 있게한다. routes에 넣어 놓는 것도 잊지말자.

```js
import OrderForm from '../components/new-order/OrderForm';

import useFetchCart from '../hooks/useFetchCart';

export default function OrderPage() {
  const { cart } = useFetchCart();

  if (!cart) {
    return null;
  }

  return (
    <OrderForm cart={cart} />
  );
}
```

```js
import { Cart } from '../../types';

import Table from '../line-item/Table';

type OrderFormProps = {
  cart: Cart;
};

export default function OrderForm({ cart }: OrderFormProps) {
  if (!cart.lineItems.length) {
    return (
      <p>장바구니가 비었습니다</p>
    );
  }

  return (
    <div>
      <h2>주문하기</h2>
      <Table
        lineItems={cart.lineItems}
        totalPrice={cart.totalPrice}
      />
      {/* TODO : 배송지 입력 */}
      {/* TODO : 결제하기 */}
    </div>
  );
}
```

만드는 김에 주문 성공 페이지도 만들자.

```js
import { Link } from 'react-router-dom';

export default function OrderCompletePage() {
  return (
    <div>
      <p>
        회원 가입이 완료되었습니다.
      </p>
      <p>
        <Link to="/orders">주문 목록 확인</Link>
      </p>
    </div>
  );
}
```

주문 페이지로 이동할 수 있게 장바구니 목록에서 버튼을 추가해주자.

```js
import { useNavigate } from 'react-router-dom';
import { Cart } from '../../types';

import Table from '../line-item/Table';

import Button from '../ui/Button';

type CartViewProps = {
  cart: Cart;
};

export default function CartView({ cart }: CartViewProps) {
  const navigate = useNavigate();

  const handleClick = () => {
    navigate('/order');
  };

  if (!cart.lineItems.length) {
    return (
      <p>장바구니가 비었습니다</p>
    );
  }

  return (
    <div>
      <Table
        lineItems={cart.lineItems}
        totalPrice={cart.totalPrice}
      />
      <Button onClick={handleClick}>
        주문하기
      </Button>
    </div>
  );
}
```

## 배송 정보 입력에 필요한 스토어 구현

배송에 필요한 주문자의 이름, 배송지의 주소, 연락처를 저장하고 유효성 검사를 실시하는 스토어를 구현하자. 스토어 객체를 만들어주는 커스텀 훅도 같이 만든다.

참고로 배송지는 건물번호까지의 주소와 상세주소를 구분하여 입력하며, 우편번호도 따로 저장한다.

```js
import { singleton } from 'tsyringe';
import { Action, Store } from 'usestore-ts';

@singleton()
@Store()
export default class OrderFormStore {
  name = '';

  address1 = '';

  address2 = '';

  postalCode = '';

  phoneNumber = '';

  get valid() {
    return !!this.name.trim()
      && !!this.address1.trim()
      && !!this.address2.trim()
      && !!this.postalCode.trim()
      && !!this.phoneNumber.trim();
  }

  @Action()
  changeName(name: string) {
    this.name = name;
  }

  @Action()
  changeAddress1(address1: string, postalCode: string) {
    this.address1 = address1;
    this.postalCode = postalCode;
  }

  @Action()
  changeAddress2(address2: string) {
    this.address2 = address2;
  }

  @Action()
  changePhoneNumber(phoneNumber: string) {
    this.phoneNumber = phoneNumber.replace(/[^0-9]/g, '');
  }
}
```

```js
import { container } from 'tsyringe';

import { useStore } from 'usestore-ts';

import OrderFormStore from '../stores/OrderFormStore';

export default function useOrderFormStore() {
  const store = container.resolve(OrderFormStore);
  return useStore(store);
}
```

## 배송 정보 입력 화면 구현

배송 정보를 입력하기 위해 필요한 코드가 꽤 길어서, `OrderForm`에 바로 만들어주지 않고 `ShippingForm` 컴포넌트로 분리해줬다.

주소 입력은 우편번호 검색 기능을 통해 자동으로 채워지게 하고, 상세 주소만 직접 입력받는다.

이전에 만들어 놓은 UI 요소 `TextBox`를 통해 input을 구성하고, 추가적으로 더 필요한 속성들을 `TextBox`에서 추가해주면 된다.

```js
import styled from 'styled-components';

import TextBox from '../ui/TextBox';
import Button from '../ui/Button';

import useOrderFormStore from '../../hooks/useOrderFormStore';

const Container = styled.div`
  h3 {
    font-size: 2rem;
  }

  input {
    width: 50rem;
  }
`;

const PostalCodeField = styled.div`
  > div {
    display: inline-block;
    margin-right: 1rem;
  }

  input {
    width: 10rem;
  }
`;

export default function ShippingForm() {
  const [{
    name, address1, address2, postalCode, phoneNumber,
  }, store] = useOrderFormStore();

  const handleClickSearchPostalCode = () => {
    // TODO: 우편번호 검색
  };

  const handleChangeName = (value: string) => {
    store.changeName(value);
  };

  // TODO: 우편번호 검색하면 자동으로 채워짐
  const handleChangeAddress1 = (value: {
    address: string;
    postalCode: string;
  }) => {
    store.changeAddress1(value.address, value.postalCode);
  };

  const handleChangeAddress2 = (value: string) => {
    store.changeAddress2(value);
  };

  const handlePhonNumber = (value: string) => {
    store.changePhoneNumber(value);
  };

  return (
    <Container>
      <h3>받는 사람</h3>
      <TextBox
        label="이름"
        placeholder="받는 분 이름"
        value={name}
        onChange={handleChangeName}
      />
      <PostalCodeField>
        <TextBox
          label="우편번호"
          value={postalCode}
          readOnly
        />
        <Button onClick={handleClickSearchPostalCode}>
          우편번호 검색
        </Button>
      </PostalCodeField>
      <TextBox
        label="주소"
        value={address1}
        readOnly
      />
      <TextBox
        label="상세 주소"
        value={address2}
        onChange={handleChangeAddress2}
      />
      <TextBox
        label="전화번호"
        type="tel"
        value={phoneNumber}
        onChange={handlePhonNumber}
      />
    </Container>
  );
}
```

```js
// ... 

type TextBoxProps = {
  label: string;
  placeholder?: string;
  type?: 'text' | 'number' | 'password' | 'tel';
  value: string;
  onChange?: (value: string) => void;
  readOnly?: boolean;
}

export default function TextBox({
  label,
  placeholder = undefined,
  type = 'text',
  value,
  onChange = undefined,
  readOnly = false,
}: TextBoxProps) {
  const id = useRef(`textbox-${Math.random().toString().slice(2)}`);

  const handleChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    if (!onChange) return;
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
        readOnly={readOnly}
      />
    </Container>
  );
}
```

{% hint style="info" %}
input의 type을 tel로 설정하면 모바일 환경에서는 숫자 키패드가 나온다.
{% endhint %}

## 우편번호 검색 기능

직접 주소를 입력받지 않고 우편번호 검색 기능을 사용하는 이유로 두가지가 있다.

1. 배송 시 우편번호가 필요하지만, 평소에 쓰일 일이 없어 외우고있는 사람이 많지 않다. (설상가상 예전에 6자리에서 5자리로 완전히 바뀜)
2. 우편번호를 검색하면 표준화된 주소를 제공해주기 때문이다.

우편번호 검색 서비스로는 [Daum 우편번호 서비스](https://postcode.map.daum.net/guide)를 활용할 수 있다.

```js
<script src="//t1.daumcdn.net/mapjsapi/bundle/postcode/prod/postcode.v2.js"></script>
<script>
    new daum.Postcode({
        oncomplete: function(data) {
            // 팝업에서 검색결과 항목을 클릭했을때 실행할 코드를 작성하는 부분입니다.
            // 예제를 참고하여 다양한 활용법을 확인해 보세요.
        }
    }).open();
</script>
```

스크립트를 통해 포스트코드를 불러오고, `new daum.Postcode`로 해당 서비스의 사용이 가능하다.

우선 해당 서비스의 스크립트를 `index.html`에 불러온다. 순서대로 스크립트가 실행되기 때문에 반드시 먼저 불러와줘야 한다.

```js
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>Shop</title>
</head>
<body>
    <div id="root"></div>
    <script src="//t1.daumcdn.net/mapjsapi/bundle/postcode/prod/postcode.v2.js"></script>
    <script type="module" src="./src/main.tsx"></script>
</body>
</html>
```

그냥 `new daum.Postcode({ ... })`를 이용하려고 하면 타입스크립트가 `daum`이 뭔지 인식하지 못 한다. 타입을 잡아주자.

```js
// new-order/daum.postcode.d.ts
declare namespace daum {
    export type PostcodeResult = {
      address: string;
      zonecode: string;
    }

    export class Postcode {
      constructor({ oncomplete, width, height }: {
        oncomplete: (data: PostcodeResult) => void;
        width: string;
        height: string;
      });

      embed(element: HTMLElement | null);
    }
  }
```

추가적으로 사용자가 선택한 주소에 대해 어떤 데이터를 받아올건지에 대한 속성(건물명 등)이나 Postcode와 함께 사용할 수 있는 함수를 추가 및 변경하고 싶다면 공식 문서를 참고하자.

강의에서는 주소 검색창을 새 창에 띄우지 않고 모달로 보여주기 위해 embed 함수의 타입을 정의해뒀다. 모달로 사용할 컴포넌트를 새로 생성하자.

```js
import { useRef } from 'react';

import { useEffectOnce } from 'usehooks-ts';

import styled from 'styled-components';

const Container = styled.div`
  // ...
`;

type AddressSearchProps = {
  close: ()=> void;
  changeAddress: ({ address, zonecode }: {
    address: string;
    zonecode: string;
  }) => void;
}

export default function AddressSearch({
  close, changeAddress,
}: AddressSearchProps) {
  const refElement = useRef<HTMLDivElement>(null);

  useEffectOnce(() => {
    new daum.Postcode({
      oncomplete(data) {
        const { address, zonecode } = data;
        changeAddress({ address, zonecode });
        close();
      },
      width: '100%',
      height: '100%',
    }).embed(refElement.current);
  });

  return (
    <Container id="address-search-container" onClick={close}>
      <div ref={refElement} />
    </Container>
  );
}
```

- `useEffectOnce`를 사용하여 우편번호 검색 서비스를 딱 한번만 실행되게 했고, `new daum.Postcode`로 우편검색 서비스를 생성한다.
- `Container`의 스타일로 인해 화면 중간에 모달창으로 우편번호 검색창이 나타난다.
- `embed`의 인자로 우편번호 검색창을 띄울 HTML element를 넘겨주게 돼있는데, 컴포넌트의 렌더링 상태와 관련없이 일관성 있는 element를 전달해주기 위해 `useRef`를 사용했다.
- `oncomplete`는 주소 선택 후 실행할 함수를 의미하며, 사용자가 입력한 주소의 정보를 `data`를 통해 전달 받으므로, 해당 값을 가지고 `ShippingForm`에 있던 주소1, 우편번호 textbox를 변경하도록 함수를 실행시켰다.
- `oncomplete`에서 `close` 함수를 실행시켜 주소를 선택하면 자동으로 검색창이 닫힌다.
- `useBoolean`을 사용하여 value값의 상태에 따라 `AddressSearch` 모달창을 제어했다. `setFalse` 함수를 넘겨받아 `Container`의 onClick 함수로 달아놔서 바깥 영역을 클릭하면 자동으로 우편번호 검색창이 꺼진다.

```js
export default function ShippingForm() {
  const [{
    name, address1, address2, zonecode, phoneNumber,
  }, store] = useOrderFormStore();

  // ...

  const {
    value: searching, setTrue: openSearch, setFalse: closeSearch,
  } = useBoolean();

  const handleClickSearchPostalCode = () => {
    openSearch();
  };

  const handleChangeAddress1 = (value: {
    address: string;
    zonecode: string;
  }) => {
    store.changeAddress1(value.address, value.zonecode);
  };

  // ...

  return (
    <Container>
      {/* ... */}

      <PostalCodeField>
        <TextBox
          label="우편번호"
          value={zonecode}
          readOnly
        />
        <Button onClick={handleClickSearchPostalCode}>
          우편번호 검색
        </Button>
      </PostalCodeField>

      {/* ... */}

      {searching && (
        <AddressSearch
          close={closeSearch}
          changeAddress={handleChangeAddress1}
        />
      )}
    </Container>
  );
}
```

## Test

### I.wait

구현하면서 스스로 기능에 대한 테스트를 할 때, 로그인, 배송지 입력 등 번거롭게 작업을 해야하는 경우가 있다. 이 때 E2E 테스트에서 텍스트 박스를 채워주는 등의 번거로운 작업을 해놓고 `I.wait`로 여유있게 대기시간을 설정해놓으면 번거로운 작업 없이 기능을 테스트 해볼 수 있다.
