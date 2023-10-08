## 0. 공부하게 된 계기

HTTP 헤더에 대해 자세히 알면 좋으니까! 시작했다.

## 1. Content-Type

참고  
https://2ham-s.tistory.com/307  
https://velog.io/@ryan-son/multipartform-data-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0-vs-applicationjson

리액트로 프론트 엔드를 공부하던 와중에 application/json 타입으로 통신을 할 때도 있었고,  
multipart/form-data로 통신을 할 때도 있었다.  
결론부터 말하자면 string 등의 타입만 보낼 때는 json이 편리하지만, 음악/사진/동영상 등은 form data가 편리하다고 생각한다.

### application/json

JSON이란 JavaScript Object Notation의 줄임말로, 데이터를 전송하거나 저장할 때 많이 사용되는 경량의 데이터 교환 방식을 말한다.  
다음은 장점과, application/json 형식으로 바이너리 데이터를 취급할 때의 단점을 정리했다.

- 장점
  - 파일과 함께 전달되는 데이터의 연관 관계를 표현하기 용이.
  - 아스키 데이터를 전달하기에 편함.
- 단점
  - 바이너리 데이터를 다루기 위해서는 인코딩이 필요한데, 이를 위해서는 33~37%(base64와 같은 경우) 더 용량이 커짐.
  - 서버는 디코딩, 클라이언트는 인코딩을 위한 오버헤드가 추가.
  - 인코딩한 파일 컨텐츠의 문자열이 길어져서 서버측에서 파일을 읽지 못하는 경우가 발생할 수 있음.

### multipart/form-data

이 타입은 MIME 데이터 스트림의 규칙을 준수한다.  
(프론트) `append` 메소드로 key-value 값을 하나씩 추가해주고, 같은 key를 가진 값을 여러 개 넣을 수 있다.  
단, `set`으로 넣으면 덮어쓰인다.  
_숫자를 넣어도 문자열이 되고, 배열을 넣어도 콤마로 구분한 문자열이 된다. 또한 객체를 넣으면 무시가 된다._
위 특징을 잘 참고해서 사용해야 한다.

- 장점
  - application/json에서 발생하는 오버헤드, 용량 등을 감수하지 않아도 됨.
- 단점
  - 파일과 함께 전달된 데이터의 연관 관계 표현이 제한됨.

### 추가사항

데이터를 보내는 각각의 방식에는 장단점이 있다.  
검색을 하다 알게된 사실인데, form-data와 json으로 각각을 동시에 보내는 방법도 있다.  
두 가지의 장점을 살리기 위해서 고안된 방법 같은데, form 안에 json을 넣어서 보냄으로써 구현된다.

## 2. Cache-Control

참고 및 출처  
https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching  
https://hudi.blog/http-cache/  
https://toss.tech/article/smart-web-service-cache  
https://www.blog-dreamus.com/post/cache-control-%EC%9D%B4-%ED%95%84%EC%9A%94%ED%95%9C-%EC%9D%B4%EC%9C%A0  
https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Cache-Control

캐시는 **기본적으로 자주 사용되는 데이터를 임시로 복사해두는 임의의 장소**를 의미하고, 캐싱은 캐시에 저장하는 행위를 말한다.  
일반적으로 캐싱은 캐시에 저장된 데이터에 접근하는 시간에 비해 원본 데이터에 접근하는 시간이 오래 걸리는 경우 사용한다.

참고로 브라우저에서 캐시가 동작하려면 "Disable cache" 등의 브라우저 옵션이 꺼져 있어야 한다.

### HTTP 캐시 종류

![http caching](https://github.com/mochang2/development-diary/assets/63287638/8c23d7f4-f0c2-40ab-ac92-930a29e96081)

캐시는 크게 2개로 나눌 수 있다.

- Private Cache: 웹 브라우저에 저장되는 캐시. 다른 사람이 접근할 수 없음. 서버 응답에 Authorization 헤더가 포함되어 있으면 이 장소에 저장하지 않음.
- Shared Cache: 브라우저와 서버 사이에 동작하는 캐시.
  - Proxy Cache: (포워드) 프록시에서 동작하는 캐시.
  - Managed Cache: CND 서비스 또는 리버스 프록시에서 동작하는 캐시.

(이거는 내가 잘못 생각했던 부분인데, Cache-Control은 Shared Cache에 저장하는 행위에 대해서도 control할 수 있다)

### 캐시 동작 방식

캐시는 다음과 같이 동작한다.

1. (Private Cache)Client의 요청 시 유효 기간이 지나지 않은 리소스가 있는지 확인한다. 만약 있으면 디스크 또는 메모리에서 읽어와 사용한다. 이를 fresh라고 표현한다.

![200 from cache](https://github.com/mochang2/development-diary/assets/63287638/44dfcecf-42ea-49f9-a1c2-231ef42a0a63)

2. (Shared Cache or Server)유효 기간이 지났다면(이를 stale이라고 표현한다) 캐시가 유효한지 재검증을 수행한다. 해당 문서가 유효하다면 304 상태 코드를 보내준다(일반적으로 Body는 보내지 않음으로써 네트워크 비용을 절약한다).

![304 from revalidation](https://github.com/mochang2/development-diary/assets/63287638/258d092d-8408-4113-ba30-4305b6d2e9a0)

이때, 재검증 시각에 대한 내용은 갱신된다.  
예를 들어 15:00:00에 "max-age=15"를 가진 응답을 받았다고 하자.  
이후 15:00:20에 재요청을 보내면 유효 기간이 지났다고 판단하고 재검증을 수행한다.  
만약 서버로부터 304응답을 받았다면 15:00:35 전에 보내는 리소스 요청은 다시 Private Cache에서 읽어와 사용한다.

(자세한 재검증 관련 헤더는 아래에서 다루겠다)

3. 유효하지 않다면 Server에 요청을 전달하고 응답을 받는다. 이때 Server로부터 유효하지 않다는 응답과 새로운 응답을 위해 각각 별도의 요청을 보낼 필요가 없이 효율적으로 동작한다.

### 속성

더 많은 내용들이 있는데, 자주 사용하는 내용들 위주로 정리했다.

- 캐시 능력
  - public: Private Cache, Shared Cache 모두에 캐싱.
  - private: Private Cache에만 캐싱.
  - no-cache: 캐시된 복사본을 Client에게 보여주기 전에 반드시 서버에 재검증 요청을 요구. "max-age=0"과 같음.
  - only-if-cached: Client는 캐싱된 응답만을 받으며 새로운 데이터를 내려받지 않음.
- 만료
  - max-age=\<seconds\>: 리소스가 최신 상태라고 판단할 최대 시간.
  - s-maxage=\<seconds\>: max-age 또는 Expires 헤더를 재정의. Shared Cache에서 리소스가 최신 상태라고 판단할 최대 시간.
- 재검증과 리로딩
  - must-revalidate: 캐시를 사용하기 이전에 반드시 리소스를 검증해야 하며, 만료된 리소스는 사용되어서는 안 됨.
  - proxy-revalidate: Shared Cache에서 must-revalidate 옵션.
  - immutable: response body가 계속해서 변하지 않을 것이라는 것을 나타냄. 응답이 만료되지 않은 경우라면 클라이언트는 재검증을 전송해서는 안 됨.
- 기타
  - no-store: 캐시를 사용하지 않음.

_참고1_  
캐시를 사용하지 않기 위해서 다음과 같이 사용 가능하다.

```text
Cache-Control: no-cache, no-store, must-revalidate
```

_참고2_  
"max-age=0"(no-cache)의 경우, 일부 모바일 브라우저의 경우 웹 브라우저를 껐다 켜기 전까지 리소스가 만료되지 않도록 하는 경우가 있다.

### 부가적인 헤더

아래에 부가적인 헤더 사용은 서버에서 결정한다.  
하나만 쓸 수도 있고 여러 가지를 같이 혼용해서 쓸 수도 있다.

#### Age

Age 헤더는 캐시 응답 때 나타나는데, max-age 시간 내에서 얼마나 흘렀는지 초 단위로 알려준다.  
*max-age=3600*을 설정한 경우, 1분이 지나면

```text
Age: 60
```

이 캐시 응답 헤더에 포함된다.

#### Expires

Cache-Control과 별개로 응답에 Expires라는 헤더를 줄 수도 있다.

```text
Expires: Sun, 8 Oct 2023 07:28:00 GMT
```

응답 컨텐츠가 언제 만료되는지를 나타내며, Cache-Control의 max-age가 있는 경우 이 헤더는 무시된다.  
Cache-Control 헤더가 캐시 제어를 더 세밀하게 설정할 수 있기 때문에, Expires를 혼용해서 사용하는 것은 일반적으로는 네트워크 리소스 낭비를 야기한다.

#### Last-Modified와 If-Modified-Since

문서 유효 시간 체크하는 방법이다.

최초 요청 시 다음과 같은 응답을 내려준다.
Server는 리소스의 마지막 수정 날짜(Last-Modified)를 제공한다.

```text
HTTP/1.1 200 OK
Content-Type: text/html
Cache-Control: max-age=3600
Last-Modified: Sat, 03 Sep 2022 00:00:00 GMT
```

캐시 기간이 초과된 이후 다음 요청 시 다음과 같은 내용을 보낸다.  
이전 응답에서 Last-Modified 값을 If-Modified-Since의 값으로 넣는다.

```text
GET /index.html HTTP/1.1
Host: example.com
Accept: text/html
If-Modified-Since: Sat, 03 Sep 2022 00:00:00 GMT
```

원본 리소스에 아무런 변경이 없다면 서버는 아래와 같은 응답을 보낸다.  
Last-Modified 값에 대한 변경은 없다.

```text
HTTP/1.1 304 Not Modified
Content-Type: text/html
Cache-Control: max-age=3600
Last-Modified: Sat, 03 Sep 2022 00:00:00 GMT
```

#### Etag(Entity tag)와 If-None-Match

아래와 같은 경우 단순히 유효 시간을 체크하는 것으로 충분하지 않을 수 있다.

- 문서가 주기적으로 업데이트가 되지만, 바뀐 내용이 없는 경우
- 변경된 내용이 있지만, 응답상으로는 의미가 크지 않은 경우
- 유효한 시간을 정하기 어려운 경우
- 초보다 정밀한 단위의 설정이 필요한 경우

최초 요청 시 다음과 같은 응답을 내려준다.
Server는 특정 버전의 리소스를 구분하기 위한 식별자인 Etag를 제공한다.

```text
HTTP/1.1 200 OK
Content-Type: text/html
Cache-Control: max-age=3600
ETag: "abcdefu"
```

캐시 기간이 초과된 이후 다음 요청 시 다음과 같은 내용을 보낸다.  
이전 응답에서 Etag 값을 If-None-Match의 값으로 넣는다.

```text
GET /index.html HTTP/1.1
Host: example.com
Accept: text/html
If-None-Match: "abcdefu"
```

원본 리소스에 아무런 변경이 없다면 서버는 아래와 같은 응답을 보낸다.  
Etag 값에 대한 변경은 없다.

```text
HTTP/1.1 304 Not Modified
Content-Type: text/html
Cache-Control: max-age=3600
Last-Modified: "abcdefu"
```

Last-Modified와 Etag를 같이 사용하는 경우 복잡성이 높아지며 일부 불필요한 오버헤드가 발생할 수 있다.  
그러나 일부 상황에서는 두 가지 방법을 조합함으로써 캐시 효율성을 높일 수 있다.  
Last-Modified는 리소스 변경 여부를 확인하는 간단한 방법을 제공하고, Etag는 보다 정확하게 캐시 유효성 검사를 수행할 수 있기 때문이다.

_참고) Etag 생성 방법_

- 해시
- 마지막으로 수정된 timestamp의 해시
- 개정번호

등 다양함.

_참고) weak ETag vs strong ETag_

- weak ETag: 만들기는 쉬우나 비교하기에는 효율성이 떨어짐. ex) etag: W/'asdfadfs'
- strong ETag: 비교하기에는 이상적이지만(byte 단위 비교 등) 효율적이지 않음. ex) etag: 'asdfasdf'
