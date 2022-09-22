## 0. 공부하게 된 계기

cms 프로젝트에서 로그인, 회원가입을 구현하기 위해 token과 token storage에 대해 정리했다.  
어떤 것들이 절대적으로 좋거나 나쁘거나 하지 않은 것 같다.  
상황에 따라, 그리고 목적에 따라 필요한 것을 고르면 된다.  
아래에 나오는 모든 저장소는 (크롬 브라우저 기준) 'Application' 탭에서 저장된 내용을 확인할 수 있다.

## 1. token

토큰은 서버에게 이 요청을 보내는 사용자가 정당한 사용자가 맞음을 알려주기 위해 요청에 추가해서 보내는 것을 말한다.  
나는 토큰을 쿠키, 쿠키 + 세션, JWT, JWT with refresh token으로 구분하는 게 맞다고 생각한다.  
정보보안기사 공부할 때 질리도록 봤으니까 http의 특성 등은 생략하겠다.

### 쿠키

쿠키는 사용자의 정보를 담아, 클라이언트 측(브라우저)에 저장하는 방법을 말한다.  
일반적으로 서버보다는 클라이언트가 보안상 취약하다고 말하기 때문에 쿠키는 안전하지 않다고 많이 얘기한다.  
물론 http-only(스크립트 언어가 쿠키에 접근할 수 없음) 속성과 secure(https일 때만 쿠키에 접근할 수 있음) 속성을 이용하면 안전하다고 이야기하지만,  
사용자에 대한 정보가 들어있는 쿠키가 직접 통신으로 왔다갔다하는 것은 일반적으로 안전하지 않다.  
따라서 자주 사용하는 방식이 아니다.

### 세션

쿠키와 달리 사용자의 정보를 서버 측에 저장하는 방법을 말한다.  
http의 cookie header를 이용하여 통신하지만 사용자의 정보가 직접 왔다갔다하는 것이 아닌,  
서버 측에 저장된 세션에 대응되는 세션 아이디가 왔다갔다한다.  
쿠키보다는 안전하다고 하지만 서버 측에서는 세션 저장을 위해 별도의 공간을 관리해야 한다(일반적으로 db를 사용하는 것 같다).

### JWT

Json Web Token의 약자이다.  
JWT는 '.'을 기준으로 header, payload, signature로 구성되어 있다.  
header에는 알고리즘과 타입을, payload에는 토큰 발급자, 토큰 제목, 토큰의 만료 기간 등이 포함되며, signature은 header와 payload의 값을 각각 인코딩하고, 인코딩한 값을 서버의 키를 이용해 해싱한다.  
사용자의 요청에서 받은 signature 값과, 서버가 새로 계산한 signature 값이 다르면 사용자 인증이 실패하게 된다.  
JWT의 장점은 구현이 쉽다는 것과 서버에 키 외에 저장할 공간이 별도로 필요하지 않다는 것이다.  
덕분에 세션이나 후술할 JWT with refresh token과 달리 db에 기록하지 않아도 된다.  
다만 토큰의 만료기간이 짧으면 사용자의 이용이 불편해지고, 길면 보안에 취약해진다는 단점이 존재한다.

### JWT with refresh token

JWT의 단점을 보완하기 위해 나왔다.  
이 방식은 token을 두 가지로 분류한다.  
각각 access token과 refresh token이다.  
처음에 사용자가 인증되면 서버는 사용자에게 짧은 유효기간을 가진 access token과 refresh token(구현 방식에 따라 refresh token을 서버에서 저장하는 방법도 있다고 한다)을 발급한다.  
사용자는 요청에 해당 access token을 서버에 보내고, 서버는 access token의 유효 기간이 남아 있다면 정상적인 응답을 하고,  
유효 기간이 만료되었다면 refresh token을 확인한다.  
refresh token이 만료되지 않았다면, 새로운 access token을 발급해주지만,  
만료되었다면, 재인증(로그인 등)을 요청한다.  
JWT의 장점은 세션 및 쿠키, JWT의 단점을 커버한 것이라고 생각하면 된다.  
다만 가장 큰 단점은 구현이 어렵다는 것이다.  
다음 링크는 refresh token을 구현한 코드이다. https://cotak.tistory.com/102

## 2. token storage

사용자가 받은 토큰을 클라이언트는 어디에 저장하는 게 나을까?라는 고민을 하게 된다.  
세 가지 후보가 있다.  
각각 웹 스토리지(로컬, 세션 스토리지)와 쿠키이다.

### 웹 스토리지

공통적으로 5~10MB의 크기를 갖는다.  
XSS에는 취약하지만 CSRF에는 쿠키보다 강하다(라고 하지만 Authorization 헤더에 항상 담겨서 같이 넘어가기 때문에 CSRF에 강하다고는 말 못 할 거 같다).

- 로컬 스토리지
  - 서버에서 접근할 수 없고 persistent cookie와 비슷하다.
  - 반영구적으로 저장이 가능하다.
  - 브라우저 종료시 사라지지 않는다.
- 세션 스토리지
  - 서버에서 접근할 수 없고 session cookie와 비슷하다.
  - 브라우저 종료시에 사라진다.

### 쿠키

최대 4KB까지 저장이 가능하다.  
http request시 따로 코드를 구현하지 않아도 자동으로 포함 가능하다.  
httponly 설정을 추가하면 XSS 공격을 막을 수 있다.  
다만 여전히 CSRF 공격에는 취약하다.  
다음 링크에서 구현한 코드를 참고할 수 있다. https://velog.io/@neity16/NodeJS-Token-%EC%A0%80%EC%9E%A5-%EC%9C%84%EC%B9%98%EC%9D%98-%EA%B3%A0%EC%B0%B0

## 3. 전송

토큰을 헤더에 보낼지, body에 담을지에 대한 고민을 했다.  
결론부터 말하자면 이것은 크게 중요하지 않다.  
헤더에 넣어서 보낼 때는 일반적인 토큰은 보통 authorization header에, 쿠키는 cookie 헤더에 넣는다.  
결국 각각 장단점이 있기 때문에 프론트와 백의 소통과 api명세가 가장 중요하다.

## 4. 인증 타입

### Basic

사용자 아이디와 암호를 Base64 인코딩한 값을 사용한다.  
RFC 7617을 참고하면 자세히 알 수 있다.

### Bearer

JWT 혹은 OAuth에 대한 토큰을 사용한다.  
Authorization 헤더에 Bearer ${token}을 넣음으로써 전달할 수 있다.  
RFC 6750을 참고하면 자세히 알 수 있다.

### Digest

서버에서 난수 데이터 문자열을 클라이언트에 보내서 사용한다.  
클라이언트는 사용자 정보와 nonce를 포함하여 해시값을 사용하여 응답한다.  
RFC 7616을 참고하면 자세히 알 수 있다.

### HOBA

전자 서명 기반 인증이다.  
RFC 7486을 참고하면 자세히 알 수 있다.

### Mutual

암호를 이용한 C/S 간 상호 인증을 말한다.

## 5. 결론

~보안을 최소한으로 챙기면서 구현의 편의성을 위해 token은 JWT를, 저장 방식은 웹 스토리지로 결정했다.~  
쿠키를 사용하기로 다시 변경했다.  
cookie에서 same site(strict, lax, none)라는 속성을 사용하면 CSRF에 대한 방어도 가능하며,  
백의 코드는 거의 동일하지만 프론트의 코드가 매우 짧아지기 때문이다.

## 6. 추가사항

https://han41858.tistory.com/54 참고.
토큰 저장소로 사용되지는 않지만 브라우저 저장소의 하나로 indexedDB라는 것이 있다.  
쿠키가 문서 내부에 간단한 문자열 데이터를 저장, 로컬 저장소가 Json 데이터를 문자열로 변환하여 저장,  
세션 저장소가 Json 데이터를 해당 탭 세션에 저장하는 것이라면,  
indexedDB는 key를 이용해 index되는 구조화된 데이터를 저장하는 용도이다.  
파일이나 블롭 등의 데이터를 저장할 수도 있다.

### 특징

- key/value 저장
- Transaction 기능 지원
- key 범위의 쿼리와 index 지원
- 웹 스토리지에 비해 훨씬 많은 데이터를 저장 가능

### 사용 패턴

애플리케이션 블록을 방지하기 위해 모두 비동기로 이루어진다.

순서는 다음과 같다.

1. 데이터베이스를 연다.
2. 객체 저장소(Object store)를 생성한다. 객체 저장소란 데이터를 담는 공간으로 여러 개의 key/value 값으로 형성된다. 하나의 indexedDB 안에 여러 개의 객체 저장소가 존재할 수 있지만 각각의 이름은 고유해야 한다.
3. transaction을 시작하고 데이터베이스 작업을 요청한다. 모든 데이터 읽기 및 쓰기는 transaction 내에서 수행된다.
4. DOM eventListener를 사용하여 요청이 완료될 때까지 기다리고 결과를 확인한다.

### 예시 코드

```javascript
// 0. 브라우저가 indexedDB를 지원하는지 확인
if (!window.indexedDB) {
  console.log('사용 못 함')
} else {
  // 다음 단계
}

// 1. 데이터베이스 열기
const request = indexedDB.open('DB 이름', version)
// 해당 DB가 존재하지 않거나 현재 DB 버전이 원하는 버전보다 낮으면 request.onupgradeneeded가 호출되고,
// DB 버전을 업그레이드한다. 또는 DB가 아직 존재하지 않을 때도 트리거되므로 DB 초기화를 수행할 수 있다.
// https://stackoverflow.com/questions/34300166/why-does-indexeddb-use-a-version 여기 질문에도 나와있듯이
// 버전에 따라 어떤 동작을 할 수 있도록 지시할 수 있다.
// 성공적으로 DB가 열리면 request.onSuccess가, 에러가 발생하면 request.onerror가 호출된다.

request.onsuccess = () => {
  console.log('success')
  const database = request.result
}
request.onupgradeneeded = () => {
  const database = request.result
}
request.onerror = () => {
  console.error('error')
}

// 2. 객체 저장소 생성
const option = {
  keyPath: 'id', // id를 제공하는데 필요한 인덱스 필드의 이름을 지정
  autoIncrement: true,
}
const someStore = database.createObjectStore('store 이름', option)

// 3. transaction 시작
const transaction = database.transaction(someStore, 'transaction mode') // 첫 번째 인자는 객체 저장소의 이름을 말함
// 두 번째 인자는 'readonly', 'readwrite', 'versionchange' 가 가능함
// someStore이란 objectStore에 어떠한 모드로 transaction 시작하기
// transaction.oncomplete, transaction.onerror에 대해 콜백함수를 실행할 수 있음

const transaction = database
  .transaction('object store', 'readwrite')
  .objectStore('object store')
// 위에서 한 행위에 대해 추가적으로 'object store'이란 테이블 선택
transaction.add(entry) // 테이블을 선택한 뒤에 데이터 추가
transaction.put(entry)
transaction.get(key)
transaction.getAll()
transaction.delete(key)

return transaction.complete // DB에 변화가 생겼다면 이걸 return해야 됨
```
