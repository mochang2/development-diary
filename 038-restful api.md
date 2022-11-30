## 0. 공부하게 된 계기

클라이언트, 디바이스 종류가 다양해지면서 순수한 API 서버로서의 백엔드가 필요해졌다.  
동시에 자원에 대한 행위를 명시하는 REST API가 각광받기 시작했다(과거에 SOAP이라는 복잡한 형식을 대체함).

API를 설계하면서 RESTful API가 무엇인지 확실히 알고 갈 필요가 있다고 느꼈다.  
실제로 모든 규칙을 지킬 수 있는 것은 아니지만 최대한 기준을 따라가는 것이 설계나 협업에서 혼란을 줄일 수 있는 길이라고 생각한다.

참고  
https://gmlwjd9405.github.io/2018/09/21/rest-and-restful.html  
https://sanghaklee.tistory.com/57  
https://www.youtube.com/watch?v=RP_f5dMoHFc  
https://velog.io/@kjh03160/%EA%B7%B8%EB%9F%B0-REST-API%EB%A1%9C-%EA%B4%9C%EC%B0%AE%EC%9D%80%EA%B0%80

## 1. REST의 정의

_cf) API란? 소프트웨어가 다른 소프트웨어로부터 지정된 형식으로 요청, 명령을 받을 수 있는 수단_

REST란 'REpresentational State Transfer' 의 약자로 자원을 이름(자원의 표현)으로 구분하여 해당 자원의 상태(정보)를 주고 받는 모든 것을 의미한다.  
이를 HTTP의 개념과 합쳐보면 HTTP **URI**를 통해 자원(Resource)을 명시하고, HTTP **Method**를 통해 해당 자원에 대한 **CRUD**하는 것을 의미한다.

### 등장 배경

- 애플리케이션 분리 및 통합
- 다양한 클라이언트의 등장
- 서버 프로그램은 다양한 브라우저와 안드로이폰, 아이폰과 같은 모바일 디바이스에서도 통신을 할 수 있어야 함

### 특징

1. **Server-Client 구조**

자원이 있는 쪽이 server, 자원을 요청하는 쪽이 client이다.  
server: API를 제공하고 비즈니스 로직 처리 및 저장을 책임진다.  
client: 사용자 인증이나 context(세션, 로그인 정보) 등을 직접 관리하고 책임진다.

2. **Stateless**

HTTP는 stateless 하므로 REST 역시 stateless하다.

client의 context를 server에 저장하지 않는다.  
server는 각각의 요청을 완전히 별개의 것으로 인식하고 처리한다.  
각 API 서버는 client의 요청만을 단순 처리한다.  
다만 이전 요청이 DB를 수정하여 DB에 의해 바뀌는 것은 허용한다.

Server의 처리 방식에 일관성을 부여하고 부담이 줄어들며, 서비스의 자유도가 높아진다.

3. **Cacheable**

웹 표준 HTTP 프로토콜을 그대로 사용하므로 웹에서 사용하는 기존의 인프라(캐싱 포함)를 그대로 활용할 수 있다.  
HTTP 표준에서 사용하는 Last-Modified 태그나 E-Tag를 이용하면 캐싱 구현이 가능하다.

4. **Layered System**

client는 REST API server만 호출한다.  
REST server는 다중 계층으로 구성될 수 있다.  
API server는 순수 비즈니스 로직을 수행하고 그 앞단에 보안, 로드밸런싱, 암호화, 사용자 인증 등을 추가하여 구조상의 유연성을 줄 수 있다.

5. **Uniform Interface**

URI로 지정한 resource에 대한 조작을 통일되고 한정적인 인터페이스로 수행한다.  
HTTP 표준 프로토콜에 따르는 모든 플랫폼에서 사용이 가능하다.  
특정 언어나 기술에 종속되지 않는다.  
왜냐하면 server와 client는 독립적인 존재로서 진화 (해야) 하기 때문이다.

uniform interface를 위해서는 아래 4가지를 만족해야 한다.  
보통의 (우리가 부르는) REST API는 위 2가지는 잘 만족하지만 나머지 두 가지를 잘 만족하지 않는다.

- Identification of resources
- Manipulation of resources through representations
- _Self-descriptive messages_

self-descriptive하기 위해서는 (response와 같은 경우) 아래와 같이 content-type에 확실히 명시해야 한다.

```text
HTTP/1.1 200 OK
Content-Type: application/json-patch+json

[ { "op": "remove", "path": "/a/b/c" } ]
```

- _Hypermedia as the engine of application state(HATEOAS)_

홈페이지 -> 글 목록 -> 글 쓰기 -> 글 저장 -> 생성된 글 보기 -> 목록 ... 이렇게 상태를 전이하는 것을 애플리케이션 상태 전이라고 하고 항상 해당 페이지에 있던 링크를 따라간다면 HATEOAS라고 할 수 있다.  
아래와 같은 HTML은 HATEOAS라고 할 수 있다.

```text
HTTP/1.1 200 OK
Content-Type: text/html
```

```html
<html>
  <head> </head>
  <body>
    <a href="/test"> test </a>
  </body>
</html>
```

6. **Code-on-Demand(optional)**

Server로부터 스크립트를 받아서 Client에서 실행한다.  
반드시 충족할 필요는 없다.

## 2. RESTful 하려면 어떡해야 될까

RESTful API란 REST API를 REST하게 사용하는 것을 이야기한다.  
다만 REST API에는 표준이 없다.  
따라서 다음에 제시할 사용법은 가장 자세하게 명시되어 있는 블로그의 내용을 인용한 것이다.

### 1) URL

1. URL 마지막에 '/'을 포함하지 않는다.
2. 계층적 구조는 '/'로 표현한다.
3. 띄어쓰기를 대체하기 위해 '\_' 대신 '-'를 사용한다.
4. 영어 대문자가 아닌 소문자를 사용한다.
5. 확장자를 포함하지 않는다.
6. 행위는 URL에 포함하지 않고 method로 표현한다. 단, 컨트롤 리소스를 의미하는 URL은 예외적으로 동사를 허용한다.

함수처럼, 컨트롤 리소스를 나타내는 URL은 동작을 포함하는 이름을 짓는다.

```text
http://api.test.com/posts/duplicating // bad
http://api.test.com/posts/duplicate // good
```

### 2) HTTP headers

1. Content-Location

POST 메서드를 통한 요청은 멱등하지 않다.  
따라서 요청의 응답 헤더에 새로 생성된 리소스를 식별할 수 있는 Content-Location을 명시한다.

```text
HTTP/1.1 200 OK
Content-Location: /users/1
```

2. 알맞는 Content-Type을 명시한다.
3. Retry-After

비정상적인 방법(DoS, Brute Force)으로 API를 이용하려는 경우 429 오류 응답을 내보낸다.

```text
HTTP/1.1 429 Too Many Requests
Retry-After: 3600
```

4. Link

페이징 처리를 위해 사용한다.

- next: The link relation for the immediate next page of results.
- last: The link relation for the last page of results.
- first: The link relation for the first page of results.
- prev: The link relation for the immediate previous page of results.

```text
HTTP/1.1 200 OK
Link: <https://api.test.com/users?page=3&per_page=100>; rel="next",
  <https://api.test.com/users?page=50&per_page=100>; rel="last"
```

### 3) HTTP method

1. POST, GET, PUT, DELETE 4가지는 반드시 제공한다.
2. OPTIONS, HEAD, PATCH를 사용하여 완성도 높은 API를 만든다.

### 4) HTTP status

1. 의미에 맞는 HTTP status를 리턴한다.
2. 500번대 에러는 사용자에게 나타내지 마라

### 5) Use HATEOAS

응답 객체에 해당 리소스의 상태가 전이될 수 있는 link들을 함께 제공한다.  
link들을 통해 리소스의 다음 상태 전이 정보를 동적으로 제공한다.

다음 3가지를 포함한다.

- `rel`: 변경될 리소스의 상태 관계(self는 자기 자신)
- `href`: 요청 URL
- `method`: 요청 method

다음과 같이 사용한다.

```text
201 Created
```

```json
{
  "id": 1,
  "name": "hak",
  "createdAt": "2018-07-04 14:00:00",
  "links": [
    {
      "rel": "self",
      "href": "http://api.test.com/users/1",
      "method": "GET"
    },
    {
      "rel": "delete",
      "href": "http://api.test.com/users/1",
      "method": "DELETE"
    },
    {
      "rel": "update",
      "href": "http://api.test.com/users/1",
      "method": "PATCH",
      "more_info": "http://api.test.com/docs/user-update",
      "body": {
        "name": "{The value to be modified}"
      }
    },
    {
      "rel": "user.posts",
      "href": "http://api.test.com/users/1/posts",
      "method": "GET"
    }
  ]
}
```

### 6) Collection

1. Paging

Collection에 대한 GET 요청의 경우 한 번에 모든 결과를 응답하지 않고 적당한 크기로 데이터 셋을 나눠서 응답한다.  
요청에 대한 key값은 개발자가 편한 것으로 정해도 된다.

```text
HTTP/1.1 200 OK
Link:
<https://api.test.com/users?offset=10&limit=10>; rel="next",
<https://api.test.com/users?offset=50&limit=10>; rel="last",
<https://api.test.com/users?offset=0&limit=10>; rel="first",
<https://api.test.com/users?offset=0&limit=0>; rel="prev",
[
    {1, ...},
    {2, ...},
    ...
    {10,...},
    "links": [
      {
        "rel": "next",
        "method": "GET",
        "link": "https://api.test.com/users?offset=10&limit=10
      },
      {
        "rel": "last",
        "method": "GET",
        "link": "https://api.test.com/users?offset=50&limit=10
      },
      {
        "rel": "first",
        "method": "GET",
        "link": "https://api.test.com/users?offset=0&limit=10
      },
      {
        "rel": "prev",
        "method": "GET",
        "link": "https://api.test.com/users?offset=0&limit=0
      },
    ]
]
```

2. Ordering

Collection에 대한 GET 요청의 경우 클라이언트의 요청에 맞게 정렬해 응답한다.  
`order`라는 key를 사용한다.

3. Filtering

Collection에 대한 GET 요청의 경우 리스트 검색 조건을 요청할 수 있다.

4. Field-Selecting

Collection(리스트)에 대한 GET 요청의 경우 리스트 결과의 일부분만 선택해서 응답받을 수 있다.
