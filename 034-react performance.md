## 0. 공부하게 된 계기

성능을 향상시키는 적이 있냐는 질문을 많이 들어봤다.  
미리 방법을 공부한 뒤 프로젝트에 언제든지 도입할 수 있도록 해보고자 한다.

지금부터 진행하는 테스트는 `CRA(v5.0.1)`, `npm run eject`를 실행한 코드를 이용했다.  
webpack은 v5이다.

## 1. re-rendering

### 1) memoization

[when to use memoization in react](https://im-developer.tistory.com/198) 이런 글들은 수두룩하지만 너무 추상적인 표현인 것 같다.  
직접 성능을 비교할 방법을 알아보고 싶었다.
이를 위해 re-rendering 시간을 측정하기 위한 3가지 툴을 비교해봤다.

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
shell script도 이용해보고, 브라우저가 아닌 node.js로도 돌려봤지만 이게 제일 나은 대안인 것 같다.

```javascript
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
    // elapsed 기준으로 내림차순
    return benchmark2.times.elapsed - benchmark1.times.elapsed
  }

  suite
    .add('w memo', performanceCheck1, { minSamples: 50 })
    .add('wo memo', performanceCheck2, { minSamples: 50 })
    .on('cycle', function (event) {
      // add eventlistener
      // name, operations/second(Hz), runned samples 출력
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
실제로는 컴포넌트에 대략 n번 정도 re-rendering이 될 거 같다\~ 라고 가정하고 `minSamples`를 조정해야 한다.  
특정 횟수보다 적게 re-rendering되는 컴포넌트는 오히려 메모이제이션을 활용하는 것이 더 느리기 때문이다(예전에 증명한 글을 봤는데 그 글을 못 찾겠다. 그런데 어찌보면 당연한 말이다. 메모이제이션이 일종의 캐싱과 같은 역할을 하기 때문이다).  
또한 메모리는 무제한으로 사용 가능한 자원이 아니기 때문에 배포 단계에서 react 성능을 체크할 수 있는 사이트나 툴을 다시 이용해보는 것이 좋겠다.

#### dependency compare

위 방법을 통해 `memo`, `useCallback`, `useMemo` 등의 메모이제이션을 활용하는 함수, 훅의 성능을 테스트할 수 있었다.  
다만 확실히 학습하고자 언제 re-rendering 되는지 dependency를 조금 더 뜯어봤다.

dependency가 변경되면 re-rendering될 때 훅으로 감싼 코드를 다시 실행한다.  
react는 dependency 변경을 확인하기 위해 shallow compare한다(`Object.is()`로 비교한다고 한다).  
이를 확인하고자 다음 코드로 테스트했다.

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
  // 만약 App 내부에 선언하면, useMemo가 버튼을 누를 때마다 re-rendering됨
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

[위](https://github.com/mochang2/development-diary/blob/main/034-react%20performance.md#compare)에서 봤듯이 react에서 모든 비교는 shallow comparison이라고 생각해도 된다(불변성 유지!).  
`React.memo`도 마찬가지이다.

그래서 아래와 같이 deep compare을 도와주게 하는 방법이 존재한다.

```jsx
function moviePropsAreEqual(prevMovie, nextMovie) {
  return (
    prevMovie.title === nextMovie.title &&
    prevMovie.releaseDate === nextMovie.releaseDate
  )
}

const MemoizedMovie = memo(Movie, moviePropsAreEqual)
```

[github issue](https://github.com/facebook/react/issues/14463)에 다음과 같은 글이 있다.

> You should always use React.memo LITERALLY, as comparing the tree returned by the Component is always more expensive than comparing a pair of props properties)

[TOAST UI](https://ui.toast.com/weekly-pick/ko_20190731)에 나와있듯이 props가 자주 변경되지 않는 컴포넌트면 사용하는 것이 좋다고 한다.

여기서 궁금증이 생겼다.  
애초에 렌더링된 컴포넌트는 메모리에 있는 것이기 때문에 `React.memo`의 효율성을 따지기 위해서는 단순히 re-rendering이 효과적인지 따지는 것이 아니라 props comparison의 성능과 메모이제이션의 성능을 비교해야 하지 않을까?  
만약 deep compare한다면 `React.memo`의 성능은 어떻게 될까?

다음과 같이 코드를 작성한 뒤 `benchmark`를 이용해 테스트했다.

```jsx
export function ParentComponent1() {
  // react.memo를 사용한 컴포넌트를 자식으로 가짐
  const [count, setCount] = useState(0)

  return (
    <div className="App">
      <header>I'm parent</header>
      <main>
        <button onClick={() => setCount(count + 1)}>+1</button>
        <div>{count}</div>
        <MemoizedComponent
          count1={1}
          // props 수십 개 선언
        />
      </main>
    </div>
  )
}

export function ParentComponent2() {
  // react.memo를 사용하지 않은 컴포넌트를 자식으로 가짐
  const [count, setCount] = useState(0)

  return (
    <div className="App">
      <header>I'm parent</header>
      <main>
        <button onClick={() => setCount(count + 1)}>+1</button>
        <div>{count}</div>
        <ChildComponent
          count1={1}
          // props 수십 개 선언
        />
      </main>
    </div>
  )
}

function ChildComponent({ ...rest }) {
  return <div>I'm child</div>
}

function compareProps(prevProps, nextProps) {
  // not best practice
  // 일부러 느리게 비교하기 위해서 만듦
  const prevPropsKeys = Object.keys(prevProps)
  const nextPropsKeys = Object.keys(nextProps)

  return (
    prevPropsKeys.length === nextPropsKeys.length &&
    prevPropsKeys.every((key) => prevProps[key] === nextProps[key]) &&
    nextPropsKeys.every((key) => prevProps[key] === nextProps[key])
  )
}

const MemoizedComponent = memo(ChildComponent, compareProps)
```

결과는 97번 렌더링하는데 메모이제이션을 사용한 컴포넌트가 11.367s, 메모이제이션을 사용하지 않은 컴포넌트가 11.178s 걸렸다.  
미세하지만 메모이제이션을 사용하지 않은 게 더 유리했다.  
물론 이 예시에서는 일부러 느린 compare 함수를 사용했으며 `ChildComponent`가 더 무거웠다면 다른 결과가 나왔을 것이다.

따라서 `React.memo`를 사용하기 전 해당 **컴포넌트의 props가 자주 변경되는지**와 자주 변경되지 않는다면 **메모이제이션이 compare보다 값어치가 있는지**를 확인해야 한다는 결론을 얻었다.

### 2) key props

해당 내용과 관련된 자세한 것은 [virtual DOM 파트](https://github.com/mochang2/development-diary/blob/main/029-virtual%20DOM.md#diffing-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98)에 기록해놨다.

간단히만 정리하자면 `key`는 모든 JSX가 기본적으로 갖는 property 값이다.  
전역적으로 unique할 필요는 없지만 형제 요소들 간에는 unique해야 한다.

보통 `return`에서 `Array.map`을 이용할 때 많이 사용되기 때문에 index를 `key`값으로 주는 경우가 많다.  
일반적인 경우에서는 문제가 되지 않지만 해당 데이터가 수정될 때 다시 정렬되는 과정을 거친 후에 rendering되는 데이터라면 재조정 과정에서 휴리스틱 알고리즘의 이점을 얻지 못한다.  
이럴 때는 형제 요소 간 노드의 순서가 변해도 unique하게 노드를 추적할 수 있는 `id` 값 등을 주는 것이 좋다.

### 3) state 분리

리액트는 기본적으로 재귀적으로 컴포넌트를 렌더링 한다.  
그러므로, 부모가 렌더링 되면 자식도 렌더링 된다.  
렌더링 그 자체로는 문제가 되지 않는다.  
렌더링은 리액트가 DOM의 변화가 있는지 확인하기 위한 절차일 뿐이다.  
그러나 렌더링은 시간이 소요되며, UI 변화가 없는 불필요한 렌더링은 시간을 소비한다.

부모에서 관리하던 state를 자식 요소로 내린다면 형제 요소 간의 불필요한 re-rendering을 발생시키지 않을 수 있다.

```tsx
// 분리 이전
function App() {
  const [data, setData] = useState<DataType[] | null>(null)
  const [categoryOption, setCategoryOption] = useState(DEFAULT_OPTION)
  const [searchText, setSearchText] = useState('')
  const [page, setPage] = useState(1) // 이 state 분리

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

`categoryOption`, `searchText`가 변경될 때 `Table`가 re-render되는 것은 어쩔 수 없다.  
하지만 `page`가 변경되면서 `DataList`가 re-render될 때, `Filter`가 re-render될 필요는 없다.

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

function FaqList({ data, categoryOption, searchText }: DataListProps) {
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

### 1) minify

빌드 산출물의 리소스 크기 자체를 줄이는 방법이다.  
JS나 CSS에서 여백, 빈칸, 주석 등을 제거하고 이름이 긴 변수는 짧게 줄임으로써 브라우저에 전달되는 코드 양을 줄인다.

#### style 최소화

webpack을 사용한 CSS minify는 [terser-webpack-plugin](https://github.com/mochang2/development-diary/blob/main/026-FE%20development%20environment.md#production-mode)을 사용하거나 [css-minimizer-webpack-plugin](https://yamoo9.gitbook.io/webpack/webpack/webpack-plugins/minimize-css-files)을 사용할 수 있다.

나는 `styled-components`를 많이 사용하기 때문에 해당 방법에 대해서만 정리하고자 한다.  
원래는 `babel-plugin-styled-components`을 설치한 뒤 바벨 설정 파일을 수정해야 하지만 v4부터 *babel macro*라는 것을 제공한다고 한다.  
사용법은 엄청 간단하다.

```jsx
import styled from 'styled-components'

// 아래처럼만 바꾸기

import styled from 'styled-components/macro'
```

(스크린샷을 안 찍었지만 실제로 아래 방법을 이용하면 번들 크기가 줄어들었다)

#### comment 제거

CRA에서 기본적인 설정이 되어 있다.

```javascript
module.exports = function (webpackEnv) {
  return {
    // ...
    optimization: {
      minimize: isEnvProduction,
      minimizer: [
        new TerserPlugin({
          terserOptions: {
            // ...
            output: {
              comments: false,
              // format: { comments: false } 로도 가능
            },
          },
          extractComments: false, // 추가. 하면 LICENSE 파일이 생기지 않음.
        }),
      ],
    },
  }
}
```

![remove comments](https://user-images.githubusercontent.com/63287638/193710169-3bee06d1-0a10-4f7c-88da-b193a3f1ddd8.png)

위는 주석을 제거하기 전, 아래는 주석을 제거한 후이다.

_참고) source map 제거_  
처음에는 `<hash>.js.map` 파일에 일부 주석이 그대로 남아 있어서 terser plugin이 이상한가 싶었다~해당 프로젝트 github issue도 다 뒤져보고 소스 코드도 다 뒤져봤다 ㅠㅠ~.  
알고보니 해당 이름의 파일은 source map 파일이고 진짜 빌드 파일은 `<hash>.js` 파일이었다.

source map 파일은 브라우저 내에서 원본 소스 코드를 확인할 수 있도록 도와주는 디버깅 파일이다.  
따라서 다음과 같은 이유로 실제 배포 시에는 제거되어야 한다.

1. 내부 코드가 노출된다. 난독화한 이유가 없어진다.
2. 메모리 부족 이슈가 발생할 수 있다. 이는 `devtool` 옵션을 `source-map`이 아닌 (development 모드에서 사용되는)`cheap-module-source-map`으로 바꿔서 해결할 수도 있다.

CRA는 간단히 `package.json`을 바꿔서 해결할 수 있다.

```json
{
  "scripts": {
    "bulld": "GENERATE_SOURCEMAP=false react-scripts build",
    // 또는
    "build": "react-scripts build && rm build/static/**/*.map"
  }
}
```

webpack 설정을 바꿀 수도 있다.  
간단히 `devtool: false` 설정하면 된다.

### 2) mocking 코드 제거

처음에는 `mockServiceWorker.js` 파일이 빌드 산출물에 minify도 되지 않고 그대로 포함돼서 없앨 방법을 찾아봤다.  
[github issue](https://github.com/mswjs/msw/issues/291)를 확인해보니 public 폴더에 있는 것은 그대로 복사되며, build에 포함된다고 해도 성능에는 문제가 없다고 한다.  
`index.html`에서 해당 파일을 다운로드 하지 않으므로 당연한 말이었다.

그래도 혹여 편-안하지 않다면 단순히 `package.json`의 scripts 부분에 수동으로 지워주게끔 추가해주자.

### 3) console.~~ 제거

CRA에서는 기본적으로 옵션이 설정되어 있지 않다.  
다음 옵션으로 제거 가능하다.

```javascript
module.exports = function (webpackEnv) {
  return {
    // ...
    optimization: {
      minimize: isEnvProduction,
      minimizer: [
        new TerserPlugin({
          terserOptions: {
            compress: {
              drop_console: true,
            },
          },
        }),
      ],
    },
  }
}
```

### 4) 안 쓰는 node_modules 제거

[CRA does not have devDependency](https://github.com/npm/npm/issues/2854)에서 알 수 있듯이 빌드할 때 이미 react 동작에 필요한 모듈들만 남는다고 한다.  
[npm prune --production](https://github.com/serverless/serverless/issues/569)을 통해 해당 동작을 진행하는 것 같다.  
실제로 안 쓰는 패키지 마구 설치한 뒤 build를 해도 산출물 용량이 동일하다.

### 5) tree shaking

참고: https://medium.com/naver-fe-platform/webpack%EC%97%90%EC%84%9C-tree-shaking-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0-1748e0e0c365 , https://helloinyong.tistory.com/305 , https://jforj.tistory.com/166 , https://huns.me/development/2265

tree shaking이란 코드를 빌드할 때 실제로 쓰지 않는 코드들을 제외한다는 의미이다.

#### side effect

tree shaking을 하기 전에 side effect를 알아야 한다.  
사용하지 않더라도 다른 코드에 영향을 끼칠 수 있다고 판단하여 빌드 산출물에 포함하는 경우가 있다.  
다음과 같은 경우이다.

1. 전역 함수를 사용하는 경우(`Object`, `Math`, `String`, `RegExp` 등)
2. 함수 실행 코드에서 멤버 변수를 변경하고 반환하는 경우
3. class 내부에서 `static` property를 사용하는 경우(단, static method는 side effect 발생하지 않음)

아래는 webpack3까지는 side effect라고 분류했는데 4부터 tree shaking을 자동으로 진행한다.

1. import한 모듈을 사용한 함수를 사용하지 않는 경우
2. import한 모듈을 다시 export default한 경우

webpack이 볼 때는 side effect지만 개발자가 볼 때는 아닌 경우가 있다.  
이런 경우 `package.json`에 이를 명시해줌으로써 export를 제거할 수 있다.

```json
{
  // ...
  "sideEffects": false, // 모든 모듈에서 side effect가 없는 경우
  "sideEffects": ["./src/somewhere"] // side effect가 존재하는 파일. regexp 사용 가능
}
```

#### import 됐지만 사용 안되는 모듈 제거

**아래 내용은 단순히 참고만 하자. import한 외부 라이브러리에 따라, 상황에 따라 다를 수 있다. 상황에 맞게 설정하자.**

다음과 같은 순서를 적용해야 한다.

1. ES6 모듈을 적용한다.

모든 모듈은 `import`, `export` 키워드를 사용한다.  
따라서 commonJS는 tree shaking이 되지 않는다.  
babel 설정도 추가해야 한다.

```javascript
// babel.config.js
{
  //
  "presets": [
    [
        "@babel/preset-env",
        {
            "modules": false
        }
    ],
],
}
```

2. side effect를 추가한다.

테스트해볼 코드는 다음과 같다.

한참 헤맨 부분인데 외부 라이브러리로 테스트하려고 했는데 [lodash](https://stackoverflow.com/questions/58741044/why-webpack-doesnt-tree-shake-the-lodash-when-using-import-as)나 [moment](https://luke-tofu.tistory.com/entry/Goodbye-Moment-feat-Dayjs)는 commonJS를 사용해 tree shaking이 제공되지 않는다고 한다.

```jsx
// App.jsx
import { ParentComponent1, ParentComponent2 } from './components/Memotest'
import dayjs from 'dayjs'
import * as utils from './lib/utils'

function App() {
  const today = utils.getToday()

  return (
    <div className="App">
      <header className="App-header">
        <ParentComponent1 />
        <ParentComponent2 />
      </header>
      <main>{today}</main>
    </div>
  )
}

// lib/utils.js
export const getToday = () => {
  return new Date().toISOString().slice(0, 10)
}

export const getDate = (datetime: string) => {
  return datetime.split(' ')[0]
}
```

위 코드에서 `dayjs`와 `utils.getDate`는 사용되고 있지 않다.  
tree shaking의 효과를 보기 위해 다음과 같은 순서로 실행했다.

**결과 1**  
tree shaking 미적용

![1](https://user-images.githubusercontent.com/63287638/193747023-e02e258c-7fa8-4ba1-8395-b6d171331efc.png)

**결과 2**
tree shaking 적용, `"sideEffects": false`, `import * as utils from './lib/utils`  
tree shaking 적용, `"sideEffects": false`, `import { getToday } from './lib/utils`

![2,3](https://user-images.githubusercontent.com/63287638/193747025-53d02473-375e-4d1d-b118-39616832e918.png)

**결과 3**
tree shaking 미적용, `import dayjs from 'dayjs` 삭제, `import { getToday } from './lib/utils`
tree shaking 미적용, `import dayjs from 'dayjs` 삭제, `import * as utils from './lib/utils`

**tree shaking 적용 x, 사용 안 하는 모듈 전부 삭제, default import**

![4,5](https://user-images.githubusercontent.com/63287638/193747026-d6cedcfc-f335-4e48-a510-0a6fcde4caaa.png)

두 가지 결론을 얻었다.

1. tree shaking보다 안 쓰는 코드가 삭제가 더 효과적이다(TODO: 이는 내가 한 실험에서만 편향된 결과일 수 있다. 실전에서 테스트해보고 내용을 추가하자).
2. default로 `import`하는 것과 부분 `import`가 차이가 없다. 이는 webpack4부터 자동으로 최적화해줘서라고 한다.

_side effects 실험_  
side effects가 효과가 있는지에 대한 실험은 [stack overflow](https://stackoverflow.com/questions/49160752/what-does-webpack-4-expect-from-a-package-with-sideeffects-false)에 있으니 여기를 참고하자.  
결론: side effects를 가진 모듈이 적을수록 번들 사이즈가 줄어든다.

## 3. 초기 렌더링 빠르게 하기

### 1) additional HTML wrapper 최소화

컴포넌트 구역을 나누기 위해 의미없는 `<div>` 등으로 감싸는 것보다 `<React.Fragment>`로 감쌈으로써 화면에 그릴 element를 최소화할 수 있다.  
단순히 구역을 나눌 때는 해당 방법을 적용할 수 있겠지만, 컴포넌트를 감싸는 wrapper까지 바꿀 필요는 없다고 생각한다.  
페이지마다 동일한 스타일을 보장해야되는 컴포넌트일 경우 특히 그렇다.

### 2) SSR

CSR vs SSR로 나눠서 작성하는 것은 너무 길어질 것 같다.  
정말 좋은 참고 자료이다: https://www.youtube.com/watch?v=YuqB8D6eCKE&t=549s

- CSR
  - 장점
    - 화면 깜박임이 없음
    - 초기 로딩 이후 구동 속도가 빠름
    - TTV(Time To View)와 TTI(Time To Interact) 사이 간극이 없음
    - 서버 부하 분산
  - 단점
    - 초기 로딩 속도가 느림
    - SEO에 불리
- SSR
  - 장점
    - 초기 구동 속도가 빠름
    - SEO에 유리함
  - 단점
    - 화면 깜빡임이 있음
    - TTV와 TTI 사이 간극이 있음
    - 서버 부하가 있음

CSR에서 렌더링에 대한 부하를 클라이언트가 나눴던 것을 다시 SSR을 통해 서버에게 부담을 전가할 수 있다.  
대신 초기 렌더링 때 많은 정적 파일을 클라이언트에 주지 않아도 된다.

https://medium.com/jspoint/a-beginners-guide-to-react-server-side-rendering-ssr-bf3853841d55 를 참고하면 Express로 SSR을 구현하는 방법이 상세하게 나와 있다.  
하지만 설정이 너무 복잡하고 백엔드에도 react, babel 관련된 모듈들을 설치해야 한다.  
공부 목적으로는 상관없겠지만 백엔드와 프론트엔드가 하는 일을 분리하기 위해서라도 나는 Next와 같은 SSR 프레임워크를 쓰는 게 맞다고 생각한다.

### 3) code splitting

code splitting은 SPA라 해도 해당 페이지에서 필요한 모듈만 브라우저에서 로딩할 수 있도록 코드를 chunk 단위로 분해하는 것이다.

_참고 chunk_  
chunk란 하나의 덩어리라는 뜻으로, 코드 스플리팅 시 생성되는 JS 파일 조각을 의미한다.

#### require.ensure

```javascript
require.ensure(
  ['dependency'],
  function (require) {
    var module = require('경로 또는 모듈') // 이렇게 require해서 사용하면 됨
  },
  'chunk name'
)
```

코드 아무 곳에서 `require.ensure`로 시작하는 함수를 호출할 수 있다.  
commonJS 방식이므로 이런 것만 있다는 것만 알고 넘어가겠다.

#### import

webpack을 이용한 code splitting 방식이다.  
`@babel/plugin-syntax-dynamic-import` 모듈을 설치하여 바벨 설정에 적용한 뒤 다음과 같이 코드를 짠다.

```javascript
import('./math').then((math) => {
  console.log(math.add(16, 26))
})
```

CRA를 하면 기본적으로 사용할 수 있는 방식이라고 한다.  
해당 방식을 사용하면 `catch`를 붙여서 에러 핸들링을 할 수 있는 장점이 있다.  
또한 webpack이 해당 코드가 필요할 때마다 비동기적으로 로딩해주므로 추가적으로 설정할 부분이 적다.

webpack chunk 파일에 대한 캐시 방법에 대해서는 https://www.zerocho.com/category/Webpack/post/58ad4c9d1136440018ba44e7 를 참고하자.  
hash와 chunkhash의 옵션을 제공한다.

#### React.lazy()

react에서 기본적으로 제공해주는 `React.lazy()`가 있다.  
[공식 문서](https://reactjs.org/docs/code-splitting.html)에서 3가지 상황에 대해서 추천한다.

1. App 최상단에서 page 컴포넌트(routing)
2. user interaction으로 화면에 보이는 컴포넌트
3. 페이지가 길어서 처음 viewport에 나오지 않는 컴포넌트

##### 테스트

나는 간단하게 page routing에 대해 code splitting을 적용해봤다.

코드는 엄청 간단하다.  
기존 코드를 주석처리하고 `pages`에서 import한 컴포넌트를 `lazy`로 감싸줬다.  
아쉽게도 아직까지는 `lazy`로 감싸는 컴포넌트는 default export된 컴포넌트에 대해서만 기능을 제공한다고 한다.

```tsx
import { Suspense, lazy } from 'react'
import { BrowserRouter, Route, Routes } from 'react-router-dom'
// import {
//   Coupon,
//   Faq,
//   FaqCategoryManagement,
//   FaqDetail,
//   FaqRegister,
//   FaqUpdate,
//   Inquiry,
//   Main,
//   Notice,
//   Page404
// } from 'pages'
import ROUTE from 'routes/routeMap'

const Coupon = lazy(() => import('pages/Coupon'))
const Faq = lazy(() => import('pages/Faq'))
const FaqCategoryManagement = lazy(() => import('pages/FaqCategoryManagement'))
const FaqDetail = lazy(() => import('pages/FaqDetail'))
const FaqRegister = lazy(() => import('pages/FaqRegister'))
const FaqUpdate = lazy(() => import('pages/FaqUpdate'))
const Inquiry = lazy(() => import('pages/Inquiry'))
const Main = lazy(() => import('pages/Main'))
const Notice = lazy(() => import('pages/Notice'))
const Page404 = lazy(() => import('pages/Page404'))

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>loading...</div>}>
        <Routes>
          <Route path={ROUTE.main} element={<Main />} />
          <Route path={ROUTE.coupon} element={<Coupon />} />
          <Route path={ROUTE.notice} element={<Notice />} />
          <Route path={ROUTE.faqRegister} element={<FaqRegister />} />
          <Route path={ROUTE.faqUpdate} element={<FaqUpdate />} />
          <Route path={ROUTE.faqDetail} element={<FaqDetail />} />
          <Route path={ROUTE.faq} element={<Faq />} />
          <Route
            path={ROUTE.faqCategoryManagement}
            element={<FaqCategoryManagement />}
          />
          <Route path={ROUTE.inquiry} element={<Inquiry />} />
          <Route path="*" element={<Page404 />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  )
}
```

lazy loading을 적용하는 부분을 `<Suspense>`로 감싸준다.  
`fallback`은 chunk 파일을 불러오는 도중 보여줄 로딩과 같은 요소를 넣는다.  
에러 핸들링은 [여기](https://reactjs.org/docs/code-splitting.html#error-boundaries)를 참고하자.

코드를 수정한 후 빌드하니 다음과 같은 chunk 파일들이 엄청 많이 생겼다.

![build result](https://user-images.githubusercontent.com/63287638/194357358-13f97bbf-3862-4ffc-b976-ee6f861bc5a2.jpg)

**빌드 산출물 비교**

1. Chrome Dev Tools
   ctrl + shift + p를 눌러 'show coverage'를 검색하면 현재 페이지에서 쓰이지 않지만 파싱된 코드의 비율을 볼 수 있다.

![before code splitting(coverage)](https://user-images.githubusercontent.com/63287638/194357371-2cc498fe-8334-4ab2-ac63-497602beb9be.jpg)

![after code splitting(coverage)](https://user-images.githubusercontent.com/63287638/194357350-063529b0-9c3a-4eac-95dd-e8fc5a78bb98.jpg)

위 사진은 code spliiting 전, 아래 사진은 code splitting 후이다.  
(예제 코드가 `<Main>`은 비어있는 컴포넌트고 `<Faq>`만 구현된 거라 좀 편향되긴 했지만) 99%나 되던 비율이 46%까지 떨어졌다.

2. `@babel/webpack-bundle-analyzer`
   [eject 없이 webpack-bundle-analyzer 사용하기](https://velog.io/@code-bebop/eject-%EC%97%86%EC%9D%B4-CRA%EC%9D%98-webpack.config-%EC%88%98%EC%A0%95%ED%95%98%EA%B8%B0)를 참조했다.

![before bundle analyzer](https://user-images.githubusercontent.com/63287638/194357375-acf32d69-1287-480f-becf-f7f9178fed38.jpg)

![after bundle analyzer](https://user-images.githubusercontent.com/63287638/194357365-a6a972f5-7765-45c1-ae69-6d1aeb817913.jpg)

841KB나 되던 main 번들이 689KB까지 줄어들었다.

## 4. 서버측 응답 최적화하기

TODO: 다뤄본 부분이 아니라서 서비스 배포한 뒤에 추가하자

CDN  
web worker
content compression

## 5. 성능 측정 방법

### 1) 성능과 관련된 키워드

- FPS(Frame Per Second)
  - 초당 화면에 그려지는 프레임의 개수. 60 이상이 사람이 느끼기에 불편하지 않은 수치라고 함.
- FTTB(First Time To Byte)
  - HTTP를 요청했을 때 처음 byte(정보)가 브라우저에 도달하는 시간. 작을수록 유저 경험과 SEO에 유리.
- LCP(Largest Content Paint)
  - 사용에 불편이 없을 정도로 주요 컨텐츠들이 로드된 시점.
- FID(First Input Delay)
  - 버튼 클릭 등 interaction이 발생할 때, 그에 따른 반응이 일어나는데 얼만큼의 시간이 걸리는지.
- CLS(Cumulative Layout Shift)
  - 레이아웃이 얼마나 안정되어 있는지를 나타내는 지표. 텍스트를 읽고 있는데 갑자기 높이를 차지해 버린 이미지 때문에 스크롤을 내리는 등 차례차례 다른 시점에 생성되는 요소들 때문에 자꾸만 요소들이 기존의 위치에서 벗어나는 일 등이 발생하면 수치가 커짐.

### 1) Chrome Devtools

[공식 문서](https://developer.chrome.com/docs/devtools/evaluate-performance/)가 너무 잘 나와 있어서 간단히 정리만 하겠다.

1. 녹화를 한다.
2. CPU 윗 파트(FPS)가 붉은 줄이거나 CPU 파트가 보라색, 초록색으로 가득차 있다면 해당 부분을 위주로 분석한다.
3. Main 파트를 확인한 뒤 spread를 클릭하면 bottleneck과 해당 bottleneck을 발생시키는 코드 위치를 파악할 수 있다.

예시 코드는 다음 부분에서 bottleneck이 발생한다.

```javascript
app.update = function (timestamp) {
  for (var i = 0; i < app.count; i++) {
    var m = movers[i] // movers = document.querySelectorAll('.mover')
    if (!app.optimize) {
      var pos = m.classList.contains('down')
        ? m.offsetTop + distance
        : m.offsetTop - distance
      if (pos < 0) pos = 0
      if (pos > maxHeight) pos = maxHeight
      m.style.top = pos + 'px'
      if (m.offsetTop === 0) {
        m.classList.remove('up')
        m.classList.add('down')
      }
      if (m.offsetTop === maxHeight) {
        m.classList.remove('down')
        m.classList.add('up')
      }
    } else {
      var pos = parseInt(m.style.top.slice(0, m.style.top.indexOf('px')))
      m.classList.contains('down') ? (pos += distance) : (pos -= distance)
      if (pos < 0) pos = 0
      if (pos > maxHeight) pos = maxHeight
      m.style.top = pos + 'px'
      if (pos === 0) {
        m.classList.remove('up')
        m.classList.add('down')
      }
      if (pos === maxHeight) {
        m.classList.remove('down')
        m.classList.add('up')
      }
    }
  }
  frame = window.requestAnimationFrame(app.update)
}
```

위 코드에서 비효율성을 초래하는 부분은 높이를 확인할 때 `pos`를 확인하냐 `offsetTop`을 확인하냐의 차이인 것 같다.  
혹시 `offsetTop`과 관련된 비효율성이 있는지 싶어서 검색해봤는데 해당 내용은 없다.  
결국 모든 `mover`에 대해 `offsetTop`을 알기 위해 지속적으로 DOM에 접근해서 느려진 것 같다.

#### FPS

원래 FPS가 performance 탭에 바로 존재한다고 하는데, (정확히 언제인지는 모르겠지만) 언젠가부터 사라진 것 같다.  
지금은 확인하기 위해서는 ctrl + shift + p를 누른 후 'show frame per second'를 입력하면 현재 화면의 FPS를 확인할 수 있다.

![fps](https://user-images.githubusercontent.com/63287638/194481200-f051cb7a-7e4c-4b9d-8b72-183b1e1c34db.jpg)

### 2) React Developer Tools

[블로그](https://medium.com/wantedjobs/react-profiler%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%97%AC-%EC%84%B1%EB%8A%A5-%EC%B8%A1%EC%A0%95%ED%95%98%EA%B8%B0-5981dfb3d934)에 사용법은 상세히 나와 있다.  
Profiler와 General을 이용하면 어느 컴포넌트를 렌더링할 때 느린지, 어떤 state가 변경될 때 불필요한 렌더링이 발생하는지 알 수 있다.

하지만 블로그 예시처럼 두 개의 컴포넌트를 직접적으로 비교하는 것은 자주 사용하지 못할 것 같다.  
테스트하기 위해 수정해야될 코드가 너무 많기 때문이다.  
그리고 렌더링 시간이 매번 달라지기 때문에 절대적인 렌더링 시간보다 bottleneck을 찾는데 사용해야 될 것 같다.
