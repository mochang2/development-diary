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

### Props Drilling

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

`B`의 `setState()`가 호출되어, `B`의 리렌더링이 렌더링 큐로 들어간다.
리액트는 트리 최상단에서 부터 렌더링 패스를 시작한다.
`A`는 업데이트가 필요하다고 체크 되어 있지 않을 것이므로, 지나간다.
`B`는 업데이트가 필요한 컴포넌트로 체크되어 있으므로, `B`를 리렌더링 한다. `B`는 `C`를 리턴한다.
`C`는 원래 업데이트가 필요한 것으로 간주되어 있지 않았다. 그러나, 부모인 `B`가 렌더링 되었으므로, 리액트는 그 하위 컴포넌트인 `C`를 렌더링 한다. `C`는 `D`를 리턴한다.
`D`도 마찬가지로 렌더링이 필요하다고 체크되어 있지 않았지만, `C`가 렌더링된 관계로, 그 자식인 `D`도 렌더링 한다.
컴포넌트를 렌더링 하는 작업은, 기본적으로, 하위에 있는 모든 컴포넌트 또한 렌더링 하게 된다.

> 일반적인 렌더링의 경우, 리액트는 props가 변경되어 있는지 신경쓰지 않는다. 부모 컴포넌트가 렌더링 되어 있기 때문에, 자식 컴포넌트도 무조건 리렌더링 된다.

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

#### Re-render 코드로 살펴보기 - Props Drilling

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

`handleCount`가 발생해도 `<Button >`만 re-render 된다.

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

### Context API

built-in  
provider로 감싸져 있으면 값을 사용하지 않는 컴포넌트도 리렌더링 => children을 감싼 패턴으로 변경하면 hocs 같은 느낌? 다만 provider hell이 될 수 있다고 함.

https://dev.rase.blog/21-10-07-context-and-state-management/
context api 는 상태관리 도구로 사용하면 props drilling 을 회피한다는 장점보다 잃는 것이 더 많다.
단지 props drilling을 해결하기 위한 방법 정도.

context API - https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&ved=2ahUKEwjEiOa1-4v6AhWZm1YBHVHtBCsQFnoECAcQAQ&url=https%3A%2F%2Fcodemacaw.com%2F2021%2F11%2F21%2Fprevent-unnecessary-re-rendering-when-using-context-api%2F&usg=AOvVaw0hmf6N2lhTO0csUEhcV7L5 ,

https://chatoo2412.github.io/javascript/react/react-context-as-a-state-management-tool/

redux
recoil - https://velog.io/@yiyb0603/TypeScript-React-Recoil%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-TodoList-%EB%A7%8C%EB%93%A4%EC%96%B4%EB%B3%B4%EA%B8%B0
2022.09 현재 1 버전이 출시되지 않음. 1점대가 아니기 때문에 변경 사항이 많이 생김. migration 하는 것은 개발자가 감당해야 할 몫. 1점대가 아니면 real에서는 잘 안 씀.
jotai - 1버전이 넘음

언제 re-rendering 되는지. 어떠한 철학을 가지고 만들었는지.

## 3. 서버 상태 vs 클라이언트 상태

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
