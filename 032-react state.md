## 0. 공부하게 된 계기

좋은 react 개발자가 되기 위해서는 가독성 좋은, 성능 좋은, 유지 보수 가능한 코드를 짜야 된다.  
성능이나 디자인 패턴을 이해하고 싶다면 react에서 상태 관리가 어떻게 이루어지며 어떠한 원리를 가지고 있는지 파악할 필요가 있다.

## 1. Component State

### Props vs State

다음과 같은 특징을 지닌다.

- props
  - 변경되면 안 됨
  - 컴포넌트가 호출받을 때 전달받는 값
  - 호출받고 코드가 읽혀지는 시점에서 값이 고정됨
- state
  - 변경될 수 있음
  - 컴포넌트가 실행될 때 내부적으로 가진 값
  - 비동기적으로 변경됨

props와 state와 관련해서 자주 실수하는 부분이 있다.

1. 전달받은 props로 state를 초기화한다.

일반적으로 props를 state를 초기화하는데만 사용하고 props가 업데이트 되지 않거나 하위 component가 props 변화에 반응하지 않아도 된다면 문제가 없다.

그래서 다음과 같은 경우처럼 사용하면 문제 없이 변화가 화면에 잘 반영된다.

```jsx
function Main() {
  const [users, setUsers] = useState(fakeUsers)

  const getFakeUsers = () => {
    return new Promise((resolve) =>
      setTimeout(
        () =>
          resolve(
            {
              id: '1',
              name: 'Robin',
            },
            {
              id: '2',
              name: 'Dennis',
            }
          ),
        2000
      )
    )
  }

  useEffect(() => {
    const fetchUsers = async () => {
      const data = await getFakeUsers()
      setUsers(data)
    }

    fetchUsers()
  }, [])

  return (
    <div>
      <h1>부모 컴포넌트</h1>
      <List list={users} />
    </div>
  )
}

function List({ list }) {
  return (
    <ul>
      {list.map((item) => (
        <Item key={item.id} item={item} />
      ))}
    </ul>
  )
}

function Item({ item }) {
  const [name, setName] = useState(item)

  const handleNameChange = (event) => {
    setName({ ...name, name: element.value })
  }

  return (
    <li>
      {item.name}
      <input type="text" value={name.name} onChange={handleNameChange} />
    </li>
  )
}
```

하지만 이러한 방식이 react의 anti-pattern으로 불리는 이유는 다음과 같은 경우가 존재하기 때문이다.

```jsx
function ParentComponent() {
  const [initialValue, setInitialValue] = useState('Initial value')

  useEffect(() => {
    setTimeout(() => {
      setInitialValue('Changed value')
    }, 1000)
  }, [])

  useEffect(() => {
    console.log(initialValue)
  }, [initialValue])

  return <ChildComponent initialValue={initialValue} />
}

function ChildComponent({ initialValue }) {
  const [inputValue, setInputValue] = useState(initialValue)

  const handleChange = (e) => {
    setInputValue(e.target.value)
  }

  return <input value={inputValue} onChange={handleChange} />
}
```

~실용적이지 않은 예시지만 그냥 그러려니 하자~  
위 코드는 1초 뒤 다음과 같은 결과를 보인다.

![state change result1](https://user-images.githubusercontent.com/63287638/191727884-56762ba3-976d-4df9-bf1c-20bfca679ec2.jpg)

![state change result2](https://user-images.githubusercontent.com/63287638/191728146-fdf1f08d-31a3-4c07-a1d1-c5f2d3705363.jpg)

분명 `useEffect`에서는 `initialValue`가 바뀌었다는 것을 보여주지만 `ChildComponent`의 UI에는 해당 변경이 반영되지 않는다.

위와 같이 코드를 짰을 때 하위 컴포넌트에도 해당 변경이 반영되기를 원한다면 `setInitialValue`를 같이 props로 넘겨주는 방식으로 구조를 변경해야 한다.

2. state는 비동기적으로 변경된다.

```jsx
// 주석은 `button`을 1번 눌렀을 때 console에 찍히는 내용이다.
function App() {
  const [count, setCount] = useState(0)

  const addCount = () => {
    console.log(count) // 0
    setCount(count + 1)
    console.log(count) // 0
  }

  useEffect(() => {
    console.log(count) // 1
  }, [count])

  return (
    <div>
      <button onClick={addCount}>+1</button>
      <h1>{count}</h1>
    </div>
  )
}
```

setState는 비동기적이다.  
setState가 호출되는 시점은 해당 setState가 포함된 모든 함수가 실행된 이후이다.  
이는 react에서 효율적으로 렌더링하기 위해 상태값 변경에 대해 batching 처리를 하기 위해서이다.

> batching: 성능을 최적화하기 위해 state 업데이트를 하나의 re-render가 발생하도록 그룹화하는 것

그래서 아래와 같은 경우는 `button`을 1번 눌렀을 때 10이 아닌 4가 더해진다.

```jsx
function App() {
  const [count, setCount] = useState(0)

  const addCount = () => {
    setCount(count + 1)
    setCount(count + 2)
    setCount(count + 3)
    setCount(count + 4) // re-render은 1번만 발생
  }

  useEffect(() => {
    console.log(count) // 4
  }, [count])

  return (
    <div>
      <button onClick={addCount}>+1</button>
      <h1>{count}</h1>
    </div>
  )
}
```

만약 동기적으로 동작하게 만들어 +1, +2, +3, +4를 전부 수행하고 싶다면 아래처럼 setState의 callback을 활용하거나 `useReducer`, `useEffect`, `useRef` 등과 같은 훅을 이용할 수 있다.

```jsx
function App() {
  const [count, setCount] = useState(0)

  const addCount = () => {
    setCount((count) => count + 1)
    setCount((count) => count + 2)
    setCount((count) => count + 3)
    setCount((count) => count + 4) // re-render는 1번만 발생
  }

  useEffect(() => {
    console.log(count) // 10
  }, [count])

  return (
    <div>
      <button onClick={addCount}>+1</button>
      <h1>{count}</h1>
    </div>
  )
}
```

_cf) automatic batching_

비동기적 update에 추가적인 설명을 덧붙이자면 렌더링 최적화를 위해 react 18부터 automatic batching이 추가되었다.

기존에는 react의 batching이 시점에 대한 일관성이 없었다.  
`Timeout`, `Promise` 등 모든 다른 이벤트를 다룰 때는 setState마다 re-render가 발생했다.  
아래와 같이 말이다.

```jsx
function App() {
  const [count, setCount] = useState(0)
  const [flag, setFlag] = useState(false)

  function handleClick() {
    fetchSomething().then(() => {
      // React 17 and earlier does NOT batch these:
      setCount(count + 1) // Causes a re-render
      setFlag(!flag) // Causes a re-render
    })
  }

  return (
    <div>
      <button onClick={handleClick}>+1</button>
    </div>
  )
}
```

하지만 `root`를 `createRoot`를 통해서 생성하면 `Timeout`, `Promise` 등 모든 다른 이벤트를 다룰 때도 batching이 되도록 만들 수 있다.  
참고로 CRA로 만들면 자동으로 아래와 같은 코드가 생성된다.

```jsx
// index.jsx
const root = ReactDOM.createRoot(document.getElementById('root'))

root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
)
```

```jsx
// App.jsx
function App() {
  const [count, setCount] = useState(0)
  const [flag, setFlag] = useState(false)

  function handleClick() {
    fetchSomething().then(() => {
      // React 18 batches these:
      setCount(count + 1) // does not cause a re-render
      setFlag(!flag) // does not cause a re-render
    })
  }

  return (
    <div>
      <button onClick={handleClick}>+1</button>
    </div>
  )
}
```

만약 위와 같은 코드에서 batching을 적용하고 싶지 않다면 각각의 setState를 별도의 callback으로 감싸거나 `createRoot`를 이용하지 않으면 된다.

### React의 불변성

[JS의 불변성](https://github.com/mochang2/development-diary/blob/main/025-fundamentals%20of%20js.md#8-immutability)에서 다뤘듯이 불변성이란 **메모리 영역에서 값을 변경할 수 없다**는 의미이다.
react에서 불변성은 새로운 개념이 아니라 JS의 불변성이라는 개념을 지켜가면서 state와 props를 이용할 수 있도록 하는 아이디어를 react에 녹여낸 것이다.

[아래](https://github.com/mochang2/development-diary/blob/main/030-react.md#%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-%EB%A6%AC%EB%A0%8C%EB%8D%94%EB%A7%81)에서 다루듯이 react는 state가 변경되면 컴포넌트가 re-render된다.  
react에서 불변성을 지켜주는 이유는 state가 변경되었는지 파악하기 위함이다.

react는 state가 변경되었는지 확인할 때 얕은 비교를 수행한다.  
즉, object의 내부를 하나하나 비교하는 것이 아니라 주소값(참조값)만 비교한다.  
만약 불변성을 지켜주지 않는 방식으로 setState를 호출하면 변화된 값이 화면에 반영되지 않을 수 있다.

react에서 불변성을 지킴으로써 다음과 같은 이점을 얻는다.

1. 효율적인 상태 업데이트(계산 리소스가 적은 shallow compare)
2. side effect 방지 및 프로그래밍 구조의 단순성

#### 불변성을 유지하는 방법

함수형 프로그래밍을 생각하면 단순하다.  
간단하게 Array로 설명을 하면, `push`나 `splice` 등을 사용하지 않고 spread 연산자나 고차함수(`map`, `filter` 등)을 이용하는 것이다.

```jsx
// state에 어떤 값을 추가해야 되는 상황
function Component() {
  const [data, setData] = useState([])

  const handleData = (event) => {
    setData([...data, event.target.dataset.id])
  }
}
```

## 2. 전역 상태 관리

전역 상태 관리 라이브러리가 등장하기 전에는 [Container-Presenter 패턴](https://github.com/mochang2/development-diary/blob/main/028-architecture.md#container-presenter)을 이용해서 컴포넌트 간 데이터를 공유했다.  
props drilling을 사용했다는 뜻이다.

전역 상태 관리에 들어가기 전에 props drilling의 단점을 먼저 짚고 넘어가고자 한다.

### 1) Props Drilling

props drilling이란 react의 컴포넌트 트리에서 데이터를 전달하기 위해 상위 컴포넌트에서 하위 컴포넌트로 props를 계속해서 내려주는 것을 의미한다.

전역 변수를 사용한다면 데이터가 어디서 초기화되고 갱신되며 사용하는지 판단하기 쉽지 않다.  
props drilling을 이용해서 props를 따라간다면 코드를 실행하지 않고도 어디서 선언됐고 사용됐는지 쉽게 파악할 수 있으며 전역 변수를 사용할 때 항상 문제가 되는 사이드 이펙트를 덜 걱정할 수 있다.

하지만 컴포넌트 depth가 증가할수록 이 장점은 희미해진다.  
특히 다음과 같은 상황을 마주한다면 말이다.

- 일부 데이터의 자료형을 바꾸게 되는 경우 `{ user: { name: 'Joe West' } } -> { user: { firstName: 'Joe', lastName: 'West' } }`
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

> (3번) 예를 들어, `A` > `B` > `C` > `D` 순서의 컴포넌트 트리가 있다고 가정해보자. `B`에 카운터를 올리는 버튼이 있고, 이를 클릭했다고 가정해보자.

`B`의 `setState()`가 호출되어, `B`의 리렌더링이 렌더링 queue로 들어간다.
리액트는 트리 최상단부터 렌더링 path를 시작한다.
`A`는 업데이트가 필요하다고 체크 되어 있지 않을 것이므로, 지나간다.
`B`는 업데이트가 필요한 컴포넌트로 체크되어 있으므로, `B`를 리렌더링 한다. `B`는 `C`를 리턴한다.
`C`는 원래 업데이트가 필요한 것으로 간주되어 있지 않았다. 그러나, 부모인 `B`가 렌더링 되었으므로, 리액트는 그 하위 컴포넌트인 `C`를 렌더링 한다. `C`는 `D`를 리턴한다.
`D`도 마찬가지로 렌더링이 필요하다고 체크되어 있지 않았지만, `C`가 렌더링된 관계로, 그 자식인 `D`도 렌더링 한다.
컴포넌트를 렌더링 하는 작업은, 기본적으로, 하위에 있는 모든 컴포넌트 또한 렌더링 하게 된다.

위와 같은 경우 경우, 리액트는 props가 변경되었는지 신경쓰지 않는다. 부모 컴포넌트가 렌더링 되어 있기 때문에, 자식 컴포넌트도 무조건 리렌더링 된다.

위 3번은 react의 [diffing 알고리즘](https://ko.reactjs.org/docs/reconciliation.html#motivation) 때문에 발생하며 이를 간단하게 증명할 수 있다.

```tsx
function App() {
  const [message, setMessage] = useState('')

  const handleMessage = () => {
    setMessage(message + 'click ')
  }

  useEffect(() => {
    console.log('re-render app')
  })

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
  useEffect(() => {
    console.log('re-render title')
  })

  return <div>{message}</div>
}
```

위와 같은 경우는 `message`를 바꾸면 두 번째 `<Title />`은 props를 전달받지 않더라도 `<App />`과 두 개의 `<Title />` 모두에서 re-rendering 발생한다(`re-render app` -> `re-render title` -> `re-render title`).

#### re-render 코드로 살펴보기 - Props Drilling

props drilling을 사용할 때 중간에 단순히 `props`를 전달받는 컴포넌트는 어떻게 될까?  
앞서 이야기했듯이 `props`가 바뀌거나 부모가 re-render 되면서 본인도 re-render 된다.

```tsx
function Main() {
  const [count, setCount] = useState(0)

  const handleCount = () => {
    setCount(count + 1)
  }

  useEffect(() => {
    console.log('re-render main')
  })

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
  useEffect(() => {
    console.log('re-render middleware')
  })

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
  useEffect(() => {
    console.log('re-render button')
  })

  return <button onClick={onChange}>{count}</button>
}
```

`handleCount`가 실행될 때마다 세 컴포넌트 모두 re-render 된다.

#### re-render 코드로 살펴보기 - Recoil

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

  useEffect(() => {
    console.log('re-render main')
  })

  return (
    <div>
      여기는 메인 페이지입니다.
      <button onClick={handleCount}>click me</button>
      <Middleware />
    </div>
  )
}

function Middleware() {
  useEffect(() => {
    console.log('re-render middleware')
  })

  return (
    <div>
      여기는 미들웨어입니다.
      <Button />
    </div>
  )
}

function Button() {
  const count = useRecoilValue(CountState)

  useEffect(() => {
    console.log('re-render button')
  })

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

  useEffect(() => {
    console.log('re-render main')
  })

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

`handleCount`가 발생해도 `<Button>`만 re-render 된다.

3. 자식에서 변화를 발생시킨 뒤 부모에서 해당 값 구독하기

```tsx
function Main() {
  const count = useRecoilValue(CountState)

  useEffect(() => {
    console.log('re-render main')
  })

  return (
    <div>
      여기는 메인 페이지입니다. {count}
      <Middleware />
    </div>
  )
}

function Middleware() {
  useEffect(() => {
    console.log('re-render middleware')
  })

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

  useEffect(() => {
    console.log('re-render button')
  })

  return <button onClick={handleCount}>click here</button>
}
```

![자식에서 변화를 발생시킨 뒤 부모에서 해당 값 구독하기](https://user-images.githubusercontent.com/63287638/189563575-b882f840-05ec-45a3-b3a4-a880cee51926.jpg)

`handleCount`가 실행될 때마다 세 컴포넌트 모두 re-render 된다.

**결론**

결론적으로 '전역 상태 관리 라이브러리가 re-render 효율성을 가진다'는 말이 틀린 말은 아니다.  
다만 특정한 구조가 아니고서는 '반드시 더 효율적이다'고 얘기할 수는 없는 것 같다.

### 2) Context API

참고  
https://dev.rase.blog/21-10-07-context-and-state-management/  
https://codemacaw.com/2021/11/21/prevent-unnecessary-re-rendering-when-using-context-api/  
https://chatoo2412.github.io/javascript/react/react-context-as-a-state-management-tool/  
https://velog.io/@velopert/react-context-tutorial#context-%EC%97%90%EC%84%9C-%EC%83%81%ED%83%9C-%EA%B4%80%EB%A6%AC%EA%B0%80-%ED%95%84%EC%9A%94%ED%95%9C-%EA%B2%BD%EC%9A%B0  
https://hong-jh.tistory.com/m/entry/Context-API%EB%8A%94-%EC%99%9C-%EC%93%B0%EA%B3%A0-%EA%B7%B8%EB%A0%87%EB%8B%A4%EB%A9%B4-Redux%EB%8A%94-%ED%95%84%EC%9A%94%EC%97%86%EC%9D%84%EA%B9%8C  
https://velopert.com/3606  
https://stackoverflow.com/questions/67467924/how-to-reduce-react-context-hell

우선 확실히 짚고 넘어가야 할 점은 context api는 **상태 관리 도구가 아니다.**  
실제로 [공식 문서](https://reactjs.org/docs/context.html)에도 context api에 대해 state management라는 용어는 등장하지도 않는다.  
context api는 props drilling을 해결하기 위한 방법으로 등장했다.  
오직 전역적으로 상태를 공유해주기 위해서 사용되며 **상태 관리를 하지 않는다.**

_cf) 참고_  
`redux`, `react-dom-router`, `styled-component` 등이 이 context api를 활용하여 구현됐다.

#### 상태 관리의 특징

`redux`와 많이 비교되기 때문에 `redux`의 예시를 들겠다.

1. 초기 값을 저장한다. `redux`에서 `store`의 초기값을 지정할 수 있다.
2. 현재 값을 읽을 수 있다. `redux`에서 `mapStateToProps`나 `useSelector`를 통해서 state를 읽을 수 있다.
3. 값 업데이트가 가능하다. `redux`에서 reducer에 action을 dispatch해서 state를 업데이트할 수 있다.

context api는 이러한 기능을 제공해주지 않는다(사용자가 직접 이러한 기능을 제공하게끔 만드는 것이다).

#### Context

그렇다면 context란 무엇일까?  
context란 단순히 react component에서 props가 아닌 또 다른 방식으로 컴포넌트 간 값을 전달하는 방법이다.

[velopert 블로그](https://velog.io/@velopert/react-context-tutorial#context%EA%B0%80-%EA%BC%AD-%EC%A0%84%EC%97%AD%EC%A0%81%EC%9D%B4%EC%96%B4%EC%95%BC-%ED%95%9C%EB%8B%A4%EB%8A%94-%EC%83%9D%EA%B0%81%EC%9D%84-%EB%B2%84%EB%A6%AC%EC%9E%90)를 인용하자면

> Context는 전역 상태 관리를 할 수 있는 수단일 뿐이고, 상태 관리 라이브러리는 상태 관리를 더욱 편하고, 효율적으로 할 수 있게 해주는 기능들을 제공해주는 도구입니다.

주로 사용자 로그인 정보, 애플리케이션 설정, 테마 등 전역적으로 state가 사용될 때 사용된다.

#### 장단점

아래 코드 예시를 보기 전에 장단점을 먼저 정리하고자 한다.

- 장점
  - built in이기 때문에 별도의 서드 파티 라이브러리 설치가 필요 없다. 이 때문에 애플리케이션이 가벼워질 수 있다.
  - 러닝 커브가 낮다.
  - (전역 상태에 대해서도 공유가 가능하지만) 전역적이지 않고 특정 컴포넌트 간 상태를 공유할 때 다루기에 용이하다.
- 단점
  - 불필요한 리렌더링이 발생한다.
    - 메모이제이션이나 `children`을 감싼 형태를 통해 극복할 수 있다.
    - 또한 context를 나눔(context 간 관심사 분리)으로써 해결 가능하다.
    - 이는 또다른 문제점인 provider hell(또는 wrapper hell)을 야기한다.
    - provider hell은 `reduce`를 통해 해결할 수 있다(현재는 이 방법에 **best practice**라고 생각한다).
    - 결국 불필요한 리렌더링 발생이 단점이라고 말하기 좀 애매한 것 같다.
  - 특정 '컴포넌트 간'이 아닌 전역적으로 상태를 공유해야 한다면 이미 다른 훌륭한 라이브러리가 많다.

#### 코드 예시

![context api component structure](https://user-images.githubusercontent.com/63287638/196319728-a7e7a17d-575a-41c5-b22f-0c4ae3405577.png)

위 사진은 코드 예시로 사용한 컴포넌트 구조이다.

![screen](https://user-images.githubusercontent.com/63287638/196320189-88939ad8-cd00-48a0-906f-c7a8f9411056.jpg)

위 사진은 실제 화면이다.

##### 불필요한 re-rendering 발생

`ChildComponent3`와 `GrandChildComponent2`의 공통 조상이 `ParentComponent`이므로 `ParentComponent`에서 `CounterContext.Provider`로 감싸줬다.

```jsx
// Context.js
import { useState, useEffect, createContext, useContext } from 'react'

const CounterContext = createContext()

export function ParentComponent() {
  const [count, setCount] = useState(0)

  useEffect(() => {
    console.log('re-render parent')
  })

  return (
    <CounterContext.Provider value={[count, setCount]}>
      <div>
        여기는 메인 페이지입니다.
        <ChildComponent1 />
        <ChildComponent2 />
        <ChildComponent3 />
      </div>
    </CounterContext.Provider>
  )
}

function ChildComponent1() {
  useEffect(() => {
    console.log('re-render child1')
  })

  return (
    <div>
      <span>여기는 child1입니다.</span>
      <GrandChildComponent1 />
      <GrandChildComponent2 />
    </div>
  )
}

function ChildComponent2() {
  useEffect(() => {
    console.log('re-render chlid2')
  })

  return (
    <div>
      <span>여기는 child2입니다.</span>
    </div>
  )
}

function ChildComponent3() {
  const [, setCount] = useContext(CounterContext)

  useEffect(() => {
    console.log('re-render child3')
  })

  return (
    <div>
      <span>여기는 child3입니다.</span>
      <button onClick={() => setCount((prev) => prev + 1)}>click me!</button>
    </div>
  )
}

function GrandChildComponent1() {
  useEffect(() => {
    console.log('re-render grandchild1')
  })

  return (
    <div>
      <span>여기는 grandchild1입니다.</span>
    </div>
  )
}

function GrandChildComponent2() {
  const [count] = useContext(CounterContext)

  useEffect(() => {
    console.log('re-render grandchild2')
  })

  return (
    <div>
      <span>여기는 grandchild1입니다. count: {count}</span>
    </div>
  )
}
```

```jsx
// App.js

function App() {
  return (
    <div className="App">
      <ParentComponent />
    </div>
  )
}
```

위 예시에서 `ChildComponent3`의 `button`을 누르면 `ParentComponent`를 제외하고 전부 re-rendering된다.

![context api re rendering](https://user-images.githubusercontent.com/63287638/196320057-1fcac460-5f78-4d9b-924f-6f90d3f08de4.jpg)

위와 같은 상황에서 `ChildComponent1`, `ChildComponent2`, `GrandChildComponent1`은 사용하지도 않는 context 때문에 불필요하게 re-rendering됐다.  
이러한 상황을 막기 위한 해결책 중 하나가 memoization이다(실제로 많은 블로그에서 그렇게 이야기한다).  
다만 나는 이 방법이 완벽한 해결책이라고 생각하지 않는다.  
왜냐하면 결국 메모이제이션은 props를 비교하기 위한 비용이 발생하기 때문이다.

##### 불필요한 re-rendering 제거

아래는 또다른 해결책이다.  
`CounterProvider`라는 컴포넌트로 `ParentComponent`를 감싼 뒤 `props.children`을 이용하여 rendering하는 것이다.

```jsx
// Context.js
import { useState, useEffect, createContext, useContext } from 'react'

const CounterContext = createContext()

export function CounterProvider({ children }) {
  const [count, setCount] = useState(0)

  return (
    <CounterContext.Provider value={[count, setCount]}>
      {children}
    </CounterContext.Provider>
  )
}

export function ParentComponent() {
  useEffect(() => {
    console.log('re-render parent')
  })

  return (
    <div
      style={{
        display: 'flex',
        justifyContent: 'space-around',
      }}
    >
      여기는 메인 페이지입니다.
      <ChildComponent1 />
      <ChildComponent2 />
      <ChildComponent3 />
    </div>
  )
}

function ChildComponent1() {
  useEffect(() => {
    console.log('re-render child1')
  })

  return (
    <div>
      <span>여기는 child1입니다.</span>
      <GrandChildComponent1 />
      <GrandChildComponent2 />
    </div>
  )
}

function ChildComponent2() {
  useEffect(() => {
    console.log('re-render chlid2')
  })

  return (
    <div>
      <span>여기는 child2입니다.</span>
    </div>
  )
}

function ChildComponent3() {
  const [, setCount] = useContext(CounterContext)

  useEffect(() => {
    console.log('re-render child3')
  })

  return (
    <div>
      <span>여기는 child3입니다.</span>
      <button onClick={() => setCount((prev) => prev + 1)}>click me!</button>
    </div>
  )
}

function GrandChildComponent1() {
  useEffect(() => {
    console.log('re-render grandchild1')
  })

  return (
    <div>
      <span>여기는 grandchild1입니다.</span>
    </div>
  )
}

function GrandChildComponent2() {
  const [count] = useContext(CounterContext)

  useEffect(() => {
    console.log('re-render grandchild2')
  })

  return (
    <div>
      <span>여기는 grandchild1입니다. count: {count}</span>
    </div>
  )
}
```

```jsx
// App.js

function App() {
  return (
    <div className="App">
      <CounterProvider>
        <ParentComponent />
      </CounterProvider>
    </div>
  )
}
```

![context api re rendering](https://user-images.githubusercontent.com/63287638/196320893-ec43293f-581c-4bd6-9c2c-0e6f5b0c9b60.jpg)

위와 같은 방법으로 `ChildComponent1`, `ChildComponent2`, `GrandChildComponent1`에서의 불필요한 렌더링을 막을 수 있다.  
다만 이러한 방법은 또 다른 문제점을 야기할 수 있다.  
바로 provider hell이다.

불필요한 re-rendering을 막기 위해서는 다음과 같은 방법이 필요하다.

1. context의 관심사를 분리해야 한다. 예를 들어 style 테마 context와 인증 토큰 context를 분리하지 않는다면 style 테마가 변경될 때 인증 토큰을 사용하는 component도 re-render된다. 위 코드에서도 state와 setState의 context를 분리해줬다면 버튼 클릭 시 `ChildComponent3`가 re-rendering되는 것까지 막을 수 있다.
2. provider component로 감싸줘야 한다.

극단적일 경우 아래와 같은 상황도 발생한다고 한다.

![provider hell](https://user-images.githubusercontent.com/63287638/196321524-99fb2ba9-4d7b-453b-a0f5-4d7da3b165fe.jpeg)

##### provider hell 해결

다행히 위와 같이 극단적으로 depth가 깊어지는 것을 해결하는 방법이 있다.  
`Array.prototype.reduce`을 사용해서 Provider를 하나로 묶을 수 있다.

```jsx
import { SampleProvider } from './contexts/sample'
import { AnotherProvider } from './contexts/another'

const AppProvider = ({ contexts, children }) => {
  return contexts.reduce(
    (prev, context) =>
      React.createElement(context, {
        children: prev,
      }),
    children
  )
}

const App = () => {
  return (
    <AppProvider contexts={[SampleProvider, AnotherProvider]}>
      <div>
        <SomeComponents />
      </div>
    </AppProvider>
  )
}
```

#### provider scope

consumer는 가장 가까운 조상 provider를 참조한다.  
만약 조상 provider가 없다면 default로 fall back된다.

```javascript
const defaultValue = { name: 'unknown' }
const SectionContext = createContext(defaultValue)
const SectionProvider = SectionContext.Provider
const SectionConsumer = SectionContext.Consumer

const App = () => (
  <div>
    <SectionProvider value={{ name: 'header' }}>
      <header>
        <Link />
        <SectionProvider value={{ name: 'floating bar' }}>
          <aside>
            <Link />
          </aside>
        </SectionProvider>
      </header>
    </SectionProvider>

    <SectionProvider value={{ name: 'content' }}>
      <article />
      <div>
        <Link />
      </div>
    </SectionProvider>
  </div>
)

const Link = () => {
  const sendAnalyticsEvent = (sectionName) => {
    /*
     * Link 컴포넌트가 어느 provider 안에 위치하는지에 따라
     * 'unknown' | 'header' | 'floating bar' | 'content' 가 출력됨
     */
    console.log(`Link in ${sectionName} has clicked.`)
  }

  return (
    <SectionConsumer>
      {({ name }) => <a onClick={() => sendAnalyticsEvent(name)}>click me</a>}
    </SectionConsumer>
  )
}
```

#### 결론

다시 한 번 말하지만 context api는 전역 상태 관리 도구는 아니다.  
하지만 전역 상태를 공유하기 위해 props drilling 대신 사용할 수 있는 훌륭한 built-in 도구이다.

### Redux

참고 및 사진 출처  
https://ridicorp.com/story/how-to-use-redux-in-ridi/  
https://devlog-h.tistory.com/m/26  
https://velog.io/@404/%EB%A6%AC%EB%8D%95%EC%8A%A4-2.-%EB%A6%AC%EB%8D%95%EC%8A%A4%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0  
https://amyhyemi.tistory.com/m/103  
https://yong-nyong.tistory.com/m/12  
https://medium.com/js-imaginea/best-practices-with-react-and-redux-application-1e94a6f214a0  
https://velog.io/@seongkyun/useSelector-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0  
https://velog.io/@kim6515516/useSelector%EC%83%81%ED%83%9C%EC%A1%B0%ED%9A%8C-useDispatch%EC%95%A1%EC%85%98-%EB%94%94%EC%8A%A4%ED%8C%A8%EC%B9%98  
https://redux.js.org/usage/structuring-reducers/prerequisite-concepts#note-on-immutability-side-effects-and-mutation

[redux 공식 문서](https://redux.js.org/)에 따르면 redux는 다음과 같은 상황에 사용하기 좋다고 한다.

- 앱의 여러 위치에서 필요한 많은 양의 state들이 존재할 때 (전역 상태가 필요하다고 느껴질 때)
- state가 자주 업데이트 될 때
- state를 업데이트 하는 로직이 복잡할 때
- 앱이 중간 또는 큰 사이즈의 코드를 갖고 있고 많은 사람들에 의해 코드가 관리될 때
- 상태가 업데이트되는 시점을 관찰할 필요가 있을 때

특히 react를 사용할 때 많이 나타나는 현상들이다.

#### 원칙

1. 하나의 애플리케이션 안에는 하나의 store만 사용

덕분에 애플리케이션 디버깅이 쉽다.

2. state 업데이트는 오직 action을 통해서

상태를 변화시키는 의도를 정확하게 표현할 수 있고, 상태 변경에 대한 추적이 용이해진다.

3. state 업데이트를 일으키는 reducer는 순수 함수

reducer는 이전 state와 action 객체를 파라미터로 받는다.  
파라미터 외의 값에는 의존하면 안 된다.  
`useState`로 관리하는 state처럼 불변성을 유지해야 한다.  
같은 파라미터로 호출된 reducer는 같은 output을 반환해야 한다.

#### 장점

1. 데이터 흐름 파악이 쉬움

[아키텍처](https://github.com/mochang2/development-diary/blob/main/028-architecture.md#%EC%82%AC%EC%A0%84%EC%A7%80%EC%8B%9D-flux%EC%99%80-redux)에서 다룬 적 있지만 간단히만 다루겠다.  
`redux`(flux 패턴)는 MVC 패턴의 단점을 극복하기 위해 사용되었다.

![mvc](https://user-images.githubusercontent.com/63287638/196336237-d0cdd816-20e7-4270-8263-cc1fb12e0f38.jpg)

위 사진에서 볼 수 있듯이 MVC는 **양방향 데이터 흐름**이 가장 큰 문제였다.  
따라서 데이터 흐름을 파악하기 힘든 구조였다.

이와 달리 flux 패턴은 단방향 데이터 흐름을 가지고 있기에 데이터 흐름을 파악하기 쉽다.

2. 미들웨어

API 요청 시 (다만 이제는 `swr`, `react-query` 등의 등장으로 사용할 필요가 줄어듦), Next 없이 SSR 사용 시(`recoil`이나 `jotai`에는 없는 안정성이 있음) 미들웨어의 도움을 받을 수 있다.

~직접 프로젝트에서 사용한 적이 없기 때문에 코드 예시에서는 제외했다.~

3. 안정성

현재(2022.10) [react-redux](https://www.npmjs.com/package/react-redux)의 버전은 8.0.4이다.  
충분히 많은 버전이 나와서 안정성이 검증됐으며 커뮤니티도 넓어서 best practice를 찾기 쉽다.

4. 바뀐 값을 읽는 component에 대해서만 re-rendering

context를 분리해야 하는 context api와 달리 불변성을 유지하면(redux 또한 shallow comparison) 불필요한 re-rendering을 방지할 수 있다.

#### 단점

1. 많은 양의 보일러 플레이트

아래 코드 예시를 보면서 더 알아보자.

2. 러닝 커브

`redux`를 제대로 사용하기 위해서는 위에서 이야기한 미들웨어가 필수적이다.

#### useReducer

`redux`를 이해하기 전 `useReducer`라는 훅에 대해 정리하고자 한다.  
`useReducer`는 build-in hook으로 `useState`의 대체이다.  
다만 차이점은 `useState`가 컴포넌트 내부에서 state를 다뤘다면 `useReducer`는 state 업데이트 로직을 컴포넌트로부터 분리한다.  
그래서 컴포넌트 내부 state가 많아지거나 state의 타입이 복잡해질 때 사용하면 좋다고 한다(개인적인 생각으로는 그 전에 컴포넌트 분리를 우선하는 게 좋을 거 같다).

다음에서 볼 수 있듯이 `redux`와 매우 비슷한 사용법을 보인다.

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'decrement':
      return { ...state, count: state.count - 1 }
    case 'increment':
      return { ...state, count: state.count + 1 }
    default:
      throw new Error('Unsupported action type:', action.type)
  }
}

function Main() {
  const [number, dispatch] = useReducer(reducer, { count: 0 })
  // 사용법 [state, dispatch] = useReducer(reducer, initialState, init)

  return (
    <>
      <h1>Count: {number.count}</h1>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
    </>
  )
}
```

#### 코드 예시(middleware는 없음)

`useSelector`, `useDispatch` 등의 hook이 등장하기 전에는 `connect`라는 HoC를 사용했었다.  
HoC는 [HoC](https://github.com/mochang2/development-diary/blob/main/030-react.md#cf-hochigher-order-component)에서 정리했듯이 hook이 나오기 전에 재사용이나 conditional rendering을 위해 사용했었다.

내가 생각하기에 `redux`에서 HoC와 hook의 특징은 다음과 같다.

- HoC
  - component에서 state 초기화, state 업데이트 로직을 분리할 수 있다.
  - 부모 컴포넌트가 re-rendering될 때 컴포넌트의 props가 바뀌지 않았다면 re-rendering을 자동으로 방지한다.
  - 코드의 양이 좀 더 많다.
- hook
  - 코드의 양이 좀 더 적다.
  - component 내부에 state 업데이트 로직이 존재한다.
  - 부모 컴포넌트가 re-rendering될 때 props가 바뀌지 않았다면 `React.memo`를 사용해서 re-rendering을 방지해야 한다.

취향에 따라 하나의 프로젝트에서 하나의 방법으로 통일해서 사용하는 것이 좋은 것 같다.

아래 예시는 [context api](https://github.com/mochang2/development-diary/blob/main/032-react%20state.md#context)와 같은 컴포넌트 구조를 사용했다.

```jsx
// index.js
import { Provider } from 'react-redux'
import store from './stores/reducers/reducer'

const root = ReactDOM.createRoot(document.getElementById('root'))

root.render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
)
```

```jsx
// components/App.js
import { useEffect } from 'react'
import { useDispatch, useSelector } from 'react-redux'
import { increase } from '../stores/actions/actions'

export function ParentComponent() {
  return (
    <div>
      여기는 메인 페이지입니다.
      <ChildComponent1 />
      <ChildComponent2 />
      <ChildComponent3 />
    </div>
  )
}

function ChildComponent1() {
  return (
    <div>
      <span>여기는 child1입니다.</span>
      <GrandChildComponent1 />
      <GrandChildComponent2 />
    </div>
  )
}

function ChildComponent2() {
  return (
    <div>
      <span>여기는 child2입니다.</span>
    </div>
  )
}

function ChildComponent3() {
  const dispatchCount = useDispatch()

  return (
    <div>
      <span>여기는 child3입니다.</span>
      <button onClick={() => dispatchCount(increase())}>click me!</button>
    </div>
  )
}

function GrandChildComponent1() {
  return (
    <div>
      <span>여기는 grandchild1입니다.</span>
    </div>
  )
}

function GrandChildComponent2() {
  const count = useSelector((state) => state.count)

  return (
    <div>
      <span>여기는 grandchild2입니다. count: {count}</span>
    </div>
  )
}
```

```jsx
// stores/reducers/reducer.js
import { legacy_createStore } from 'redux'
// createStore은 deprecated
// 공식 문서는 redux toolkit의 configureStore 사용을 권장함

const initialState = {
  count: 0,
}

const rootReducer = (state = initialState, action) => {
  switch (action.type) {
    case 'INCREASE':
      return { ...state, count: state.count + 1 }
    default:
      return state
  }
}

export default legacy_createStore(rootReducer)
```

```jsx
// stores/actions/actions.js

export const increase = () => ({
  type: 'INCREASE',
  // payload를 이용해서 reducer에서 action.payload를 이용해 업데이트 하기도 함
})
```

`state`에 변화가 있을 때 context 분리 등의 작업을 하지 않아도 `GrandChildComponent3`만 re-rendering된다는 장점이 있다.
다만 확실히 `count`, `setCount(count + 1)` 이러한 간단한 작업을 하는데 말도 안되게 많은 양의 코드가 필요하다.  
그리고 만약 미들웨어가 추가되면 코드 양이 훨씬 늘어날 것이다.

#### useSelector 효율적으로 사용하기

```jsx
const App = () => {
  const state = useSelector((state) => state)
  const name = state.User.name
  const age = state.User.age
  const address = state.User.address
  // const number = state.User.number // User안에 number 값도 있음

  return (
    <div>
      {`회원인 ${name}씨의 나이는 ${age}세이고, ${address}에 거주한다.`}
    </div>
  )
}
```

```jsx
const App = () => {
  const name = useSelector((state) => state.User.name)
  const age = useSelector((state) => state.User.age)
  const address = useSelector((state) => state.User.address)
  // const number = useSelector((state)=> state.User.number); User안에 number 값도 있음

  return (
    <div>
      {`회원인 ${name}씨의 나이는 ${age}세이고, ${address}에 거주한다.`}
    </div>
  )
}
```

코드는 첫 번째가 더 깔끔해보이지만 두 번째 `App` 컴포넌트가 더 효율적이다.  
왜냐하면 첫 번째 `App`은 `state.User.number`이 변경되어도 re-rendering된다.  
반면 두 번째 `App`은 `state.User.number`이 변경되어도 re-rendering되지 않는다.

### Recoil, Jotai

참고 및 사진 출처  
http://blog.hwahae.co.kr/all/tech/tech-tech/6099/  
https://velog.io/@turret1234/React-Jotai%EB%84%8C-%EB%88%84%EA%B5%AC%EB%83%90  
https://blog.logrocket.com/jotai-vs-recoil-what-are-the-differences/  
https://recoiljs.org/  
https://jotai.org/  
https://programming119.tistory.com/m/263

`recoil` 코드 예시는 [위](https://github.com/mochang2/development-diary/blob/main/032-react%20state.md#1-props-drilling)에서 props drilling의 비효율성을 증명할 때 썼고, `jotai`도 기본적인 사용법은 크게 다르지 않으므로 생략하겠다.

`recoil`의 특징 중에 동시성 모드 등 여러 react 기능들과 호환이 가능하다는 것이 있는데, 현재는 동시성 모드가 정식으로 출시된 것이 아니기 때문에 해당 내용은 제외했다.

#### 공통점

`jotai`는 `recoil`의 영향을 받아 만들어진 모듈이다.  
그래서 철학이나 사용법에 공통점이 존재한다.

1. atomic

![atom](https://user-images.githubusercontent.com/63287638/196420157-5fb81821-6c8d-4a15-bb3f-81480dd3ddcc.png)

context api는 컴포넌트의 state를 공통된 상위 요소까지 끌어올려야만 공유할 수 있으며, 이 과정에서 거대한 트리가 re-rendering되는 것을 야기하기도 한다.  
또한 context는 단일 값만 저장할 수 있으며, 여러 값들의 집합을 담을 수 없다.

이와 반대로 `recoil`과 `jotai`는 state를 atom이라는 작고 가벼운 것으로 잘게 쪼개 관리한다.  
각각의 atom들은 원할 때(on-demand) 독립적으로 사용될 수 있다(다만 `recoil`의 selector를 쓴다면 atom 간 종속적인 관계가 생길 수는 있다).  
이러한 atomic한 특징(state가 점진적이고 분산되어 있음) 덕분에 code splitting에 유리하다.

2. 낮은 러닝 커브

`useState` 또는 `useReducer`를 사용하는 것과 사용법이 많이 다르지 않다.  
단순히 컴포넌트 간 상태 공유를 위해서 사용한다면 가장 쉽다(selector 등의 기능이 추가되면 나름 복잡해지긴 하지만).

3. async query

`recoil`과 `jotai` 모두 async query를 제공한다.

```javascript
// recoil
const currentUserNameQuery = selector({
  key: 'CurrentUserName',
  get: async ({ get }) => {
    const response = await myDBQuery({
      userID: get(currentUserIDState),
    })
    return response.name
  },
})
```

```javascript
// jotai
const fetchCountAtom = atom(
  (get) => get(countAtom),
  async (_get, set, url) => {
    const response = await fetch(url)
    set(countAtom, (await response.json()).count)
  }
)
```

#### 차이점

[jotai 공식 문서](https://jotai.org/docs/basics/comparison)에 있는 내용을 포함해 각종 블로그들에 나와 있는 내용을 같이 정리했다.

1. 버전

2022년 10월 기준으로 `recoil`의 버전은 0.7.6이 최신이다.  
`jotai`는 1.8.6이 최신이다.  
`jotai`가 훨씬 빠르게 업데이트 되고 있다.  
만약 `recoil`를 사용한다면 v1 이전이기 때문에, 만약 v1이 되어 많은 변경 사항이 생기면 migration을 전부 개발자가 감당해야 하는 문제가 존재한다.  
(근데 v1 겁나게 안 내놓는게 페이스북에서 만드는 라이브러리들의 특징이기도 하단다)

2. 번들 사이즈

`recoil` v0.7.6의 번들 사이즈(unpacked)는 2.2MB이다.  
`jotai` v1.8.6의 번들 사이즈는 901KB이다.  
atom 상태의 변경불가능한 스냅샷을 찍는 기능들이 `recoil`에는 내장되어 있는 반면 `jotai`는 없다(순전히 전역 상태 관리를 위한 기능은 크게 다른 것 같진 않다).  
대신 공식적으로 `Immer`, `Optics`, `Redux`, `Zustand`와 같은 모듈과 같이 사용할 수 있는 방법들을 제공한다.

_cf) `jotai`가 강조하는 두가지 특징_
Primitive: 리액트 기본 state 함수인 `useState` 와 유사한 인터페이스
Flexible: atom들끼리 서로 결합 및 상태에 관여할 수 있고, *다른 라이브러리들과 원할한 결합을 지원*한다.

3. 다운로드 횟수

일반적으로 다운로드 횟수가 많으면 커뮤니티가 더 활성화 되어 있고, best practice에 대한 좋은 자료들이 많다.  
2022년 10월 기준으로 recoil이 조금 더 많이 다운로드되고 있다.

4. key 사용 여부

```javascript
// recoil
const counterState = atom({ key: 'counter', default: 0 })
```

```javascript
// jotai
const counterAtom = atom(0)
```

`recoil`은 atom을 선언할 때 key 값이 전역적으로 unique한 값이어야 한다.  
`jotai`는 atom을 선언할 때 key 값이 필요 없어서 보일러 플레이트 양이 미세하게 더 적다.  
[atoms in atom](https://jotai.org/docs/guides/atoms-in-atom)에 따르면 `jotai`는 atom들을 referential equality를 통해 구분한다고 한다.
이는 다음과 같은 잠재적인 문제를 가지고 있다.

> This could be a potential issue. For example, if you want to identify an atom for debugging, you are going to add `counterAtom.debugLabel = "counter"` anyway. One other difference is that if your module with atoms was updated, React Fast Refresh will not be able to preserve the old state, since all new atoms are no longer referentially equal to old ones (which works in Recoil because it compares the key string).

React Fast Refresh는 HMR를 대체할 react 기능이라고 한다.

~추후에 `jotai`를 사용하다가 문제를 겪게 되면 실제 사례를 추가하자.~

5. Provider

`recoil`은 최상위 component에서 `RecoilRoot`로 감싸줘야 한다.  
반면 `jotai`는 기본적으로 Provider를 사용하지 않아도 된다.  
Provider가 없더라도 atom을 선언할 때 설정된 기본 값을 가진 atom을 전역적으로 사용할 수 있다.
이는 `jotai` 가 내부적으로 react 의 context api를 이용하기 때문에 가능한 것인데, context api도 사실 Provider 없이 사용 가능하다.  
다만, Provier가 없다면 하위 component들에서 구독하는 context들이 re-rendering이 되지 않지만, `jotai` 는 이것을 가능하게 만들었으며, Provider 하위 component 전체가 re-rendering되는 (혹은 메모이제이션을 추가적으로 해주어야하는) 불편함을 개선했다.
`jotai`에서 만약 Provider를 사용하면 해당 state를 공유할 component를 범위를 제한할 수 있다.

## 3. 서버 상태 vs 클라이언트 상태

swr vs react-query 특징만 비교하고 best practice는 추후에 추가.

axios(fetch) vs react-query(swr)을 비교하는 게 아님. 각자 하는 역할이 다른 것임.

참고  
https://fe-developers.kakaoent.com/2022/220224-data-fetching-libs/

react 상태관리

client side state - local state + global state
server side state(서버로부터 받아오는 데이터)

기존에는 리덕스와 같은 상태 관리 라이브러리에 Global State와 Server State를 전부 포함하는 방법으로 프로그래밍을 했었는데요(그래서 비동기 로직이 포함되었으면 thunk나 saga와 같은 미들웨어 사용했음), 위와 같은 data fetching 라이브러리를 사용함으로써 상태 관리 라이브러리에서 비동기 로직을 제거하여 관심사가 분리되고 선언적으로 프로그래밍할 수 있게 되었습니다.

##

data fetching 라이브러리는 위에서 말한 리덕스의 단점들을 해결합니다.

- 선언적으로 프로그래밍 할 수 있음(장황하지 않은 코드)
- 동일한 API 요청이 여러 번 호출될 경우 한 번만 실행
- 데이터가 dirty 해진 경우 적절한 시점에 알아서 업데이트
- Global State와 Server State의 관심사를 분리

+) 비동기 처리 필요 x. 자동 캐싱. 서버 데이터에 대해서는 useState 훅 사용 안 해도 됨.

##

swr vs react-query

공통적으로
query, caching, polling, parallel queries, initial data, window focus re-fetching, network status re-fetching
제공

swr: 가벼움. dev-tool 사용하려면 서드 파티 라이브러리 필요. 전역 에러 핸들링 제공. 데이터를 조작할 수 있지만 mutation이 불편. 캐시 GC 기능 없음.

react-query: dev tool. 전역 에러 핸들링 제공. mutation 훅 제공. 캐시 GC 기능.

https://goongoguma.github.io/2021/11/04/React-Query-vs-SWR/
https://velog.io/@seohee0112/React-Query-vs-SWR
https://tech.madup.com/react-query-vs-swr/
