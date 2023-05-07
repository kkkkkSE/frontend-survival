# 장바구니 상품 담기

장바구니에 상품을 담는다는 것은 어떤 의미인가? 말그대로 상품이 장바구니 안에 들어간다의 느낌이 아니라, 상품을 주문하기 위해 선택한 옵션과 수량, 가격이 포함되어 있는 주문서를 만드는 느낌에 더 가깝다.

사용자가 선택한 구성을 장바구니 페이지에서 보여주기 위해 복잡하고 많은 로직이 들어갈 수 있는데, 이는 백엔드와 충분히 소통하여 어디까지 로직을 처리하고 어떤 방식으로 데이터를 받아 사용할건지 결정해야 한다.

이번 강의에서는 그런 복잡한 로직은 백엔드에서 담당하고, 프론트엔드에서는 상품의 옵션과 수량을 컨트롤 하는 것에 집중했다.

1. 옵션 보여주기 + 옵션 선택
2. 수량 선택 + 수량에 맞는 가격 보여주기
3. 장바구니 담기 기능 -> 성공 시 메세지 띄우기

## 구현할 페이지 준비

상품 상세 페이지에서 만들어뒀던 `AddToCartForm` 컴포넌트에서 위의 기능을 구현한 컴포넌트를 모두 담아놓을 예정이다.

상품 상세 페이지에서 연결되는 부분이니 `/product-detail/form`에서 관련 컴포넌트를 관리하자.

```js
export default function AddToCartForm() {
  return (
    <div>
      <Options />
      <Quantity />
      <Price />
      <SubmitButton />
    </div>
  );
}
```

## 수량 선택하기

비즈니스 로직을 수행할 스토어를 만들자.

```js
import { singleton } from 'tsyringe';
import { Action, Store } from 'usestore-ts';

@singleton()
@Store()
export default class ProductFormStore {
  quantity = 1;

  @Action()
  changeQuantity(quantity: number) {
    if (quantity <= 0) {
      return;
    }
    if (quantity > 10) {
      return;
    }
    this.quantity = quantity;
  }
}
```

스토어를 만들었다면 스토어 객체를 생성할 수 있게 커스텀 훅을 만들고, 뷰를 만들어주자.

```js
import { container } from 'tsyringe';
import { useStore } from 'usestore-ts';

import ProductFormStore from '../stores/ProductFormStore';

export default function useProductFormStore() {
  const store = container.resolve(ProductFormStore);
  return useStore(store);
}
```

```js
import styled from 'styled-components';

import useProductFormStore from '../../../hooks/useProductFormStore';

import Button from '../../ui/Button';

const Container = styled.div`
  input{
    width: 5rem;
  }
`;

export default function Quantity() {
  const [{ quantity }, store] = useProductFormStore();

  const handleClickDecrease = () => {
    store.changeQuantity(quantity - 1);
  };

  const handleClickIncrease = () => {
    store.changeQuantity(quantity + 1);
  };

  return (
    <Container>
      <Button onClick={handleClickDecrease}>
        -
      </Button>
      <input type="text" value={quantity} readOnly />
      <Button onClick={handleClickIncrease}>
        +
      </Button>
    </Container>
  );
}
```

## 가격 표시

```js
import styled from 'styled-components';

import useProductDetailStore from '../../../hooks/useProductDetailStore';
import useProductFormStore from '../../../hooks/useProductFormStore';

import numberFormat from '../../../utils/numberFormat';

const Container = styled.div`
    margin-block: 0.5rem;
`;

export default function Price() {
  const [{ product }] = useProductDetailStore();
  const [{ quantity }] = useProductFormStore();

  return (
    <Container>
      {numberFormat(product.price * quantity)}
      원
    </Container>
  );
}
```

상품에 대한 가격 정보와 수량 정보를 각 스토어 객체를 통해서 받은 후 곱해서 출력할 수 있다.

만약 이렇게 비즈니스 로직이 포함되어 있는게 불편하다면, `ProductFormStore`에서 금액을 계산하는 메서드나 getter를 만들어 사용해줘도 된다.

```js
export default function Price() {
  const [{ product }] = useProductDetailStore();
  const [, productFormStore] = useProductFormStore();

  useEffect(() => {
    productFormStore.setProduct(product);
  }, [product, productFormStore]);

  return (
    <Container>
      {numberFormat(productFormStore.price)}
      원
    </Container>
  );
}
```

```js
export default class ProductFormStore {
  product: ProductDetail = nullProductDetail;

  // ...

  @Action()
  setProduct(product: ProductDetail) {
    this.product = product;
  }

  get price() {
    return this.product.price * this.quantity;
  }
}
```

만약 상품 정보가 바뀌게 되면 가격 정보가 바뀔 수 있으므로 의존성 배열에 `product`도 추가했다. 하지만 해당 컴포넌트에서 `product`의 변화를 감지한다는 것이 구조상, 흐름상 어색하게 느껴지니 상위 컴포넌트(page 컴포넌트 등)에서 처리해주는 것이 더 좋을 것 같다.

## 장바구니 추가 버튼

장바구니에 상품을 추가하려면 POST 요청을 보내야한다. 요청을 보내기 위해 필요한 상품 id와 옵션, 사용자가 선택한 옵션, 수량 정보를 저장할 상태를 추가하고, 요청이 완료되면 성공 메세지를 띄우기 위한 트리거도 상태에 추가한다. 이는 `setProduct`에 의해 초기화된다.

> 가격 표시 컴포넌트에서 `ProductDetailStore`에 의해 상품 상세 페이지에 처음 접근하면 자동으로 `setProduct`를 실행한다. 가격 표시 컴포넌트만 구현했을 때는 몰랐지만, `setProduct`에 `addToCart`를 위해 상태들을 초기화하는 로직이 추가되는 순간 `setProduct`가 실행되는 구간이 많이 어색하다는 사실을 알 수 있다.
강의에서는 `setProduct` 실행하는 위치를 `useFetchProduct` 안으로 옮겨줬지만, 개인 선호도에 따라 Page 컴포넌트 등으로 이동시킬 수 있다.

또한 `addToCart` 내에서 POST 요청에 필요한 데이터 형태로 가공 후 요청하고, 일부 상태를 필요한 값으로 바꿔준다.

```js
@singleton()
@Store()
export default class ProductFormStore {
  product: ProductDetail = nullProductDetail;

  productId = '';

  options: ProductOption[] = [];

  selectedOptionItems: ProductOptionItem[] = [];

  quantity = 1;

  done = false;

  async addToCart() {
    this.resetDone();

    await apiService.addProductToCart({
      productId: this.productId,
      options: this.options.map((option, index) => ({
        id: option.id,
        itemId: this.selectedOptionItems[index].id,
      })),
      quantity: this.quantity,
    });

    this.complete();
  }

  @Action()
  setProduct(product: ProductDetail) {
    this.product = product;
    this.productId = product.id;
    this.options = product.options;
    this.selectedOptionItems = this.options.map((i) => i.items[0]);
    this.quantity = 1;
    this.done = false;
  }

  @Action()
  changeQuantity(quantity: number) {
    // ...
  }

  @Action()
  resetDone() {
    this.done = false;
  }

  @Action()
  complete() {
    this.quantity = 1; // 장바구니 담으면 초기화
    this.done = true;
  }

  get price() {
    return this.product.price * this.quantity;
  }
}
```

```js
export default class ApiService {
  private instance = axios.create({
    baseURL: API_BASE_URL,
  });

  // ...

  async addProductToCart({ productId, options, quantity }: {
    productId: string;
    options: {
      id: string;
      itemId: string;
    }[];
    quantity: number;
  }): Promise<void> {
    await this.instance.post('/cart/line-items', {
      productId, options, quantity,
    });
  }
}
```

장바구니 담기 버튼은 이전에 버튼을 만들어둔걸 활용하자.

```js
export default function SubmitButton() {
  const [{ done }, store] = useProductFormStore();

  const handleClick = () => {
    store.addToCart();
  };

  if (done) {
    return (
      <p>장바구니에 담았습니다</p>
    );
  }

  return (
    <Button onClick={handleClick}>
      장바구니에 담기
    </Button>
  );
}
```

## 옵션 선택하기

옵션 선택은 `Options` 컴포넌트에서 이루어진다. 해당 컴포넌트에서 다루는 핵심은 옵션이 변경되면 옵션 변경에 필요한 옵션 id와 선택한 옵션 아이템 id를 얻어 스토어 내의 `selectedOptionItems`를 변경시키는 것이다.

> 참고로 상품 상세 페이지가 로드됐을 때, `setProduct`에 의해 `selectedOptionItems`는 모두 첫번째 옵션으로 초기화되어 있다.

```js
import useProductFormStore from '../../../hooks/useProductFormStore';

import { ChangeFunction } from './type';

import Option from './Option';

export default function Options() {
  const [{ product, selectedOptionItems }, store] = useProductFormStore();

  const handleChange: ChangeFunction = ({ optionId, optionItemId }) => {
    store.changeOptionItem({ optionId, optionItemId });
  };

  return (
    <div>
      {product.options.map((option, index) => (
        <Option
          key={option.id}
          option={option}
          selectedItem={selectedOptionItems[index]}
          onChange={handleChange}
        />
      ))}
    </div>
  );
}
```

```js
import { ProductOption, ProductOptionItem } from '../../../types';
import { ChangeFunction } from './type';

import ComboBox from '../../ui/ComboBox';

interface OptionProps{
    option: ProductOption;
    selectedItem: ProductOptionItem;
    onChange: ChangeFunction;
  }

export default function Option({
  option, selectedItem, onChange,
}: OptionProps) {
  const handleChange = (item: ProductOptionItem | null) => {
    if (!item) {
      return;
    }

    onChange({
      optionId: option.id,
      optionItemId: item.id,
    });
  };

  return (
    <ComboBox
      label={option.name}
      selectedItem={selectedItem}
      items={option.items}
      itemToId={(item) => item.id}
      itemToText={(item) => item.name}
      onChange={handleChange}
    />
  );
}
```

```js
import { useRef } from 'react';

import styled from 'styled-components';

const Container = styled.div`
  label {
    margin-right: .5rem;
  }
`;

type ComboBoxProps<T> = {
  label: string;
  selectedItem: T;
  items: T[];
  itemToId: (item: T) => string;
  itemToText: (item: T) => string;
  onChange: (item: T | null) => void;
}

export default function ComboBox<T>({
  label, selectedItem, items, itemToId, itemToText, onChange,
}: ComboBoxProps<T>) {
  const id = useRef(`combobox-${Math.random().toString().slice(2)}`);

  const handleChange = (event: React.ChangeEvent<HTMLSelectElement>) => {
    const { value } = event.target;
    const selected = items.find((item) => itemToId(item) === value);
    onChange(selected ?? null);
  };

  return (
    <Container>
      <label htmlFor={id.current}>
        {label}
      </label>
      <select
        id={id.current}
        onChange={handleChange}
        value={itemToId(selectedItem)}
      >
        {items.map((item) => (
          <option key={itemToId(item)} value={itemToId(item)}>
            {itemToText(item)}
          </option>
        ))}
      </select>
    </Container>
  );
}
```

```js
export default class ProductFormStore {
  // ...

  @Action()
  changeOptionItem({ optionId, optionItemId }: {
    optionId: string;
    optionItemId: string;
  }) {
    this.selectedOptionItems = this.product.options.map((option, index) => {
      const item = this.selectedOptionItems[index];

      return option.id !== optionId
        ? item
        : option.items.find((i) => i.id === optionItemId) ?? item;
    });
  }

  // ...
}
```

콤보박스에서 옵션 값을 변경하면 변경된 옵션을 상위 컴포넌트로 전달하며, `Options` 컴포넌트로 전달될 때 옵션 id와 선택된 옵션 아이템 id로 바뀌어 전달된다.

이 두가지 데이터로 스토어의 `changeOptionItem`가 실행된다. 기존의 `selectedOptionItems`을 변경하기 위해 상품의 모든 옵션이 담긴 `product.options`를 map으로 순회하며 새로운 `selectedOptionItems`를 반환시킨다.

예를 들면, 

1. 초기화 된 옵션이 `색상 - 블랙`, `사이즈 - S` 였고
2. 사용자가 색상을 레드로 변경했다면, 
3. `Options`은 *`색상` 옵션의 고유 id*와 *`레드` 옵션 값의 고유 id*를 받아 `changeOptionItem`를 실행한다.
4. 그러면 내부에서 상품의 전체 옵션에 대해 각각 체크하면서 사용자가 변경한 옵션인지 확인하고,
5. 사용자가 변경한 옵션을 만나면 `색상 - 레드`를 `selectedOptionItems`에 저장시킨다.

## Test

### 테스트 항목

- 버튼 클릭 시 수량 변경이 제대로 이뤄지는지, 수량 제한이 제대로 됐는지
- 수량에 따라 가격이 맞게 변하는지
- 장바구니 담기 버튼이 잘 활성화되어 있는지
- 장바구니 담기 기능과 담은 후 메세지
- 상품 옵션 변경이 제대로 되는지

### 비즈니스 로직 테스트

위 코드에서 스토어는 비즈니스 로직을 분리하기 위한 용도로 사용되기도 했는데, 이 때 비즈니스 로직에 대해 따로 테스트해주면 좋다. 컴포넌트 사용자 상호작용에 대한 테스트를 하는 것보다 더 간단하게 테스트 할 수 있다.

```js
describe('ProductFormStore', () => {
  let store: ProductFormStore;

  beforeEach(() => {
    store = new ProductFormStore();
  });

  describe('changeQuantity', () => {
    context('with correct value', () => {
      it('changes quantity', () => {
        store.changeQuantity(3);

        expect(store.quantity).toBe(3);
      });
    });

    context('with incorrect value', () => {
      it("doesn't changes quantity", () => {
        store.changeQuantity(-1);
        store.changeQuantity(11);

        expect(store.quantity).toBe(1);
      });
    });
  });
});
```

만약 테스트를 할 수 있는 양 또는 시간에 한계가 있다면, 비즈니스 로직에 가까운 부분을 테스트하는 것을 추천한다.

### 테스트에서 가격 반영하기

테스트할 때 fixture를 가져와 쓰곤 하는데, 가격 계산 기능 같은 경우 어떤 상품이냐에 따라 가짜 데이터에서의 가격과 실제 화면에서 도출된 가격이 완전히 같지 않을 수도 있다. 이 때, 테스트 로직을 다음과 같이 변경해줄 수 있다.

- fixture에 만들어놓은 값을 테스트 코드에 반영한다.
- 실제로 사용한 스토어 객체에서 값을 직접 받아와 변수에 저장하여 해당 변수를 테스트 코드에 반영한다.
