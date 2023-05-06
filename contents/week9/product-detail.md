# 상품 상세 페이지

1. 상품 상세 데이터 받아오기 : `useFetchProduct` 커스텀 훅 만들기, 로딩 + 에러(Not found 등) 처리, Store 생성
2. 상품 상세 데이터 보여주기 : `ProductDetail` 컴포넌트
3. 조합하기 : `ProductDetailPage`에서 1, 2 합치기

## 구현할 페이지 준비

`useParams`는 React Router에서 제공하는 훅으로, Route path를 가져온다. 상세 페이지는 `useParams`를 이용해 상품의 id를 받아와 상품 데이터를 조회한다.

```js
export default function ProductDetailPage() {
  const params = useParams();

  const { loading, error } = useFetchProduct({
    productId: String(params.productId),
  });

  if (loading) return <p>Loading...</p>;

  if (error) return <p>Error!</p>;

  return (
    <ProductDetail />
  );
}
```

상품 목록 컴포넌트에서는 데이터를 받아오는 동안 로딩, 에러 처리를 하지 않았지만 여기서는 스토어에서 이를 처리하게 만들었다.

## 상품 데이터 받아오기 - loading, error

사실 상품 정보를 받아온다고 했지만, `ProductDetailPage`에서는 loading, error 상태만 받아오도록 했다. 만약 이 페이지에서 상품 데이터를 받아와 내려주게 된다면, 장바구니 담기 기능을 추가했을 때 props drilling 문제가 생길 수 있다고 했다.

따라서 상품 정보는 `ProductDetail` 컴포넌트에서 다른 방법으로 받아가도록 하고, `ProductDetailPage`에서는 상품 정보를 다루지 않는다.

지금부터 Fetch Product 시의 로딩, 에러 상태를 받아올 수 있게 해보자. 우선 스토어부터 작성.

```js
// productStore.ts
@singleton()
@Store()
export default class ProductStore {
  product: ProductDetail = nullProductDetail;

  loading = true;

  error = false;

  async fetchProduct({ productId }: {
    productId: string;
  }) {
    this.startLoading();

    try {
      const product = await apiService.fetchProduct({ productId });
      this.setProduct(product);
    } catch {
      this.setError();
    }
  }

  @Action()
  private startLoading() {
    this.product = nullProductDetail;
    this.loading = true;
    this.error = false;
  }

  @Action()
  private setProduct(product: ProductDetail) {
    this.product = product;
    this.loading = false;
    this.error = false;
  }

  @Action()
  private setError() {
    this.product = nullProductDetail;
    this.loading = false;
    this.error = true;
  }
}
```

로딩, 에러 상황에 따라 직접 각각의 상태를 바꿔줘도 되지만, 맘에 드는 방식을 선택하면 된다. 위 코드는 강의에서 나온 방식이고, 개인적으로 플로우를 이해하기에 훨씬 좋은 방법 같다.

스토어에서의 Fetch Product를 위한 Axios 부분 작성.

```js
// ApiService.ts
export default class ApiService {
  // ...

  async fetchProduct({ productId } : {
    productId : string
  }): Promise<ProductDetail> {
    const { data } = await this.instance.get(`/products/${productId}`);
    return data;
  }
}
export const apiService = new ApiService();
```

다 됐으면 위에서 만든 스토어를 이용해 `useFetchProduct` 커스텀 훅 작성.

```js
// useFetchProduct.ts
import { useEffect } from 'react';
import { container } from 'tsyringe';
import { useStore } from 'usestore-ts';

import ProductStore from '../stores/ProductStore';

const useFetchProduct = ({ productId } : {
    productId: string;
}) => {
  const store = container.resolve(ProductStore);

  const [{ loading, error }] = useStore(store);

  useEffect(() => {
    store.fetchProduct({ productId });
  }, [store, productId]);

  return ({
    loading,
    error,
  });
};

export default useFetchProduct;
```

앞에서도 언급했지만, 해당 커스텀 훅에서는 직접적으로 상품 데이터를 가져오지 않는다. (그래서 사실 `useFetchProduct`라는 이름은 동작과 조금 거리가 있을지도..)

대신 데이터를 가져오는 과정에 있어서 변하는 로딩, 에러 상태만 가져와 `ProductDetailPage` 컴포넌트에 반영하고 상황에 맞는 화면을 보여줄 수 있다.

### 참고 - Null Object

데이터가 null인 상황(초기값)을 대비하여 Null Object를 만들어준다. (`product : ProductDetail | null = null`로 해줘도 되지만, 취향 차이인 것 같다.) 참고로 강의에서는 Null Object를 `types.ts` 파일에 만들어뒀다.

```js
export const nullProductDetail: ProductDetail = {
  id: '',
  category: { id: '', name: '' },
  images: [],
  name: '',
  price: 0,
  options: [],
  description: '',
};
```

## 상품 데이터 받아오기 - Product Detail

앞에서는 loading, error 상태를 받아오기 위한 구성을 했다. 하지만 `ProductDetail` 컴포넌트에서 상품 상세 정보를 보여주려면 상품 데이터를 받아와야 한다. 위에서 만들어뒀던 코드들을 약간 추가+수정해서 상품 정보도 받아오게 변경해보자.

우선 기존의 `ProductStore`의 이름을 `ProductDetailStore`로 변경하여 좀 더 자세히 의도를 드러내자.

```js
// ProductDetailStore.ts
@singleton()
@Store()
export default class ProductDetailStore {
  // ...
}
```

그리고 `useFetchProductDetail` 커스텀 훅을 생성하여 `ProductDetailStore`에 있는 상태들을 가져다 쓸 수 있게 하자.

```js
// useProductDetailStore.ts
import { container } from 'tsyringe';
import { useStore } from 'usestore-ts';

import ProductDetailStore from '../stores/ProductDetailStore';

const useProductDetailStore = () => {
  const store = container.resolve(ProductDetailStore);
  return useStore(store);
};

export default useProductDetailStore;
```

이렇게 만들어두면 `ProductDetail` 컴포넌트에서 `useProductDetailStore`를 통해 상품 정보를 받아올 수 있게 된다.

`useProductDetailStore`가 생겼으니 기존의 `useFetchProduct`에서 데이터를 가져오는 방식도 바꿔주자.

```js
// useFetchProduct.ts
import { useEffect } from 'react';

import useProductDetailStore from './useProductDetailStore';

const useFetchProduct = ({ productId } : {
    productId: string;
}) => {
  const [{ loading, error }, store] = useProductDetailStore();

  useEffect(() => {
    store.fetchProduct({ productId });
  }, [store, productId]);

  return { loading, error };
};

export default useFetchProduct;
```

## 상품 데이터 보여주기

```js
import styled from 'styled-components';

import useProductDetailStore from '../../hooks/useProductDetailStore';

import Images from './Images';
import Description from './Description';

const Container = styled.div`
  display: flex;
  justify-content: space-between;

  aside {
    width: 38%;
  }

  article {
    width: 60%;
  }
`;

export default function ProductDetailView() {
  const [{ product }] = useProductDetailStore();

  return (
    <Container>
      <aside>
        <Images images={product.images} />
      </aside>
      <article>
        <h2>{product.name}</h2>
        <AddToCartForm /> {/* 추후 추가 예정 */}
        <Description value={product.description} />
      </article>
    </Container>
  );
}
```

`useProductDetailStore`를 통해 상품 상세 데이터를 받아와 화면을 구성했다. 스타일이 들어가거나 데이터를 추가적으로 가공할 필요가 있을 때, 컴포넌트를 분리해주면 깔끔하게 정리가 가능하다.

```js
// Images.tsx
import styled from 'styled-components';

import { Image } from '../../types';

const Thumbnail = styled.img.attrs({
  alt: 'Product Image',
})`
  display: block;
  width: 100%;
  aspect-ratio: 1/1;
`;

export default function Images({ images } : {
  images : Image[]
}) {
  const [image] = images;

  if (!image) {
    return null;
  }

  return (
    <Thumbnail src={image.url} />
  );
}
```

```js
// Description.tsx
import styled from 'styled-components';

function key(value: string, index: number) {
  return `${index}-${value}`;
}

const Container = styled.div`
  li {
    min-height: 1rem;
    line-height: 1.4;
  }
`;

export default function Description({ value } : {
    value : string
  }) {
  if (!value.trim()) {
    return null;
  }

  const lines = value.split('\n');

  return (
    <Container>
      <ul>
        {lines.map((line, index) => (
          <li key={key(line, index)}>
            {line}
          </li>
        ))}
      </ul>
    </Container>
  );
}
```

Description은 데이터가 들어올 때 여러 줄의 문자열이 한 문장으로 합쳐져있는데, 화면에서 보기 좋게 뿌리기 위해 split과 map을 이용하여 가공했다.

또한 key로 잡아줄만한 뚜렷하게 구분되는 데이터(id처럼)가 없어서 index와 자른 문자열을 합쳐서 key로 지정했는데, 렌더링 구문에서 key에 직접적으로 index를 사용하게 되면 eslint가 경고를 준다. 위 코드에선 경고를 없애기 위해 함수를 따로 빼는 꼼수를 썼다.

## Test

### 테스트 항목

상품 상세 페이지와 관련해서 테스트를 할만한 항목은 아래와 같다.

- routes : 올바른 path, 올바르지 않은 path로 이동 시 각각 맞는 화면이 나오는 지 체크
  - 올바른 path일 때, 정상적으로 상품 정보를 가져오는지 확인
  - 올바르지 않은 path일 때, Not found 관련 페이지 나오는지 확인
- 상품 디테일 컴포넌트 : 올바른 path로 이동하여 페이지가 로드된 후를 체크
  - 로딩 화면 체크
  - 상품 정보(텍스트) 체크
  - 상품 정보 중 한 가지 이상이 없을 때

### 테스트 시 가짜 데이터 불러올 때

- jest.mock 으로 Fetch 커스텀 훅 가로채서 데이터 불러오기
- 테스트 코드 내에서 `container.resolve(Store)` -> store 내 Action으로 데이터 불러오기(`beforeEach`로 빼주면 편하게 사용할 수 있다.)

둘 중 편한 걸로 사용하자.
