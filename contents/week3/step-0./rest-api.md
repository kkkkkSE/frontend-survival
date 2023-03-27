# REST API

### REST

REST(Representational State Transfer)는 API를 사용하는 방식에 대한 조건을 부과하는 아키텍처로, HTTP URI(Uniform Resource Identifier)를 통해 **Resource를 명시**하고, **HTTP Method를 사용**하여 Resource(URI)에 대해 **CRUD Operation을 적용**하는 것을 말한다.

REST API, RESTful API는 이 아키텍처 스타일을 따르는 API이다.

{% hint style="info" %}
REST에서의 CRUD Operation

* GET(Read) : 리소스 조회
* POST(Create) : 전달한 데이터를 처리/생성
* PUT, PATCH(Update) : 리소스를 수정(대체)
* DELETE(Delete) : 리소스 제거
{% endhint %}

###

### REST 구성 요소

* Resource(자원) : 모든 자원에는 고유한 URL이 있고, 이를 이용해서 자원과 관련된 조작을 서버에 요청한다.
* Verb(행위) : HTTP 프로토콜의 Method를 사용한다.
* Representation of Resource(표현) : 클라이언트가 자원과 관련된 조작을 요청하면, 서버는 적절한 응답을 보낸다.

###

### REST 특징

* Uniform Interface : 통일되고 한정적인 인터페이스를 사용한다.
  * 자원을 식별할 식별자(URI) 사용
  * 특정 표현을 통해 자원을 조작
  * Self-descriptiveness : REST API 메시지만 보고도 쉽게 이해 할 수 있어야 함
* Client-Server 구조 : 요청하는 쪽이 클라이언트, 응답하는 쪽이 서버로 분리되어 있다. (서로간 의존성이 줄어듬)
* Stateless : 작업을 위한 상태정보를 따로 저장하지 않아 서버는 상태에 신경쓰지 않고 요청만을 단순히 처리한다.
* Cacheable : 캐싱 기능을 사용하여 응답시간이 빨라질 수 있다.
* Layered System : 서버는 다계층으로 구성될 수 있어 보안, 사용자 인증 등을 추가할 수 있고 프록시와 같은 네트워크 기반 중간 매체도 사용할 수 있다.

###

### REST API 설계 규칙

* URI는 자원을 표현해야 한다.
  * URL에서 동사보다 명사를, 대문자보다 소문자를 사용한다.
  * URL 가독성을 위해 언더바(\_)보다 하이픈(-)을 사용한다.
  * 파일 확장자는 포함하지 않는다.
* 행위는 Method로 표현한다.
  * 각 Method를 알맞는 상황에 사용한다.
* 슬래시는 계층 관계를 나타낸다.
  * 마지막 문자로 슬래시를 사용하지 않는다.

```http
GET /members/delete/1 (X)
DELETE /members/1 (O)
```

* 리소스 간에 관계가 있다면 이런식으로 표현할 수 있다.
* Collection은 복수형, Document는 단수형으로 표기한다.

```http
DELETE /posts/{postid}/comments/{commentid}
```

###

### (참고) API 요청과 응답 구성(HTTP Request, Response)

**클라이언트 요청(HTTP Request)** 에는 다음과 같은 요소가 포함되어 있다.

* Start Line :
  * HTTP 버전
  * Request target : 서버는 고유한 리소스 식별자로 각 리소스를 식별한다. 보통 리소스 식별자로 리소스의 경로를 지정하는 URL(Uniform Resource Locator)을 사용한다.
  * Method : 4가지 또는 5가지의 HTTP 메서드를 사용하여 리소스에 수행해야 하는 작업을 알려준다.
* [Header](https://developer.mozilla.org/ko/docs/Glossary/Request\_header) : 서버에 요청할 때 메세지와 관련 없는 부가적인 정보를 전달할 수 있다.
* Body : 요청할 실제 내용이 담겨있다. 보통 POST 메서드 사용 시 Body를 사용한다.

**서버 응답(HTTP Response)** 에는 다음과 같은 요소가 포함되어 있다.

* Status line :
  * HTTP 버전
  * Status Code : 응답 성공 또는 실패를 알리는 3자리 코드가 있다.
    * 2XX : 성공 응답
    * 3XX : URL 리디렉션
    * 4XX : 클라이언트의 요청이 부적절
    * 5XX : 서버 문제로 응답 실패
  * Status Text : 응답 메세지의 상태를 간략하게 설명해준다.
* [Header](https://developer.mozilla.org/ko/docs/Glossary/Response\_header) : 서버에서 응답할 때 메세지와 관련 없는 부가적인 정보를 전달할 수 있다.
* Body : 클라이언트가 요청한 데이터 작성 방식에 맞게 데이터를 응답한다. 요청한 데이터가 없을 땐 Body가 존재하지 않을 수 있다.
