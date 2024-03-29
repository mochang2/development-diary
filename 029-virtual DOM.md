## 0. 공부하게 된 계기

[vanilla.js로 virtual DOM 만들기](https://junilhwang.github.io/TIL/Javascript/Design/Vanilla-JS-Virtual-DOM/#_1-%E1%84%87%E1%85%B3%E1%84%85%E1%85%A1%E1%84%8B%E1%85%AE%E1%84%8C%E1%85%A5-%E1%84%85%E1%85%A9%E1%84%83%E1%85%B5%E1%86%BC-%E1%84%80%E1%85%AA%E1%84%8C%E1%85%A5%E1%86%BC), [만들어 가며 알아보는 React: React는 왜 성공했나](https://techblog.woowahan.com/8311/)를 읽어보며 react에서의 virtual DOM이 중요한 요소라는 것을 깨달았고 직접 만들어보며 공부할 필요가 있다고 생각했다.

참고  
https://www.youtube.com/watch?v=PN_WmsgbQCo  
https://www.youtube.com/watch?v=1ojA5mLWts8  
https://ko.reactjs.org/docs/reconciliation.html  
https://velog.io/@juno7803/React-Reconciliation-%EC%9E%AC%EC%A1%B0%EC%A0%95  
https://basemenks.tistory.com/316  
https://velopert.com/3236

## 1. 브라우저의 동작

### DOM

DOM은 Document Object Model의 약자로 여기서 document는 HTML 문서를 말한다.  
DOM이란 HTML(input, span, div 등의 태그로 이루어진)이란 코드로 설계된 웹페이지가 브라우저 안에서 화면에 나타나고 이벤트에 반응하고 값을 입력받는 등 기능들을 수행할 객체들로 실체화된 형태이다.  
각종 `Node`들이 트리 구조를 이루고 있는 형태를 지닌다.  
여기서 `Node`란 모든 HTML element들이 상속받는 객체로, 추상 클래스와 같은 역할을 하는 것을 말한다.

DOM은 JS 객체는 아니며 DOM API를 이용하면 JS를 통해 제어가 가능하다(심지어 python도 beautifulsoup 라이브러리를 활용해 가능함).

### 브라우저 렌더링 과정

![rendering phase](https://user-images.githubusercontent.com/63287638/188289634-598c7290-78bb-4dd8-9b8c-9b565dfd90e8.png)

**parsing** -> **style** -> **layout** -> **painting** -> **composite**를 거친 후에 **rendering**된다.  
그 후 JS나 CSS를 통해 DOM이나 CSSOM에 변화가 생길 경우 **reflow** 또는 **repaint**의 과정을 수행한다.

참고로 브라우저에서 가장 성능이 중요한 부분이 이 렌더링 과정이다.

#### 1) Parsing + Style

브라우저가 HTML과 CSS를 파싱하고 읽어들이는 과정이다.  
HTML와 CSS을 파싱하여 node로 이루어진 트리를 생성하며 각각 생성하며 이를 DOM tree, CSSOM tree라고 부른다.  
이 두 가지를 합치면(DOM tree에 스타일을 매칭시켜주는 과정을 거치면) render tree라고 한다.

![render tree](https://user-images.githubusercontent.com/63287638/188289817-2549f2c3-06b1-40d6-b1f2-ec828e881446.png)

render tree에는 페이지 렌더링에 필요한 노드만 포함되어 `display: none;`과 같은 속성은 render tree에서 제외된다.  
단, `visibility: hidden;`은 레이아웃에서 공간을 차지한다.

#### 2) Layout

viewport 내에서 노드의 정확한 위치와 크기를 계산하는 과정이다.

re-rendering될 때를 **Reflow**라고 하며 다음과 같은 상황에서 발생한다.

- DOM의 추가/삭제
- CSS 속성에서 기하학적(높이/넓이/위치 등)의 변화 - margin, padding, width, height ...

#### 3) Paint

~래스터화라고도 함~

render tree의 각 노드를 화면의 실제 픽셀로 변환하는 단계이다.  
layout와 관계없는 CSS 속성을 적용하는 과정이다.  
픽셀로 변환된 결과는 포토샵의 레이어처럼 생성되어 개별 layer로 관리된다.

re-rendering될 때를 **Repaint**라고 하며 다음과 같은 상황에서 발생한다.

- CSS 속성에서 기하학적 변화가 발생하지 않은 경우 - color, background, transform ...

#### 4) Composite

화면에 표시하기 위해 페이지에서 페인트된 부분을 올바른 순서로 합치는 과정이다.  
transform, opacity, will-change 등을 사용했을 때 해당 과정을 거친다.

좋은 비유는 아니지만 Stacking Context와 관련이 있는 것처럼 받아들이면 이해가 된다.  
화면에 layer를 그리더라도 다른 layer 아래에 깔려서 안 보이는 부분이 있을 수 있다.  
이처럼 차곡차곡 painting을 쌓는 과정이다.

### DOM 조작의 비효율성

사용자와 어떠한 interaction을 통해 DOM에 변화가 발생하면 그때마다 render tree를 재생성하는 비효율성이 발생한다.  
위에서 이야기했듯이 performance에서 가장 큰 부분을 차지하는 것이 rendering 과정이고 이때 reflow, repaint도 다시하게 된다.

특히 SPA에서는 AJAX를 통해 페이지를 '서버'가 아닌'브라우저'에서 관리하므로 DOM 조작의 최적화가 필요해졌다.

## 2. virtual DOM

DOM을 virtual DOM과 구분 짓기 위해 real DOM이라고 사용하겠다.

![virtual DOM vs real DOM](https://user-images.githubusercontent.com/63287638/188290423-b312a25f-625c-4d71-a201-816bc8df40c6.png)

virtual DOM은 real DOM의 구조만 간결히 흉내낸 JS 객체이며 real DOM의 추상화 버전이다.  
virtual DOM은 real DOM과 같은 class 등의 속성들을 포함하지만 DOM API를 가지고 있지는 않다.

DOM 요소에 직접적으로 조작하면 느리기 때문에 (DOM API 등이 없어) 상대적으로 가볍고 브라우저에 종속적이지 않은 virtual DOM(old virtual DOM과 new virtual DoM)을 비교하여 변경된 내용만 real DOM에 적용한다.  
또다른 특징은 *변화를 모아서 한 번에 처리*하는 일종의 batch 작업으로, real DOM에서 100가지의 변화가 있다고 하더라고 레이아웃을 100번씩 변화시키는 것이 아니라 이러한 변화를 묶어서 한 번만 레이아웃을 변화시킨다(DOM fragment의 변화를 묶어서 적용한 다음 real DOM에 던져준다).  
이를 통해 브라우저 내에서 발생하는 렌더링 과정의 비효율성을 줄이면서 성능이 개선된다.  
다만 [아래](#4-vanilla-js%EB%A1%9C-react-poc-%EB%A7%8C%EB%93%A4%EA%B8%B0)에서 이야기하겠지만 batch를 위한 이 작업에서 반드시 virtual DOM이 필요한 것은 아니다.

### 동작 원리

virtual DOM은 html객체에 기반한 JS 객체로 표현된다.

```html
<ul id="items">
  <li>li1</li>
  <li>li2</li>
</ul>
```

위와 같은 HTML 코드가 virtual DOM에서는 아래와 같은 JS 객체로 표현된다.

```javascript
let domNode = {
  tagName: 'ul',
  props: { id: 'items' },
  children: [
    {
      tagName: 'li',
      textContent: 'li1',
    },
    {
      tagName: 'li',
      textContent: 'li2',
    },
  ],
};
```

virtual DOM은 메모리 상에서 동작하며 위 과정을 자동화, 추상화해놓은 것이다.  
virtual DOM 동작 과정을 정리하자면 다음과 같다.

1. 전체 virtual DOM이 업데이트된다.
2. virtual DOM을 업데이트 이전의 시점과 비교한다.
3. 실제로 바뀐 부분만 real DOM에서 바꾼다.
4. real DOM에서의 변화가 스크린에 그려진다.

## 3. React에서의 virtual DOM

react에서 virtual DOM이 하는 역할은 위에서 설명한 역할과 거의 비슷하다.

### 재조정(reconciliation)

[react 공식문서](https://ko.reactjs.org/docs/reconciliation.html)에 따르면

> virtual DOM (VDOM)은 UI의 이상적인 또는 “가상”적인 표현을 메모리에 저장하고 ReactDOM과 같은 라이브러리에 의해 “실제” DOM과 동기화하는 프로그래밍 개념입니다. 이 과정을 재조정이라고 합니다.

react에서 함수형 컴포넌트는 다음과 같은 내부 변환 과정을 거친다.

```javascript
// jsx
function Component() {
  return (
    <div className="box">
      <h1>hello world</h1>
    </div>
  );
}

// babel이 변환
function Component() {
  return React.createElement(
    'div',
    {
      className: 'box',
    },
    React.createElement('h1', null, 'hello world')
  );
}

// react 객체. 내부적으로 표현되는 방식
const element = {
  type: 'div',
  props: {
    className: 'box',
    children: [
      {
        type: 'h1',
        textContent: 'hello world',
      },
    ],
  },
};
```

위 `element`는 `ReactDOM.render`라는 함수에 의해서 실제 DOM 요소가 된다.  
react의 element는 DOM 요소의 가상 버전으로, light && immutable && stateless하다는 특징을 가지고 있다.

### diffing

```jsx
function ComponentA() {
  return (
    <div>
      <ComponentB />
    </div>
  );
}
```

react의 element는 불변하다는 속성을 가지고 있기 때문에 위 코드에서 `<ComponentB />` 를 `<ComponentC />`로 변경할 수 있는 유일한 방법은 새로운 요소를 만들어 `ReactDOM.render`로 전송하는 것 뿐이다.  
이때 virtual DOM을 사용해 바뀐 부분을 효율적으로 파악한다.

모든 react DOM object는 그에 대응하는 virtual DOM object가 존재한다.  
데이터가 변경되면 바뀐 데이터를 바탕으로 `React.createElement`를 통해 JSX element를 렌더링하고 이를 바탕으로 virtual DOM을 업데이트하고 이전 virtual DOM과 비교한다.
이러한 과정을 **diffing**이라고 한다.

#### diffing 알고리즘

react에서 state나 props가 갱신되면 `render` 함수는 새로운 react 엘리먼트 트리를 반환한다.  
이때 diffing 알고리즘이 동작한다.

기존에 트리를 비교하는 알고리즘은 아무리 빨라도 O(n^3)의 시간 복잡도를 지녔다.  
하지만 react는

1. 서로 다른 타입의 두 엘리먼트는 서로 다른 트리를 만들어낸다.
2. `key` prop을 통해, 여러 렌더링 사이에서 어떤 자식 엘리먼트가 변경되지 않아야 할지 표시해 줄 수 있다.

라는 가정에 기반하여 O(n)의 시간 복잡도를 지닌 휴리스틱 알고리즘을 구현했다.  
해당 가정들은 아래에서 더 자세히 설명하겠다.

O(n)의 시간 복잡도를 가진 diffing 알고리즘은 루트 노드부터 DOM tree를 bfs의 일종인 level-by-level로 탐색하여 html element 타입에 따라 다르게 행동한다.

**1) 비교하는 두 element 타입이 다른 경우**  
각 HTML 태그마다 자기만의 규칙이 있어서 그 아래 들어가는 자식 태그가 한정적이다.  
예를 들어 `<ul>`이나 `<ol>` 태그 바로 밑에는 `<li>` 태그만 담기게 되거나 `<img>`나 `<iframe>`는 자식 element를 절대 갖지 않는다.

만일 `<div>` 태그 안에 `<p>` 태그가 있었는데 `<div>`가 `<span>` 태그로 바뀐다면, 설령 자식 태그가 그대로라 할지라도 CSSOM(`display: block -> inline-block;`)이 변하기 때문에 렌더링 결과물은 어차피 달라진다.  
이런 점을 생각했을 때, 부모 태그의 타입이 바뀌는 순간 그 아래 모든 자식 태그들은 굳이 탐색할 것 없이 새로 render하는 편이 더 효율적이라고 가정한다.

물론 이러한 가정 때문에 비효율적일 수도 있다.  
아래와 같은 예시는 `<Counter />` 컴포넌트가 동일한데 `<div>`가 `<section>`으로 바뀌었다고 `<Counter />`도 re-render하기 때문이다.

```jsx
// before
<div>
  <Counter />
</div>

// after
<section>
  <Counter />
</section>
```

**2) 비교하는 두 element 타입이 같은 경우**

컴포넌트나 DOM 요소가 타입만 바뀌지 않는다면 react는 최대한 렌더링을 하지 않는 방향으로 최소한의 변경 사항만 업데이트 한다.  
예를 들어 `className`이 바뀌거나 `font-weight` 등의 CSS 속성이 바뀔 때 해당 속성만을 갱신하는 것이다.
이게 가능한 이유가 앞서 설명한 virtual DOM 덕분이다.

**3) 자식에 대한 재귀처리**

```jsx
function Component() {
  return (
    <ul>
      {array.map((item, index) => {
        return <li key={index}>{itme}</li>;
      })}
    </ul>
  );
}
```

위와 같은 코드에서 `<li>`안에 `key`라는 props를 전달해주지 않으면 react에서 경고를 발생시킨다.  
이는 diffing 알고리즘의 효율성과 관련이 있는 부분이다.

자식들을 처리할 때, react는 기본적으로 동시에 두 리스트를 차례대로 분석하여 차이점이 있으면 변경을 생성한다.
예를 들어, 자식의 끝에 엘리먼트를 추가하면, 두 트리 사이의 변경은 태그 하나를 추가하는 것으로 종료된다.
하지만 리스트의 맨 앞(또는 중간)에 엘리먼트를 추가하는 경우에는 다르다.  
마치 python에서 list(linked list가 아닌 자료구조)에 insert를 수행하는 작업과 같은 비효율성이 발생한다.

```html
<!-- before -->
<ul>
  <li>first</li>
  <li>second</li>
</ul>

<!-- after -->
<ul>
  <li>zero</li>
  <!-- first가 zero가 됐군. 새로 렌더링! -->
  <li>first</li>
  <!-- second가 first가 됐군. 새로 렌더링! -->
  <li>second</li>
  <!-- 추가됐네. 새로 렌더링! -->
</ul>
```

이러한 문제를 해결해주는 것이 `key` props이다.  
자식들이 `key`를 가지고 있다면, react는 `key`를 통해 기존 트리와 이후 트리의 자식들이 일치하는지 확인한다.

`key`를 사용하면 위 예시 코드가 아래와 같이 작동한다.

```html
<!-- before -->
<ul>
  <li key="a">first</li>
  <li key="b">second</li>
</ul>

<!-- after -->
<ul>
  <li key="c">zero</li>
  <!-- 추가됐네. 새로 렌더링! -->
  <li key="a">first</li>
  <!-- 아까 그 녀석이군. 순서만 바꾸자. -->
  <li key="b">second</li>
  <!-- 아까 그 녀석이군. 순서만 바꾸자. -->
</ul>
```

+) [추가사항](https://ko.reactjs.org/docs/lists-and-keys.html)
`key`는 오로지 형제 요소들 사이에서만 유일한 값이면 되고, 전역에서 유일할 필요는 없다.  
`key`로 사용되기 좋은 것은 unique한 id값이나 uuid 등(`stable attributes, which persist between re-renders`) 이고, 최후의 수단으로 배열의 index를 `key`로 사용할 수 있다.  
하지만 index를 사용하는 방법은 sort된 array에 새로운 값이 추가되거나 중간값이 삭제된 뒤 다시 sort가 필요한 경우 비효율적으로 작동한다.

_참고) vs Vue_  
위에서 이야기한 것처럼 React는 부모 엘리먼트가 변경되었다면 하위 모든 자식 엘리먼트를 re-rendering한다.  
반면 Vue는 변경된 부분만 re-rendering하고, 변경이 없는 요소는 이를 건너뛰어 성능을 최적화한다.  
이러한 이유로 Vue가 미세하지만 성능이 더 좋다고 이야기하는 것 같다.  
~나중에 Vue의 diffing update도 한 번 확인해보면 좋겠다.~

### performance

무조건 virtual DOM이 빠른 것이 아니다.  
정보 제공만 하는 웹페이지라면 인터렉션이 발생하지 않기 때문에 일반 DOM의 성능이 더 좋을 수도 있다.

SPA로 제작된 큰 규모의 웹 페이지라면 virtual DOM을 사용해서 브라우저 연산 양을 줄여 성능을 개선할 수 있고 그래서 react가 요새 가장 인기 있는 라이브러리인 것 같다.

### React에서의 렌더링

참고 및 사진 출처  
https://yceffort.kr/2022/04/deep-dive-in-react-rendering  
https://vercel.com/blog/how-react-18-improves-application-performance

react에서 렌더링은 컴포넌트가 현재 props와 state의 상태에 기초하여 UI를 어떻게 구성할지 컴포넌트를 호출한 후 재조정(Virtual DOM diffing -> renderer가 Real DOM에 이 내용을 삽입)하는 작업을 말한다.  
간단히 말하면 렌더링은 DOM 업데이트를 말하며, 따라서 가시적인 변경이 없어도 컴포넌트가 렌더링될 수 있다.

렌더링이 일어나는 동안, react는 컴포넌트의 루트에서 시작해 아래쪽으로 쭉 훑어보면서, 업데이트가 필요하다고 플래그가 지정되어 있는 모든 컴포넌트를 찾는다(diffing algorithm).  
만약 플래그가 지정되어 있는 컴포넌트를 만난다면, 클래스 컴포넌트의 경우 `classComponentInstance.render()`를, 함수형 컴포넌트의 경우 `FunctionComponent()`를 호출하고, 렌더링된 결과를 저장한다.

전체 컴포넌트에서 이러한 렌더링 결과물을 수집하고, 리액트는 새로운 오브젝트 트리와 비교하며, 실제 DOM을 의도한 출력처럼 보이게 적용해야 하는 모든 변경 사항을 수집한다.  
이렇게 비교하고 계산하는 과정을 위에서 말한 재조정(_reconciliation_)이라고 한다.  
즉, re-rendering될 때 재조정이 발생한다.

react는 이 단계를 의도적으로 다음 두 가지 단계로 분류하였다.

- Render phase: 컴포넌트를 렌더링하고 변경사항을 계산하는 모든 작업(element 추가, 수정, 삭제 등 reconciler가 컴포넌트의 변경을 DOM에 적용하기 위해 수행하는 일을 scheduler에 등록)
- Commit phase: 돔에 변경사항을 적용하는 과정(DOM 조작을 일괄처리한 후 React가 콜스택을 비움. 이후 브라우저가 paint 시작)

![react phases](https://github.com/mochang2/development-diary/assets/63287638/4d8dc6dc-82f7-46bc-9674-4ae3f2d7cc3d)

_cf) 권고하지 않는 사항_

모든 컴포넌트는 그 안에 또 다른 컴포넌트 선언을 가지고 있지 않는다.  
예를 들면 아래와 같은 상황처럼 말이다.

```jsx
function ParentComponent() {
  function ChildComponent() {
    return <div>I'm child</div>;
  }

  return <div>I'm parent</div>;
}
```

왜냐하면 렌더링할 때마다 매번 새로운 참조를 만들게 되어 매번 새로운 트리를 생성하는 비효율성이 발생하기 때문이다.

## 4. vanilla JS로 react PoC 만들기

다른 블로그들의 영감을 받아(?) ~사실 거의 비슷하게~ [react PoC](https://www.npmjs.com/package/vanilla-to-react)를 만들어봤다.  
덕분에 react의 특징을 조금이나마 알 수 있었다.

다음은 직접 만들며, 그리고 자료들을 찾아보며 알게 된 사실이다.  
직접 만들어 볼 생각을 하지 않았다면 평생 몰랐을 것 같다.

1. JSX는 가독성을 많이 높여준다(사실 이는 내가 JSX에 익숙해져서 그런걸지도 모르겠다).
2. `useState` 훅은 내부적으로 클로저 개념을 사용해 state를 관리한다(다만 언제 이 state가 필요가 없게 되어 삭제하고 메모리를 관리하는지는 모르겠다).
3. virtual DOM은 DOM의 경량화된 트리 형태의 객체이다.
4. react의 re-rendering 성능을 도와주는 것은 diffing algorithm이다.
5. virtual DOM 자체는 성능상 이점을 주지 않는다. 변경 사항을 한 번에 적용하는 batch도 `innerHTML`을 한 번만 적용하는 방식 등으로 작업해도 state 100번 변경에 대해 re-rendering을 1번만 하게 도와줄 수 있다.

4, 5와 관련해서, 그렇다면 virtual DOM으로 해결하려고 하는 것은 무엇인가?  
https://velopert.com/3236 에 의하면 다음과 같다.

> 그러면, virtual DOM 이 해결하려고 하는건 무엇이냐? DOM fragment를 관리하는 과정을 수동으로 하나하나 작업 할 필요 없이, 자동화하고 추상화하는거에요. 그 뿐만 아니라, 만약에 이 작업을 여러분들이 직접 한다면, 기존 값 중 어떤게 바뀌었고 어떤게 바뀌지 않았는지 계속 파악하고 있어야하는데 (그렇지 않으면 수정할 필요가 없는 DOM 트리도 업데이트를 하게 될 수도 있으니까요), virtual DOM이 이걸 자동으로 해주는 거에요. 어떤게 바뀌었는지, 어떤게 바뀌지 않았는지 알아내주죠.
> 마지막으로, DOM 관리를 virtual DOM 이 하도록 함으로써, 컴포넌트가 DOM 조작 요청을 할 때 다른 컴포넌트들과 상호작용을 하지 않아도 되고, 특정 DOM 을 조작할 것 이라던지, 이미 조작했다던지에 대한 정보를 공유할 필요가 없습니다. 즉, 각 변화들의 동기화 작업을 거치지 않으면서도 모든 작업을 하나로 묶어줄 수 있다는거죠.

아마 내가 ~실력이 좋아~ react PoC를 제대로 만들었다면 virtual DOM의 목적을 더 잘 이해할 수 있었을 것 같다.  
state가 변경되면 해당 컴포넌트부터 Virtual DOM을 비교하는 것이 아니라, app 최상단부터 파악하기 때문이다.  
아쉽지만 시간을 너무 잡아먹는 부분이라 여기까지만 알아보겠다.

+) renderer 부분을 보완하고자 한다면: [Fiber 아키텍처](https://bumkeyy.gitbook.io/bumkeyy-code/frontend/a-deep-dive-into-react-fiber-internals)

Fiber 아키텍처는 React의 내부 재귀 렌더링 알고리즘을 **비동기적**으로 작동하도록 변경했다.  
이를 통해 React는 작업을 나누고 우선 순위를 지정하여 더 효율적으로 작업을 수행할 수 있게 되었다.  
다른 말로 증분 렌더링이 가능해졌다고 한다.  
Fiber 아키텍처를 통해 React는 우선순위가 높은 작업을 먼저 처리하고(예를 들어 scroll 유저 인터렉션 -> button 클릭 -> fetch한 데이터 렌더링), 필요한 경우 중단하고 나중에 다시 시작할 수 있게 되었다.

(그 이전에는 Stack을 통해 재조정 햇는데, 동기적으로 하나의 큰 태스크를 실행했다. 이 때문에 이 콜 스택이 전부 처리되기 전까지는 메인 스레드가 다른 작업을 할 수 없었고, 앱은 일시적으로 무반응 상태가 되거나 버벅였다고 한다)

다음은 각각 ChatGPT가 알려준 Fiber 아키텍처 이전과 이후의 diffing update이다.

> 1. 컴포넌트 렌더링: React 컴포넌트의 렌더링 작업은 가상 DOM에 대한 변경 사항을 생성합니다. 이 변경 사항은 이전 렌더링 결과와 비교하여 업데이트해야 할 부분을 식별하는 데 사용됩니다.
> 2. 가상 DOM 비교: 이전 렌더링 결과와 현재 렌더링 결과를 비교하면서 두 가상 DOM 트리 사이의 차이점을 찾습니다. 이 과정에서 React는 노드의 유형, 속성, 자식 요소 등을 비교하여 변경된 부분을 식별합니다.
> 3. 업데이트 처리: 비교 과정을 통해 변경된 부분을 식별한 후, React는 해당 변경 사항을 실제 DOM에 반영합니다. 이전에는 변경 사항이 있는 모든 요소를 실제 DOM에 직접 업데이트했기 때문에 성능 문제가 발생할 수 있었습니다.

> 1. 렌더링 작업 분할: Fiber 아키텍처는 렌더링 작업을 여러 작은 단위인 "Fiber"로 분할합니다. 이러한 Fiber는 작업의 우선순위를 가지고 있어 중단하고 다시 시작할 수 있습니다. 이렇게 작업을 분할함으로써 React는 더 작은 단위의 작업을 처리하면서 렌더링 작업을 우선순위에 따라 조정할 수 있습니다.
> 2. 가상 DOM 비교: React는 두 가상 DOM 트리를 비교하여 변경 사항을 찾는 "Reconciliation" 과정을 수행합니다. 이전과 동일하게 노드의 유형, 속성, 자식 요소 등을 비교하여 변경된 부분을 식별합니다.
> 3. 업데이트 처리: 변경 사항이 식별되면 React는 이를 "작업 단위"로 스케줄링합니다. 작업 단위는 Fiber로 표현되며, 각각의 Fiber는 업데이트되어야 할 컴포넌트에 대한 정보와 업데이트 작업을 수행하는 메서드를 포함합니다.
> 4. 우선순위와 조화: React는 Fiber를 우선순위에 따라 처리하면서 중단하고 재개할 수 있습니다. 이를 통해 React는 브라우저의 렌더링 작업이나 다른 긴급한 작업을 위해 우선순위를 조정할 수 있습니다.
> 5. 업데이트 적용: React는 업데이트 작업을 실제 DOM에 적용합니다. 이때 변경된 부분만 실제 DOM에 업데이트되며, 전체 DOM을 다시 렌더링할 필요가 없습니다. 이를 "증분 업데이트"라고도 합니다.

_cf) `useTransition`, `useDeferredValue`_  
작업 우선 순위를 지정해서 업데이트를 진행할 수 있도록 도와주는 훅이 React 18에 나왔다.  
참고: https://velog.io/@ktthee/React-18-%EC%97%90-%EC%B6%94%EA%B0%80%EB%90%9C-useDeferredValue-%EB%A5%BC-%EC%8D%A8-%EB%B3%B4%EC%9E%90, https://react.dev/reference/react/useTransition
