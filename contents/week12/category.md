# 카테고리 관리

사용자 관리와 다르게 카테고리 관리에서는 카테고리를 불러오고, 수정하는 것까지 해볼 예정이다.

카테고리를 받아오는 과정은 사용자 목록을 받아올 때와 거의 같다. 이 부분은 빠르게 넘어가보자.

## 카테고리 목록 페이지

사용자 웹에서는 카테고리를 얻어 헤더에 뿌려주기만 했지만, 관리자 웹에서는 이 카테고리를 추가, 수정하는 기능이 포함되어야 한다. 또한 만들어놓은 카테고리에 대해 사용자에게 보여줄건지, 보이지 않게 할건지도 설정할 수 있도록 hidden이라는 프로퍼티가 추가됐다.

```js
import { Link } from 'react-router-dom';

import styled from 'styled-components';

import useFetchCategories from '../hooks/useFetchCategories';

const Container = styled.div`
  // ...
`;

export default function CategoryListPage() {
  const { categories, loading, error } = useFetchCategories();

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
      <h2>Categories</h2>
      <table>
        <thead>
          <tr>
            <th>이름</th>
            <th>표시</th>
            <th>행동</th>
          </tr>
        </thead>
        <tbody>
          {categories.map((category) => (
            <tr key={category.id}>
              <td>{category.name}</td>
              <td>{category.hidden ? '감춤' : '보임'}</td>
              <td>
                <Link to={`/categories/${category.id}/edit`}>수정</Link>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
      <Link to="/categories/new">
        카테고리 추가
      </Link>
    </Container>
  );
}
```

## 카테고리 목록 받아오기

```js
import useFetch from './useFetch';

import { Category } from '../types';

export default function useFetchCategories() {
  interface Data {
    categories: Category[];
  }

  const { data, error, loading } = useFetch<Data>('/categories');

  return {
    categories: data?.categories ?? [],
    error,
    loading,
  };
}
```

## 카테고리 추가하기

우선 카테고리 추가 요청을 보내기 위한 코드부터 작성하자.

데이터를 받아오는 과정은 모두 커스텀 훅에 넣어 숨겼던 것처럼, 카테고리를 호출하는 것은 커스텀 훅에게 맡기고 카테고리 추가 페이지 컴포넌트에선 필요할 때 커스텀 훅에 지시하기만 하자.

그리고 카테고리를 추가하게 되면 카테고리를 다시 받아와야 하는데, 데이터를 받아오는 것은 SWR가 담당하고 있으니 카테고리 추가 후 강제로 다시 카테고리를 받아오게 하자. [공식 문서](https://swr.vercel.app/ko/docs/mutation#bound-mutate)에 안내되어 있다. mutate를 사용하면 캐시를 초기화 시키면서 데이터를 다시 받아온다.

```js
import useFetch from './useFetch';

import { Category } from '../types';

import { apiService } from '../services/ApiService';

export default function useFetchCategories() {
  interface Data {
    categories: Category[];
  }

  const {
    data, error, loading, mutate,
  } = useFetch<Data>('/categories');

  return {
    categories: data?.categories ?? [],
    error,
    loading,
    async createCategory({ name }: {
      name: string;
    }) {
      await apiService.createCategory({ name });
      mutate();
    },
  };
}
```

```js
export default class ApiService {

  // ...

  async createCategory({ name } : { name : string }) : Promise<void> {
    await this.instance.post('/categories', { name });
  }
}
```

그리고 카테고리 추가 화면 구성하기

```js
import { useNavigate } from 'react-router-dom';

import { Controller, useForm } from 'react-hook-form';

import styled from 'styled-components';

import Button from '../components/ui/Button';

import useFetchCategories from '../hooks/useFetchCategories';

const Container = styled.div`
  h2 {
    margin-bottom: 2rem;
    font-size: 2rem;
  }

  [type=submit] {
    display: block;
    margin-block: 1rem;
  }
`;

export default function CategoryNewPage() {
  const navigate = useNavigate();

  const { createCategory } = useFetchCategories();

  type FormValues = {
    name: string;
  };

  const { handleSubmit, control } = useForm<FormValues>();

  const onSubmit = async (data: FormValues) => {
    await createCategory({
      name: data.name,
    });
    navigate('/categories');
  };

  return (
    <Container>
      <h2>New Category</h2>
      <form onSubmit={handleSubmit(onSubmit)}>
        <Controller
          control={control}
          name="name"
          defaultValue=""
          render={({ field: { onChange, value } }) => (
            <div>
              <label htmlFor="input-name">이름</label>
              <input
                id="input-name"
                value={value}
                onChange={onChange}
              />
            </div>
          )}
        />
        <Button type="submit">
          등록
        </Button>
      </form>
    </Container>
  );
}
```

`Controller`는 React Hook Form에서 제공하는 컴포넌트로, 일반적인 HTML 요소가 아닌 다른 UI 컴포넌트(외부 라이브러리에서 온 컴포넌트도 포함)를 React Hook Form이 제어하도록 할 때 사용한다.

- control : 해당 UI 컴포넌트를 사용할거라면 반드시 써줘야 함
- name : Form에서 제어할 data 객체의 프로퍼티명
- defaultValue : default value
- render : 실제로 렌더링할 부분, 함수 형태로 전달
  - { field } : render 함수의 인자로 넘길 수 있는 부분으로, field 객체 안에 onChange, value, ref 등 input과 같은 요소가 가질 수 있는 속성들이 포함되어 있다. 각 속성에 대하여 렌더링 될 외부 UI 컴포넌트에 전달하여 사용할 수 있다.

강의에서는 (`TextBox`와 같은) 외부 UI 컴포넌트를 사용하진 않았지만, 아마 `Controller`를 사용하는 예시를 보여주기 위해 사용한 것 같다. `Controller`없이 사용하려면 다음과 같이 코드를 짤 수도 있다.

```js
/* eslint-disable react/jsx-props-no-spreading */

export default function CategoryNewPage() {
  const navigate = useNavigate();

  const { createCategory } = useFetchCategories();

  type FormValues = {
    name: string;
  };

  const { register, handleSubmit } = useForm<FormValues>();

  const onSubmit = async (data: FormValues) => {
    await createCategory({
      name: data.name,
    });
    navigate('/categories');
  };

  return (
    <Container>
      <h2>New Category</h2>
      <form onSubmit={handleSubmit(onSubmit)}>
        <label htmlFor="input-name">이름</label>
        <input {...register('name')} id="input-name" />
        <Button type="submit">
          등록
        </Button>
      </form>
    </Container>
  );
}
```

## 카테고리 수정하기

수정 페이지에 들어오면 기존의 카테고리 정보를 받아와야 한다. GET 요청이므로 SWR를 이용해서 데이터를 받아오자.

카테고리 정보를 수정하는 건 PATCH 요청이므로 ApiService에 따로 작성해주고, 카테고리를 생성할 때처럼 커스텀 훅에서 해당 함수를 내보내서 원할 때 업데이트 되도록 해주자. 변경 사항이 반영될 수 있게 mutate 해주기.

```js
export default class ApiService {

  // ...

  async updateCategory({ categoryId, name, hidden }: {
    categoryId : string;
    name: string;
    hidden: boolean;
  }): Promise<void> {
    await this.instance.patch(`/categories/${categoryId}`, {
      name, hidden,
    });
  }
}

```

```js
import useFetch from './useFetch';

import { apiService } from '../services/ApiService';

import { Category } from '../types';

export default function useFetchCategory({ categoryId }: {
  categoryId: string;
}) {
  const url = `/categories/${categoryId}`;

  const {
    data, error, loading, mutate,
  } = useFetch<Category>(url);

  return {
    category: data,
    error,
    loading,
    async updateCategory({ name, hidden }: {
      name: string;
      hidden: boolean;
    }) {
      await apiService.updateCategory({ categoryId, name, hidden });
      mutate();
    },
  };
}
```

카테고리 수정 페이지 구현

```js
/* eslint-disable react/jsx-props-no-spreading */
import { useNavigate, useParams } from 'react-router-dom';

import { useForm } from 'react-hook-form';

import styled from 'styled-components';

import Button from '../components/ui/Button';

import useFetchCategory from '../hooks/useFetchCategory';

const Container = styled.div`
  h2 {
    margin-bottom: 2rem;
    font-size: 2rem;
  }

  [type=submit] {
    display: block;
    margin-block: 1rem;
  }
`;

export default function CategoryEditPage() {
  const params = useParams();

  const categoryId = String(params.id);

  const navigate = useNavigate();

  const {
    category, loading, error, updateCategory,
  } = useFetchCategory({ categoryId });

  type FormValues = {
    name: string;
    hidden: boolean;
  };

  const { register, handleSubmit } = useForm<FormValues>();

  const onSubmit = async (data: FormValues) => {
    await updateCategory({
      name: data.name,
      hidden: data.hidden,
    });
    navigate('/categories');
  };

  if (loading) {
    return (
      <p>Loading...</p>
    );
  }

  if (error || !category) {
    return (
      <p>Error!</p>
    );
  }

  return (
    <Container>
      <h2>Edit Category</h2>
      <form onSubmit={handleSubmit(onSubmit)}>
        <div>
          <label htmlFor="input-name">이름</label>
          <input
            {...register('name')}
            id="input-name"
            defaultValue={category.name}
          />
        </div>
        <div>
          <label htmlFor="input-hidden">감춤</label>
          <input
            {...register('hidden')}
            id="input-hidden"
            type="checkbox"
            defaultChecked={category.hidden}
          />
        </div>
        <Button type="submit">
          수정
        </Button>
      </form>
    </Container>
  );
}
```
