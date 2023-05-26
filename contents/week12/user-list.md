---
description: SWR로 GET 요청 보내기
---

# 사용자 목록

사용자를 관리하는 것은 어디까지 관리하냐에 따라 많이 복잡해질 수도 있다. 사용자의 개인 정보를 다루는 것이 문제가 될 수 있으면서도 고객 지원 업무에 도움이 되기도 하기 때문에 사용자 관리는 어디까지, 어떻게 할 지 많이 고려해야 한다. 강의에서는 단순히 사용자 목록 정도만 확인하기로 했다.

기존 방식으로 사용자 목록을 불러왔다면 다음과 같은 과정을 거쳤을 것이다.

1. ApiService에서 Axios를 통해 API 호출하는 코드를 작성하고
2. 유저 관련 스토어를 만들어 API 호출 + 사용자 목록을 담을 상태를 만든다.
3. 커스텀 훅에서 유저 관련 스토어 객체를 만들어 반환시킨 다음
4. 컴포넌트에서 커스텀 훅을 통해 스토어 객체에 있는 사용자 목록을 사용했을 것이다.

이 과정이 SWR을 사용하면 얼마나 간단해질지 한 번 확인해보자.

## 구현할 페이지 준비

```js
import styled from 'styled-components';

import useFetchUsers from '../hooks/useFetchUsers';

const Container = styled.div`
  h2 {
    margin-bottom: 2rem;
    font-size: 2rem;
  }

  table {
    th, td {
      padding: .5rem;
    }
  }
`;

export default function UserListPage() {
  const { users, loading, error } = useFetchUsers();

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
      <h2>Users</h2>
      <table>
        <thead>
          <tr>
            <th>이름</th>
            <th>이메일</th>
            <th>역할</th>
          </tr>
        </thead>
        <tbody>
          {users.map((user) => (
            <tr key={user.id}>
              <td>{user.name}</td>
              <td>{user.email}</td>
              <td>{user.role}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </Container>
  );
}
```

스토어 객체에서 받아왔던 것과 동일하게 커스텀 훅을 거쳐서 데이터를 받아오도록 한다.

## 사용자 목록 받아오기

SWR를 이용해 READ 요청을 하기 위해 커스텀 훅(`useFetch`) 내에서 `useSWR` 훅을 만들어준다. 요청할 URL의 Path와 fetcher를 전달해주면 훅 내부에서 데이터를 요청하고, 데이터와 에러, 로딩 상태도 같이 내보내준다.

참고로 mutate는 강제로 캐시를 초기화하여 데이터를 불러오게 하는 함수로, 데이터가 수정되거나 삭제했을 때 사용한다. 매번 사용하는건 아니고, 카테고리에서 사용해볼 예정이다.

```js
import useSWR from 'swr';

import { apiService } from '../services/ApiService';

const API_BASE_URL = process.env.API_BASE_URL || 'http://localhost:3000/admin';

export default function useFetch<Data>(path: string) {
  const url = `${API_BASE_URL}${path}`;

  const {
    data, error, isLoading, mutate,
  } = useSWR<Data>(url, apiService.fetcher());

  return {
    data,
    error,
    loading: isLoading,
    mutate,
  };
}
```

```js
export default class ApiService {

  // ...

  fetcher() {
    return async (url: string) => {
      const { data } = await this.instance.get(url);
      return data;
    };
  }
}
```

이제 `useFetch`를 가지고 사용자 목록을 가져오는 커스텀 훅을 작성하자.

```js
import useFetch from './useFetch';

import { User } from '../types';

export default function useFetchUsers() {
  interface Data {
    users: User[];
  }

  const { data, error, loading } = useFetch<Data>('/users');

  return {
    users: data?.users ?? [],
    error,
    loading,
  };
}
```

EZ.
