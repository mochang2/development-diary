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

## 5. 전역 상태 관리

전역 상태 관리 라이브러리가 등장하기 전에는 [Container-Presenter 패턴](https://github.com/mochang2/development-diary/blob/main/028-architecture.md#container-presenter)을 이용해서 컴포넌트 간 데이터를 공유했다.  
props drilling을 사용했다는 뜻이다.

전역 상태 관리에 들어가기 전에 props drilling의 단점을 먼저 짚고 넘어가고자 한다.

### Props Drilling

props drilling이란 react의 컴포넌트 트리에서 데이터를 전달하기 위해 상위 컴포넌트에서 하위 컴포넌트로 props를 계속해서 내려주는 것을 의미한다.

전역 변수를 사용한다면 데이터가 어디서 초기화되고 갱신되며 사용하는지 판단하기 쉽지 않다.  
props drilling을 이용해서 props를 따라간다면 코드를 실행하지 않고도 어디서 선언됐고 사용됐는지 쉽게 파악할 수 있으며 전역 변수를 사용할 때 항상 문제가 되는 사이드 이펙트를 덜 걱정할 수 있다.

하지만 컴포넌트 depth가 증가할수록 이 장점은 희미해진다.  
특히 다음과 같은 상황을 마주한다면 말이다.

- 일부 데이터의 자료형을 바꾸게 되는 경우 `{ user: { name: 'Joe West' } } -> { user: { firstName: 'Joe', lastName: 'West' } })`
- 필요보다 많은 property를 전달하다가 컴포넌트를 분리하는 과정에서 필요 없는 property가 계속 남는 경우
- 필요보다 적은 property를 전달하면서 동시에 defaultProps를 과용한 결과로 필요한 property가 전달되지 않은 상황을 문제를 인지하지 어려운 경우 (또한 컴포넌트 분리 과정에서도)
- property의 이름이 중간에서 변경되어서 값을 추적하는데 쉽지 않아지는 경우

추가적으로 props drilling의 단점으로 re-rendering의 비효율성이 있다고 예상했다.  
~아무리 검색해도 리렌더링 비효율성이란 단점은 나오지 않았을 때 잘못된 결론이라는 것을 알았어야 됐다...~  
하지만 잘못된 판단이었다.

아래 내용은 props drilling이 전역 상태 관리 라이브러리와는 다르게 실제로 비효율적인 re-render을 발생하는지 실험하기 위한 과정이다.

#### 컴포넌트 리렌더링

컴포넌트는 다음과 같은 세 가지 상황에서 re-render 된다.

1. update in state(`setState`). `props` 변화가 발생하는지 상관없이 해당 컴포넌트의 모든 자식 요소들도 re-render 된다.
2. update in props. 부모로부터 물려받은 `props`에 변화가 발생하면 해당 컴포넌트는 re-render 된다.
3. re-rendering of parent component.

위 3번은 react의 [diffing 알고리즘](https://ko.reactjs.org/docs/reconciliation.html#motivation) 때문에 발생하며 이를 간단하게 증명할 수 있다.

```tsx
function App() {
  const [message, setMessage] = useState('')

  const handleMessage = () => {
    setMessage(message + 'click ')
  }

  console.log('re-render app')

  return (
    <>
      <div onClick={handleMessage}>click here</div>
      <Title message={message} />
      <Title />
    </>
  )
}

interface TitleProps {
  message?: string
}
function Title({ message }: TitleProps) {
  console.log('re-render title')

  return <div>{message}</div>
}
```

위와 같은 경우는 `message`를 바꾸면 두 번째 `<Title />`은 props를 전달받지 않더라도 `<App />`과 두 개의 `<Title />` 모두에서 re-rendering 발생한다(`re-render app` -> `re-render title` -> `re-render title`).

#### Re-render 코드로 살펴보기 - Props Drilling

props drilling을 사용할 때 중간에 단순히 `props`를 전달받는 컴포넌트는 어떻게 될까?  
앞서 이야기했듯이 `props`가 바뀌거나 부모가 re-render 되면서 본인도 re-render 된다.

```tsx
function Main() {
  const [count, setCount] = useState(0)

  const handleCount = () => {
    setCount(count + 1)
  }

  console.log('re-render main')

  return (
    <div>
      여기는 메인 페이지입니다.
      <Middleware count={count} onChange={handleCount} />
    </div>
  )
}

interface MiddlewareProps {
  count: number
  onChange: () => void
}
function Middleware({ count, onChange }: MiddlewareProps) {
  console.log('re-render middleware')

  return (
    <div>
      여기는 미들웨어입니다.
      <Button count={count} onChange={onChange} />
    </div>
  )
}

interface ButtonProps {
  count: number
  onChange: () => void
}
function Button({ count, onChange }: ButtonProps) {
  console.log('re-render button')

  return <button onClick={onChange}>{count}</button>
}
```

`onChange`가 실행될 때마다 세 컴포넌트 모두 re-render 된다.

#### Re-render 코드로 살펴보기 - Recoil

그렇다면 `recoil`을 사용한다면 어떻게 될까?

```typescript
// 아래 state를 사용
const CountState = atom<number>({
  key: 'count-state',
  default: 0,
})
```

1. 부모에서 전역 변수에 대해 변화를 발생시킨 뒤 부모와 자식에서 해당 값을 구독하기

```tsx
function Main() {
  const [count, setCount] = useRecoilState(CountState)

  const handleCount = () => {
    setCount(count + 1)
  }

  console.log('re-render main')

  return (
    <div>
      여기는 메인 페이지입니다.
      <button onClick={handleCount}>click me</button>
      <Middleware />
    </div>
  )
}

function Middleware() {
  console.log('re-render middleware')

  return (
    <div>
      여기는 미들웨어입니다.
      <Button />
    </div>
  )
}

function Button() {
  const count = useRecoilValue(CountState)

  console.log('re-render button')

  return <button>{count}</button>
}
```

![부모에서 전역 변수에 대해 변화를 발생시킨 뒤 부모와 자식에서 해당 값을 구독하기](https://user-images.githubusercontent.com/63287638/189563571-d4ba66a5-108d-4567-b77a-1da7130e5a23.jpg)

`handleCount`가 실행될 때마다 세 컴포넌트 모두 re-render 된다.

2. 부모에서 전역 변수에 대해 변화를 발생시킨 뒤 자식에서만 해당 값을 구독하기

```tsx
function Main() {
  const setCount = useSetRecoilState(CountState)

  const handleCount = () => {
    setCount(Math.floor(Math.random() * 1000))
  }

  console.log('re-render main')

  return (
    <div>
      여기는 메인 페이지입니다.
      <button onClick={handleCount}>click me</button>
      <Middleware />
    </div>
  )
}

// 나머지 컴포넌트는 1)과 동일
```

![부모에서 전역 변수에 대해 변화를 발생시킨 뒤 자식에서만 해당 값을 구독하기](https://user-images.githubusercontent.com/63287638/189563570-a2eeaeca-b99a-4d8b-bc71-43e8938f453d.jpg)

`handleCount`가 발생해도 `<Button >`만 re-render 된다.

3. 자식에서 변화를 발생시킨 뒤 부모에서 해당 값 구독하기

```tsx
function Main() {
  const count = useRecoilValue(CountState)

  console.log('re-render main')

  return (
    <div>
      여기는 메인 페이지입니다. {count}
      <Middleware />
    </div>
  )
}

function Middleware() {
  console.log('re-render middleware')

  return (
    <div>
      여기는 미들웨어입니다.
      <Button />
    </div>
  )
}

function Button() {
  const setCount = useSetRecoilState(CountState)

  const handleCount = () => {
    setCount(Math.floor(Math.random() * 1000))
  }

  console.log('re-render button')

  return <button onClick={handleCount}>click here</button>
}
```

![자식에서 변화를 발생시킨 뒤 부모에서 해당 값 구독하기](https://user-images.githubusercontent.com/63287638/189563575-b882f840-05ec-45a3-b3a4-a880cee51926.jpg)

`handleCount`가 실행될 때마다 세 컴포넌트 모두 re-render 된다.

**결론**

결론적으로 '전역 상태 관리 라이브러리가 re-render 효율성을 가진다'는 말이 틀린 말은 아니다.  
다만 특정한 구조가 아니고서는 '반드시 더 효율적이다'고 얘기할 수는 없는 것 같다.

### Context API

context API - https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&ved=2ahUKEwjEiOa1-4v6AhWZm1YBHVHtBCsQFnoECAcQAQ&url=https%3A%2F%2Fcodemacaw.com%2F2021%2F11%2F21%2Fprevent-unnecessary-re-rendering-when-using-context-api%2F&usg=AOvVaw0hmf6N2lhTO0csUEhcV7L5
redux
recoil - https://velog.io/@yiyb0603/TypeScript-React-Recoil%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-TodoList-%EB%A7%8C%EB%93%A4%EC%96%B4%EB%B3%B4%EA%B8%B0
jotai

언제 re-rendering 되는지. 어떠한 철학을 가지고 만들었는지.
