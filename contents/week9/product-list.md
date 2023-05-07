# 상품 목록 페이지

상품 목록을 보여주기 위해서 아래 단계를 거쳐야한다.

1. 상품 목록을 받아오기 : `useFetchProducts` hook 만들기
2. 상품 목록을 보여주기 : `Products` 컴포넌트로 구현하기
3. 조합하기 : `ProductListPage`에서 1, 2 합치기

상품 목록 띄우기에 성공했다면, 카테고리 별 상품 목록도 만들어보자.

1. Header에 카테고리 별 링크 만들기 : `useFetchCategories` hook 만들기
2. 카테고리 별 상품 받아오기 : 카테고리 id를 이용하여 상품 받아오기

> 이것저것 만들다보면 폴더 구조가 복잡해질 수 있다. 상황에 따라, 필요에 따라 폴더 구조를 본인이 잘 컨트롤 할 수 있도록 만들어가자. 컴포넌트도 마찬가지다. 너무 길어져서 구조를 알아보기 힘들다면 분리해도 된다.

## 구현할 페이지 준비

`useFetchProducts` 내에서 데이터를 어떤 형태로, 어떤 종류의 데이터를 내보낼건지 등을 선택해야 한다. 필요에 따라 error나 loading 상태 등을 추가로 받아올 수 있다. 강의에서는 상품 데이터만 받아왔다.

```js
import Products from '../components/product-list/Products';

import useFetchProducts from '../hooks/useFetchProducts';

export default function ProductListPage() {
  const { products } = useFetchProducts();

  return (
    <div>
      <h2>Products</h2>
      <Products products={products} />
    </div>
  );
}
```

## 상품 받아오기 - useFetchProducts

```js
import { useFetch } from 'usehooks-ts';

import { ProductSummary } from '../types';

// TODO : 환경변수로 빼기
const apiBaseUrl = 'https://shop-demo-api-01.fly.dev';

export default function useFetchProducts() {
  type Data = {
    products: ProductSummary[];
  };

  const { data } = useFetch<Data>(`${apiBaseUrl}/products`);

  return {
    products: data?.products ?? [],
  };
```

products가 비어있는 값이 되지 않기 위해 data가 없을 때 빈 배열이 출력되도록 하자.

`useFetch`에서 error도 제공하니 사용할거면 아래처럼 사용하자. 데이터가 아직 받아오지 않은 상태를 loading으로 사용할수도 있다.

```js
  const { data, error } = useFetch<Data>(`${apiBaseUrl}/products`);

  return {
    products: data?.products ?? [],
    loading: !data,
    error,
  };
}
```

## 상품 보여주기 - Products 컴포넌트

처음에는 데이터가 잘 받아와지는지 확인하기 위해 문자열로 띄워본다.

```js
import { ProductSummary } from '../../types';

interface ProductProps{
    products: ProductSummary[];
}

export default function Products({ products } : ProductProps) {
  return (
    <div>
      {products.map((product) => (
        <p key={product.id}>{JSON.stringify(product)}</p>
      ))}
    </div>
  );
}
```

잘 나오는걸 확인했다면 화면구성 시작.

- 순서가 없는 리스트니까 ul, li 사용
- 스타일 적용(styled-components)
- 제품 클릭 시 상세 정보 보여줌
  - 상세 정보 컴포넌트 `Product` 생성

```js
import { Link } from 'react-router-dom';
import styled from 'styled-components';

import { ProductSummary } from '../../types';

import Product from './Product';

const Container = styled.div`
  ul {
    display: flex;
    flex-wrap: wrap;
  }

  li {
    width: 20%;
    padding: 1rem;
  }

  a {
    display: block;
  }
`;

type ProductsProps = {
  products: ProductSummary[];
}

export default function Products({ products }: ProductsProps) {
  if (!products.length) {
    return null;
  }

  return (
    <Container>
      <ul>
        {products.map((product) => (
          <li key={product.id}>
            <Link to={`/products/${product.id}`}>
              <Product product={product} />
            </Link>
          </li>
        ))}
      </ul>
    </Container>
  );
}
```

`Product` 컴포넌트도 만들기(처음엔 데이터 전달 잘 되는지부터!)

```js
import { ProductSummary } from '../../types';

interface ProductProps{
    product: ProductSummary;
  }

export default function Product({ product } : ProductProps) {
  return (
    <p>{JSON.stringify(product)}</p>
  );
}
```

잘 받아와는지 확인했으면 화면 구성

```js
import styled from 'styled-components';

import { ProductSummary } from '../../types';

const Thumbnail = styled.img.attrs({
  alt: 'Thumbnail',
})`
  display: block;
  width: 100%;
  aspect-ratio: 1/1;
`;

type ProductProps = {
  product: ProductSummary;
}

export default function Product({ product }: ProductProps) {
  return (
    <div>
      <Thumbnail src={product.thumbnail.url} />
      <div>{product.name}</div>
      <div>
        {numberFormat(product.price)}
        원
      </div>
    </div>
  );
}
```

### 참고 - numberFormat

```js
// /src/utils/numberFormat.ts
export default function numberFormat(value: number) {
  return new Intl.NumberFormat().format(value);
}
```

`Intl.NumberFormat()`는 현재 로케일에 맞춰 숫자를 문자열로 변환시켜준다.

ex) 영어권이면 `12345` -> `12,345`, 독일권이면 `12345` -> `12.345`

특정 나라의 통화로 사용하고 싶으면 괄호 안 인자에 옵션을 추가해주면 된다.

### 참고 - Thumbnail

이미지에 대한 스타일 적용을 위해 스타일 컴포넌트로 따로 분리해줬다.

대체 텍스트같은는 경우에 따라 다르지만, 대체 텍스트가 있는게 오히려 방해가 될 수도 있다고 한다. 따라서 서비스를 이용하는 유저가 어떤 것을 더 선호하는지에 따라 대체 텍스트를 지정하는 것이 좋다.

### 참고 - Text

한정된 공간 안에서 텍스트를 표시할 때, 글자 단위로 줄띄움을 할지 아니면 단어 단위로 줄띄움을 할지 등을 선택해야 한다. 사용자가 어떤 환경에서 서비스를 이용할 지 모르므로, 글자가 넘치면(text-overflow) '...'을 넣는 등의 대비가 되어있어야 한다.

## 상품 보여주기 - Store

상품 목록을 Store를 통해 받아오고 관리할 수 있다.

```js
import axios from 'axios';

import { singleton } from 'tsyringe';
import { Action, Store } from 'usestore-ts';

import { ProductSummary } from '../types';

// TODO : 환경변수로 빼기
const apiBaseUrl = 'https://shop-demo-api-01.fly.dev';

@singleton()
@Store()
export default class ProductsStore {
  products: ProductSummary[] = [];

  async fetchProducts() {
    this.setProducts([]);

    const { data } = await axios.get(`${apiBaseUrl}/products`);
    const { products } = data;

    this.setProducts(products);
  }

  @Action()
  setProducts(products: ProductSummary[]) {
    this.products = products;
  }
}
```

`useFetchProducts`에서 `useFetch`가 아닌 Store를 통해 데이터를 받아오도록 변경하면 된다.

```js
// /src/hooks/useFetchProducts.ts
import { useEffect } from 'react';

import { container } from 'tsyringe';
import { useStore } from 'usestore-ts';

import ProductsStore from '../stores/ProductsStore';

export default function useFetchProducts() {
  const store = container.resolve(ProductsStore);

  const [{ products }] = useStore(store);

  useEffect(() => {
    store.fetchCategories();
  }, [store]);

  return ({
    products,
  });
}
```

## 카테고리 불러오기 - Store

Products 불러오는 것과 다를 것 없음. 카테고리로 변경해주자.

```js
import axios from 'axios';

import { singleton } from 'tsyringe';
import { Action, Store } from 'usestore-ts';

import { Category } from '../types';

// TODO : 환경변수로 빼기
const apiBaseUrl = 'https://shop-demo-api-01.fly.dev';

@singleton()
@Store()
export default class CategoriesStore {
  categories: Category[] = [];

  async fetchCategories() {
    this.setCategories([]);

    const { data } = await axios.get(`${apiBaseUrl}/categories`);
    const { categories } = data;

    this.setCategories(categories);
  }

  @Action()
  setCategories(categories: Category[]) {
    this.categories = categories;
  }
}
```

```js
import { useEffect } from 'react';

import { container } from 'tsyringe';
import { useStore } from 'usestore-ts';

import CategoriesStore from '../stores/CategoriesStore';

export default function useFetchCategories() {
  const store = container.resolve(CategoriesStore);

  const [{ categories }] = useStore(store);

  useEffect(() => {
    store.fetchCategories();
  }, [store]);

  return ({
    categories,
  });
}
```

코드에서 보다시피 중복되는 코드가 있고, 앞으로도 계속 중복해서 사용할 것 같은 부분을 따로 빼주자.

여기서는 axios를 통해 데이터를 받아오는 부분, URL이 대상만 다르고 이외에는 동일한 코드를 사용하는 걸 볼 수 있다.

```js
// /src/services/ApiService.ts
import axios from 'axios';

import { Category, ProductSummary } from '../types';

const API_BASE_URL = process.env.API_BASE_URL
                     || 'https://shop-demo-api-01.fly.dev';

export default class ApiService {
  private instance = axios.create({
    baseURL: API_BASE_URL,
  });

  async fetchCategories(): Promise<Category[]> {
    const { data } = await this.instance.get('/categories');
    const { categories } = data;
    return categories;
  }

  async fetchProducts(): Promise<ProductSummary[]> {
    const { data } = await this.instance.get('/products');
    const { products } = data;
    return products;
  }
}

export const apiService = new ApiService();
```

참고로 `axios.create()`는 axios에서 사용할 인스턴스를 정의하는 메서드로, 사용할 설정을 정의하거나 인터셉트를 추가하는 등의 작업을 할 수 있다.

코드에서는 baseURL가 정의된 인스턴스를 이용하여 바로 get 요청을 하도록 만들어져있다.

이 코드를 활용하면 데이터 요청 코드는 다음과 같이 바뀔 수 있다.

```js
import { singleton } from 'tsyringe';
import { Action, Store } from 'usestore-ts';

import { Category } from '../types';

import { apiService } from '../services/ApiService';

@singleton()
@Store()
export default class CategoriesStore {
  categories: Category[] = [];

  async fetchCategories() {
    this.setCategories([]);

    const categories = await apiService.fetchCategories();

    this.setCategories(categories);
  }

  @Action()
  setCategories(categories: Category[]) {
    this.categories = categories;
  }
}
```

apiService에서 받아온 데이터는 프로미스를 반환하므로 await와 함께 써주자.

## 카테고리 별 상품 목록 불러오기

- 카테고리 id를 받아오기
  - `Link`에 담아둔 쿼리 파라미터(카테고리 id)를 사용
- 받아온 카테고리 id에 해당하는 상품만 불러오기

```js
import { useSearchParams } from 'react-router-dom';

import Products from '../components/product-list/Products';

import useFetchProducts from '../hooks/useFetchProducts';

export default function ProductListPage() {
  const [params] = useSearchParams();

  const categoryId = params.get('categoryId') ?? undefined;

  const { products } = useFetchProducts({ categoryId });

  return (
    <div>
      <h2>Products</h2>
      <Products products={products} />
    </div>
  );
}
```

참고로 `useSearchParams()`는 React Router에서 제공하는 훅으로, 쿼리 문자열을 가져오는 데 사용된다. 내부적으로 `location.search`가 사용되고 있다.

해당 훅으로 가져온 쿼리 스트링에 다양한 메서드를 사용할 수 있으며, 여기서 사용한건 `get` 메서드이다. `get`은 인자로 쿼리 이름을 넣어주면 값을 반환해준다.

전체 상품을 가져올 땐 카테고리 id가 없을수도 있으므로, `get`에서 반환된 값이 없으면 카테고리 id에 undefined이 저장되도록 한다.

이후에는 Products를 받아오는 모든 관련된 파일에 카테고리 id를 받을 수 있도록 고쳐주면 된다. 아무 값도 받지 않을 경우를 대비하여 옵셔널 프로퍼티를 써주자.

```js
export default function useFetchProducts({
  categoryId,
} : {categoryId? : string}) {
  const store = container.resolve(ProductsStore);

  const [{ products }] = useStore(store);

  useEffect(() => {
    store.fetchProducts({ categoryId });
  }, [store, categoryId]);

  return ({
    products,
  });
}
```

카테고리 id가 바뀔 때마다 상품을 새로 불러와야 하므로 useEffect로 바꾼 뒤 `categoryId`를 의존하게 만들어줘야 한다.

```js
export default class ProductsStore {
  products: ProductSummary[] = [];

  async fetchProducts({ categoryId } : { categoryId?: string }) {
    this.setProducts([]);

    const products = await apiService.fetchProducts({ categoryId });

    this.setProducts(products);
  }

  @Action()
  setProducts(products: ProductSummary[]) {
    this.products = products;
  }
}
```

```js
export default class ApiService {

  // ...

  async fetchProducts(
    { categoryId } : {categoryId?: string},
  ): Promise<ProductSummary[]> {
    const { data } = await this.instance.get('/products', {
      params: { categoryId },
    });
    const { products } = data;
    return products;
  }
}

export const apiService = new ApiService();
```

axios에서 쿼리 파라미터를 넘겨줄 때, path에 직접 써도 되고 위 코드처럼 따로 적어줘도 된다.

## Test

이제 상품 목록을 불러오는 것과 관련하여 테스트할 수 있다.

라우트 테스트에서는 전체 목록을 불러왔을 때 어떤 상품이 보이는지, 카테고리 목록에서 해당 카테고리 상품은 보이고 다른 카테고리 상품이 보이지 않는지 등을 체크할 수 있다.

또한 codeceptjs로도 메뉴에서 전체 상품 또는 카테고리 메뉴를 클릭했을 때 정상적으로 상품들이 보이는지도 체크할 수 있다.

> TODO : 테스트 코드 추가
