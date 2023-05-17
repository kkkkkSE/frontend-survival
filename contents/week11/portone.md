# 주문하기

## 포트원 결제 요청

PG란 Payment Gateway의 약자로 전자결제대행사, 즉 온라인에서 결제 수단을 제공하는 서비스를 제공하는 회사이다. 카드사와 가맹점 사이에서 카드 결제의 승인, 취소, 정산을 처리하는 것이다.

[포트원](https://portone.io/korea/ko)은 여러 PG사를 하나의 API로 사용할 수 있게 해주는 통합 결제 솔루션이다. 요새는 카드결제 뿐만 아니라 토스나 네이버페이 등 다른 결제 시스템을 사용하는 사람도 많아서 다양한 방법으로 결제할 수 있게끔 할 수 있다.

보통 카드결제 시스템을 붙이려면 사업자 신분이어야 하지만, 포트원을 사용하면 비용을 지불하지 않아도 되고, 또 사업자가 아니어도 테스트 용도로 사용할 수 있어서 지금 우리가 써보기에 적절하다.

## 결제 방식

- 인증 결제 : 카드 정보와 함께 카드 소지자의 개인정보를 입력해야 하는 결제 방식. 보안이 안전한 편이다.
- 비인증 결제 : 카드 정보만으로 결제하는 방식. 편리하지만 보안이 취약할 수 있다.

강의에서는 인증 결제 시스템을 사용한다.

## 결제 프로세스

1. 결제 요청 : 결제창이 떠서 정보를 입력하고 제출하여 결제 요청 (F/E)
2. 결제 결과 확인 : 결제 처리 후 결제 요청에 대한 결과를 받음 (F/E)
3. 결제 금액 위변조 검증 : 프론트에서 백엔드로 결제 결과를 전송하여 위변조 검증 (B/E)
4. 결제 완료 : 백엔드에서 위변조 검증 후 통과하면 주문 완료 페이지로 이동

## 구현 전 준비사항

- PG상점 아이디
- 가맹점 식별코드

포트원에 로그인하여 [결제 연동]을 클릭하면, 결제 대행사를 추가할 수 있다. 원하는 결제 대행사를 선택하여 추가 버튼을 누르면 기타 정보를 입력하는 창이 나오는데, 테스트 용으로 사용할 경우 PG상점 아이디가 자동으로 채워져있지만 실사용할 땐 직접 넣어줘야하니 참고하자. 개발할 때 사용되는 항목이다.

PG사로 카카오페이를 선택하면 부담없이 테스트할 수 있다며 강의에서 강력히 추천했다.

[상점, 계정 관리]-[내 식별코드, API Keys]를 클릭하면 가맹점 식별코드가 나오는데, 이것도 개발할 때 사용되는 항목이다.

## 결제 구현

포트원 V2가 나왔지만 아직 베타라서 강의에선 [V1](https://developers.portone.io/docs/ko/sdk/javascript-sdk/readme)를 사용한다. 나중에 V2가 안정화되어 변경하고 싶다면 [공식 문서](https://developers.portone.io/docs/ko/v2-payment/v2)를 참고하여 바꿔주자.

인증결제 방식을 구현하기 위해 [문서](https://developers.portone.io/docs/ko/auth/guide/readme)에 나와있는 순서대로 따라가보자.

1. V2는 npm에서 설치할 수 있지만 V1은 스크립트 태그로 라이브러리를 불러와야 한다.

    ```js
    <script src="https://cdn.iamport.kr/v1/iamport.js"></script>
    ```

2. 라이브러리를 가져오면 `window`객체에 `IMP`라는 객체가 생기는데, `IMP`는 아임포트를 나타내며 이는 포트원의 바뀌기 전 이름이다. 이 객체를 공식 문서에 나와있는 것처럼 초기화해주면 문제가 생긴다. 스크립트 태크를 이용해 라이브러리를 가져왔기 때문에 타입 정보가 필요하기 때문이다.

    우편번호 서비스를 사용할 때 타입 정보가 없어 직접 정의해줬던 것처럼 해줄수도 있지만, 강의에서는 `Reflect`를 이용해 강제로 `IMP`를 가져와 초기화시켰다.

    ```js
    Reflect.get(window, 'IMP').init('가맹점_식별코드');
    ```

    공식 문서에 따르면, 이는 단 한번만 이루어져야 하므로 두 번 이상 실행되지 않는 `main`(`index.html` 파일과 직접적으로 연결된 ts파일)에 코드를 심어줬다.

    {% hint style="info" %}
    외부인이 보면 안되는 가맹점 식별코드, PG상점 아이디, API Base URL 같은 것은 `.env` 파일에 정의해놓고 환경변수로 사용한다.

    ```text
    API_BASE_URL=https://shop-demo-api-03.fly.dev
    PORTONE_IMP=<가맹점 식별코드>
    PORTONE_PG_CODE=<PG사 코드>.<PG상점아이디>
    ```

    {% endhint %}

    초기화 이후에는 꼭 서버를 다시 시작해주자.

3. 결제창을 생성하는 함수를 만들자. 어떤 항목을 받을 수 있는지, 응답 처리는 어떻게 하는지 등은 문서에 있다. `request_pay`는 비동기 함수로, 문서에서는 콜백함수를 통해 비동기 작업의 결과에 따른 처리를 해주지만, 결제창을 불러올 때 async/await 문을 사용해주기 위해서 콜백함수 대신 Promise를 반환해주는 걸로 변경했다.

    기존에 Axios를 사용했을 때와 마찬가지로 따로 파일을 만들어줬다.

    ```js
    // services/PaymentService.ts
    const PG_CODE = process.env.PORTONE_PG_CODE || '';

    type Product = {
      name: string;
      price: number;
    };

    type Buyer = {
      name: string;
      email: string;
      phoneNumber: string;
      address: string;
      postalCode: string;
    };

    type PaymentResponse = {
      success: boolean;
      error_code: string;
      error_msg: string;
      imp_uid: string | null;
      merchant_uid: string;
    }

    export default class PaymentService {
      // 앞서 말했던 타입 문제로 Reflect.get 으로 IMP 객체 사용
      instance = Reflect.get(window, 'IMP');

      async requestPayment({ merchantId, product, buyer }: {
        merchantId: string;
        product: Product;
        buyer?: Buyer;
      }): Promise<{
        merchantId: string;
        transactionId: string;
      }> {
        return new Promise((resolve, reject) => {
          this.instance.request_pay({
            pg: PG_CODE,
            pay_method: 'card',
            merchant_uid: merchantId,
            name: product.name,
            amount: product.price,
            buyer_email: buyer?.email,
            buyer_name: buyer?.name,
            buyer_tel: buyer?.phoneNumber,
            buyer_addr: buyer?.address,
            buyer_postcode: buyer?.postalCode,
          }, (response: PaymentResponse) => {
            if (response.success) {
              resolve({
                merchantId: response.merchant_uid,
                transactionId: response.imp_uid ?? '',
              });
            } else {
              reject(Error(response.error_msg));
            }
          });
        });
      }
    }

    export const paymentService = new PaymentService();
    ```

    결제에 실패했을 때, PG사에서 전달해주는 에러 메세지를 그대로 사용하기 위해 Error로 에러 메세지를 그대로 전달하고 있다.(한글로 예쁘게 전달해 줌)

    바이어 정보는 강의에서 사용하지 않았지만 사용할 때 위처럼 정의해두면 된다. 이처럼 결제창으로 전달하는 정보같은 경우는 필요한만큼 커스텀해도 된다.

4. 3번을 컴포넌트에서 간편하게 사용할 수 있도록 커스터 훅을 작성하자.

    ```js
    import { paymentService } from '../services/PaymentService';

    import { Cart } from '../types';

    export default function usePayment(cart: Cart) {
      return {
        async requestPayment() {
          const now = new Date();
          const date = now.toISOString().slice(0, 10).replace(/-/g, '');
          const time = now.getTime();
          const nonce = Math.random().toString().slice(-5);
          const merchantId = `ORDER-${date}-${time}${nonce}`;

          const result = await paymentService.requestPayment({
            merchantId,
            product: {
              name: cart.lineItems[0].product.name,
              price: cart.totalPrice,
            },
          });

          return result;
        },
      };
    }
    ```

    `merchantId`는 보통 가맹점이 정한 양식에 따라 다른 주문과 겹치지 않게 작성하면 되고, `product.name`도 모든 상품의 이름을 나타낼 수 없으니 대표 상품만 나타내거나, 대표 상품 외 n건 등으로 작성할 수 있다.

5. 결제 버튼과 결제 시스템을 연동해주자. `OrderForm`에서 바로 처리해줄 수도 있지만, 깔끔하게 분리하기 위해 `PaymentButton`을 만들어줬다.

    ```js
    import { useState } from 'react';
    import { useNavigate } from 'react-router-dom';
    import styled from 'styled-components';

    import useOrderFormStore from '../../hooks/useOrderFormStore';
    import usePayment from '../../hooks/usePayment';

    import { Cart } from '../../types';

    import Button from '../ui/Button';

    const Container = styled.div`
      p {
        margin-block: 2rem;
        color: ${(props) => props.theme.colors.primary};
        font-size: 2rem;
        font-weight: bold;
      }
    `;

    type PaymentButtonProps = {
      cart: Cart;
    };

    export default function PaymentButton({ cart }: PaymentButtonProps) {
      const navigate = useNavigate();

      const [{ valid }] = useOrderFormStore();

      const { requestPayment } = usePayment(cart);

      const [error, setError] = useState('');

      const handleClick = async () => {
        setError('');

        try {
          const { merchantId, transactionId } = await requestPayment();

          // TODO: B/E로 주문 및 결제 정보 전달.

          navigate('/order/complete');
        } catch (e) {
          if (e instanceof Error) {
            setError(e.message);
          }
        }
      };

      return (
        <Container>
          <Button onClick={handleClick} disabled={!valid}>
            결제
          </Button>
          {error && (
            <p>{error}</p>
          )}
        </Container>
      );
    }
    ```

    사용자가 받는 사람의 정보를 입력하고, 유효성 검사에 통과하면(Store에서 이뤄짐) 버튼이 활성화된다. 이제 결제 버튼을 클릭하면 결제창이 나오고, 결제가 정상적으로 이루어지면 결제 성공 페이지로 이동한다. 결제 도중 문제가 생기면 에러 메세지를 받아 화면에 띄워준다. 에러 메세지는 PG사에서 전달해준다.

## 주문 처리

결제창을 띄우고, 결제를 하는 것까지는 완료됐다. 이제 해당 결제 결과를 백엔드에 전달하여 주문을 처리하는 일만 남았다.

주문을 처리하는 API 호출을 위한 메서드를 작성하자.

```js
export default class ApiService {
  // ...

  async createOrder({ receiver, payment }: {
    receiver : {
      name: string;
      address1: string;
      address2: string;
      postalCode: string;
      phoneNumber: string;
    },
    payment : {
      merchantId : string;
      transactionId : string;
    }
  }): Promise<void> {
    await this.instance.post('/orders', {
      receiver, payment,
    });
  }
}
```

주문 처리 API 호출은 주문 관련 스토어였던 `OrderFormStore`에서 함께 처리해주면 좋을 것 같다.

```js
  async order({ merchantId, transactionId }: {
    merchantId: string;
    transactionId: string;
  }) {
    apiService.createOrder({
      receiver: {
        name: this.name,
        address1: this.address1,
        address2: this.address2,
        postalCode: this.zonecode,
        phoneNumber: this.phoneNumber,
      },
      payment: { merchantId, transactionId },
    });
  }
```

```js
export default function PaymentButton({ cart }: PaymentButtonProps) {
  const navigate = useNavigate();

  const [{ valid }, store] = useOrderFormStore();

  const { requestPayment } = usePayment(cart);

  const [error, setError] = useState('');

  const handleClick = async () => {
    setError('');

    try {
      const { merchantId, transactionId } = await requestPayment();

      store.order({ merchantId, transactionId });

      navigate('/order/complete');
    } catch (e) {
      if (e instanceof Error) {
        setError(e.message);
      }
    }
  };

  return (
    <Container>
      <Button onClick={handleClick} disabled={!valid}>
        결제
      </Button>
      {error && (
        <p>{error}</p>
      )}
    </Container>
  );
}
```

이제 주문이 잘 되는 걸 확인할 수 있다!

백엔드에서 주문 완료 처리가 되면, 자동으로 장바구니를 비워줘서 이후에 장바구니를 확인하면 비어있는 것을 확인할 수 있다.

차후에 다른 PG사를 이용해보거나, 다른 정보를 전달하고 처리하는 등의 작업을 연습해보면 좋을 것 같다.
