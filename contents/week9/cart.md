# 장바구니 페이지

상품 목록 페이지와 흡사하게 만들면 된다.

1. 장바구니 목록 받아오기 : `useFetchCart` 커스텀 훅 만들기
2. 장바구니 목록 보여주기 : `CartView` 컴포넌트로 구현하기
3. 조합하기 : `CartPage`에서 1, 2 합치기

## 구현할 페이지 준비

```js
export default function CartPage() {
  // 1. 장바구니 목록 받아오기
  const { cart } = useFetchCart();

  if (!cart) {
    return null;
  }

  return (
    <div>
      <h2>장바구니</h2>
      {/* 2. 장바구니 목록 보여주기 */}
      <CartView cart={cart} />
    </div>
  );
}
```

장바구니 목록을 보여줄 컴포넌트 이름이 `Cart`가 아닌 이유는, `Cart`라는 이름을 가진 타입이 있기 때문이다. 같은 이름을 사용할 수 없으므로, 이 때 사용할 수 있는 방법은 두가지가 있다.

1. 겹치는 이름을 겹치지 않게 다른 이름으로 변경한다. 강의에서 `Cart` 컴포넌트를 `CartView`로 바꾼 것과 같다.
2. `Cart` 타입을 가져올 때, `as`를 사용하여 다른 이름으로 바꿔 사용한다.

```js
import { Cart as CartType } from '../../type.ts'
```

## 장바구니 목록 받아오기

원래라면 데이터 요청을 보낼 때 사용자의 액세스 토큰을 넘겨줘서 사용자의 장바구니를 가져와야 하지만, 여기서는 모두가 공유하는(?) 장바구니를 가져온다.

```js
// useFetchCart.ts
import { useEffect } from 'react';

import { container } from 'tsyringe';
import { useStore } from 'usestore-ts';

import CartStore from '../stores/CartStore';

export default function useFetchCart() {
  const store = container.resolve(CartStore);

  const [{ cart }] = useStore(store);

  useEffect(() => {
    store.fetchCart();
  }, [store]);

  return { cart };
}
```

```js
// CartStore.ts
import { singleton } from 'tsyringe';
import { Action, Store } from 'usestore-ts';

import { Cart } from '../types';

import { apiService } from '../services/ApiService';

@singleton()
@Store()
export default class CartStore {
  cart: Cart | null = null;

  async fetchCart() {
    this.setCart(null);

    const cart = await apiService.fetchCart();

    this.setCart(cart);
  }

  @Action()
  setCart(cart: Cart | null) {
    this.cart = cart;
  }
}
```

```js
export default class ApiService {
  private instance = axios.create({
    baseURL: API_BASE_URL,
  });

  // ...

  async fetchCart(): Promise<Cart> {
    const { data } = await this.instance.get('/cart');
    return data;
  }
}

export const apiService = new ApiService();
```

## 장바구니 목록 보여주기

```js
import styled from 'styled-components';

import { Cart } from '../../types';

import numberFormat from '../../utils/numberFormat';

import LineItemView from './LineItemView';

const Container = styled.div`
  table {
    width: 100%;
  }

  th, td {
    padding: .5rem;
    text-align: left;
  }
`;

type CartViewProps = {
  cart: Cart;
};

export default function CartView({ cart }: CartViewProps) {
  if (!cart.lineItems.length) {
    return (
      <p>장바구니가 비었습니다</p>
    );
  }

  return (
    <Container>
      <table>
        <thead>
          <tr>
            <th>제품</th>
            <th>단가</th>
            <th>수량</th>
            <th>금액</th>
          </tr>
        </thead>
        <tbody>
          {cart.lineItems.map((lineItem) => (
            <LineItemView
              key={lineItem.id}
              lineItem={lineItem}
            />
          ))}
        </tbody>
        <tfoot>
          <tr>
            <th colSpan={3}>
              합계
            </th>
            <td>
              {numberFormat(cart.totalPrice)}
              원
            </td>
          </tr>
        </tfoot>
      </table>
    </Container>
  );
}
```

```js
import { Link } from 'react-router-dom';

import { OrderLineItem } from '../../types';

import numberFormat from '../../utils/numberFormat';

import Options from './Options';

interface LineItemViewProps {
    lineItem: OrderLineItem;
}

export default function LineItemView({ lineItem } : LineItemViewProps) {
  return (
    <tr>
      <td>
        <Link to={`/products/${lineItem.product.id}`}>
          {lineItem.product.name}
        </Link>
        <Options options={lineItem.options} />
      </td>
      <td>
        {numberFormat(lineItem.unitPrice)}
        원
      </td>
      <td>{lineItem.quantity}</td>
      <td>
        {numberFormat(lineItem.totalPrice)}
        원
      </td>
    </tr>
  );
}
```

```js
import { OrderOption } from '../../types';

interface OptionsProps{
    options: OrderOption[];
}

export default function Options({ options } : OptionsProps) {
  if (!options.length) {
    return null;
  }

  const text = options
    .map((option) => `${option.name}: ${option.item.name}`)
    .join(', ');

  return (
    <div>
      (
      {text}
      )
    </div>
  );
}
```

`table` 태그는 `ul` 안에 `li`가 있어야 하는 것처럼 정해진 구조가 있다. `table` 태그 없이 `tr`을 사용할 수 없고, `table` 내에서 `thead`나 `tbody`가 빠졌을 경우 경고를 준다. `table`을 사용할 땐 구조에 유의하자.

### 참고 - CLI에서 POST 하기

처음 장바구니가 비어있었는데, 아직 장바구니 추가 기능을 만들지 않아 장바구니 목록이 잘 나오는지 확인할 수가 없었다. 그래서 커맨드로 장바구니에 상품을 추가해줬다.

```bash
curl -X POST https://shop-demo-api-01.fly.dev/cart/line-items \
  -H "Content-Type: application/json" \
  --data '
    {
      "productId": "0BV000PRO0001",
      "options": [
        {
          "id": "0BV000OPT0001",
          "itemId": "0BV000ITEM001"
        },
        {
          "id": "0BV000OPT0002",
          "itemId": "0BV000ITEM005"
        }
      ],
      "quantity": 1
    }
  '
```

## Test

### 테스트 항목

- 장바구니의 합계가 잘 나오는지 체크
- 장바구니에 넣은 상품의 옵션, 수량 등
- 빈 장바구니

### Backdoor

백도어란, 컴퓨터 보안 분야에서 사용되는 용어로 시스템에 접근하기 위해 정상적인 인증 절차를 무효화하는 악성 코드의 유형이다. 네트워크나 응용 프로그램 등의 보안 조치를 우회하여 시스템에 접근할 수 있는 방식을 말한다.

이러한 방식을 프론트엔드를 테스트에도 적용할 수 있다. 예를 들어 장바구니의 기능이 제대로 수행되는지 테스트해보려면 장바구니를 초기화하는 등의 과정을 거쳐야한다. 하지만 테스트 환경에서 실제 유저가 장바구니에 접근하여 추가하거나 비우는 등의 동작을 하려면 액세스 토큰이 필요한데, 테스트 환경에서는 불가능하다.

이를 해결하기 위해 실제 유저가 사용하는 것과 비슷한 환경을 만들어 테스트를 수행할 수 있도록 백도어 path를 만들어 사용한다.

단, 보안 등의 문제가 있을 수 있으므로 백도어는 테스트 용도로만 사용되어야 하며, **실제 서비스에는 포함시키지 않아야** 한다.

```js
// tests/backdoor.ts
const { I } = inject();

const BACKDOOR_BASE_URL = 'http://localhost:3000/backdoor';

export = {
    setupDatabase(){
        I.amOnPage(`${BACKDOOR_BASE_URL}/setup-database`);
        I.see('OK');
    }
}
```

### 포함된 텍스트 찾기

화면 내에 텍스트를 찾을 때, 문자열이 정확히 'A'인지보다 문자열에 'A'가 포함되었는지를 파악하는게 편할 때가 많다. 만약 문자열이 변수에 저장되어 있고, 그 변수에 'A'가 포함되어 있는지 확인할 때 다음과 같은 방법을 사용할 수 있다.

- 정규표현식 사용하기 : `screen.getByText(/찾는문자열/)`에서 찾는 문자열 대신 변수명을 사용하려면 다음과 같이 사용해야 한다.

```js
screen.getByText(new RegExp(변수명));
```

- container 사용하기 : container는 컴포넌트 렌더링 결과물을 담고있는 변수다. 이를 검증하기 위해 expect와 함께 사용하며, 이를 사용하여 직접 찾는 문자열이 화면에 있는지 파악할 수 있다.

```js
expect(container).toHaveTextContent(변수명);
```

{% hint style="info" %}
단, container 변수는 컴포넌트의 내부 구현에 매우 의존적이어서 구현이 변경됐을 때 테스트에 실패할 확률이 높다. 또한 컴포넌트의 출력 결과에 직접 접근하기 때문에 내부 상태나 메서드를 테스트하기에 한계가 있다.

요즘에는 컴포넌트의 출력 결과물이 아닌 사용자 상호작용에 기반한 테스트를 선호하며, React Testing Library와 같은 라이브러리가 이에 해당한다.
{% endhint %}
