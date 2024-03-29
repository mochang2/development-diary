## 0. 공부하게 된 계기

react에 대해 면접 전에 부랴부랴 외우기 보다 미리 '이해'하고 싶어서 정리하려고 한다.

react는 왜 등장했는가에 대해 여러 글을 읽고 나의 방식대로 정리해봤다.  
react 이전에는 SPA 프레임워크로서 angular가 존재했다.  
angular는 jQuery를 써서 DOM을 조작(느리고)했고 앱이 커질수록 user interaction에 대한 추적이 어려워졌으며 러닝 커브도 존재했다고 한다.
이에 불편함을 느껴 facebook(현 meta)에서 개발을 한 것이 react이며 다음과 같은 장점으로 현재 대세 라이브러리가 되었다.

1. (이거는 논란의 여지가 있지만) react는 라이브러리이며 단지 UI이다.  
   그래서 자율도가 높고 다른 분야에서도 활용할 수 있다.  
   react를 ReactNative 라이브러리와 결합시키면 앱을 만들 수 있으며, React360 라이브러리와 결합시키면 VR을, ReactElectron과 결합시키면 데스크톱 앱을 만들 수 있다.  
   따라서 자바스크립트를 기반으로 모바일, VR, 데스크톱 어떤 것이든 만들 수 있는 크로스 플랫폼 역할을 한다.
2. JSX를 이용해 선언적 프로그래밍이 가능하다.
3. diffing algorithm으로 빠르게 re-rendering 할 수 있다(이때 virtual DOM을 이용하지만 virtual DOM 자체가 re-rendering을 빠르게 하지는 않는다. [이는 virtual DOM 없이도 가능하지만 virtual DOM은 단순히 이를 추상화, 자동화한 것이다.](https://velopert.com/3236)).
4. component를 통해 재사용성과 독립성을 얻어냈다.

+) 추가사항 위 1번과 관련해서 React 패키지의 구성 요소를 알아보겠다.

React 패키지는 크게 다섯 가지로 구성되어 있다.

1. Core: Component가 정의되어 있다. 다른 패키지에 의존성이 없어서 다양한 플랫폼(브라우저, 모바일)에 올려서 사용 가능하다.
2. Renderer: `react-dom`, `react-native-renderer` 등 호스트 렌더링 환경에 의존한다. 아래 Reconciler와 Legacy-events 패키지와 의존성이 있다.
3. Event(Legacy-events): 기존 웹에서의 event를 wrapping하여 사용하기 쉽게 만든 모듈이다.
4. Scheduler: React에서 task(`setState` 등)를 비동기로 실행하기 위한 모듈이다.
5. Reconciler: 컴포넌트 호출 및 Virtual DOM 재조정(변경 사항 파악)을 담당한다. Renderer과 분리되어 있기 때문에 다양한 플랫폼에서 서로 다른 조합을 가지고 Core를 사용할 수 있다.

### 목차

- [명령형 프로그래밍 vs 선언형 프로그래밍](https://github.com/mochang2/development-diary/blob/main/030-react.md#1-%EB%AA%85%EB%A0%B9%ED%98%95-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8Dimperative-programming-vs-%EC%84%A0%EC%96%B8%ED%98%95-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8Ddeclarative-programming)
- [hook과 함수형 컴포넌트 vs 클래스형 컴포넌트](https://github.com/mochang2/development-diary/blob/main/030-react.md#2-hook%EA%B3%BC-%ED%95%A8%EC%88%98%ED%98%95-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-vs-%ED%81%B4%EB%9E%98%EC%8A%A4%ED%98%95-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8)
- [컴포넌트 합성](https://github.com/mochang2/development-diary/blob/main/030-react.md#3-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-%ED%95%A9%EC%84%B1)
- [포탈](https://github.com/mochang2/development-diary/edit/main/030-react.md#4-%ED%8F%AC%ED%83%88)
- [forwardRef](https://github.com/mochang2/development-diary/edit/main/030-react.md#5-forwardref)

## 1. 명령형 프로그래밍(imperative programming) vs 선언형 프로그래밍(declarative programming)

[React 공식 페이지](https://ko.reactjs.org/)에서는 react를 선언형이라고 설명한다.  
UI 라이브러리/프레임워크에서 '선언적'이라는 특징이 react만의 고유한 특징이 아니라, svelte, vue 등도 갖는 특징이다.

### 개념

명령형 프로그래밍은 코드로 원하는 결과를 달성해 나가는 **과정**에만 관심을 두는 프로그래밍 스타일이다.  
흔히들 'HOW'로 표현한다.

반면 선언형 프로그래밍은 필요한 것을 어떤 것인지 **기술**하는데 관심을 두는 프로그래밍 스타일이다.
흔히들 'WHAT'으로 표현한다.  
함수형 프로그래밍(`map`, `filter` 등의 추상화를 이용하는 고차함수)이 선언형 프로그래밍의 한 종류이다.

### 예시

아래 예시는 단순히 명령형 프로그래밍과 선언형 프로그래밍의 차이를 알기 위한 예시이다.

```javascript
// // 명령형 프로그래밍
// 원하는 것을 달성하는 '방법'에 중점
let string = 'THis is the midday show with Cheryl Waters';
let urlFriendly = '';

for (var i = 0; i < string.length; i++) {
  if (string[i] === ' ') {
    urlFriendly += '-';
  } else {
    urlFriendly += string[i];
  }
}

console.log(urlFriendly);

// // 선언형 프로그래밍
// 추상화를 이용
const string = 'This is the midday show with Cheryl Waters';
const urlFriendly = string.replace(/ /g, '-');

console.log(urlFriendly);
```

아래는 'hello world'를 쓰기 위한 실제 DOM 조작과 관련된 예시이다.

```javascript
// vanilla.js의 절차적 프로그래밍
const app = document.querySelector('#app');
const header = document.createElement('h1');
header.innerText = 'hello world';

app.appendChild(header);
```

```jsx
// react.js의 선언형 프로그래밍
function App() {
  return (
    <div id="app">
      <h1>hello world</h1>
    </div>
  );
}

// 바벨을 이용하지 않는다면
ReactDOM.render(<h1>hello world</h1>, document.querySelector('#app'));
```

(이상한 예시지만) 만약 위 코드에서 'hello'와 'world'를 각각의 `<h1>`으로 분리하고 싶다고 하자.  
vanilla.js는 아마 `const header2 = document.createElement('h1')`를 선언한 뒤 `appendChild`를 한 번 더 써야될 것이다.  
반면 react는 기존 `<h1>`태그를 수정한 뒤 아래에 `<h1>`태그를 추가해주면 된다.

### 선언형 프로그래밍의 장점

1. 가독성이 좋다(이거는 내가 단순히 jsx에 익숙해져서 그럴 수도 있는 것 같다).
2. 추상화를 이용하기 때문에 재사용하기 쉽다.
3. 결과 예측이 쉽다. 위 예시에서 vanilla.js는 최소 둘 이상의 변수가 필요하지만, react는 변수가 전혀 필요가 없다. 변수에 어떤 값들이 저장되었는지 트래킹을 해야되는데, 변수가 많이 사용되면 디버깅이 어려워진다.
4. 유지보수가 좋다(위 예시에서는 와닿지 않을 수도 있지만 많은 데이터를 갖는 object 형식을 사용한다고 생각하면 와닿을 것이다).

+) 추가사항  
만약 react에서 DOM을 변경할 때 기존 element를 완전히 제거하고 새로운 element를 생성한다면 다음과 같은 문제가 발생한다.

1. performance가 저하한다.
2. 화면 깜빡임 발생하여 UX가 좋지 않다.

이를 해결하기 위해 react에서는 [**virtual DOM**](https://github.com/mochang2/development-diary/blob/main/029-virtual%20DOM.md)과 **reconciliation**이라는 개념을 도입했다.

## 2. hook과 함수형 컴포넌트 vs 클래스형 컴포넌트

[react 공식문서](https://ko.reactjs.org/docs/hooks-overview.html)에서 effect hook에 대해 side effect를 수행하는 훅이라고 설명한다.

> Effect Hook, 즉 `useEffect`는 함수 컴포넌트 내에서 side effects를 수행할 수 있게 해줍니다. React class의 `componentDidMount` 나 `componentDidUpdate`, `componentWillUnmount`와 같은 목적으로 제공되지만, 하나의 API로 통합된 것입니다.

hook에 대해 본격적으로 들어가기 전에 side effect는 무엇인지 먼저 알아봤다.

### Side Effect

side effect라는 말은 순수 함수에서 나온 개념이다.  
순수 함수는 같은 input에 대해 항상 같은 output을 내뱉는 함수를 말한다~그 외의 특징이 더 있지만 여기서 기술하는 것은 함수형 프로그래밍이 아니므로 패스~  
따라서 함수형 프로그래밍에서 사용하는 순수 함수는 예측 가능하며 테스트가 쉽다는 특징이 있다.
side effect는 부작용이라는 뜻으로, **순수 함수에서 결과값을 예측하기 어렵게 만드는 외부 환경 등**을 말한다.

react에서도 크게 다른 뜻이 아니다.  
react 컴포넌트 안에서 데이터를 가져오거나 구독하고, DOM을 직접 조작하는 작업을 모든 작업을 side effect(또는 짧게 effect)라고 한다.  
왜냐하면 이것은 다른 컴포넌트에 영향을 줄 수도 있는 부분이며, 렌더링 과정에서는 구현할 수 없는 작업이기 때문이다.

side effect는 다음 사항들을 포함한다.

- BE에 API 요청하는 행위
- `document`나 `window`를 조작하는 browser API 사용하는 행위
- `setTimeout`이나 `setInterval` 등 예측 불가능한 timer API를 사용하는 행위

```jsx
function App({ name }) {
  document.title = name;
  // This is a side effect. Don't do this in the component body!

  return <h1>{name}</h1>;
}
```

위와 같이 side effect를 컴포넌트 내부에서 바로 동작시킨다면 컴포넌트 렌더링 과정에 영향을 끼치게 된다.  
이는 react rendering 규칙 중 하나인 **렌더링은 '순수'해야 하고 side effect가 없어야 한다**는 것을 무시한 것이다.  
따라서 side effect는 컴포넌트 렌더링 과정과 '분리'되어야 한다.  
해당 기능을 제공해주는 것이 `useEffect`와 같은 hook이다.

_rendering 로직 참고_  
렌더링 로직은 다음과 같은 행위를 할 수 없다.

- 존재하는 변수나 객체 변경
- `Math.random()` `Date.now()`와 같은 랜덤 값 생성
- 네트워크 요청
- state를 업데이트

_useEffect 참고_  
`useEffect` hook은 react에게 컴포넌트가 렌더링 이후에 어떤 일을 수행해야 하는지를 말해준다.  
react는 hook을 통해 넘긴 함수(effect)를 기억했다가 DOM 업데이트를 수행한 이후에 해당 함수를 호출한다.

`useEffect`에 전달되는 함수는 모든 렌더링마다 다른데, state가 제대로 업데이트 되는지에 대한 걱정 없이 effect 내부에서 그 값을 읽도록 하기 위해 의도된 부분이라고 한다.  
덕분에 리렌더링할 때마다 이전과 다른 effect로 교체된 상태를 전달할 수 있다.

### 함수형 컴포넌트로 전환된 이유

현재는 모든 react 코드가 함수형으로 작성되고 있다.  
그 이유는 무엇일까?

#### `this`를 사용하지 않음

클래스형 컴포넌트는 state를 다룰 때 항상 `this.`로 시작한다.

```jsx
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0,
    };
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }
  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```

```jsx
import { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

위에 쓴 각각의 컴포넌트는 결과적으로 같은 동작을 하며 UI를 그린다.  
하지만 클래스형 컴포넌트가 훨씬 많은 양의 코드가 필요하며 (위에 나온 예시는 아니지만)JS에서 위치에 따라 `this`가 다르게 동작하기 때문에 이벤트 핸들러에 함수 바인딩 과정도 복잡했다.  
반면 함수형 컴포넌트는 훨씬 코드가 간단하며 함수 바인딩이 편해졌다.

#### 생명주기가 없음

![lifecycle](https://user-images.githubusercontent.com/63287638/189896667-fab16e4b-7e79-4967-bcf7-69c8cd5b88a1.png)

위 사진은 클래스형 컴포넌트에서 이용했던 컴포넌트 생명 주기를 나타낸 것이다.  
`componentDidMount`는 컴포넌트가 마운트된 후, `componentDidUpdate`는 컴포넌트가 업데이트된 후, `componentWillUnmount`는 컴포넌트가 언마운트되기 직전에 실행된다.  
클래스형 컴포넌트가 버그 없이 의도한대로 동작하기 위해서는 위에 나온 것들을 올바르게 이해하고 사용했어야만 했다.

반면에 함수형 컴포넌트는 생명 주기 함수가 따로 존재하지 않는다.  
이유는 '모든 라이프 사이클 훅은 `React.Component`에 존재하기 때문'이다.  
클래스형 컴포넌트는 `extends`로 `React.Component`에 접근할 수 있었지만 함수형 컴포넌트는 그렇지 않다.

대신 함수형 컴포넌트는 공식문서에 나왔듯이 `useEffect` 훅을 이용해 `componentDidMount` 나 `componentDidUpdate`, `componentWillUnmount`를 대체할 수 있다.

#### 렌더링 성능

참고: https://djoech.medium.com/functional-vs-class-components-in-react-231e3fbd7108 , https://betterprogramming.pub/react-class-vs-functional-components-2327c7324bdd

[과거 react 공식문서](https://reactjs.org/blog/2015/10/07/react-v0.14.html#stateless-functional-components)에서 미래에는 함수형 컴포넌트의 performance 향상을 기대한다~ 라고 이야기했다.  
이것 때문에 계속 찾아봤는데 성능이 다르다는 결과가 없었다...
https://jsfiddle.net/69z2wepo/136096/ 에서 렌더링 속도도 측정했으나 서로 빠른 정도가 왔다갔다한다.

만~약 함수형 컴포넌트와 클래스형 컴포넌트 간의 성능 차이가 나더라도 구조로 인한 성능 차이 더 크다.  
따라서 렌더링 성능면에서는 함수형인지 클래스형인지를 신경쓸 필요는 없는 것 같다.

#### 번들 사이즈

간결함으로 인한 이점이 하나 더 있다.  
바로 번들 사이즈가 줄어드는 것이다.

react, redux, CRA의 저자인 Dan Abramov가 [본인의 블로그](https://medium.com/@dan_abramov/making-sense-of-react-hooks-fdbde8803889)에서 언급한 내용이다.

> In terms of the implementation size, the Hooks support increases React only by ~1.5kB (min+gzip). While this isn’t much, it’s also likely that adopting Hooks could reduce your bundle size because code using Hooks tends to minify better than equivalent code using classes.

이 글을 보고 실제로 4개의 컴포넌트를 함수형 컴포넌트와 클래스형 컴포넌트로 작성한 내용에 대해 빌드를 해봤다(react v18.2, webpack v5).

![build bundle sizes](https://user-images.githubusercontent.com/63287638/194302539-36cdbdcf-ec95-40c9-a0b4-dcc733ab8775.png)

위 사진이 함수형 컴포넌트의 번들 결과물(181,471 byte), 아래 사진이 클래스형 컴포넌트의 번들 결과물(182,957 byte)이다.

바벨로 컴파일한 결과를 통해 봤을 때도 확연한 차이가 있다.

```jsx
// 함수형 컴포넌트
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

// 컴파일 결과
function Welcome(props) {
  return _react2.default.createElement('h1', null, 'Hello, ', props.name);
}
```

```jsx
// 클래스형 컴포넌트
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}

// 컴파일 결과
var Welcome = (function (_React$Component) {
  _inherits(Welcome, _React$Component);

  function Welcome() {
    _classCallback(this, Welcome);

    return _possibleConstructorReturn(
      this,
      (Welcome.__proto__ || Object.getPrototypeOf(Welcome)).apply(
        this,
        arguments
      )
    );
  }

  _createClass(Welcome, [
    {
      key: 'render',
      value: function render() {
        return _reaact2.default.createElement(
          'h1',
          null,
          'Hello, ',
          this.props.name
        );
      },
    },
  ]);

  return Welcome;
})(_react2.default.Component);
```

결과적으로 번들 사이즈 측면에서는 함수형 컴포넌트가 유리하다.

### Hook

react v16.8에서 hook이라는 개념이 등장하면서 함수형 컴포넌트가 대세가 되었다.  
이전에도 함수형 컴포넌트는 존재했으나 그 당시의 함수형 컴포넌트는 단순한 JS 함수로서, stateless한 컴포넌트였다.  
즉, 클래스형 컴포넌트처럼 `this.state`와 같은 문법을 사용할 수 없었다.  
그래서 그 당시에 굳이 함수형 컴포넌트를 쓰고 싶었다면 별도로 클래스형 컴포넌트를 생성하거나 부모 컴포넌트로 'lift the state up'한 뒤 자식 컴포넌트에서 받아쓰는 방법으로 구현해야 됐다고 한다.

`useState` 훅이 등장하면서 함수형 컴포넌트에서 별도로 state 관리가 가능해졌고 'lift the state up'과 같은 방법을 사용하지 않을 수 있게 되었다.  
`useState`는 JS의 [클로저](https://github.com/mochang2/development-diary/blob/main/025-fundamentals%20of%20js.md#3-closure)라는 개념을 활용하여 구현했다고 한다.

_추가 사항_  
참고  
https://overreacted.io/why-do-hooks-rely-on-call-order/  
https://pozafly.github.io/react/react-is-managing-hooks-as-an-array/

`useState`는 state를 배열(정확히 말하면 Linked List)에 넣어서 관리한다.  
그렇다면 왜 배열일까?  
고유한 key값이 있는 객체나 `Map`은 안될까?

```js
const [value1, setValue1] = useState(true);
const [value2, setValue2] = useState(true);
```

위와 같이 선언하면 사용자는 `value1`, `value2` 각각이 가리키는 값이 무엇인지 안다.  
하지만 React 입장에서는 state를 index로 접근 가능한 배열 형태로 관리하지 않는다면, `value1`, `value2`를 구분할 키값이 필요하다.  
아래와 같이 말이다.

```js
const [value1, setValue1] = useState('value1', true);
const [value2, setValue2] = useState('value2', true);
```

`useState`는 정말 많이 사용되는 훅인데 매번 key값이 중복되는지, 아닌지 체크하다보면 편의성이 떨어질 것이다.  
Context API 등에서도 key 값이 같으면 더 먼 Provider에서 제공된 값이 덮어씌어지는 것처럼 [믹스인 다중 상속 문제](https://pozafly.github.io/react/react-is-managing-hooks-as-an-array/#%EB%AF%B9%EC%8A%A4%EC%9D%B8-%EB%8B%A4%EC%A4%91-%EC%83%81%EC%86%8D-%EB%AC%B8%EC%A0%9C%EB%8B%A4%EC%9D%B4%EC%95%84%EB%AA%AC%EB%93%9C-%EB%AC%B8%EC%A0%9C)가 발생하는 것을 피해야 하기 때문이다.

그렇다면 JS의 `Symbol` 기능을 선택한다면 괜찮지 않을까?

```js
const symbol1 = Symbol();

function useCustomHook() {
  const [value, setValue] = useState(symbol1);
  return [value, setValue];
}
```

React는 동일한 로직을 반복해서 선언하지 않도록 custom hook 기능을 제공한다.  
하지만 `Symbol`을 사용한다면 위와 같이 `Symbol`이 custom hook이 외부에서 선언되기 때문에 `useCustomHook`을 여러 곳에서 반복적으로 호출할 수 없다.  
`useCustomHook`을 사용하는 각각의 컴포넌트가 다른 value를 가지고 있어야 되는데 모두 동일한 value를 가리키게 되기 때문이다.

마지막으로 hook 간에 데이터 전달이 불가능해진다.  
[React Spring](https://medium.com/@drcmda/hooks-in-react-spring-a-tutorial-c6c436ad7ee4)이라는 서로 연쇄되는 값을 이용해 애니메이션을 만들 수 있는 모듈이 있다.

```js
const [{ pos1 }, set] = useSpring({ pos1: [0, 0], config: fast });
const [{ pos2 }] = useSpring({ pos2: pos1, config: slow });
const [{ pos3 }] = useSpring({ pos3: pos2, config: slow });
```

key값을 이용해 state를 관리한다면 같은 훅 간에 데이터를 전달하고 연쇄되는 동작을 표현할 수 없다.

이러한 이유 때문에 React 팀은 개발자가 key값을 관리할 필요 없는 배열을 선택한 것 같다.

_참고) hook 사용 규칙_

1. 최상위(at the Top Level)에서만 hook을 호출해야 한다. 컴포넌트가 렌더링 될 때마다 항상 동일한 순서로 hook이 호출되기 위해서는 반복문, 조건문 또는 중첩된 함수 내에서 hook을 호출하면 안 된다.
2. react 함수 내에서만 hook을 호출해야 한다. 다만 react 함수 컴포넌트나 custom hook 내부에서 hook을 호출할 수는 있다. 해당 규칙을 지킨다면 모든 상태 관련 로직을 소스코드에서 명확하게 보이도록 코드를 짤 수 있다.

## 3. 컴포넌트 합성

특정 컴포넌트들은 어떤 자식 엘리먼트가 들어올지 미리 예상할 수 없는데, 이를 '일반화'(맞는 표현일지는 모르겠지만) 시킴으로써 재사용 가능하게 만들 수 있다.  
특히 `props.children`을 이용하여 `Sidebar`나 `Dialog`를 만들 때 유용한 방식이다.

> react는 render시 jsx 코드를 `React.createElement(component, props, ...children)`를 이용해 변환한다는 점을 이용한다.

아래는 대시보드에서 페이지에 고정적으로 사용되는 `VerticalNavBar`를 이용하여 컴포넌트 합성을 진행한 예시이다.

```jsx
// typescript를 사용한다면 children type에 대해서 https://itchallenger.tistory.com/394 참고

// Content.jsx
const Wrapper = styled.div`
  padding-left: ${({ paddingLeft }) => paddingLeft};
`;
function Content({ paddingLeft, children }: ContentProps) {
  return <Wrapper paddingLeft={paddingLeft}>{children}</Wrapper>;
}
```

```jsx
// Header.jsx
const Wrapper = styled.div`
  padding: ${({ paddingLeft }) => `20px 4vw 30px ${paddingLeft}`};
`;
function Header({ paddingLeft }) {
  return <Wrapper paddingLeft={paddingLeft}>header</Wrapper>;
}
```

```jsx
// VerticalNavBar.jsx
const Wrapper = styled.div`
  width: ${({ width }) => width};
`;
function VerticalNavBar({ width }) {
  return <Wrapper width={width}>{/* some content */}</Wrapper>;
}
```

```jsx
// VerticalLayout.jsx
import Content from 'components/common/Content';
import Header from './Header';
import VerticalNavBar from './VerticalNavBar';

const NAV_BAR_WIDTH = '326px';

function VerticalLayout({ children }) {
  return (
    <React.Fragment>
      <VerticalNavBar width={NAV_BAR_WIDTH} />
      <Header paddingLeft={NAV_BAR_WIDTH} />
      <Content paddingLeft={NAV_BAR_WIDTH}>{children}</Content>
    </React.Fragment>
  );
}
```

```jsx
// Page.tsx
import { VerticalLayout } from 'components/layout';

// 합성된 컴포넌트 이용
function Page() {
  return <VerticalLayout>{/* Page 내용 */}</VerticalLayout>;
}
```

`props.children` 자체에 새로운 props를 전달해주고 싶을 때가 있지만 만약 그렇다면 구조를 잘 짰는지 다시 생각해보자.  
기본적으로 props는 read-only이므로 `{children}` 이외의 방식은 `children`의 props를 변형시킬 수 있다.  
도저히 답이 안 나오면 `React.cloneElement`를 사용할 수 있으며 사용법은 https://velog.io/@hyunjoogo/React-children-Component%EC%97%90-props-%EC%A0%84%EB%8B%AC%ED%95%98%EA%B8%B0 와 https://www.delftstack.com/ko/howto/react/react-pass-props-to-children/ 를 참고하자.

컴포넌트 합성시 반드시 `props.children`을 사용해야 하는 것은 아니다.  
아래는 [공식문서](https://ko.reactjs.org/docs/composition-vs-inheritance.html)의 예시이다.

```jsx
// general component
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">{props.title}</h1>
      <p className="Dialog-message">{props.message}</p>
    </FancyBorder>
  );
}

// specific component
function WelcomeDialog() {
  return (
    <Dialog title="Welcome" message="Thank you for visiting our spacecraft!" />
  );
}
```

### _cf) HOC(Higher Order Component)_

참고: https://ko.reactjs.org/docs/higher-order-components.html , https://velopert.com/3537 , https://www.robinwieruch.de/react-higher-order-components/

HOC란 컴포넌트를 입력으로 받아 새 컴포넌트를 반환하는 함수이다.  
react의 naming convention으로 `with`로 함수의 이름이 시작하며 Redux의 `connect`와 같은 라이브러리에서도 자주 사용되는 방식이다.  
hook이 현재 react에서 대세지만 HOC가 완전히 사라진 것은 아니며 사용법을 익혀둘 필요가 있다.

주의할 점은 다음과 같다.

- HOC 내부에서 원본 컴포넌트를 변경하지 않도록 해야 한다.
- render 메서드 안에서 사용하면 안된다(렌더링될 때마다 계속해서 새로운 컴포넌트를 만들어내므로).
- static method는 복사되지 않기 때문에 따로 복사해서 넣어줘야 한다(오픈 라이브러리를 가끔 보면 `hoistStatics`와 같은 함수가 해당 역할을 해주는 코드를 볼 수 있음).
- ref는 전달되지 않으므로 `React.forwardRef`라는 기능을 사용해야 한다.

HOC의 장점은 다음과 같다.

1. **재사용**이 가능하다.  
   HOC에서 `class` 자체를 return 함수로써 클래스형 컴포넌트에서 사용 가능한 예제이다.
   다음 코드 예제를 보면 `CommentList` 컴포넌트와 `BlogPost` 컴포넌트가 하는 역할은 똑같다.

```jsx
// DataSource: localStorage와 같은 전역 데이터 소스

class CommentList extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      comments: DataSource.getComments(),
    };
  }

  componentDidMount() {
    // 변화감지를 위해 리스너를 추가
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    // 리스너를 제거
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    // 데이터 소스가 변경될때 마다 comments를 업데이트
    this.setState({
      comments: DataSource.getComments(),
    });
  }

  render() {
    return (
      <div>
        {this.state.comments.map((comment) => (
          <Comment comment={comment} key={comment.id} />
        ))}
      </div>
    );
  }
}

class BlogPost extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      blogPost: DataSource.getBlogPost(props.id),
    };
  }

  componentDidMount() {
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    this.setState({
      blogPost: DataSource.getBlogPost(this.props.id),
    });
  }

  render() {
    return <TextBlock text={this.state.blogPost} />;
  }
}
```

위 두 클래스를 동시에 다룰 HOC를 선언하고 사용한다.

```jsx
function withSubscription(WrappedComponent, selectData) {
  return class extends React.Component {
    constructor(props) {...}
    componentDidMount() {...}
    componentWillUnmount() {...}
    handleChange() {...}

    render() {
      // ... 래핑된 컴포넌트를 새로운 데이터로 랜더링
      // 컴포넌트에 추가로 props를 내려줄 수 있음
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}

// 새로운 컴포넌트 반환
const CommentListWithSubscription = withSubscription(
  CommentList, // 해당 컴포넌트는 비즈니스 로직이 빠진 컴포넌트
  (DataSource) => DataSource.getComments()
);

const BlogPostWithSubscription = withSubscription(
  BlogPost, // 해당 컴포넌트는 비즈니스 로직이 빠진 컴포넌트
  (DataSource, props) => DataSource.getBlogPost(props.id)
);
```

2. **conditinal rendering**이 가능하며 HOC를 통해 conditional rendering 로직을 분리할 수 있다.
   함수형 컴포넌트에서 사용하는 방식으로  
   `const multiply = (multiplier) => (multiplicand) => multiplicand * multiplier`  
   `const product = multiply(3)(4)`와 같은 원리를 이용한다.  
    다음과 같이 `TodoList` 컴포넌트를 렌더링하고 싶은 `App` 컴포넌트가 있다고 하자.

```jsx
const fetchData = () => {
  return { data: TODOS };
};

const App = () => {
  const { data } = fetchData();

  return <TodoList data={data} />;
};
```

여기서

1. `data`가 null일 때
2. 빈 배열일 때
3. 로딩 중일 때

등등 모든 예외를 처리한다면 `App` 컴포넌트는 다음과 같아야 된다.

```jsx
const fetchData = () => {
  return { data: null, isLoading: true };
};

const App = () => {
  const { data, isLoading } = fetchData();

  if (isLoading) return <div>Loading data.</div>;
  if (!data) return <div>No data loaded yet.</div>;
  if (!data.length) return <div>Data is empty.</div>;

  return <TodoList data={data} />;
};
```

이를 HoC를 이용하면 로직 처리를 분리함으로써 좀더 우아하게 바꿀 수 있다.

```jsx
const withConditionalFeedback = (Component) => (props) => {
  if (props.isLoading) return <div>Loading data.</div>;
  if (!props.data) return <div>No data loaded yet.</div>;
  if (!props.data.length) return <div>Data is empty.</div>;

  return <Component {...props} />;
};

const TodoList = withConditionalFeedback(BaseTodoList);
// BaseTodoList는 위 예시의 TodoList에서 같은 컴포넌트

const App = () => {
  const { data, isLoading } = fetchData();

  return <TodoList data={data} isLoading={isLoading} />;
};
```

다음과 같은 방식으로 configuration을 추가할 수도 있다.

```jsx
const withConditionalFeedback =
  ({ loadingFeedback, noDataFeedback, dataEmptyFeedback }) =>
  (Component) =>
  (props) => {
    if (props.isLoading) return <div>{loadingFeedback || 'Loading data.'}</div>;

    if (!props.data)
      return <div>{noDataFeedback || 'No data loaded yet.'}</div>;

    if (!props.data.length)
      return <div>{dataEmptyFeedback || 'Data is empty.'}</div>;

    return <Component {...props} />;
  };

// ...

const TodoList = withConditionalFeedback({
  loadingFeedback: 'Loading Todos.',
  noDataFeedback: 'No Todos loaded yet.',
  dataEmptyFeedback: 'Todos are empty.',
})(BaseTodoList);
```

마지막으로 `loadingFeedback`, `noDataFeedback`, `dataEmptyFeedback` 자체를 HOC로 만들 수 있다.

```jsx
const withLoadingFeedback = (Component) => (props) => {
  if (props.isLoading) return <div>Loading data.</div>;
  return <Component {...props} />;
};

const withNoDataFeedback = (Component) => (props) => {
  if (!props.data) return <div>No data loaded yet.</div>;
  return <Component {...props} />;
};

const withDataEmptyFeedback = (Component) => (props) => {
  if (!props.data.length) return <div>Data is empty.</div>;
  return <Component {...props} />;
};

const compose = (...fns) =>
  fns.reduceRight(
    (prevFn, nextFn) =>
      (...args) =>
        nextFn(prevFn(...args)),
    (value) => value
  );

const TodoList = compose(
  withLoadingFeedback,
  withNoDataFeedback,
  withDataEmptyFeedback
)(BaseTodoList);

// compose는 다음과 같은 결과를 return한다
const TodoList = withLoadingFeedback(
  withNoDataFeedback(withDataEmptyFeedback(BaseTodoList))
);
```

+) 참고로 [공식문서](https://ko.reactjs.org/docs/composition-vs-inheritance.html)에서 '합성'은 좋은 구조이지만 '상속'을 사용하는 좋은 사례는 발견하지 못했다고 한다.

## 4. 포탈

참고  
https://jeonghwan-kim.github.io/2022/06/02/react-portal  
https://velog.io/@dfd1123/react-create-portal-%EC%97%90-%EB%8C%80%ED%95%98%EC%97%AC  
https://ko.reactjs.org/docs/portals.html

> Portal은 부모 컴포넌트의 DOM 계층 구조 바깥에 있는 DOM 노드로 자식을 렌더링하는 최고의 방법을 제공합니다.

어플리케이션 마운트되는 위치를 이동한다는 의미이다.

우형 개발자 [김정환님 블로그](https://jeonghwan-kim.github.io/2022/06/02/react-portal)에서 _모달과 같은 경우 메인 어플리케이션 UI 컨텍스트에 영향을 주지 않기 때문에 다른 돔에서 모달 앨리먼트가 변하더라도 메인 어플리케이션이 돌아가는 돔에는 영향을 주지 않을 것_ 이라고 생각해 portal이 유의미하다고 생각했다고 한다.  
하지만 모달을 보여줄지 말지 state 관리는 보통 `App` 컴포넌트 하위에서 관리하기 때문에 개인적으로는 성능과 관련되어 유의미한 기능은 아니라고 생각한다.

다른 이점이 있다면 메인 돔 외부에 엘리먼트 일부를 그림으로써 `App` 컴포넌트의 CSS 상속을 피하고, React 어플리케이션 컴포넌트 트리 구조를 따라 이벤트 버블링도 사용할 수 있다.

```html
<!-- index.html -->
<body>
  <noscript>You need to enable JavaScript to run this app.</noscript>
  <div id="root"></div>
  <div id="modal-root"></div>
  <!-- 추가 -->
</body>
```

```js
// App.js
import ReactDOM from 'react-dom';

const Modal = ({ children }) => (
  <div className="Modal">
    {children}
    <button>Button</button>
  </div>
);

const PortalModal = (props) => {
  const modalRoot = document.querySelector('#modal-root'); // 모달이 마운트 될 엘리먼트.
  // 모달 앨리먼트를 modalRoot에 마운트.
  return ReactDOM.createPortal(<Modal {...props} />, modalRoot);
};

function App() {
  const handleClick = (e) => {
    // button에서 발생한 이벤트 버블링됨.
    console.log(e.target); // <button>
  };

  return (
    <div className="App" onClick={handleClick}>
      App
      <PortalModal>Modal</PortalModal>
    </div>
  );
}

export default App;
```

### vs shadow DOM

혹시 둘이 뭐가 다른가 싶으면 [여기](https://github.com/mochang2/development-diary/blob/main/050-shadow%20dom.md#vs-react-portal)를 확인하자.

## 5. forwardRef

일반적인 어플리케이션에서 사용하는 기능은 아니다.  
보통 라이브러리 개발자들이 많이 사용한다고 한다.

일반 함수나 클래스 컴포넌트는 ref를 인자로 받지도 않고, props에서도 사용할 수 없다.  
하지만 `forwardRef` API를 활용하면 하위 컴포넌트의 두 번째 인자로 ref가 입력되어 ref를 상위에 전달할 수 있다.

아래는 간단한 예시이다.

```jsx
// Input.jsx

import { forwardRef } from 'react';

export default forwardRef(function (props = {}, ref) {
  return (
    <div>
      <p>Input</p>
      <input ref={ref} />
    </div>
  );
});
```

## 6. flushSync

[react state 2번](https://github.com/mochang2/development-diary/blob/main/032-react%20state.md#props-vs-state)에서 알아봤듯이 state는 비동기적으로 변경된다.  
이로 인해 API 호출 시 문제가 발생한 적이 있어, 해당 기억을 기록하고자 한다.

```tsx
'use client';

// import ...

type TResponse = {
  totalCount: number;
  projects: TProject[];
};

const INITIAL_PROJECT_DATA: TResponse = {
  totalCount: 0,
  projects: [],
};

const App = () => {
  const { getFirstPage, getLastPage } = usePagination();

  const [projectData, setProjectData] =
    useState<TResponse>(INITIAL_PROJECT_DATA);
  const [page, setPage] = useState(getFirstPage());

  const searchParams = useSearchParams();

  const [inputPage, handleInputPage, setInputPage] = useInput(
    getFirstPage().toString()
  );

  const COUNT_PER_PAGE = 5;
  const isOpenSearchParams = searchParams.get(OPEN) === ALL_PROJECTS;

  useEffect(() => {
    if (isOpenSearchParams) {
      fetchProjectData();
    }
  }, [isOpenSearchParams]);

  function clickPaginationArrow(page: number) {
    setPage(page);
    handleInputPage({
      target: { value: page.toString() },
    } as TInputChangeEvent);
    fetchProjectData(); // 1 페이지일 때 Pagination 컴포넌트에서 ">" 버튼을 누르면 2 페이지에 대한 API를 호출할 것이라고 기대
  }

  async function fetchProjectData() {
    // API 연동
  }

  return (
    <>
      <BottomSheet isOpen={isBottomSheetOpen} onClose={closeBottomSheet}>
        {/* 렌더링 될 내용 */}
        <Pagination
          page={page}
          inputPage={inputPage}
          totalCount={projectData.totalCount}
          countPerPage={COUNT_PER_PAGE}
          handleInputPage={handleInputPage}
          clickPaginationArrow={clickPaginationArrow}
        />
      </BottomSheet>
    </>
  );
};

export default App;
```

위 코드는 페이지네이션에 input 변경 시 또는 페이지네이션의 화살표("<", ">") 클릭 시 자동으로 해당 페이지 API를 호출하고자 했던 내용이다.  
하지만 page가 원하는 대로 불리지 않았다.  
예를 들어 현재 1 페이지에서 ">"를 누르면 input은 2로 올바르게 변경되었지만 API는 1페이지를 호출했다.  
이 문제를 해결하기 위해 `flushSync`라는 기능이 있다는 것을 알고 `setPage`와 `handleInputPage` 부분을 `flushSync`로 감싸봤지만, 올바르게 동작하지 않았다.

알고 보니 `flushSync`는 좀 다른 문제를 해결하기 위해 마련된 기능이었다.  
아주 심플하게 말한다면 `flushSync`는 `async/await`과 같은 역할(또는 Vue의 `await nextTick()`)이라고 할 수 있겠다.

```tsx
import { useState } from 'react';
import { flushSync } from 'react=-dom';

const App = () => {
  const [count, setCount] = useState(0);

  function handleClick() {
    flushSync(() => {
      setCount(count + 1);
      console.log(count); // 가장 먼저 실행됨 0
    });

    // 이 시점에 DOM에 렌더링되는 count는 이미 +1된 상태임
    console.log(count); // 가장 마지막에 실행됨 0
  }

  useEffect(() => {
    console.log(count); // 두 번째로 실행됨 1
  }, [count]);

  return (
    <div>
      <button onClick={handleClick}>+1</button>
      {count}
    </div>
  );
};
```

위 코드는 DOM에 변경이 반영되는 시점이 워낙 빨라서 잘 확인이 안 되지만 [공식 문서](https://react.dev/reference/react-dom/flushSync) 예시를 보면 더 확실히 알 수 있다.  
이처럼 `flushSync`는 state의 비동기 업데이트를 동기적으로 메모리에서 반영하기 위한 용도가 아닌, DOM 업데이트에서 반영하기 위한 용도이다.

아래와 같이 input에 `task`을 추가할 때, 그에 맞춰 스크롤이 가장 마지막 `task`를 따라 이동하고 싶다면 `flushSync`를 유용하게 사용할 수 있다.  
다만, 공식 문서에서 이야기한 것처럼 `flushSync`는 성능에 안 좋은 영향을 끼칠 수 있고 `Suspense`와 사용한다면 강제로 fallback state를 표시할 수 있으므로 사용을 최소화하는 것이 좋다.

```json
[
  {
    "id": "0",
    "task": "업무1"
  },
  {
    "id": "1",
    "task": "업무2"
  },
  {
    "id": "2",
    "task": "업무3"
  },
  {
    "id": "3",
    "task": "업무4"
  },
  {
    "id": "4",
    "task": "업무5"
  },
  {
    "id": "5",
    "task": "업무6"
  }
]
```

```jsx
import { useRef, useState } from 'react';
import { flushSync } from 'react-dom';
import mockTodos from './mock.json';

function App() {
  const [todos, setTodos] = useState(mockTodos);
  const [taskInput, setTaskInput] = useState('');

  const listRef = useRef();

  const handleSubmit = (e) => {
    e.preventDefault();

    if (!taskInput) return;
    setTaskInput('');
    handleAdd(taskInput);
  };

  const handleAdd = (input) => {
    setTodos([...todos, { id: todos.length, task: input }]);
    listRef.current.scrollTop = listRef.current.scrollHeight;
  };

  return (
    <section>
      <h1>Todos</h1>
      <ul ref={listRef} style={{ height: '100px', overflowY: 'scroll' }}>
        {todos.map((todo) => (
          <li key={todo.id}>{todo.task}</li>
        ))}
      </ul>
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          placeholder="input 기입"
          value={taskInput}
          onChange={(e) => setTaskInput(e.target.value)}
        />
        <button>input 추가</button>
      </form>
    </section>
  );
}

export default App;
```

_cf) 페이지네이션 해결 방법_  
React class component에서 `setState`의 callback으로 API를 호출하는 것처럼, `useEffect`를 사용했다.  
`useEffect`의 dependencies로 `page`를 전달했고, `page`가 변경될 때마다 API를 호출하도록 하니 원하는 대로 동작했다.
