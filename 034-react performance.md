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
  "sideEffects": ["./src/somewhere"] // side effect가 존재하지만 tree shaking의 대상으로 지정. regexp 사용 가능
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

additional html wrapper => <></>

> wrapper 최소화

공용 컴포넌트, page 최상단은 wrapper로 감쌈
나머지는 <></>로 처리하고자 함
https://jelvix.com/blog/is-react-js-fast

chunk file

3. code splitting(chunk.js): 유저가 당장 필요한 정보에 우선순위를 두어 순서대로 로딩. dynamic import(React.lazy) / 여러 entry points
   https://velog.io/@y1andyu/React-%EC%BD%94%EB%93%9C-%EB%B6%84%ED%95%A0  
   https://medium.com/humanscape-tech/react%EC%97%90%EC%84%9C-%ED%95%B4%EB%B3%B4%EB%8A%94-%EC%BD%94%EB%93%9C-%EC%8A%A4%ED%94%8C%EB%A6%AC%ED%8C%85-code-splitting-56c9c7a1baa4

SSR

2. SSR 도입(항상 성능 향상은 아니다): https://d2.naver.com/helloworld/7804182

## 4. 서버측 응답 최적화하기

CDN  
web worker
content compression

## 5. 성능 측정 방법
