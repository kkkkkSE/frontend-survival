# 개발 전 준비사항

## 용어 정의

어떤 것을 만드는지에 대한 용어를 정의한다. 소통에서 사용하는 용어를 정확하게 정의해두지 않으면 혼선을 줄 수 있다.

ex) 상품에 대한 용어를 굿즈, 아이템, 프로덕트 등등 여러 용어를 자유롭게 사용하는 것보다 '상품 = Product' 이런식으로 확실히 정해놓는 것이 좋다.

- Product: 상품
  - Summary: 상품에 대한 요약 정보
  - Detail: 상품에 대한 상세 정보
  - Image: 상품 이미지
  - Option: 상품에 대한 상세 옵션 종류 (색상, 크기 등)
    - OptionItem: 옵션에 대한 상세 옵션 값 (옵션이 색상이라면 이건 Blue, Red 같은 걸 의미함)
    - 접두어는 Product
- Category: 상품에 대한 분류
- Cart: 장바구니
  - LineItem: 장바구니에 담긴 것 (상품, 옵션, 수량 등을 동시에 다룸)
    - 여기서도 Option과 OptionItem을 사용한다.
      - 용어는 동일하지만 Product와 다른 구성을 갖기 때문에, 여기서는 Order라는 접두어를 활용한다.
    - 시스템을 분리할 수 있다면, 근본적으로 나누는 걸 추천(상품 정보 확인 / 장바구니 / 주문).
- Order: 주문
  - 여기서도 동일한 LineItem 활용.
- User: 사용자

## 구현할 기능 정리

사용자가 사용할 수 있는 기능을 정리한다. 비즈니스 우선순위에 따라 기능을 개발할 순서를 결정한다. (매출이 발생할 수 있는 가장 핵심적인 것부터)

ex) 상품에 대한 정보 확인은 필수다. 결제 시스템, 로그인, 장바구니 등이 없어도 상품 정보 확인 후 판매자에게 연락을 취해 구매를 할 수 있다. (옛날의 배달 시스템을 떠올리면 됨)

    1. 상품 목록 확인
    2. 상품 상세 정보 확인
    3. 장바구니에 상품 담기
    4. 주문하기 → 배송지 입력, 결제
    5. 주문 목록 확인
    6. 주문 상세 확인
    7. 로그인
    8. 회원 가입

## 화면 구성

사용자에게 제공할 화면 구성을 정리한다.

1. 홈 페이지 - `/`
2. 상품 목록 페이지 - `/products`
3. 상품 상세 페이지 - `/products/{id}`
4. 장바구니 페이지 - `/cart`
5. 주문 페이지 - `/order`
6. 주문 완료 페이지 - `/order/complete`
7. 주문 목록 페이지 - `/orders`
8. 주문 상세 페이지 - `/orders/{id}`
9. 로그인 페이지 - `/login`
10. 회원 가입 페이지 - `/signup`
11. 회원 가입 완료 페이지 - `/signup/complete`

## API

백엔드에서 제공하는 API 명세서. 받아올 수 있는 데이터의 종류와 형태, 요청 형태 등을 체크한다.

## 개발환경 세팅

개발환경 세팅 + CodeceptJS 세팅(아샬님의 [기본환경 세팅](https://github.com/ahastudio/til/blob/main/react/20230205-setup-react-project.md), [CodeceptJS 세팅](https://github.com/ahastudio/CodingLife/tree/main/20211012/react#codeceptjs-%EC%82%AC%EC%9A%A9))

## 필요한 라이브러리 설치

- [React Router](https://github.com/remix-run/react-router)
- [styled-components](https://github.com/styled-components/styled-components)
- [styled-reset](https://github.com/zacanger/styled-reset)
- [usehooks-ts](https://github.com/juliencrn/usehooks-ts)
- [Axios](https://github.com/axios/axios): REST API 사용을 위한 HTTP 클라이언트. Fetch 보다 덜 번거롭고, 에러 처리가 간편하다.
- [tsyringe](https://github.com/microsoft/tsyringe)
- [reflect-metadata](https://github.com/rbuckton/reflect-metadata)
- [usestore-ts](https://github.com/seed2whale/usestore-ts)
- [jest-dom](https://github.com/testing-library/jest-dom): React Testing Library에서 활용할 수 있는 **DOM 확인용 Matcher 모음**
- [MSW](https://github.com/mswjs/msw)

## E2E 테스트 준비

CodeceptJS를 이용한 테스트를 준비한다. 아래 기능 테스트를 통과하는 것을 목표로 삼는다. (9주차 목표)

1. 상품 목록
    1. 모든 상품 보기
    2. 특정 카테고리의 상품 보기
2. 상품 상세
3. 장바구니
    1. 장바구니가 비어있는 경우
    2. 장바구니에 상품을 담은 경우

## Styles

Reset CSS, defaultTheme, GlobalStyle 세팅 및 다크 모드를 위한 styled.d.ts 작성(8주차 노트 참고)

## Routes

사용자 정의 라우터를 사용 (createBrowserRouter) => routes 파일과 Layout 준비(7주차 노트 참고)

## Test Helper

테스트 코드에서 지원하지 않는 기능에 대비하여 React Testing Libarary의 render를 한번 감싼 테스트용 헬퍼 함수를 준비한다.

아래는 styled-components의 Theme를 제공하기 위한 `ThemeProvider`, React Router의 `Link`를 지원하기 위한 `MemoryRouter`를 한번 감싼 모습이다.

```js
// test-helpers.tsx
import { render as originalRender } from '@testing-library/react'; // 실제로 렌더되는 부분의 구분을 위한 별칭

import React from 'react';

import { MemoryRouter } from 'react-router-dom';

import { ThemeProvider } from 'styled-components';

import defaultTheme from './styles/defaultTheme';

export function render(element: React.ReactElement) {
  return originalRender((
    <MemoryRouter initialEntries={['/']}>
      <ThemeProvider theme={defaultTheme}>
        {element}
      </ThemeProvider>
    </MemoryRouter>
  ));
}
```

## Types

위에서 언급했던 **용어 + REST API 스펙**에 맞춰 타입을 작성한다.

실전에서는 프론트엔드가 타입을 주도할수도, 백엔드와 협의를 통해 타입을 정할수도 있다. 타입을 정의하다보면 용어집이 수정될수도 있음.

```js
export type Category = {
  id: string;
  name: string;
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
}

export type ProductOptionItem = {
  id: string;
  name: string;
};

export type ProductOption = {
  id: string;
  name: string;
  items: ProductOptionItem[];
};

export type ProductDetail = {
  id: string;
  category: Category;
  images: Image[];
  name: string;
  price: number;
  options: ProductOption[];
  description: string;
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

export type Cart = {
  lineItems: LineItem[];
  totalPrice: number;
}
```

## MSW 세팅

REST API 스펙에 맞춰서 MSW 핸들러를 준비한다.

```js
import { rest } from 'msw';

import { ProductSummary } from '../types';

import fixtures from '../../fixtures';

const BASE_URL = 'https://shop-demo-api-01.fly.dev';

const productSummaries: ProductSummary[] = fixtures.products
  .map((product) => ({
    id: product.id,
    category: product.category,
    thumbnail: { url: product.images[0].url },
    name: product.name,
    price: product.price,
  }));

const handlers = [
  rest.get(`${BASE_URL}/categories`, (req, res, ctx) => (
    res(ctx.json({ categories: fixtures.categories }))
  )),
  rest.get(`${BASE_URL}/products`, (req, res, ctx) => (
    res(ctx.json({ products: productSummaries }))
  )),
  rest.get(`${BASE_URL}/products/:id`, (req, res, ctx) => {
    const product = fixtures.products.find((i) => i.id === req.params.id);
    if (!product) {
      return res(ctx.status(404));
    }
    return res(ctx.json(product));
  }),
  rest.get(`${BASE_URL}/cart`, (req, res, ctx) => (
    res(ctx.json(fixtures.cart))
  )),
  rest.post(`${BASE_URL}/cart/line-items`, (req, res, ctx) => (
    res(ctx.status(201))
  )),
];

export default handlers;
```
