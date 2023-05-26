# 개발 전 준비사항

관리자 웹은 사용자가 이용하는 웹과 분리해서 새로운 페이지로 만든다. 강의에서는 사용자 웹을 포트 8080, 관리자 웹을 8000번으로 띄워서 동시에 접속했다.

관리자 웹은 상품의 등록, 수정을 제외하면 단순 CRUD로 구성된다. 이전에 만들었던 코드를 재활용해서 구현할 예정이다. 이를 위해 SWR과 React Hook Form을 사용한다.

## SWR

기존에 우리가 웹을 구현할 땐, 프론트엔드에서 상태 관리를 적극적으로 했다. 이에 반해 상태 관리는 백엔드에 맡기고, 프론트엔드에선 캐시와 백엔드와의 동기화에 집중하며 웹을 구현할 수도 있다.

지금까지는 프론트엔드에서 상태 관리를 하는 방향으로 구현해봤으니, 이번에는 백엔드에 의존하여 동기화와 캐싱에만 집중할 수 있도록 SWR를 사용해볼 예정이다. 단순 CRUD 애플리케이션을 구축할거라 SWR을 선택했지만, 좀 더 복잡하게 구현하려면 react-query(TanStack-Query로 이름 변경됨)를 사용할 수도 있다.

공식문서에서 설명하는 `useSWR`의 형태는 다음과 같다.

```js
import useSWR from 'swr'
 
function Profile() {
  const { data, error, isLoading } = useSWR('/api/user', fetcher)
 
  if (error) return <div>failed to load</div>
  if (isLoading) return <div>loading...</div>
  return <div>hello {data.name}!</div>
}
```

fetcher 함수는 **데이터를 반환하는 비동기 함수**를 의미하며, 어떠한 것이라도 상관없다. Fetch API, Axios 모두 사용할 수 있다.

SWR을 사용하면 주기적으로 데이터를 확인해주기 때문에 누군가 어떤 데이터를 변경하게 되면 완전 실시간은 아니더라도 자동으로 반영해주어 편리하다. 이는 useState처럼 화면에도 반영된다.

## React Hook Form

지금까지 우리는 사용자의 입력에 따라 실시간으로 리렌더링하여 화면에 반영하는 등의 Controlled Component(제어 컴포넌트)를 사용해왔지만, 단순 CRUD 애플리케이션을 구축할 때는 이런 것들이 거의 필요하지 않게 된다.

이러한 Uncontrolled Component를 사용하거나, 그에 준하는 편의성을 제공하는 도구를 활용할 수 있고, 이 모든 것들을 React Hook Form이 지원한다. Uncontrolled에 집중하면, 리렌더링이 줄어들어 커다란 Form의 성능 문제에서 유리하다.

```js
import { useForm, SubmitHandler } from "react-hook-form";

type Inputs = {
  example: string,
  exampleRequired: string,
};

export default function App() {
  const { register, handleSubmit, formState: { errors } } = useForm<Inputs>();
  const onSubmit: SubmitHandler<Inputs> = data => console.log(data);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input defaultValue="test" {...register("example")} />
      <input {...register("exampleRequired", { required: true })} />
      {errors.exampleRequired && <span>This field is required</span>}
      <input type="submit" />
    </form>
  );
}
```

위 코드는 React Hook Form의 기본 예제이다.

- register : submit할 데이터를 등록
- handleSubmit : submit 했을 때 실행하는 함수. 유효성 검사 실시
- errors.inputName : 유효성 검사에서 통과하지 못하면 error 반환

## Type

관리자 웹 REST API를 참고해서 Type을 새롭게 작성해야 한다.

```js
export type User = {
  id: string;
  name: string;
  email: string;
  role: string;
}

export type Category = {
  id: string;
  name: string;
  hidden?: boolean;
}

export type Image = {
  url: string;
}

export type ProductSummary = {
  id: string;
  category: Category;
  thumbnail: Image;
  name: string;
  price: number;
  hidden: boolean;
}

export type ProductOptionItem = {
  id: string;
  name: string;
  deleted: boolean;
};

export type ProductOption = {
  id: string;
  name: string;
  items: ProductOptionItem[];
  deleted: boolean;
};

export type ProductDetail = {
  id: string;
  category: Category;
  images: Image[];
  name: string;
  price: number;
  options: ProductOption[];
  description: string;
  hidden: boolean;
}

export type OrderOptionItem = {
  name: string;
};

export type OrderOption = {
  name: string;
  item: OrderOptionItem;
};

export type LineItem = {
  id: string;
  product: {
    id: string;
    name: string;
  };
  options: OrderOption[];
  unitPrice: number;
  quantity: number;
  totalPrice: number;
}

export type OrderSummary = {
  id: string;
  orderer: User;
  title: string;
  totalPrice: number;
  status: string;
  orderedAt: string;
}

type Receiver = {
  name: string;
  address1: string;
  address2: string;
  postalCode: string;
  phoneNumber: string;
}

type Payment = {
  merchantId: string;
  transactionId: string;
}

export type OrderDetail = {
  id: string;
  orderer: User;
  title: string;
  lineItems: LineItem[];
  totalPrice: number;
  receiver: Receiver;
  payment: Payment;
  status: string;
  orderedAt: string;
}
```

## ApiService

CRUD 중 Read에 관련된 것은 대부분 SWR를 사용하기 때문에, ApiService에는 대부분 Read를 제외한 나머지(Mutation)에 대한 코드만 들어있다.
