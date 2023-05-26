# 주문 관리

원래 주문 관리에 엄청 많은 기능이 들어가야 하는데(주문 취소, 환불, 부분 환불 등..), 강의에서는 주요 기능 몇 가지만 진행한다.

## 주문 목록 불러오기

SWR(`useFetch`)를 이용해서 GET 요청 실시하는 커스텀 훅 생성

```js
import useFetch from './useFetch';

import { OrderSummary } from '../types';

export default function useFetchOrders() {
  interface Data {
    orders: OrderSummary[];
  }

  const {
    data, error, loading,
  } = useFetch<Data>('/orders');

  return {
    orders: data?.orders ?? [],
    error,
    loading,

  };
}
```

## 주문 목록 페이지

```js
import { Link } from 'react-router-dom';

import styled from 'styled-components';

import useFetchOrders from '../hooks/useFetchOrders';

import numberFormat from '../utils/numberFormat';

const Container = styled.div`
    // ...
`;

export default function OrderListPage() {
  const { orders, loading, error } = useFetchOrders();

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
    <Container>
      <h2>Orders</h2>
      <table>
        <thead>
          <tr>
            <th>주문일</th>
            <th>주문자</th>
            <th>상품</th>
            <th>총 가격</th>
            <th>상태</th>
            <th>행동</th>
          </tr>
        </thead>
        <tbody>
          {orders.map((order) => (
            <tr key={order.id}>
              <td>{order.orderedAt}</td>
              <td>{order.orderer.name}</td>
              <td>{order.title}</td>
              <td>
                {numberFormat(order.totalPrice)}
                원
              </td>
              <td>{order.status}</td>
              <td>
                <Link to={`/orders/${order.id}`}>
                  자세히
                </Link>
                <Link to={`/orders/${order.id}/edit`}>
                  상태 변경
                </Link>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </Container>
  );
}
```

주문 상태의 경우에는 영어단어로 되어있어 관리자가 주문 상태를 직관적으로 알기 쉽게 하기 위해 상수로 정의해놓고 사용해도 된다.

```js
const STATUS_MESSAGES: Record<string, string> = {
  paid: '결제 완료',
  ready: '배송 준비',
  shipping: '배송 중',
  complete: '배송 완료',
  canceled: '취소',
};
```

## 주문 상세 페이지

주문 목록에서 자세히 버튼을 누르면 해당 주문에 대해 상세한 내용을 확인할 수 있다. 주문 상세 데이터를 불러오는 커스텀 훅과 주문 상세 페이지를 생성하자.

```js
import useFetch from './useFetch';

import { OrderSummary } from '../types';

export default function useFetchOrders() {
  interface Data {
    orders: OrderSummary[];
  }

  const {
    data, error, loading,
  } = useFetch<Data>('/orders');

  return {
    orders: data?.orders ?? [],
    error,
    loading,

  };
}
```

```js
import { Link, useParams } from 'react-router-dom';

import styled from 'styled-components';

import Options from '../components/order-detail/Options';

import useFetchOrder from '../hooks/useFetchOrder';

import numberFormat from '../utils/numberFormat';

import { STATUS_MESSAGES } from '../constants';
import { LineItem } from '../types';

const Container = styled.div`
  // ...
`;

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

  if (error || !order) {
    return (
      <p>Error!</p>
    );
  }

  return (
    <Container>
      <h2>Order Detail</h2>
      <dl>
        <dt>주문일시</dt>
        <dd>{order.orderedAt}</dd>
        <dt>주문자</dt>
        <dd>{order.orderer.name}</dd>
        <dt>상품</dt>
        <dd>
          <ul>
            {order.lineItems.map((lineItem: LineItem) => (
              <li key={lineItem.id}>
                {lineItem.product.name}
                <Options options={lineItem.options} />
              </li>
            ))}
          </ul>
        </dd>
        <dt>총 가격</dt>
        <dd>
          {numberFormat(order.totalPrice)}
          원
        </dd>
        <dt>배송 정보</dt>
        <dd>
          <p>
            받는 사람:
            {' '}
            {order.receiver.name}
          </p>
          <p>
            연락처:
            {' '}
            {order.receiver.phoneNumber}
          </p>
          <p>
            {order.receiver.address1}
            {' '}
            {order.receiver.address2}
            {' '}
            (우편번호:
            {' '}
            {order.receiver.postalCode}
            )
          </p>
        </dd>
        <dt>결제 정보</dt>
        <dd>
          <p>
            주문번호:
            {' '}
            {order.payment.merchantId}
          </p>
          <p>
            결제고유번호:
            {' '}
            {order.payment.transactionId}
          </p>
        </dd>
        <dt>상태</dt>
        <dd>{STATUS_MESSAGES[order.status]}</dd>
      </dl>
      <Link to={`/orders/${order.id}/edit`}>
        상태 변경
      </Link>
    </Container>
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

## 주문 상태 변경

강의에선 주문과 관련된 기능 중 목록 조회와 주문 상태만 변경할 수 있게 했다.

{% hint style="info" %}
실제 서비스에선 CS를 위해 배송 시작 버튼을 눌러 송장번호를 입력하거나, 배송지 오기입 시 수정하는 기능, 로그 기록 등 주문을 자세히 다룰 수 있는 기능이 필요하다. 추가적인 기능은 비즈니스를 어떻게 제공하고, CS를 어디까지 제공하는지에 따라 달라진다.
{% endhint %}

주문 상태 변경을 위해 `useFetchOrder`에서 주문 변경 처리를 해주는 함수를 만들어 내보내주자.

```js
import useFetch from './useFetch';

import { OrderDetail } from '../types';

import { apiService } from '../services/ApiService';

export default function useFetchOrder({ orderId }: {
  orderId: string;
}) {
  const url = `/orders/${orderId}`;

  const {
    data, error, loading, mutate,
  } = useFetch<OrderDetail>(url);

  return {
    order: data,
    error,
    loading,
    async updateOrder({ status } : {
      status: string;
    }) {
      await apiService.updateOrder({ orderId, status });
      mutate();
    },
  };
}
```

주문 상태 변경 페이지 구현

```js
/* eslint-disable react/jsx-props-no-spreading */
import { useNavigate, useParams } from 'react-router-dom';

import { useForm } from 'react-hook-form';

import styled from 'styled-components';

import Button from '../components/ui/Button';

import useFetchOrder from '../hooks/useFetchOrder';

import numberFormat from '../utils/numberFormat';

import { STATUS_MESSAGES } from '../constants';

const Container = styled.div`
  // ...
`;

export default function OrderEditPage() {
  const params = useParams();

  const orderId = String(params.id);

  const navigate = useNavigate();

  const {
    order, loading, error, updateOrder,
  } = useFetchOrder({
    orderId,
  });

  type FormValues = {
    status: string;
  };

  const { register, handleSubmit } = useForm<FormValues>();

  const onSubmit = async (data: FormValues) => {
    await updateOrder({
      status: data.status,
    });
    navigate(`/orders/${orderId}`);
  };

  if (loading) {
    return (
      <p>Loading...</p>
    );
  }

  if (error || !order) {
    return (
      <p>Error!</p>
    );
  }

  return (
    <Container>
      <h2>Order Status Transition</h2>
      <dl>
        <dt>주문일시</dt>
        <dd>{order.orderedAt}</dd>
        <dt>주문자</dt>
        <dd>{order.orderer.name}</dd>
        <dt>상품</dt>
        <dd>{order.title}</dd>
        <dt>총 가격</dt>
        <dd>
          {numberFormat(order.totalPrice)}
          원
        </dd>
      </dl>
      <form onSubmit={handleSubmit(onSubmit)}>
        <div>
          <label htmlFor="input-status">상태</label>
          <select
            {...register('status')}
            id="input-status"
            defaultValue={order.status}
          >
            {Object.keys(STATUS_MESSAGES).map((status) => (
              <option key={status} value={status}>
                {STATUS_MESSAGES[status]}
              </option>
            ))}
          </select>
        </div>
        <Button type="submit">
          변경
        </Button>
      </form>
    </Container>
  );
}
```
