## 0. 공부하게 된 계기

성능을 향상시키는 적이 있냐는 질문을 많이 들어봤다.  
미리 방법을 공부한 뒤 프로젝트에 언제든지 도입할 수 있도록 해보고자 한다.

지금부터 진행하는 테스트는 `CRA`(react v18.2.0), `npm run eject`를 실행한 코드를 이용했다.

## 1. re-rendering

### 1) memoization

re-rendering 시간을 측정하기 위해 3가지 툴을 비교해봤다.

1. [benchmark](https://www.npmjs.com/package/benchmark)

- 장점
  - 다운로드 횟수가 많으며 자료가 많음.
  - 컴포넌트 반복 실행 가능.
  - 최소 반복 횟수 지정 가능.
- 단점
  - 서드 파티 라이브러리 필요.
  - 업데이트된지 오래됨.
  - 정확한 반복 횟수는 지정 못 함.

2. [react-performance-testing](https://www.npmjs.com/package/react-performance-testing)

- 장점
  - 최근 업데이트됨.
  - 코드 변경 없이 test 파일로 실행 가능.
- 단점
  - 서드 파티 라이브러리 필요.
  - 렌더링 시간이 특정 시간 이내인지 파악이 가능하나, 반복적인 렌더링이 정확히 얼마나 걸리는지 파악 못 함(가능하다고 하더라도 별도의 스크립트 파일 필요해보임).
  - 다운로드 횟수 적음.
  - ~예제 코드의 `wait()` 부분이 실행 안 됨. 버전 문제인가...~

3. [react developer tools](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi)

- 장점
  - 서드 파티 라이브러리 설치 필요 없음.
  - UI로 파악 가능.
- 단점
  - 반복적인 실행과 렌더링 시간 파악은 수동으로 가능.
  - 브라우저 성능, 네트워크 상태에 따라 렌더링 시간이 천차만별.

자료 많은게 짱이라고 생각하고, 반복 작업에 자동화를 생각했을 때 `benchmark`가 제일 낫다고 생각했다.

아래는 내가 짠 예제 코드이다.  
shell script도 이용해보고, 브라우저가 아닌 node로도 돌려봤지만 이게 제일 나은 대안인 것 같다.

```javascript
// index.js가 변경된 것을 감지하는 hot reloading을 이용(비교할 컴포넌트를 입력)
// index.js

import React from 'react'
import ReactDOM from 'react-dom/client'
import App from 'App'
import { WithMemoization, WithOutMemoization } from 'components/Test'

const root = ReactDOM.createRoot(document.getElementById('root'))

if (process.env.NODE_ENV === 'development') {
  const Benchmark = require('benchmark')
  const suite = new Benchmark.Suite()

  function performanceCheck1() {
    // @testing-library/react의 render는 매번 새롭게 컴포넌트를 생성하기 때문에
    // 메모이제이션이 안 되는 듯
    // root.render를 사용
    root.render(<WithMemoization />)
  }

  function performanceCheck2() {
    root.render(<WithOutMemoization />)
  }

  function compare(benchmark1, benchmark2) {
    return benchmark1.times.elapsed < benchmark2.times.elapsed
  }

  suite
    .add('w memo', performanceCheck1, { minSamples: 50 })
    .add('wo memo', performanceCheck2, { minSamples: 50 })
    .on('cycle', function (event) {
      // add eventlistener
      console.log(String(event.target))
    })
    .on('complete', function () {
      // console.log('Fastest is ' + suite.filter('fastest').map('name'))
      // 공식 문서 방법
      // Hz(1초에 해당 컴포넌트가 몇 번 실행될 수 있는지를 알려줌)를 비교

      // 리렌더링에 걸리는 시간을 알기 위해 다른 방법으로 표현
      console.log(suite)
      suite.sort(compare)
      console.log('Sort by elapsed time', suite.map('name').join(' '))

      root.render(
        <React.StrictMode>
          <App />
        </React.StrictMode>
      )
    })
    .run({ async: true })
}

root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
)
```

이해를 돕기 위해 찍은 스크린샷이다.  
`suite` 객체는 다음과 같은 구조를 가지고 있다.

![suite](https://user-images.githubusercontent.com/63287638/193517617-dddd57bd-eba5-4504-a624-dad81fb8016b.jpg)

```javascript
// components/Test.js
import { useMemo } from 'react'

const length = 10000000

export function WithMemoization() {
  const value = useMemo(
    () =>
      Array.from({ length }, (_, i) => Math.floor(i / 1000)).reduce(
        (acc, curr) => acc + curr,
        0
      ),
    []
  )

  return <div>{value}</div>
}

export function WithOutMemoization() {
  const value = Array.from({ length }, (_, i) => Math.floor(i / 1000)).reduce(
    (acc, curr) => acc + curr,
    0
  )

  return <div>{value}</div>
}
```

위 결과를 실행하면 메모이제이션을 한 컴포넌트는 10초 내외의 `elapsed time`을, 메모이제이션을 하지 않는 컴포넌트는 100초 내외의 `elapsed time`을 가진다.

극단적인 결과를 보이기 위해 위와 같이 코드를 짰다.  
실제로는 컴포넌트에 대략 n번 정도 re-rendering이 될 거 같다\~ 라고 가정하고 `minSamples`를 조정해야겠다.  
특정 횟수보다 적게 re-rendering되는 컴포넌트는 오히려 메모이제이션을 활용하는 것이 더 느리기 때문이다(예전에 증명한 글을 봤는데 그 글을 못 찾겠다. 그런데 어찌보면 당연한 말이다. 메모이제이션이 일종의 캐싱과 같은 역할을 하기 때문이다).  
또한 메모리는 무제한으로 사용 가능한 자원이 아니기 때문에 배포 단계에서 react 성능을 체크할 수 있는 사이트나 툴을 다시 이용해보는 것이 좋다.

#### compare

위 방법을 통해 `memo`, `useCallback`, `useMemo` 등의 메모이제이션을 활용하는 함수, 훅의 성능을 테스트할 수 있었다.  
다만 확실히 학습하고자 언제 re-rendering 되는지 dependency를 조금 더 뜯어보고자 한다.

많은 글들을 읽어보면 dependency가 변경되면 re-rendering될 때 해당 내용을 다시 실행되며 react의 불변성 철학 때문인지 dependency도 shallow compare한다(`Object.is()`로 비교한다고 한다).  
다음 코드로 테스트했다.

```jsx
function App() {
  const [count, setCount] = useState(0)
  const value = useMemo(() => {
    console.log('re-render')
    return count * 2
  }, [count])

  return (
    <div className="App">
      <header>{count}</header>
      <main>
        <button onClick={() => setCount(count + 1)}>+1</button>
      </main>
      <footer>{value}</footer>
    </div>
  )
}
```

위에는 당연히 버튼 누를 때마다 're-render'이 출력된다.  
하지만 아래는 그렇지 않다.

```jsx
function App() {
  let count = 1
  const value = useMemo(() => {
    console.log('re-render')
    return count * 2
  }, [count])

  return (
    <div className="App">
      <header>{count}</header>
      <main>
        <button onClick={() => count++}>+1</button>
      </main>
      <footer>{value}</footer>
    </div>
  )
}

// 또는

function func() {
  // App 내부에 선언하면 useMemo가 버튼을 누를 때마다 re-rendering됨
}

function App() {
  const [count, setCount] = useState(0)
  const value = useMemo(() => {
    console.log('re-render')
    return count * 2
  }, [func])

  return (
    <div className="App">
      <header>{count}</header>
      <main>
        <button onClick={() => setCount(count + 1)}>+1</button>
      </main>
      <footer>{value}</footer>
    </div>
  )
}
```

#### `React.memo` Deep compare

[위]()에서 봤듯이 react에서 모든 비교는 shallow comparison이라고 생각해도 된다.  
`React.memo`도 마찬가지이다.

dependency optimization - deep compare ? shallow compare ? 검색해도안 나오면 useMemo로 직접 let, const 변수도 선언해보고 함수도 컴포넌트 안과 밖에 선언해봐서 체크.

### 2) key props

key props

### 3) state 분리

> <렌더링>

리액트는 기본적으로 재귀적으로 컴포넌트를 렌더링 한다. 그러므로, 부모가 렌더링 되면 자식도 렌더링 된다.
렌더링 그 자체로는 문제가 되지 않는다. 렌더링은 리액트가 DOM의 변화가 있는지 확인하기 위한 절차일 뿐이다.
그러나 렌더링은 시간이 소요되며, UI 변화가 없는 불필요한 렌더링은 시간을 소비한다.

re-rendering => props.children. memo. useMemo 등
// 아래 경우 P가 render -> C가 render되므로 memo 의미 x

```jsx
const MemoizedChildComponent = React.memo(ChildComponent)

function ParentComponent() {
  const onClick = () => {
    console.log('Button clicked')
  }

  const data = { a: 1, b: 2 }

  return <MemoizedChildComponent onClick={onClick} data={data} />
}
```

// memo를 사용해서 re-render를 막을 수 있는 상황인지 살펴보기! 특히 props를 전달받지 않는다면 가능하지 않을까
// memo를 사용하지 말하야 할 경우? https://github.com/facebook/react/issues/14463
See how your app behaves in production mode, use React's profiling builds and the DevTools profiler to see where bottlenecks are, and strategically use these tools to optimize parts of the component tree that will actually benefit from these optimizations.

### state 분리

```tsx
// 아래 예시에서 state, setState를 그대로 child component에게 전달해줬는데, 실제로는 저렇게 쓰면 안됨

// 분리 이전
function App() {
  const [data, setData] = useState<DataType[] | null>(null)
  const [categoryOption, setCategoryOption] = useState(DEFAULT_OPTION) // DEFAULT_OPTION = '전체'
  const [searchText, setSearchText] = useState('')
  const [page, setPage] = useState(1)

  useEffect(() => {
    async function fetch() {
      const {
        data: { data },
      } = await api.get('/')

      setData(data)
    }

    fetch()
  }, [])

  return (
    <>
      <Filter>
        <Selection />{' '}
        {/* categoryOption, setCategoryOption를 전달받는 컴포넌트 */}
        <SearchInput /> {/* searchText, setSearchText를 전달받는 컴포넌트 */}
      </Filter>
      {faqs ? (
        <>
          <Table />
          {/* 필터링된 데이터를 토대로, 해당 페이지의 있는 데이터들을 보여주는 부분. 별도 컴포넌트 분리x */}
          <Pagination />
          {/* page, setPage를 전달받는 컴포넌트 */}
        </>
      ) : (
        <Loading />
      )}
    </>
  )
}
```

category, searchText가 변경될 때 DataList가 re-render되는 것은 어쩔 수 없음.
하지만 Page가 변경되면서 DataList가 re-render될 때, Filter 부분이 re-render할 필요가 없음.

```tsx
// 분리 이후
function App() {
  // ... 동일
  return (
    <>
      <Filter>{/* 동일 */}</Filter>
      {faqs ? (
        <FaqList
          faqs={faqs}
          categoryOption={
            categoryOption === DEFAULT_OPTION ? '' : categoryOption
          }
          searchText={searchText}
        />
      ) : (
        <Loading />
      )}
    </>
  )
}

function DataList({ data, categoryOption, searchText }: DataListProps) {
  const filteredData = data.filter(
    (datum) =>
      datum.category.includes(categoryOption) &&
      datum.title.includes(searchText)
  )

  const [page, setPage] = useState(1)

  return (
    <>
      <Table rowCount={PER_PAGE_COUNT + 1}>
        {HEADERS.map((header, index) => (
          <HeaderCell key={index}>{header}</HeaderCell>
        ))}
        {currentPageFaqs(filteredData, page).map((datum) => (
          <DataCell key={datum.no} />
        ))}
      </Table>
      <Pagination {/* some props */} />
    </>
  )
}
```

## 2. 리소스 크기 줄이기

production mode  
tree shaking  
additional html wrapper => <></>  
content compression

성능 향상을 위한 방법 or CSR의 단점 극복하는 방법

1. tree shaking: 실제로 쓰지 않는 코드들을 제외함. https://helloinyong.tistory.com/305

2. SSR 도입(항상 성능 향상은 아니다): https://d2.naver.com/helloworld/7804182

3. code splitting(chunk.js): 유저가 당장 필요한 정보에 우선순위를 두어 순서대로 로딩. dynamic import(React.lazy) / 여러 entry points
   https://velog.io/@y1andyu/React-%EC%BD%94%EB%93%9C-%EB%B6%84%ED%95%A0  
   https://medium.com/humanscape-tech/react%EC%97%90%EC%84%9C-%ED%95%B4%EB%B3%B4%EB%8A%94-%EC%BD%94%EB%93%9C-%EC%8A%A4%ED%94%8C%EB%A6%AC%ED%8C%85-code-splitting-56c9c7a1baa4

> wrapper 최소화

공용 컴포넌트, page 최상단은 wrapper로 감쌈
나머지는 <></>로 처리하고자 함

https://jelvix.com/blog/is-react-js-fast

## 3. 초기 렌더링 빠르게 하기

chunk file  
SSR

## 4. 서버측 응답 최적화하기

CDN  
web worker

## 5. 성능 측정 방법
