## history

vanilla.js에서도 `react-router`(`react-router-dom`, `react-router-native`) 와 같은, SPA에서의 router를 구현할 수 있다.  
우선 필요한 DOM 개념 2가지를 알아야 한다.

**`CustomEvent()`**

새로운 CustomEvent를 생성하는 생성자로 다음과 같이 사용된다.

`new CustomEvent(typeArg, options)`

- `typeArg`: 이벤트의 이름을 나타내는 문자열
- `options`: 다음을 포함하는 객체
  - `detail`: 이벤트 내에 될 이벤트의 세부 정보를 나타내는 값(객체), 기본값은 null
  - `Event()` 생성자 옵션에 지정할 수 있는 모든 속성

**BrowserRouter vs HashRouter**

- BrowserRouter
  - HTML5의 history API를 활용해서 업데이트를 한다.
  - 서버에 있는 데이터들을 스크립트에 의해 가공처리한 후 생성되어 전달되는 동적인 페이지에 적합하다.
  - 해당 라우터를 이용해서 이동한 경로에서 새로 고침을 하면 에러가 나므로 추가적인 세팅이 필요하다.
- HashRouter
  - URL의 hash를 이용한 라우터로, 주소에 '#'이 붙는다.
  - 서버가 주소의 이동을 모른다. 따라서 정적인 페이지에 주로 사용된다.
  - `www.domain.com/#/path`로 요청하면 실제로는 `www.domain.com`에 대한 요청이 간다.
  - 주소 이동 후, 새로고침해도 에러가 발생하지 않는다.
  - 검색엔진이 읽지 못한다.

SPA와 같은 웹 페이지를 만들기 위한 것이 목적이므로 BrowserRouter를 사용해야 한다.  
따라서 `history.pushState()`를 이용한다.

### 코드 예시

`ROUTE_CHANGE_EVENT`는 (custom)event의 `typeArg`이다.

```javascript
// // app.js
const app = document.querySelector('#app');
const route = () => {
  const { pathname } = location;

  app.innerHTML = '';

  if (pathname === 'product-list') {
    new ProductList(); // app에 ProductList 렌더링
  } else if (pathname === 'cart') {
    // ...
  } // ...
  // 제대로된 path가 아니면 오류 페이지를 렌더링
};

window.addEventListener(ROUTE_CHANGE_EVENT, () => {
  route(); // route가 바뀔 때마다 불릴 수 있도록 설정
});
window.addEventListener('popstate', () => {
  // window.onpopstate. 앞으로 가기, 뒤로 가기 처리
  route();
});

// // utils.js
// url을 바꿔야 되는 상황이 있을 때 해당 함수 호출
const changeRoute = (url, params) => {
  history.pushState(null, '', url);
  window.dispatchEvent(new CustomEvent(ROUTE_CHANGE_EVENT, params));
};
```

_참고)_  
https://woong-jae.com/javascript/220325-spa-from-scratch-2 는 path를 key, callback을 value로 둬서 사용한 예시이다.
