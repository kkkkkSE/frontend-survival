# 주문 확인하기

## 주문 목록 보기

사실 상품 목록 가져오기랑 거의 같아서 특별히 추가적으로 설명할 건 없음.

API 호출 코드 작성

```js
export default class ApiService {
  // ...

  async fetchOrders(): Promise<OrderSummary[]> {
    const { data } = await this.instance.get('/orders');
    const { orders } = data;
    return orders;
  }
}
```

관련 상태 관리 및 비즈니스 로직을 담당할 스토어 작성.

```js
import { singleton } from 'tsyringe';
import { Action, Store } from 'usestore-ts';

import { apiService } from '../services/ApiService';

import { OrderSummary } from '../types';

@singleton()
@Store()
export default class OrderListStore {
  orders: OrderSummary[] = [];

  async fetchOrders() {
    this.setOrders([]);

    const orders = await apiService.fetchOrders();

    this.setOrders(orders);
  }

  @Action()
  setOrders(orders: OrderSummary[]) {
    this.orders = orders;
  }
}
```

주문 목록 얻기 API 호출 + 스토어 객체를 편하게 사용할 수 있게 커스텀 훅 작성

```js
import { useEffect } from 'react';

import { container } from 'tsyringe';

import { useStore } from 'usestore-ts';

import OrderListStore from '../stores/OrderListStore';

export default function useFetchOrders() {
  const store = container.resolve(OrderListStore);

  const [{ orders }] = useStore(store);

  useEffect(() => {
    store.fetchOrders();
  }, [store]);

  return { orders };
}
```

주문 목록을 보여줄 주문 목록 페이지 작성. routes에 주문 목록 페이지 추가하고, 헤더의 Nav에도 추가해주는 걸 잊지 말자.

```js
import { Link } from 'react-router-dom';

import styled from 'styled-components';

import { OrderSummary } from '../../types';

import Order from './Order';

const Container = styled.div`
  li {
    margin-block: .5rem;
    padding: 1rem;
    background: #EEE;
  }

  a {
    display: block;
    text-decoration: none;
  }
`;

type OrdersProps = {
  orders: OrderSummary[];
}

export default function Orders({ orders }: OrdersProps) {
  if (!orders.length) {
    return null;
  }

  return (
    <Container>
      <ul>
        {orders.map((order) => (
          <li key={order.id}>
            <Link to={`/orders/${order.id}`}>
              <Order order={order} />
            </Link>
          </li>
        ))}
      </ul>
    </Container>
  );
}
```

주문 목록 데이터로 화면을 구성할 `Orders` 컴포넌트 구현

```js
import { Link } from 'react-router-dom';

import styled from 'styled-components';

import { OrderSummary } from '../../types';

import Order from './Order';

const Container = styled.div`
  li {
    margin-block: .5rem;
    padding: 1rem;
    background: #EEE;
  }

  a {
    display: block;
    text-decoration: none;
  }
`;

type OrdersProps = {
  orders: OrderSummary[];
}

export default function Orders({ orders }: OrdersProps) {
  if (!orders.length) {
    return null;
  }

  return (
    <Container>
      <ul>
        {orders.map((order) => (
          <li key={order.id}>
            <Link to={`/orders/${order.id}`}>
              <Order order={order} />
            </Link>
          </li>
        ))}
      </ul>
    </Container>
  );
}
```

```js
import styled from 'styled-components';

import { OrderSummary } from '../../types';

import numberFormat from '../../utils/numberFormat';

const Container = styled.div`
  line-height: 1.5;
`;

type OrderProps = {
  order: OrderSummary;
}

export default function Order({ order }: OrderProps) {
  return (
    <Container>
      <div>
        주문 일시:
        {' '}
        {order.orderedAt}
      </div>
      <div>
        주문 코드:
        {' '}
        {order.id}
      </div>
      <div>
        상품:
        {' '}
        {order.title}
      </div>
      <div>
        결제 금액:
        {' '}
        {numberFormat(order.totalPrice)}
        원
      </div>
    </Container>
  );
}
```

주문 목록을 클릭하면 주문 id를 이용해 주문 상세 페이지로 이동한다.

## 주문 상세 내역 보기

API 호출 코드 작성

```js
async fetchOrder({ orderId }: {
  orderId: string;
}): Promise<OrderDetail> {
  const { data } = await this.instance.get(`/orders/${orderId}`);
  return data;
}
```

관련 상태 관리 및 비즈니스 로직을 담당할 스토어 작성. 상품 상세 스토어랑 거의 같음

```js
import { singleton } from 'tsyringe';
import { Action, Store } from 'usestore-ts';

import { apiService } from '../services/ApiService';

import { nullOrderDetail, OrderDetail } from '../types';

@singleton()
@Store()
export default class OrderDetailStore {
  order: OrderDetail = nullOrderDetail;

  loading = true;

  error = false;

  async fetchOrder({ orderId }: {
    orderId: string;
  }) {
    this.startLoading();

    try {
      const order = await apiService.fetchOrder({ orderId });
      this.setOrder(order);
    } catch {
      this.setError();
    }
  }

  @Action()
  private startLoading() {
    this.order = nullOrderDetail;
    this.loading = true;
    this.error = false;
  }

  @Action()
  private setOrder(order: OrderDetail) {
    this.order = order;
    this.loading = false;
    this.error = false;
  }

  @Action()
  private setError() {
    this.order = nullOrderDetail;
    this.loading = false;
    this.error = true;
  }
}
```

상품 상세 정보 때처럼 Null Object 생성해주기

```js
export const nullOrderDetail: OrderDetail = {
  id: '',
  title: '',
  status: '',
  lineItems: [],
  totalPrice: 0,
  orderedAt: '',
};
```

스토어 객체 사용할 수 있게 커스텀 훅 생성

```js
import { useEffect } from 'react';

import { container } from 'tsyringe';

import { useStore } from 'usestore-ts';

import OrderDetailStore from '../stores/OrderDetailStore';

export default function useFetchOrder({ orderId }: {
    orderId: string;
  }) {
  const store = container.resolve(OrderDetailStore);

  const [{ order, loading, error }] = useStore(store);

  useEffect(() => {
    store.fetchOrder({ orderId });
  }, [store]);

  return { order, loading, error };
}
```

상품 상세 페이지 구현. `useParams`로 주문 Id 추출.

```js
import { useParams } from 'react-router-dom';

import Order from '../components/order/Order';

import useFetchOrder from '../hooks/useFetchOrder';

export default function OrderDetailPage() {
  const params = useParams();

  const { order, loading, error } = useFetchOrder({
    orderId: String(params.id),
  });

  if (loading) {
    return (
      <p>Loading...</p>
    );
  }

  if (error) {
    return (
      <p>Error!</p>
    );
  }

  return (
    <Order order={order} />
  );
}
```

주문 상세 내역을 보여줄 `Order` 컴포넌트도 구현. 이전의 주문 목록에서 사용한 `Order`랑 다른 컴포넌트임.

```js
import styled from 'styled-components';

import { OrderDetail } from '../../types';

import Table from './Table';

const Container = styled.div`
  dl {
    display: flex;
    flex-wrap: wrap;
    line-height: 1.7;

    dt {
      width: 8rem;
    }

    dd {
      width: calc(100% - 8rem);
    }
  }
`;

type OrderProps = {
  order: OrderDetail;
};

export default function Order({ order }: OrderProps) {
  if (!order.lineItems.length) {
    return null;
  }

  return (
    <Container>
      <dl>
        <dt>주문 일시</dt>
        <dd>{order.orderedAt}</dd>
        <dt>주문 코드</dt>
        <dd>{order.id}</dd>
      </dl>
      {/* 장바구니의 line item */}
    </Container>
  );
}
```

주문 상세 내역에서 보여줄 주문한 상품 리스트는 장바구니 목록에서 상품 목록을 나열했던 `LineItemView` 컴포넌트를 재사용했다.

기존에 `LineItemView`의 위치는 `components/cart/` 내부에 있었지만, 주문 상세 내역에서 재사용하게 되었으니 폴더 구조를 재배치하는 것이 좋다.

강의에서는 `components`폴더에 `line-item`이라는 폴더를 만들고, `LineItemView`와 관련된 컴포넌트를 다 이동시켰다. 그리고 `CartView`에서 사용했던 테이블 구조도 재사용할 수 있도록 컴포넌트로 빼서 해당 폴더에 넣어줬다.

```js
// components/line-item/Table.tsx
import styled from 'styled-components';

import { OrderLineItem } from '../../types';

import numberFormat from '../../utils/numberFormat';

import LineItemView from './LineItemView';

const Container = styled.div`
  table {
    margin-block: 1rem;
    width: 100%;
  }

  th, td {
    padding: .5rem;
    text-align: left;
  }
`;

type TableProps = {
  lineItems: OrderLineItem[];
  totalPrice: number;
};

export default function Table({ lineItems, totalPrice }: TableProps) {
  if (!lineItems.length) {
    return null;
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
          {lineItems.map((lineItem) => (
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
              {numberFormat(totalPrice)}
              원
            </td>
          </tr>
        </tfoot>
      </table>
    </Container>
  );
}
```

`CartView`와 `Order` 컴포넌트에 `Table`을 적용하자.

```js
// components/cart/CartView.tsx
import { Cart } from '../../types';

import Table from '../line-item/Table';

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
    <Table
      lineItems={cart.lineItems}
      totalPrice={cart.totalPrice}
    />
  );
}

```

```js
// components/order-detail/Order.tsx
import styled from 'styled-components';

import { OrderDetail } from '../../types';

import Table from '../line-item/Table';

const STATUS_MESSAGES: Record<string, string> = {
  paid: '결제 완료',
};

const Container = styled.div`
  dl {
    display: flex;
    flex-wrap: wrap;
    line-height: 1.7;

    dt {
      width: 8rem;
    }

    dd {
      width: calc(100% - 8rem);
    }
  }
`;

type OrderProps = {
  order: OrderDetail;
};

export default function Order({ order }: OrderProps) {
  if (!order.lineItems.length) {
    return null;
  }

  return (
    <Container>
      <dl>
        <dt>주문 일시</dt>
        <dd>{order.orderedAt}</dd>
        <dt>주문 코드</dt>
        <dd>{order.id}</dd>
        <dt>처리 상태</dt>
        <dd>{STATUS_MESSAGES[order.status]}</dd>
      </dl>
      <Table
        lineItems={order.lineItems}
        totalPrice={order.totalPrice}
      />
    </Container>
  );
}
```

주문 상태 코드는 서버에서 올 때 영어로 들어오게 돼있어서 사용자에게 더 자세한 정보를 제공하기 위해 객체를 만들어 주문 상태에 따른 메세지를 전달하도록 했다.

## Test

### 테스트 항목

- 주문 목록 페이지에 있어야 할 컨텐츠(텍스트, 메뉴 등)가 잘 나오는지
- 주문 상세 페이지로 이동하면 클릭한 주문 목록의 id에 대한 상세 내용이 제대로 나오는지

codeceptjs로 E2E 테스트 시 `steps_file.ts`에서 로그인 하는 로직을 넣어놓으면 자동으로 로그인 후 테스트를 진행한다.
