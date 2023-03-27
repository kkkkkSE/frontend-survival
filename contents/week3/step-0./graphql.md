# GraphQL

GraphQL은 페이스북에서 만든 쿼리 언어다.

보통은 하나의 view를 그리기 위해 여러 번의 API의 호출이 필요하고, 호출로 받은 데이터를 조작, 조합하여 사용해야 했다.

GraphQL은 이 점을 보완하여 원하는대로 정보를 가져올 수 있고, 보다 편하게 정보를 수정할 수 있도록 만들어졌다.

###

## GraphQL 특징

* 전체 API에서 **단 하나의 Endpoint만을 사용**한다.

{% hint style="info" %}
Endpoint : 서버에서 자원(Resource)에 접근할 수 있게 도와주는 URL
{% endhint %}

* 선언적으로 쿼리를 처리하여 **직관적으로** 어떤 데이터를 받을 지 알 수 있다.
* 요청할 때의 **Query 문에 따라 응답의 구조가 달라진다.**

```js
// 요청
{
  content {
    contentId
    contentTitle
    comments {
      commentId
      commentBody
    }
  }
}
```

```js
// 응답
{
  "data": {
    "content": {
      "contentId": "ct1",
      "contentTitle": "title",
      "comments": [
        {
          "commentId": "comment1",
          "commentBody": "comment"
        }
      ]
    }
  }
}
```

위와 같이 요청에서 명시한 데이터만 받을 수 있다.

###

## GraphQL 사용하기

GraphQL에서 사용할 수 있는 메서드는 다음과 같다.

* `Query` : 읽기 전용. 단순 조회로 데이터를 가져오기 위한 메서드(Read)
* `Mutation` : 데이터를 변경한 후 가져오기 위한 메서드(Create, Update, Delete)
* `Subscription` : 실시간으로 변경된 데이터를 가져오기 위한 메서드로, GraphQL에서 서버에 데이터를 구독하면 데이터가 변화됐을 때 알려준다.

{% hint style="info" %}
위 메서드는 기술적 제한으로 반드시 써야된다기 보다는, REST API처럼 개발자들간에 암묵적으로 합의된 역할이다.
{% endhint %}

CRUD를 담당하는 `Query`와 `Mutation`를 구현하기 위해서는 **Schema**와 **Resolver**가 필요하다.

* Schema : 데이터의 구조나 표현법, 관계를 나타낸다.
* Resolver : 메서드의 요청마다 어떻게 실제 작업들이 진행될 지를 구현한다.



### 구현해보기

_아래 예시는 apollo를 이용한 예시입니다._

#### **Schema**

위에서 언급한 GraphQL의 메서드를 사용하기 위해서 Schema에 **각 메서드의 루트 타입**과 **반환될 데이터의 형태를 지정해줄 타입**(선택사항임. 타입은 직접 기재해줘도 됨)이 필요하다.

```js
type Query { // Query 루트 타입
    equipments: [Equipment] // query로 equipments 요청 시 Equipment 타입 형태로 데이터를 불러온다.
    equipment(id: Int) : [Equipment]
}

type Mutation { // Mutation 루트 타입
    deleteEquipment(id: String): Equipment // id 인자를 받아 특정 equipment 삭제
    insertEquipment( // 아래 4가지 항목을 인자로 받아 equipment 삽입
        id: String,
        used_by: String,
        count: Int,
        new_or_used: String
    ): Equipment
}

type Equipment { // Equipment 타입 정의
    id: String
    used_by: String
    count: Int
    new_or_used: String
}
```

위 예시의 데이터를 추가하는 부분에서 전달한 인자를 직접 써준 것을 볼 수 있다. 이렇게 전달 인자가 많을 경우(보통은 추가, 수정 요청 시) 인자를 직접 전달하는 대신 `input` 타입을 써줄 수 있다.

```js
input EquipmentInput {
    id: String,
    used_by: String,
    count: Int,
    new_or_used: String
}

type Mutation {
    insertEquipment( input : EquipmentInput ): Equipment
}
```

#### **Resolver**

Resolver에는 각 타입에서 사용할 함수들이 정의되어 있다. 이 함수 내에서 데이터의 조회, 추가, 수정, 삭제가 이뤄지게 만들고, 반환 값(응답 데이터)을 지정한다.

참고로 Resolver 함수는 고정적으로 `obj`, `args`, `context`, `info` 4개의 인자를 받는다. 쿼리 요청 시 직접적으로 넘겨주는 인자는 `args`에 객체 형태로 담긴다.

```js
const database = require('./database');

const resolvers = {
    Query: {
        equipments: () => database.equipments 
        equipment: ( obj, args, context, info ) => { ... } // 특정 id의 equipment 조회 관련 홤수
    },
    Mutation: { 
        deleteEquipment: ( obj, args, context, info ) => { ... }, // equipment delete 관련 함수
        insertEquipment: ( obj, args, context, info ) => { ... } // equipment insert 관련 함수
    }
}
```

****

#### **데이터 요청과 응답**

요청을 보내면 필드 이름 또는 별칭이 key로 지정되고, resolve된 값(resolver의 함수를 통해 반환된 값)이 value가 되어 key-value 쌍으로 결과가 나타난다.

* Query 요청

```js
// id가 notebook인 equipment 조회
query {
    equipment(id: "notebook") {
        id
        used_by
        count
    }
    // 다른 필드의 조회를 원한다면 여기에 이어서 필드를 입력하면 된다.
}
```

```js
// id가 notebook인 equipment 조회 결과
{
  "data": {
    "equipments":{
        "id" : "notebook",
        "used_by" : "developer",
        "count" : 24
    }
  }
}
```

* Mutation 요청

```js
// 새로운 equipment 추가 요청
mutation {
  insertEquipment (
    id: "laptop",
    used_by: "developer",
    count: 17,
    new_or_used: "new"
  ) {
    id
    used_by
    count
    new_or_used
  }
}
```

```js
// 새롭게 추가된 equipment 데이터
{
  "data": {
    "equipments":{
        "id" : 1,
        "used_by" : "developer",
        "count" : 24
    }
  }
}
// 이후 equipments 조회 시 새로운 데이터가 추가되어 조회된다.
```

###

### 다양한 방법으로 query 하기

#### Alias(별칭)

쿼리문에서 동일한 필드 내의 서로 다른 데이터를 불러오기 위해 같은 이름의 필드를 중복하여 사용할 수 없다. 이럴 때 사용하는 것이 별칭이다. 별칭을 지정하게 되면, 응답받은 데이터에서 필드 이름 대신 별칭이 key로 지정된다.

```js
query {
    notebookEquipment: equipment(id: "notebook") {
        id
        used_by
        count
    }
    laptopEquipment: equipment(id: "laptop") {
        id
        used_by
        count
    }    
}
```

```js
{
  "data": {
    "notebookEquipment":{
        "id" : "notebook",
        "used_by" : "developer",
        "count" : 24
    },
    "laptopEquipment":{
        "id" : "laptop",
        "used_by" : "developer",
        "count" : 15
    }
  }
}
```

#### Flagment

받아와야 할 필드 내의 항목이 많을 때, 일일히 다 적어준다면 코드가 매우 길어진다. Flagment를 사용하면 여러 개의 항목을 하나로 묶어서, 요청할 때 전개하여 사용할 수 있다.

```js
query {
    notebookEquipment: equipment(id: "notebook") {
        ...equipmentfeilds
    }
    laptopEquipment: equipment(id: "laptop") {
        ...equipmentfeilds
    }    
}

fragment equipmentfeilds on Equipments{
    id
    used_by
    count
    new_or_used
    supply {
        ...
    }
}
```





_예시 참고 :_ [_얄팍한 GraphQL과 Apollo_](https://www.inflearn.com/course/%EC%96%84%ED%8C%8D%ED%95%9C-graphql-apollo) _강의_

{% hint style="info" %}
TODO : GraphQL에서 쿼리하는 법 추가하기([관련 링크](https://graphql-kr.github.io/learn/queries/))
{% endhint %}
