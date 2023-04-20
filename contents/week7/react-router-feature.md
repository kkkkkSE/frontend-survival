# React Router Feature

### 클라이언트 측 라우팅

서버에 별다른 요청 없이도 URL을 업데이트하여 적절한 화면을 나타낼 수 있다.

### 중첩 라우트

중첩 구조로 사용하여 하위 경로도 지정할 수 있다.

```js
<Route path="/" element={<Root />}>
    <Route path="contact" element={<Contact />} />
    <Route path="dashboard" element={<Dashboard />} />
    <Route element={<AuthLayout />}>
        <Route path="login" element={<Login />} />
        <Route path="logout" />
    </Route>
</Route>
```

### 동적 세그먼트

세그먼트란, URL의 경로 부분에서 '/'로 나누어진 각각의 경로들을 의미한다. 동적 세그먼트로 유동적인 경로를 사용할 수 있다.

동적 세그먼트를 이용하면, `useParams()`로 동적 세그먼트에 대해 적용된 값을 key/value 쌍으로 얻을 수 있다.

```js
// App.jsx
<Route path="user/:userId" element={<UserInfo />} />
```

```js
// UserList.jsx
import { Link } from "react-router-dom";

function UserListRow({ user }) {
  return(
    // ...
    <Link to={`/user/${user.id}`}>유저 정보 보기</Link>
    // ...
  )
}
export default List;
```

```js
// UserInfo.jsx
import React from 'react';
import { useParams } from 'react-router-dom';

function UserInfo() {
  const { userId } = useParams();

  return (
    // ...
   )
}
```

### 라우트 순위 매칭

어떤 URL 주소에 대해서 이동 가능한 라우트가 2개 이상일 경우, 리액트 라우터 자체에서 우선순위를 매겨 구체적으로 일치하는 경로를 선택한다.

```js
<Route path="/teams/:teamId" loader={ ... } />
<Route path="/teams/new" loader={ ... } />
```

```js
<Link to={`/teams/new`}>링크</Link> // 둘 중 아래 라우트 적용
```

### Link 상대 경로 지정

```js
// App.jsx
<Route path="home" element={<Home />}>
  <Route path="project/:projectId" element={<Project />}>
  </Route>
</Route>
```

```js
// Home.jsx
<Home>
  <Project />
</Home>
```

```js
// Project.jsx @ /home/project/123
<Link to="abc"> {/* 이동 시 경로 : /home/project/123/abc */}
<Link to=".."> {/* 이동 시 경로 : /home */}
```

### 데이터 Load

라우트에는 비동기 함수를 포함한 함수에서 데이터를 받아오는 기능도 있다. 만약 중첩 라우트 구조에서 상위 경로와 하위 경로가 모두 비동기 데이터를 받아온다면, 병렬로 데이터를 받아온다.

```js
<Route
  path="/"
  loader={async ({ request }) => {
    const res = await fetch("/api/user.json", {
      signal: request.signal,
    });
    const user = await res.json();
    return user;
  }}
  element={<Root />}
>
  <Route
    path=":teamId"
    loader={({ params }) => {
      return fetch(`/api/teams/${params.teamId}`);
    }}
    element={<Team />}
  >
  </Route>
</Route>
```

`loader`에서 fetch 함수 자체를 반환시키면 `loader`가 자동으로 Response 객체에서 JSON 데이터를 추출한다. 따라서 위 예제의 두 라우트 중 아래에 있는 라우트처럼 사용할수도 있다.

```js
function Root() {
  const user = useLoaderData();
}

function Team() {
  const team = useLoaderData();
}
```

받아온 데이터는 이동한 컴포넌트에서 `useLoaderData`로 받아 사용할 수 있다.

### 리디렉트

`loader` 안에서 상황에 따라 유용하게 다른 페이지로 이동할 수 있다.

```js
<Route
  path="dashboard"
  loader={async () => {
    const user = await fake.getUser();
    if (!user) {
      throw redirect("/login");
    }

    const stats = await fake.getDashboardStats();
    return { user, stats };
  }}
/>
<Route
  path="project/new"
  action={async ({ request }) => {
    const data = await request.formData();
    const newProject = await createProject(data);
    return redirect(`/projects/${newProject.id}`);
  }}
/>
```

### Pending 상태 체크

`useNavigation()`을 이용하여 페이지의 Pending 상태를 받아올 수 있다. `state`라는 프로퍼티에서 상태를 읽어올 수 있으며, 상태의 종류는 다음과 같다.

* idle : Pending 상태 아님
* submitting : POST, PUT, PATCH, DELETE로 인해 라우팅이 호출되어 완료하기 전까지의 지연 상태
* loading : 다른 경로의 페이지를 불러와 완료하기 전까지의 지연 상태

```js
function Root() {
  const navigation = useNavigation();
  return (
    <div>
      <Header />
      {navigation.state === "loading" && <Main />}
      <Footer />
    </div>
  );
}
```
