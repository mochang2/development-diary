## 0. 공부하게 된 계기

https://tech.inflab.com/202208-shadow-root/  
이 블로그에서 전역 스타일에 영향을 받지 않도록 shadow dom을 이용했다고 한다.  
관심을 가지고 다른 글들도 찾아보던 차에 Vue에서 많이 사용되는 태그인 `template`과 `slot` 이야기도 나오고, 그러면 React Portal은 이와 또 어떤 관계가 있나 싶어서 공부해봤다.

참고  
https://developer.chrome.com/articles/declarative-shadow-dom/  
https://velog.io/@rageboom/%EC%9B%B9-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-Template-and-Slot  
https://wit.nts-corp.com/2019/03/27/5552  
https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_templates_and_slots  
https://enumclass.tistory.com/226  
https://solo5star.tistory.com/28  
https://tech.inflab.com/202208-shadow-root

## 1. DOM(Document Object Model) 이란

DOM은 웹 문서를 위한 인터페이스이다.  
DOM은 HTML 문서를 node나 object로 구조화하여 표현하며 `getElementById`와 같은 각종 DOM API를 사용할 수 있는 객체 모델이다.  
즉, HTML 문서 그 자체가 아니라는 말이다.  
DOM은 브라우저가 페이지에 무엇을 렌더링 할지 결정하기 위해, 혹은 자바스크립트 프로그램이 페이지의 콘텐츠 및 구조, 스타일을 수정하기 위해 사용된다.

## 2. shadow DOM 이란

웹 컴포넌트의 기술 중 하나이다.  
웹 컴포넌트는 재사용할 수 있는 커스텀 HTML element를 생성하고, 해당 요소를 캡슐화하는 기술이다.  
캡슐화를 통해 마크업, 스타일, 동작을 외부로부터 격리하여, 웹페이지의 다른 구성 요소의 간섭을 방지할 수 있게 도와준다.

특히 글로벌 스타일에 영향을 받지 않는 컴포넌트 요소를 렌더링할 때 유용하다.  
한 가지 예시로 브랜드 아이콘과 같이 어떠한 element 안에 들어가도 같은 색, 크기, 글자 모양을 유지하고 싶을 때 사용할 수 있다.

![shadow dom](https://github.com/mochang2/development-diary/assets/63287638/51874dce-0131-4619-9c7e-106159f3c244)

위 그림은 shadow dom이 DOM tree를 구성하는 방법을 나타낸다.

- shadow host: 일반적인 DOM 노드로 shadow DOM이 이 아래로 추가된다. anchor와 같이 상호 작요앟는 요소들은 shadow host가 될 수 없다.
- shadow tree: shadow DOM 내부의 DOM tree이다.
- shadow boundary: 외부하고 구분되는 shadow DOM tree를 말한다.

### 사용법

```html
<a href="#">test</a>
<span class="shadow-host">
  <a href="/to-somewhere">anchor text</a>
</span>
```

```js
const shadowEl = document.querySelector('.shadow-host');
const shadow = shadowEl.attachShadow({ mode: 'open' });
```

위와 같이 코드를 짜면 shadow tree는 DOM tree에는 포함되지만 화면에 렌더링되지는 않는다.

![dom tree](https://github.com/mochang2/development-diary/assets/63287638/2d8accd8-5699-48b6-9cbf-5263f79a03f5)

![screen](https://github.com/mochang2/development-diary/assets/63287638/c005cf91-6051-41ea-87aa-b6e98855cac1)

(실수로 `text-decoration: none;` 속성이 추가되었음...)

다음과 같이 직접 shadow tree에 콘텐츠를 추가해야 비로소 화면에 렌더링된다.

```js
const link = document.createElement('a');
link.href = shadowEl.querySelector('a').href;
link.innerHTML = `
  <span></span>
  ${shadowEl.querySelector('a').textContent}
`;

shadow.appendChild(link);
```

![before shadow style - dom](https://github.com/mochang2/development-diary/assets/63287638/3c3f1c7b-60e4-4e5d-9ec0-01dcc4e6d4da)

![before shadow style - screen](https://github.com/mochang2/development-diary/assets/63287638/0677d999-4959-403f-924e-f457a94f4046)

shadow tree에 스타일을 다음과 같이 적용할 수 있다.  
이 스타일은 글로벌 스타일에 영향을 받지 않는다.

```js
const styles = document.createElement('style');
styles.textContent = `
  a, span {
    vertical-align: top;
    display: inline-block;
    box-sizing: border-box;
  }
  a {
      height: 20px;
      padding: 1px 8px 1px 6px;
      background-color: #1b95e0;
      color: #fff;
      border-radius: 3px;
      font-weight: 500;
      font-size: 11px;
      font-family:'Helvetica Neue', Arial, sans-serif;
      line-height: 18px;
  }
  a:hover {  background-color: #0c7abf; }
  span {
      position: relative;
      top: 2px;
      width: 14px;
      height: 14px;
      margin-right: 3px;
      background: transparent 0 0 no-repeat;
      background-image: url(data:image/svg+xml,%3Csvg%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%20viewBox%3D%220%200%2072%2072%22%3E%3Cpath%20fill%3D%22none%22%20d%3D%22M0%200h72v72H0z%22%2F%3E%3Cpath%20class%3D%22icon%22%20fill%3D%22%23fff%22%20d%3D%22M68.812%2015.14c-2.348%201.04-4.87%201.744-7.52%202.06%202.704-1.62%204.78-4.186%205.757-7.243-2.53%201.5-5.33%202.592-8.314%203.176C56.35%2010.59%2052.948%209%2049.182%209c-7.23%200-13.092%205.86-13.092%2013.093%200%201.026.118%202.02.338%202.98C25.543%2024.527%2015.9%2019.318%209.44%2011.396c-1.125%201.936-1.77%204.184-1.77%206.58%200%204.543%202.312%208.552%205.824%2010.9-2.146-.07-4.165-.658-5.93-1.64-.002.056-.002.11-.002.163%200%206.345%204.513%2011.638%2010.504%2012.84-1.1.298-2.256.457-3.45.457-.845%200-1.666-.078-2.464-.23%201.667%205.2%206.5%208.985%2012.23%209.09-4.482%203.51-10.13%205.605-16.26%205.605-1.055%200-2.096-.06-3.122-.184%205.794%203.717%2012.676%205.882%2020.067%205.882%2024.083%200%2037.25-19.95%2037.25-37.25%200-.565-.013-1.133-.038-1.693%202.558-1.847%204.778-4.15%206.532-6.774z%22%2F%3E%3C%2Fsvg%3E);
  }
`;
shadow.appendChild(styles);
```

![after shadow style - dom](https://github.com/mochang2/development-diary/assets/63287638/e8670b58-223a-4865-909b-3facc8a06dbc)

![after shadow style - screen](https://github.com/mochang2/development-diary/assets/63287638/8f7daa75-2e6d-4a7f-a890-6a9476500417)

이때 다음과 같은 글로벌 스타일을 적용해보겠다.

```css
a {
  text-decoration: none;
}
```

![after global style - dom3](https://github.com/mochang2/development-diary/assets/63287638/edf0d046-82f7-46d5-916c-05bd0447cf8a)

![after global style -dom2](https://github.com/mochang2/development-diary/assets/63287638/3bb547cb-c11e-4c51-affe-76901195fe64)

![after global style - dom1](https://github.com/mochang2/development-diary/assets/63287638/bbf3b807-1b2f-49ff-a40a-db7dcf78f1c3)

![after global style - screen](https://github.com/mochang2/development-diary/assets/63287638/e0c3946f-6132-44ac-8077-6385c12a7e78)

화면에서 확인할 수 있다시피, shadow dom 내부에 있는 `<a>` 태그에는 `text-decoration: none;` 속성이 적용되지 않았다.

### vs React Portal

React Portal의 이점 중 하나가 메인 돔 외부에 엘리먼트 일부를 그림으로써 `App` 컴포넌트의 CSS 상속을 피하는 것이다.  
그래서 modal이나 popup 등이 글로벌 스타일 상속을 피할 수 있다.  
~그래서 둘을 비교하려고 했고 착각한 것 같다.~

결론부터 말하자면 React Portal은 shadow DOM을 사용하지 않았다.  
둘이 해결하려는 문제부터 다르기 때문에 적용되는 방식도 다르다.

- Shadow DOM
  - Shadow DOM은 웹 표준 기술로서, Web Components에 사용된다.
  - 일반적인 웹 개발에서 사용되는 기술이므로 React에 한정되지 않는다.
  - Shadow DOM은 캡슐화를 통해 웹 컴포넌트의 스타일과 동작을 외부로부터 격리시킨다. 외부에서 직접적으로 접근하는 것을 방지하는데 있어서 보다 강력한 캡슐화를 제공한다.
- React Portal
  - 컴포넌트를 다른 DOM 트리에 렌더링할 수 있도록 한다.
  - 보통 React에서는 컴포넌트들은 서로 자식-부모 관계에 있어야 하고, 가장 최상위에 있는 컴포넌트가 모든 자식 컴포넌트를 렌더링한다.
  - 하지만 때로는 특정 컴포넌트를 **다른 DOM 노드에 렌더링**해야 하는 경우가 있을 수 있다. 예를 들어 모달 창이나 팝업과 같은 UI 요소를 사용할 때 유용하다(컴포넌트가 물리적으로 다른 위치에 렌더링되더라도 컴포넌트 자체의 상태와 이벤트 처리를 유지할 수 있다).
  - `App` 컴포넌트의 CSS 상속을 피하는 것은 **스타일 시트를 어떻게 가져와서 사용하냐에 따라 다르다.** 경우에 따라서 피하지 못할 수 있다. 아래는 `App` 컴포넌트의 css를 상속받는 경우이다.

```css
/* app.css */

button {
  color: red;
}
```

```js
// App.js
import PortalModal from './components/Portal';
import './app.css';

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <button>abcdef</button>
        <PortalModal>Modal</PortalModal>
      </header>
    </div>
  );
}

export default App;
```

```js
// PortalModal.js

import ReactDOM from 'react-dom';

const Modal = ({ children }) => (
  <div className="Modal">
    {children}
    <button>Button</button>
  </div>
);

const PortalModal = (props) => {
  const modalRoot = document.querySelector('#modal-root');
  return ReactDOM.createPortal(<Modal {...props} />, modalRoot);
};

export default PortalModal;
```

![react portal dom](https://github.com/mochang2/development-diary/assets/63287638/77e53078-0909-4c7a-a39f-1a484a3fda11)

![react portal screen](https://github.com/mochang2/development-diary/assets/63287638/72472b56-c082-4c8a-b8d8-f72f0509de42)

`PortalModal.js`에서 `app.css` 속성 상속을 피하고 싶다면 별도의 스타일 시트를 가져오는 방식으로 바꾸던가 해야 한다.

## 3. `<template>`과 `<slot>`

`<template>`와 `<slot>`은 유연한 DOM 구조를 구현하게 해주는 elements이다.  
각각 단독으로 사용도 가능하지만, Shadow DOM과 함께 사용하면 재사용성 측면에서 주는 이점이 크다.

### `<template>`

[MDN](https://developer.mozilla.org/docs/Web/HTML/Element/template)에 따르면 `<template>`은 페이지를 불러온 순간 즉시 그려지지는 않지만, 이후 JS를 사용해 인스턴스를 생성할 수 있는 HTML 코드를 담을 방법을 제공한다.  
일반적으로 cloneNode을 이용해서 사용하여 HTML element를 복사할 수 있다.

```html
<template>
  <div
    class="f-center"
    style="
      display: flex;
      align-items: center;
      justify-content: center;
      width: 100px;
      height: 100px;
    "
  >
    <div>test box</div>
  </div>
</template>
<div class="shadow-host"></div>
```

위와 같이 선언하면 화면에 아무런 박스도 렌더링되지 않는다.  
먼저 `<template>`에 있는 내용을 복사하지 않고 직접 element를 생성하는 코드를 보겠다.

```js
const shadowEl = document.querySelector('.shadow-host');
const shadow = shadowEl.attachShadow({ mode: 'open' });

shadowEl.classList.add('f-center');

const innerBox = document.createElement('div');
innerBox.textContent = 'text box';
innerBox.style = `
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100px;
  height: 100px;
`;
shadow.appendChild(innerBox);
```

만약 element나 attribute가 더 있었다면 상당히 복잡했을 것이다.

하지만 `<template>`을 사용하면 아래와 같이 코드를 수정할 수 있다.

```js
const template = document.querySelector('template');

const shadowEl = document.querySelector('.shadow-host');
const shadow = shadowEl.attachShadow({ mode: 'open' });

shadow.appendChild(template.content.cloneNode(true));
```

특정 innerHTML만 변수의 입력을 받을 수 있도록 바꿔준다면(~바닐라 JS 직접 짜기 귀찮아서 생략...~) 더 많은, 동적인, 재사용이 가능한 컴포넌트를 만들 수 있을 것이다.

### `<slot>`

[MDN](https://developer.mozilla.org/docs/Web/HTML/Element/slot)에 따르면 `<slot>`은 웹 컴포넌트 사용자가 자신만의 마크업으로 채워 별도의 DOM 트리를 생성하고, 컴포넌트와 함께 표현할 수 있는 웹 컴포넌트 내부의 플레이스홀더이다.  
간단히 말하자면 정의한 `<slot>`에 해당 slot의 name 이 attribute로 설정된 요소를 끼워 넣는 데에 사용된다.

```html
<div class="shadow-host">
  <slot name="title">제목</slot>
  <slot name="content">내용</slot>
</div>
```

```js
const shadowEl = document.querySelector('.shadow-host');
const shadow = shadowEl.attachShadow({ mode: 'open' });

const title = document.createElement('div');
title.slot = 'title';
title.textContent = 'slot에 관하여...';

const content = document.createElement('div');
content.slot = 'content';
content.textContent = '웹 컴포넌트 없이 설명하기 쉽진 않네';

shadow.appendChild(title);
shadow.appendChild(content);
```

최종적으로 다음과 같은 모습으로 화면에 렌더링된다.

![slot - screen](https://github.com/mochang2/development-diary/assets/63287638/9b263eca-4e9e-44d6-a9d6-0c24f5fd2d9f)

_참고) 웹 컴포넌트 정의하기_  
`window.customElements.define` 메서드를 활용하여 웹 컴포넌트를 직접 정의할 수 있다.  
이것을 이용하면 컴포넌를 재사용성 있게 사용할 수 있다.  
(그리고 점차 Vue와 닮았다는 것을 느꼈다)

```html
<div class="shadow-host">
  <slot name="title">제목</slot>
  <slot name="content">내용</slot>
</div>
<my-component> </my-component>
```

```js
window.customElements.define(
  'my-component',
  class extends HTMLElement {
    constructor() {
      super();

      const shadowEl = document.querySelector('.shadow-host');
      this.root = shadowEl.attachShadow({ mode: 'open' });
    }

    connectedCallback() {
      this.root.innerHTML = `
        <div slot='title'>slot에 관하여...</div>
        <div slot='description'>웹 컴포넌트 있으니까 좀 와닿으려나</div>
      `;
    }
  }
);
```

`window.customElements.define`은 다음과 같은 세 가지 인자를 받는다.

- `name`: Name for the new custom element. Note that custom element names must contain a hyphen. (Vue에서도 한 단어로 컴포넌트를 정의하면(`myComponent`가 아닌 `Component`와 같은 네이밍) vue-eslint에서 에러를 발생시킨다. Vue과 웹 컴포넌트를 이용한 프레임워크이기 때문이다)
- `constructor`: Constructor for the new custom element.
- `options`(Optional): Object that controls how the element is defined. One option is currently supported:
