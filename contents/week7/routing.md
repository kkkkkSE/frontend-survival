# Routing

라우팅이란, 웹 애플리케이션 내에서 다른 페이지로 이동하거나, URL에 매핑되는 컴포넌트를 렌더링하도록 도와주는 기능을 말한다. React에서는 하나의 웹페이지를 하나의 큰 컴포넌트로 만들고, 그 안에서 URL에 따라 일부 내용만 변경하는 것도 가능하다.

## Location을 이용한 Routing

### Location

Location은 HTML DOM API의 인터페이스 중 하나로, 접속중인 웹페이지의 URL 정보를 제공한다. URL을 구성하는 다양한 요소를 프로퍼티로 제공하고 있으며, URL을 변경하거나 로드하는 메서드도 제공하고 있다.

{% hint style="info" %}
HTML DOM API는 HTML 요소의 속성을 읽거나 수정하고, 새로운 HTML 요소를 동적으로 생성하거나 제거하는 등 HTML 문서에 대한 다양한 조작을 할 수 있게 해준다.
{% endhint %}

참고로, Location은 document와 window 인터페이스에서도 접근할 수 있도록 연결되어 있어 `Document.location`과 `Window.location`를 통해 액세스할 수 있다.

### Location 프로퍼티

<figure><img src="/.gitbook/assets/location-prop.png" alt=""><figcaption></figcaption></figure>

- location.hostname : 도메인 주소
- location.host : hostname + port 번호
- location.pathname : 쿼리 스트링을 포함하지 않은 host 뒤의 '/' 이후 문자열
- location.hash : 해시 기호 '#' 뒤의 문자열(한글, 특수문자, 공백은 encode하여 출력, `decodeURI(uri)`로 되돌릴 수 있음)

{% hint style="info" %}
URL의 fragment는 웹페이지 내 특정 위치로 이동하기 위해 많이 사용된다.
참고로, 옛날 웹페이지에서는 자바스크립트 파일에서 hash를 이용하여 SPA처럼 새로고침 없이 페이지 일부 내용을 변경할 때 사용하곤 했다.
{% endhint %}

이 외에도 다양한 프로퍼티를 제공하고 있으니 [공식 문서](https://developer.mozilla.org/ko/docs/Web/API/Location)를 참고하자.

### 예제

다음은 pathname에 따라 다른 컴포넌트를 렌더링하도록 만든 예제이다.

```js
export default function App(){
    const path = window.location.pathname;

    return(
        <div>
            {path === '/' && <HomaPage />}
            {path === '/about' && <AboutPage />}
        </div>
    )
}
```

> 이렇게 페이지로 사용할 컴포넌트들은 `components` 폴더가 아닌 `pages` 폴더에 저장해두면 관리가 용이해진다.

위 예제처럼 사용하는 것도 좋지만, 관리해야하는 URL이 많아질수록 렌더링 코드가 복잡해지니 객체로 따로 관리할수도 있다.

```js
const pages = { // key : pathname, value : 컴포넌트
    '/' : HomaPage,
    '/about' : AboutPage,
}

export default function App(){
    const path = window.location.pathname;

    const Page = Reflect.get(pages, path) || HomaPage; // 유효하지 않은 path면 메인화면

    return(
        <div>
            <Page />
        </div>
    )
}
```

### 주의할 점

test 환경에서는 `window` 객체가 없어서 `location.pathname`을 사용하지 못하므로 path를 추상화하여 테스트해야 한다.

[React Router](./react-router.md)를 사용함으로서 해당 문제를 해결할 수 있다.
