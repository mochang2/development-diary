## 0. 공부하게 된 계기

React에 대해 면접 전에 부랴부랴 외우기 보다 미리 '이해'하고 싶어서 정리하려고 한다.

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
// Vanilla.js의 절차적 프로그래밍
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
4. 유지보수가 좋다(위 예시에서는 와닿지 않을 수도 있지만 많은 데이터를 갖는 object 형식을 사용한다고 생각하면 와닿을 것이다)

+) 추가사항  
만약 react에서 DOM을 변경할 때 기존 element를 완전히 제거하고 새로운 element르르 생성한다면 다음과 같은 문제가 발생한다.

1. performance 저하한다.
2. 화면 깜빡임 발생하여 UX가 좋지 않다.
3. state가 사라진다.

이를 해결하기 위해 react에서는 [**virtual DOM**](https://github.com/mochang2/development-diary/blob/main/029-virtual%20DOM.md)과 **reconciliation**이라는 개념을 도입했다.

## 2. 불변성(immutability)

## 3. hook과 함수형 컴포넌트 vs 클래스형 컴포넌트

hook 구현?

## 4. 컴포넌트 합성

특정 컴포넌트들은 어떤 자식 엘리먼트가 들어올지 미리 예상할 수 없는데, 이를 '일반화'(맞는 표현일지는 모르겠지만) 시킴으로써 재사용 가능하게 만들 수 있다.  
특히 `props.children`을 이용하여 `Sidebar`나 `Dialog`를 만들 때 유용한 방식이다.

> react는 render시 jsx 코드를 `React.createElement(component, props, ...children)`를 이용해 변환한다는 점을 이용한다.

아래는 대시보드에서 페이지에 고정적으로 사용되는 `VerticalNavBar`를 이용하여 컴포넌트 합성을 진행한 예시이다.

```jsx
// typescript를 사용한다면 children type에 대해서 https://itchallenger.tistory.com/394 참고

// Content.jsx
const Wrapper = styled.div`
  position: fixed;
  top: 0;
  left: 0;
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

const NAV_BAR_WIDTH = '326px' // 반응형이라면 고정되면 안됨

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
  return <VerticalLayout>{/* Page 내용 렌더링 */}</VerticalLayout>
}
```

`props.children`에게 새로운 props를 전달해주고 싶을 때가 있지만 만약 그렇다면 구조를 잘 짰는지 다시 생각해보자.  
기본적으로 props는 read-only이므로 `{children}` 이외의 방식은 `children`의 props를 변형시킬 수 있다.  
도저히 답이 안 나오면 `React.cloneElement`를 사용하고 사용법은 https://velog.io/@hyunjoogo/React-children-Component%EC%97%90-props-%EC%A0%84%EB%8B%AC%ED%95%98%EA%B8%B0 와 https://www.delftstack.com/ko/howto/react/react-pass-props-to-children/ 를 참고하자.

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
