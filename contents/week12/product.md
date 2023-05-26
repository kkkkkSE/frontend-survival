# 상품 관리

상품 관리에선 기본적으로 SWR를 이용해 데이터를 받아오긴 하지만, GET 요청을 제외한 추가, 수정 요청 및 폼 데이터 관리를 위해 React Hook Form 대신 usestore-ts를 사용했다. 아마 폼이 유동적이기 때문에 이렇게 관리하는 편이 나을 수도 있어서 인듯 하다.

## 상품 목록 페이지

우선 상품 목록을 불러오는 커스텀 훅을 작성한다. 위에서 언급한 것처럼 스토어에서 상품 추가나 수정 로직을 넣을 거라서, SWR가 가져오는 상품 목록에 대해 캐시 초기화(reload)하기 위해 refresh 함수를 하나 만들어서 내보내주자.

```js
import { ProductSummary } from '../types';

import useFetch from './useFetch';

const useFetchProducts = () => {
  const {
    data, error, loading, mutate,
  } = useFetch<{
        products : ProductSummary[]
    }>('/products');

  return ({
    products: data?.products ?? [],
    error,
    loading,
    refresh() {
      mutate();
    },
  });
};

export default useFetchProducts;
```

상품 목록 화면 생성

```js
import { Link } from 'react-router-dom';

import styled from 'styled-components';

import useFetchProducts from '../hooks/useFetchProducts';

import numberFormat from '../utils/numberFormat';

const Container = styled.div`
  // ...
`;

export default function ProductListPage() {
  const { products, loading, error } = useFetchProducts();

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
      <h2>Products</h2>
      <table>
        <thead>
          <tr>
            <th>카테고리</th>
            <th>이미지</th>
            <th>제품명</th>
            <th>가격</th>
            <th>표시</th>
            <th>행동</th>
          </tr>
        </thead>
        <tbody>
          {products.map((product) => (
            <tr key={product.id}>
              <td>
                {product.category.name}
              </td>
              <td>
                {product.thumbnail?.url && (
                  <img src={product.thumbnail.url} alt="썸네일" />
                )}
              </td>
              <td>
                {product.name}
              </td>
              <td>
                {numberFormat(product.price)}
                원
              </td>
              <td>
                {product.hidden ? '숨김' : '보임'}
              </td>
              <td>
                <Link to={`/products/${product.id}`}>
                  자세히
                </Link>
                <Link to={`/products/${product.id}/edit`}>
                  수정
                </Link>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
      <Link to="/products/new">
        상품 추가
      </Link>
    </Container>
  );
}
```

## 상품 상세 페이지

상품 상세 불러오는 커스텀 훅 작성. 상품 전체 목록 불러왔을 때처럼 mutate를 이용한 refresh 함수 만들기

```js
import useFetch from './useFetch';

import { ProductDetail } from '../types';

export default function useFetchProduct({ productId }: {
  productId: string;
}) {
  const url = `/products/${productId}`;

  const {
    data, error, loading, mutate,
  } = useFetch<ProductDetail>(url);

  return {
    product: data,
    error,
    loading,
    refresh() {
      mutate();
    },
  };
}
```

상품 상세 화면 구현

```js
import { Link, useParams } from 'react-router-dom';

import styled from 'styled-components';

import Description from '../components/product-detail/Description';

import useFetchProduct from '../hooks/useFetchProduct';

import numberFormat from '../utils/numberFormat';

const Container = styled.div`
  // ...
`;

export default function ProductDetailPage() {
  const params = useParams();

  const { product, loading, error } = useFetchProduct({
    productId: String(params.id),
  });

  if (loading) {
    return (
      <p>Loading...</p>
    );
  }

  if (error || !product) {
    return (
      <p>Error!</p>
    );
  }

  return (
    <Container>
      <h2>Product Detail</h2>
      <dl>
        <dt>이름</dt>
        <dd>{product.name}</dd>
        <dt>카테고리</dt>
        <dd>{product.category.name}</dd>
        <dt>이미지</dt>
        <dd>
          {product.images.map((image) => (
            <img
              key={image.url}
              src={image.url}
              alt="썸네일"
            />
          ))}
        </dd>
        <dt>가격</dt>
        <dd>
          {numberFormat(product.price)}
          원
        </dd>
        <dt>옵션</dt>
        <dd>
          {product.options.map((option) => (
            <div key={option.id}>
              {option.name}
              :
              {' '}
              {option.items.map((item) => item.name).join(', ')}
            </div>
          ))}
        </dd>
        <dt>설명</dt>
        <dd>
          <Description value={product.description} />
        </dd>
        <dt>표시</dt>
        <dd>{product.hidden ? '숨김' : '보임'}</dd>
      </dl>
      <Link to={`/products/${product.id}/edit`}>
        수정
      </Link>
    </Container>
  );
}
```

```js
import styled from 'styled-components';

const Container = styled.div`
  // ...
`;

const key = (line:string, index:number) => `${index}-${line}`;

type DescriptionProps = {
  value: string;
}

export default function Description({ value }: DescriptionProps) {
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

## 상품 관리 스토어

우선 ApiService에 상품 추가, 변경 요청 Axios 코드 추가부터 해놓자.

```js
export default class ApiService {
  // ...

  async createProduct({
    categoryId, images, name, price, options, description,
  }: {
    categoryId: string;
    images: Image[];
    name: string;
    price: number;
    options: ProductOption[];
    description: string;
  }): Promise<void> {
    await this.instance.post('/products', {
      categoryId, images, name, price, options, description,
    });
  }

  async updateProduct({
    productId, categoryId, images, name, price, options, description, hidden,
  }: {
    productId: string;
    categoryId: string;
    images: Image[];
    name: string;
    price: number;
    options: ProductOption[];
    description: string;
    hidden: boolean;
  }): Promise<void> {
    await this.instance.patch(`/products/${productId}`, {
      categoryId, images, name, price, options, description, hidden,
    });
  }
}
```

이제부터 스토어를 사용하여 로직을 구현할 것이다. 스토어에는 다음과 같은 로직이 필요하다.

- Form 초기화
- Form 유효성 검사
- 상품 수정 시 기존 데이터를 Form에 적용하는 `setProduct`
- 각 input(textbox 등)에 입력값을 넣으면 스토어의 상태에 적용시키는 change 로직
- 이미지 추가, 삭제, 변경 로직
- 상품 옵션과 옵션 내 선택사항 추가, 삭제, 변경 로직
- 숨김, 보임 toggle
- 에러 발생 시 에러 상태를 true로 바꿔주는 `setError`
- 에러 없이 API 호출이 정상적으로 끝났을 때 done 상태를 true로 바꿔주는 `setDone`
- 상품의 추가 및 업데이트를 처리하는(POST, PATCH 요청) 로직

강의에선 테스트 코드를 먼저 작성해서 어떤 기능을 만들건지 미리 체크하고 기능 구현을 시작한다.

```js
import ProductFormStore from './ProductFormStore';

import fixtures from '../../fixtures';

const createProduct = jest.fn();
const updateProduct = jest.fn();

jest.mock('../services/ApiService', () => ({
  get apiService() { 
    return {
      createProduct,
      updateProduct,
    };
  },
}));

const context = describe;

describe('ProductFormStore', () => {
  let store: ProductFormStore;

  beforeEach(() => {
    jest.clearAllMocks();

    store = new ProductFormStore();
  });

  describe('toggleHidden', () => {
    it('changes “hidden” to true and false alternately', () => {
      expect(store.hidden).toBeFalsy();

      store.toggleHidden();

      expect(store.hidden).toBeTruthy();

      store.toggleHidden();

      expect(store.hidden).toBeFalsy();
    });
  });

  describe('create', () => {
    const [category] = fixtures.categories;

    beforeEach(() => {
      store.reset();

      store.changeCategory(category);
      store.changeName('New Product');
      store.changePrice('123400');
      store.changeDescription('What is this?');

      store.changeImageUrl(0, 'http://example.com/images/01.jpg');
      store.addImage();
      store.changeImageUrl(1, 'http://example.com/images/02.jpg');
      store.removeImage(1);

      expect(store.images).toHaveLength(1);

      store.addOption();
      store.addOption();
      store.addOption();
      store.removeOption(2);
      store.changeOptionName(0, 'Color');
      store.changeOptionName(1, 'Size');

      expect(store.options).toHaveLength(2);

      store.addOptionItem(0);
      store.addOptionItem(0);

      store.removeOptionItem(0, 2);
      store.changeOptionItemName(0, 0, 'Black');
      store.changeOptionItemName(0, 1, 'White');

      expect(store.options[0].items).toHaveLength(2);

      store.changeOptionItemName(1, 0, 'Free');

      expect(store.valid).toBeTruthy();

      store.toggleHidden();
    });

    context('when API responds with success', () => {
      it('sets done is true', async () => {
        await store.create();

        expect(createProduct).toBeCalled();

        expect(store.done).toBeTruthy();
        expect(store.error).toBeFalsy();
      });
    });

    context('when API responds with error', () => {
      beforeEach(() => {
        createProduct.mockRejectedValue(Error('Create Product API error!'));
      });

      it('sets error is true', async () => {
        await store.create();

        expect(createProduct).toBeCalled();

        expect(store.done).toBeFalsy();
        expect(store.error).toBeTruthy();
      });
    });
  });

  describe('update', () => {
    const [product] = fixtures.products;

    beforeEach(() => {
      store.setProduct(JSON.parse(JSON.stringify(product)));

      store.changeName('New Name');
    });

    context('when API responds with success', () => {
      it('sets done is true', async () => {
        await store.update();

        expect(updateProduct).toBeCalled();

        expect(store.done).toBeTruthy();
        expect(store.error).toBeFalsy();
      });
    });

    context('when API responds with error', () => {
      beforeEach(() => {
        updateProduct.mockRejectedValue(Error('Update Product API error!'));
      });

      it('sets error is true', async () => {
        await store.update();

        expect(updateProduct).toBeCalled();

        expect(store.done).toBeFalsy();
        expect(store.error).toBeTruthy();
      });
    });
  });
});
```

비즈니스 로직을 테스트 할 때 중요한 것 중 하나는 테스트를 처리하는 과정에서 원본 데이터(fixture)가 변경될 수 있으므로, fixture 중 참조 데이터를 사용해서 로직을 테스트 할 때는 깊은 복사 처리 후 진행하자. 전개 연산자나, JSON 관련 메서드 등을 사용할 수 있다. (아니면 스토어 내에서 이 과정을 진행해도 됨)

테스트 코드 작성이 끝나면, 이 테스트가 통과할 수 있게 스토어를 작성하면 된다.

```js
import { singleton } from 'tsyringe';

import { Store, Action } from 'usestore-ts';

import { apiService } from '../services/ApiService';

import {
  Category, Image, ProductDetail, ProductOption,
} from '../types';

import { append, remove, update } from '../utils';

@singleton()
@Store()
export default class ProductFormStore {
  productId = '';

  category: Category | null = null;

  images: Image[] = [];

  name = '';

  price = '';

  options: ProductOption[] = [];

  description = '';

  hidden = false;

  error = false;

  done = false;

  get valid() {
    const price = parseInt(this.price, 10);
    return !!this.category?.id
      && this.images.length && this.images.every((i) => i.url)
      && !!this.name.trim() && Number.isInteger(price)
      && this.options.every((option) => (
        option.name && option.items.length
          && option.items.every((item) => item.name)
      ))
      && this.description.trim();
  }

  @Action()
  reset() {
    this.productId = '';
    this.category = null;
    this.images = [{ url: '' }];
    this.name = '';
    this.price = '';
    this.options = [];
    this.description = '';
    this.error = false;
    this.done = false;
  }

  @Action()
  setProduct(product: ProductDetail) {
    this.productId = product.id;
    this.category = product.category;
    this.images = product.images;
    this.name = product.name;
    this.price = product.price.toString();
    this.options = product.options;
    this.description = product.description;
    this.hidden = product.hidden;
    this.error = false;
    this.done = false;
  }

  @Action()
  changeCategory(category: Category) {
    this.category = category;
  }

  @Action()
  addImage() {
    this.images = append(this.images, { url: '' });
  }

  @Action()
  removeImage(index: number) {
    this.images = remove(this.images, index);
  }

  @Action()
  changeImageUrl(index: number, url: string) {
    this.images = update(this.images, index, (image) => ({
      ...image,
      url,
    }));
  }

  @Action()
  changeName(name: string) {
    this.name = name;
  }

  @Action()
  changePrice(price: string) {
    this.price = price;
  }

  @Action()
  addOption() {
    const option = {
      name: '',
      items: [{ name: '' }],
    };

    this.options = append(this.options, option);
  }

  @Action()
  removeOption(index: number) {
    this.options = remove(this.options, index);
  }

  @Action()
  changeOptionName(index: number, name: string) {
    this.options = update(this.options, index, (option) => ({
      ...option,
      name,
    }));
  }

  @Action()
  addOptionItem(optionIndex: number) {
    this.options = update(this.options, optionIndex, (option) => ({
      ...option,
      items: append(option.items, { name: '' }),
    }));
  }

  @Action()
  removeOptionItem(optionIndex: number, itemIndex: number) {
    this.options = update(this.options, optionIndex, (option) => ({
      ...option,
      items: remove(option.items, itemIndex),
    }));
  }

  @Action()
  changeOptionItemName(optionIndex: number, itemIndex: number, name: string) {
    this.options = update(this.options, optionIndex, (option) => ({
      ...option,
      items: update(option.items, itemIndex, (item) => ({
        ...item,
        name,
      })),
    }));
  }

  @Action()
  changeDescription(description: string) {
    this.description = description;
  }

  @Action()
  toggleHidden() {
    this.hidden = !this.hidden;
  }

  @Action()
  private setError() {
    this.error = true;
  }

  @Action()
  private setDone() {
    this.done = true;
  }

  async create() {
    try {
      await apiService.createProduct({
        categoryId: this.category?.id || '',
        images: this.images,
        name: this.name,
        price: parseInt(this.price, 10),
        options: this.options,
        description: this.description,
      });

      this.setDone();
    } catch (e) {
      this.setError();
    }
  }

  async update() {
    try {
      await apiService.updateProduct({
        productId: this.productId,
        categoryId: this.category?.id || '',
        images: this.images,
        name: this.name,
        price: parseInt(this.price, 10),
        options: this.options,
        description: this.description,
        hidden: this.hidden,
      });

      this.setDone();
    } catch (e) {
      this.setError();
    }
  }
}
```

여기서 상품 옵션에 관련된 로직에는 새로운 함수를 만들어서 사용해줬다. 상품에는 컬러, 사이즈 등과 같이 여러개의 상품 옵션이 필요하고, 상품 옵션 내에서도 다양한 선택지가 있어야 한다. 2depth로 운용하는게 쉽지 않으니 옵션을 추가, 변경, 삭제할 때 로직을 처리해주는 유틸리티 함수를 따로 만들어 사용했다.

그리고 옵션의 id의 경우, 백엔드에서 상품 등록 시 부여해주는 거라서 프론트엔드에서 처음 옵션을 추가할 땐 옵션에 대한 id 값을 가지고 있지 않다. 그래서 `types.ts` 파일의 옵션과 관련된 타입에서 id는 옵셔널하게 받도록 변경해줬다.

```js
export function append<T>(items: T[], item: T) {
  return [...items, item];
}

export function remove<T>(items: T[], index: number) {
  return [
    ...items.slice(0, index),
    ...items.slice(index + 1),
  ];
}

export function update<T>(items: T[], index: number, f: (value: T) => T) {
  return items.map((item, i) => (i === index ? f(item) : item));
}
```

마지막으로 스토어 객체 만들어 줄 커스텀 훅 생성

```js
import { container } from 'tsyringe';

import { useStore } from 'usestore-ts';

import ProductFormStore from '../stores/ProductFormStore';

export default function useProductFormStore() {
  const store = container.resolve(ProductFormStore);
  return useStore(store);
}
```

## 상품 추가하기

상품을 가져오고, 변경할 때 스토어에서 기능을 수행하기 때문에 실질적으로 상품의 목록을 가져오는 SWR를 잊어선 안된다. 추가, 변경이 끝나면 SWR의 mutate를 이용하여 상품 목록을 새로 불러와주자. 이전에 refresh라는 함수로 이를 실행할 수 있게 해두었다.

```js
import { useEffect } from 'react';

import { useNavigate } from 'react-router-dom';

import ProductNewForm from '../components/product/ProductNewForm';

import useFetchProducts from '../hooks/useFetchProducts';
import useFetchCategories from '../hooks/useFetchCategories';
import useProductFormStore from '../hooks/useProductFormStore';

export default function ProductNewPage() {
  const navigate = useNavigate();

  const { refresh } = useFetchProducts();
  const { categories } = useFetchCategories();

  const [, store] = useProductFormStore();

  useEffect(() => {
    if (!categories.length) {
      return;
    }

    store.reset();
    store.changeCategory(categories[0]); // 처음 들어오면 자동으로 셀렉트박스 첫번째꺼 선택되어 있는 것 처리
  }, [store, categories]);

  const handleComplete = () => {
    refresh();
    navigate('/products');
  };

  if (!categories.length) {
    return null;
  }

  return (
    <ProductNewForm
      categories={categories}
      onComplete={handleComplete}
    />
  );
}
```

```js
import { useEffect } from 'react';

import styled from 'styled-components';

import ComboBox from '../ui/ComboBox';
import TextBox from '../ui/TextBox';
import Button from '../ui/Button';

import Images from './Images';
import Options from './Options';

import useProductFormStore from '../../hooks/useProductFormStore';

import { Category } from '../../types';

const Container = styled.div`
  h2 {
    margin-bottom: 2rem;
    font-size: 2rem;
  }
`;

type ProductNewFormProps = {
  categories: Category[];
  onComplete: () => void;
}

export default function ProductNewForm({
  categories, onComplete,
}: ProductNewFormProps) {
  const [{
    category, images, name, price, options, description, valid, error, done,
  }, store] = useProductFormStore();

  const handleSubmit = async (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    await store.create();
  };

  useEffect(() => {
    if (done) {
      onComplete();
    }
  }, [done]);

  return (
    <Container>
      <h2>New Product</h2>
      <form onSubmit={handleSubmit}>
        <ComboBox
          label="카테고리"
          selectedItem={category}
          items={categories}
          itemToId={(item) => item?.id || ''}
          itemToText={(item) => item?.name || ''}
          onChange={(value) => value && store.changeCategory(value)}
        />
        <Images images={images} store={store} />
        <TextBox
          label="상품명"
          value={name}
          onChange={(value) => store.changeName(value)}
        />
        <TextBox
          type="number"
          label="가격"
          value={price.toString()}
          onChange={(value) => store.changePrice(value)}
        />
        <Options options={options} store={store} />
        <TextBox
          label="설명"
          value={description}
          onChange={(value) => store.changeDescription(value)}
        />
        <Button type="submit" disabled={!valid}>
          등록
        </Button>
        {error && (
          <p>상품 등록 실패</p>
        )}
      </form>
    </Container>
  );
}
```

```js
import TextBox from '../ui/TextBox';
import Button from '../ui/Button';

import OptionItems from './OptionItems';

import ProductFormStore from '../../stores/ProductFormStore';

import { ProductOption } from '../../types';

import { key } from '../../utils';

type OptionsProps = {
  options: ProductOption[];
  store: Readonly<ProductFormStore>;
}

export default function Options({ options, store }: OptionsProps) {
  return (
    <ul>
      {options.map((option, index) => (
        <li key={key(option.id ?? '', index)}>
          <TextBox
            label={`옵션 #${index + 1}`}
            value={option.name}
            onChange={(value) => store.changeOptionName(index, value)}
          />
          <Button onClick={() => store.removeOption(index)}>
            옵션 삭제
          </Button>
          <OptionItems
            optionIndex={index}
            items={option.items}
            store={store}
          />
        </li>
      ))}
      <li>
        <Button onClick={() => store.addOption()}>
          옵션 추가
        </Button>
      </li>
    </ul>
  );
}
```

```js
import styled from 'styled-components';

import TextBox from '../ui/TextBox';
import Button from '../ui/Button';

import ProductFormStore from '../../stores/ProductFormStore';

import { ProductOptionItem } from '../../types';

import { key } from '../../utils';

const Container = styled.div`
  ol {
    margin-left: 1rem;

    label {
      display: none;
    }
  }
`;

type OptionItemsProps = {
  optionIndex: number;
  items: ProductOptionItem[];
  store: Readonly<ProductFormStore>;
}

export default function OptionItems({
  optionIndex, items, store,
}: OptionItemsProps) {
  return (
    <Container>
      <ol>
        {items.map((item, index) => (
          <li key={key(item.id ?? '', index)}>
            <TextBox
              label={`옵션 아이템 #${optionIndex + 1}-${index + 1}`}
              value={item.name}
              onChange={(value) => (
                store.changeOptionItemName(optionIndex, index, value)
              )}
            />
            <Button onClick={() => store.removeOptionItem(optionIndex, index)}>
              아이템 삭제
            </Button>
          </li>
        ))}
        <li>
          <Button onClick={() => store.addOptionItem(optionIndex)}>
            아이템 추가
          </Button>
        </li>
      </ol>
    </Container>
  );
}
```

```js
import TextBox from '../ui/TextBox';
import Button from '../ui/Button';

import ProductFormStore from '../../stores/ProductFormStore';

import { Image } from '../../types';

import { key } from '../../utils';

type ImagesProps = {
  images: Image[];
  store: Readonly<ProductFormStore>;
}

export default function Images({ images, store }: ImagesProps) {
  return (
    <ul>
      {images.map((image, index) => (
        <li key={key('', index)}>
          <TextBox
            label={`이미지 #${index + 1}`}
            value={image.url}
            onChange={(value) => store.changeImageUrl(index, value)}
          />
          <Button onClick={() => store.removeImage(index)}>
            이미지 삭제
          </Button>
        </li>
      ))}
      <li>
        <Button onClick={() => store.addImage()}>
          이미지 추가
        </Button>
      </li>
    </ul>
  );
}
```

로컬에 있는 이미지를 업로드 할 순 없고, 서버에 올라와있는 이미지의 URL로 가져오는 방식으로 되어있다. 나머지는 어떻게 구현되어 있고, 어떻게 컴포넌트가 분리되어 있는지 참고하면 될 것 같다.

## 상품 수정하기

상품 추가하기와 비슷하지만, 상품 추가하기에선 페이지 접근 시 모든 Form을 초기화 시켰다면 수정 페이지에서는 데이터를 Fetch하여 Form에 입력해줘야 한다. 만약 상품 조회에 실패했다면 아무 것도 반환하지 않는다. 이 때 그냥 로딩 메세지나 실패 문구 등을 띄워주면 좋을듯.

```js
import { useEffect } from 'react';

import { useNavigate, useParams } from 'react-router-dom';

import ProductEditForm from '../components/product/ProductEditForm';

import useFetchProduct from '../hooks/useFetchProduct';
import useFetchCategories from '../hooks/useFetchCategories';
import useProductFormStore from '../hooks/useProductFormStore';

export default function ProductEditPage() {
  const navigate = useNavigate();

  const params = useParams();
  const productId = String(params.id);

  const { product, refresh } = useFetchProduct({ productId });
  const { categories } = useFetchCategories();

  const [, store] = useProductFormStore();

  useEffect(() => {
    if (!product) {
      return;
    }

    store.setProduct(product);
  }, [store, product]);

  const handleComplete = () => {
    refresh();
    navigate(`/products/${productId}`);
  };

  if (!product || !categories.length) {
    return null;
  }

  return (
    <ProductEditForm
      categories={categories}
      onComplete={handleComplete}
    />
  );
}
```

Form은 추가하기와 거의 같은데, 숨김 상태에 대한 항목이 추가됐다. 체크박스를 이용해서 숨김/해제를 변경할거라 UI 컴포넌트도 하나 만들었다.

```js
import { useEffect } from 'react';

import styled from 'styled-components';

import ComboBox from '../ui/ComboBox';
import TextBox from '../ui/TextBox';
import CheckBox from '../ui/CheckBox';
import Button from '../ui/Button';

import Images from './Images';
import Options from './Options';

import useProductFormStore from '../../hooks/useProductFormStore';

import { Category } from '../../types';

const Container = styled.div`
  h2 {
    margin-bottom: 2rem;
    font-size: 2rem;
  }
`;

type ProductEditFormProps = {
  categories: Category[];
  onComplete: () => void;
}

export default function ProductEditForm({
  categories, onComplete,
}: ProductEditFormProps) {
  const [{
    category, images, name, price, options, description, hidden,
    valid, error, done,
  }, store] = useProductFormStore();

  const handleSubmit = async (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    await store.update();
  };

  useEffect(() => {
    if (done) {
      onComplete();
    }
  }, [done]);

  return (
    <Container>
      <h2>Edit Product</h2>
      <form onSubmit={handleSubmit}>
        <ComboBox
          label="카테고리"
          selectedItem={category}
          items={categories}
          itemToId={(item) => item?.id || ''}
          itemToText={(item) => item?.name || ''}
          onChange={(value) => value && store.changeCategory(value)}
        />
        <Images images={images} store={store} />
        <TextBox
          label="상품명"
          value={name}
          onChange={(value) => store.changeName(value)}
        />
        <TextBox
          type="number"
          label="가격"
          value={price.toString()}
          onChange={(value) => store.changePrice(value)}
        />
        <Options options={options} store={store} />
        <TextBox
          label="설명"
          value={description}
          onChange={(value) => store.changeDescription(value)}
        />
        <CheckBox
          label="감추기"
          checked={hidden}
          onChange={() => store.toggleHidden()}
        />
        <Button type="submit" disabled={!valid}>
          변경
        </Button>
        {error && (
          <p>상품 수정 실패</p>
        )}
      </form>
    </Container>
  );
}
```

```js
import React, { useRef } from 'react';

import styled from 'styled-components';

const Container = styled.div`
  margin-block: .5rem;

  label,
  input {
    vertical-align: middle;
  }

  label {
    display: inline-block;
    width: 1rem;
    padding-right: .5rem;
    text-align: right;
  }
`;

type CheckBox = {
  label: string;
  checked: boolean;
  onChange: (checked: boolean) => void;
}

export default function CheckBox({
  label, checked, onChange,
}: CheckBox) {
  const id = useRef(`checkbox-${Math.random().toString().slice(2)}`);

  const handleChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    onChange(event.target.checked);
  };

  return (
    <Container>
      <label htmlFor={id.current}>
        {label}
      </label>
      <input
        id={id.current}
        type="checkbox"
        checked={checked}
        onChange={handleChange}
      />
    </Container>
  );
}
```

## 테스트

모든 구현이 끝났으면 최종적으로 E2E 테스트를 진행하여 기능이 잘 동작하는지 체크하자 !
