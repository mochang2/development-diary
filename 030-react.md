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
3. virtual DOM의 효율적인 diffing algorithm으로 빠르다.
4. component를 통해 재사용성과 독립성을 얻어냈다.

### 목차

- [명령형 프로그래밍 vs 선언형 프로그래밍](https://github.com/mochang2/development-diary/blob/main/030-react.md#1-%EB%AA%85%EB%A0%B9%ED%98%95-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8Dimperative-programming-vs-%EC%84%A0%EC%96%B8%ED%98%95-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8Ddeclarative-programming)
- [hook과 함수형 컴포넌트 vs 클래스형 컴포넌트](https://github.com/mochang2/development-diary/blob/main/030-react.md#2-hook%EA%B3%BC-%ED%95%A8%EC%88%98%ED%98%95-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-vs-%ED%81%B4%EB%9E%98%EC%8A%A4%ED%98%95-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8)
- [컴포넌트 합성](https://github.com/mochang2/development-diary/blob/main/030-react.md#3-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-%ED%95%A9%EC%84%B1)

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
let string = 'THis is the midday show with Cheryl Waters'
let urlFriendly = ''

for (var i = 0; i < string.length; i++) {
  if (string[i] === ' ') {
    urlFriendly += '-'
  } else {
    urlFriendly += string[i]
  }
}

console.log(urlFriendly)

// // 선언형 프로그래밍
// 추상화를 이용
const string = 'This is the midday show with Cheryl Waters'
const urlFriendly = string.replace(/ /g, '-')

console.log(urlFriendly)
```

아래는 'hello world'를 쓰기 위한 실제 DOM 조작과 관련된 예시이다.

```javascript
// vanilla.js의 절차적 프로그래밍
const app = document.querySelector('#app')
const header = document.createElement('h1')
header.innerText = 'hello world'

app.appendChild(header)
```

```jsx
// react.js의 선언형 프로그래밍
function App() {
  return (
    <div id="app">
      <h1>hello world</h1>
    </div>
  )
}

// 바벨을 이용하지 않는다면
ReactDOM.render(<h1>hello world</h1>, document.querySelector('#app'))
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
  document.title = name
  // This is a side effect. Don't do this in the component body!

  return <h1>{name}</h1>
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
    super(props)
    this.state = {
      count: 0,
    }
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`
  }
  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    )
  }
}
```

```jsx
import { useState, useEffect } from 'react'

function Example() {
  const [count, setCount] = useState(0)

  useEffect(() => {
    document.title = `You clicked ${count} times`
  })

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  )
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
  return <h1>Hello, {props.name}</h1>
}

// 컴파일 결과
function Welcome(props) {
  return _react2.default.createElement('h1', null, 'Hello, ', props.name)
}
```

```jsx
// 클래스형 컴포넌트
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>
  }
}

// 컴파일 결과
var Welcome = (function (_React$Component) {
  _inherits(Welcome, _React$Component)

  function Welcome() {
    _classCallback(this, Welcome)

    return _possibleConstructorReturn(
      this,
      (Welcome.__proto__ || Object.getPrototypeOf(Welcome)).apply(
        this,
        arguments
      )
    )
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
        )
      },
    },
  ])

  return Welcome
})(_react2.default.Component)
```

결과적으로 번들 사이즈 측면에서는 함수형 컴포넌트가 유리하다.

### Hook

react v16.8에서 hook이라는 개념이 등장하면서 함수형 컴포넌트가 대세가 되었다.  
이전에도 함수형 컴포넌트는 존재했으나 그 당시의 함수형 컴포넌트는 단순한 JS 함수로서, stateless한 컴포넌트였다.  
즉, 클래스형 컴포넌트처럼 `this.state`와 같은 문법을 사용할 수 없었다.  
그래서 그 당시에 굳이 함수형 컴포넌트를 쓰고 싶었다면 별도로 클래스형 컴포넌트를 생성하거나 부모 컴포넌트로 'lift the state up'한 뒤 자식 컴포넌트에서 받아쓰는 방법으로 구현해야 됐다고 한다.

`useState` 훅이 등장하면서 함수형 컴포넌트에서 별도로 state 관리가 가능해졌고 'lift the state up'과 같은 방법을 사용하지 않을 수 있게 되었다.  
`useState`는 JS의 [클로저](https://github.com/mochang2/development-diary/blob/main/025-fundamentals%20of%20js.md#3-closure)라는 개념을 활용하여 구현했다고 한다.

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
`
function Content({ paddingLeft, children }: ContentProps) {
  return <Wrapper paddingLeft={paddingLeft}>{children}</Wrapper>
}
```

```jsx
// Header.jsx
const Wrapper = styled.div`
  padding: ${({ paddingLeft }) => `20px 4vw 30px ${paddingLeft}`};
`
function Header({ paddingLeft }) {
  return <Wrapper paddingLeft={paddingLeft}>header</Wrapper>
}
```

```jsx
// VerticalNavBar.jsx
const Wrapper = styled.div`
  width: ${({ width }) => width};
`
function VerticalNavBar({ width }) {
  return <Wrapper width={width}>{/* some content */}</Wrapper>
}
```

```jsx
// VerticalLayout.jsx
import Content from 'components/common/Content'
import Header from './Header'
import VerticalNavBar from './VerticalNavBar'

const NAV_BAR_WIDTH = '326px'

function VerticalLayout({ children }) {
  return (
    <React.Fragment>
      <VerticalNavBar width={NAV_BAR_WIDTH} />
      <Header paddingLeft={NAV_BAR_WIDTH} />
      <Content paddingLeft={NAV_BAR_WIDTH}>{children}</Content>
    </React.Fragment>
  )
}
```

```jsx
// Page.tsx
import { VerticalLayout } from 'components/layout'

// 합성된 컴포넌트 이용
function Page() {
  return <VerticalLayout>{/* Page 내용 */}</VerticalLayout>
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
  )
}

// specific component
function WelcomeDialog() {
  return (
    <Dialog title="Welcome" message="Thank you for visiting our spacecraft!" />
  )
}
```

### _cf) HOC(Higher Order Component)_

참고: https://ko.reactjs.org/docs/higher-order-components.html , https://velopert.com/3537 , https://www.robinwieruch.de/react-higher-order-components/

HOC란 컴포넌트를 입력으로 받아 새 컴포넌트를 반환하는 함수이다.  
react의 naming convention으로 `with`로 함수의 이름이 시작하며 Redux의 `connect`와 같은 라이브러리에서도 자주 사용되는 방식이다.  
hook이 현재 react에서 대세지만 HOC가 완전히 사라진 것은 아니며 사용법을 익혀둘 필요가 있다.

주의할 점은 HOC를 이용할 때 반드시 HOC 내부에서 원본 컴포넌트를 변경하지 않도록 해야 한다는 것이다.

HOC의 장점은 다음과 같다.

1. **재사용**이 가능하다.  
   HOC에서 `class` 자체를 return 함수로써 클래스형 컴포넌트에서 사용 가능한 예제이다.
   다음 코드 예제를 보면 `CommentList` 컴포넌트와 `BlogPost` 컴포넌트가 하는 역할은 똑같다.

```jsx
// DataSource: localStorage와 같은 전역 데이터 소스

class CommentList extends React.Component {
  constructor(props) {
    super(props)
    this.handleChange = this.handleChange.bind(this)
    this.state = {
      comments: DataSource.getComments(),
    }
  }

  componentDidMount() {
    // 변화감지를 위해 리스너를 추가
    DataSource.addChangeListener(this.handleChange)
  }

  componentWillUnmount() {
    // 리스너를 제거
    DataSource.removeChangeListener(this.handleChange)
  }

  handleChange() {
    // 데이터 소스가 변경될때 마다 comments를 업데이트
    this.setState({
      comments: DataSource.getComments(),
    })
  }

  render() {
    return (
      <div>
        {this.state.comments.map((comment) => (
          <Comment comment={comment} key={comment.id} />
        ))}
      </div>
    )
  }
}

class BlogPost extends React.Component {
  constructor(props) {
    super(props)
    this.handleChange = this.handleChange.bind(this)
    this.state = {
      blogPost: DataSource.getBlogPost(props.id),
    }
  }

  componentDidMount() {
    DataSource.addChangeListener(this.handleChange)
  }

  componentWillUnmount() {
    DataSource.removeChangeListener(this.handleChange)
  }

  handleChange() {
    this.setState({
      blogPost: DataSource.getBlogPost(this.props.id),
    })
  }

  render() {
    return <TextBlock text={this.state.blogPost} />
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
  return { data: TODOS }
}

const App = () => {
  const { data } = fetchData()

  return <TodoList data={data} />
}
```

여기서

1. `data`가 null일 때
2. 빈 배열일 때
3. 로딩 중일 때

등등 모든 예외를 처리한다면 `App` 컴포넌트는 다음과 같아야 된다.

```jsx
const fetchData = () => {
  return { data: null, isLoading: true }
}

const App = () => {
  const { data, isLoading } = fetchData()

  if (isLoading) return <div>Loading data.</div>
  if (!data) return <div>No data loaded yet.</div>
  if (!data.length) return <div>Data is empty.</div>

  return <TodoList data={data} />
}
```

이를 HoC를 이용하면 로직 처리를 분리함으로써 좀더 우아하게 바꿀 수 있다.

```jsx
const withConditionalFeedback = (Component) => (props) => {
  if (props.isLoading) return <div>Loading data.</div>
  if (!props.data) return <div>No data loaded yet.</div>
  if (!props.data.length) return <div>Data is empty.</div>

  return <Component {...props} />
}

const TodoList = withConditionalFeedback(BaseTodoList)
// BaseTodoList는 위 예시의 TodoList에서 같은 컴포넌트

const App = () => {
  const { data, isLoading } = fetchData()

  return <TodoList data={data} isLoading={isLoading} />
}
```

다음과 같은 방식으로 configuration을 추가할 수도 있다.

```jsx
const withConditionalFeedback =
  ({ loadingFeedback, noDataFeedback, dataEmptyFeedback }) =>
  (Component) =>
  (props) => {
    if (props.isLoading) return <div>{loadingFeedback || 'Loading data.'}</div>

    if (!props.data) return <div>{noDataFeedback || 'No data loaded yet.'}</div>

    if (!props.data.length)
      return <div>{dataEmptyFeedback || 'Data is empty.'}</div>

    return <Component {...props} />
  }

// ...

const TodoList = withConditionalFeedback({
  loadingFeedback: 'Loading Todos.',
  noDataFeedback: 'No Todos loaded yet.',
  dataEmptyFeedback: 'Todos are empty.',
})(BaseTodoList)
```

마지막으로 `loadingFeedback`, `noDataFeedback`, `dataEmptyFeedback` 자체를 HOC로 만들 수 있다.

```jsx
const withLoadingFeedback = (Component) => (props) => {
  if (props.isLoading) return <div>Loading data.</div>
  return <Component {...props} />
}

const withNoDataFeedback = (Component) => (props) => {
  if (!props.data) return <div>No data loaded yet.</div>
  return <Component {...props} />
}

const withDataEmptyFeedback = (Component) => (props) => {
  if (!props.data.length) return <div>Data is empty.</div>
  return <Component {...props} />
}

const compose = (...fns) =>
  fns.reduceRight(
    (prevFn, nextFn) =>
      (...args) =>
        nextFn(prevFn(...args)),
    (value) => value
  )

const TodoList = compose(
  withLoadingFeedback,
  withNoDataFeedback,
  withDataEmptyFeedback
)(BaseTodoList)

// compose는 다음과 같은 결과를 return한다
const TodoList = withLoadingFeedback(
  withNoDataFeedback(withDataEmptyFeedback(BaseTodoList))
)
```

+) 참고로 [공식문서](https://ko.reactjs.org/docs/composition-vs-inheritance.html)에서 '합성'은 좋은 구조이지만 '상속'을 사용하는 좋은 사례는 발견하지 못했다고 한다.
