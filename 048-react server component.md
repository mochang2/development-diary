(작성중)  
(현재는 실험 버전을 넘어 공식적으로 지원한다지만 아직 자료가 적은 편인 듯. 프로젝트 진행해보며, 그리고 다른 글들을 계속 읽어보며 추가할 예정)

## 0. 공부하게 된 계기

React v18에 새로 나온 기능이다.

> Fiber 아키텍처는 React의 내부 재귀 rendering 알고리즘을 **비동기적**으로 작동하도록 변경했다.  
> 이를 통해 React는 작업을 나누고 우선 순위를 지정하여 더 효율적으로 작업을 수행할 수 있게 되었다.  
> 다른 말로 증분 rendering이 가능해졌다고 한다.  
> Fiber 아키텍처를 통해 React는 우선순위가 높은 작업을 먼저 처리하고(예를 들어, scroll 유저 인터렉션 -> button 클릭 -> fetch한 데이터 rendering), 필요한 경우 중단하며 나중에 다시 시작할 수 있게 되었다.

React Server Component(이하 RSC, RCC는 React Client Component: 기존 React에서 사용하던 컴포넌트 개념)는 Fiber 아키텍처의 이점을 활용하여 서버 측에서 컴포넌트를 처리하고, 필요한 부분만을 클라이언트로 전송한다.  
그리고 내가 진행하는 프로젝트에서 Next를 쓸 예정인데, Next는 v13부터 RSC를 공식적으로 지원하고 있다.  
공부하지 않을 이유가 없다.

문법적인 측면이나 RSC / RCC가 쓸 수 있는 API 등은 공식 문서에 더 잘 나와 있으니까 자세히 적진 않았다.

참고  
https://www.plasmic.app/blog/how-react-server-components-work  
https://yceffort.kr/2022/01/how-react-server-components-work  
https://ui.toast.com/weekly-pick/ko_20210119  
https://github.com/reactjs/rfcs/blob/bf51f8755ddb38d92e23ad415fc4e3c02b95b331/text/0000-server-components.md  
https://www.webscope.io/blog/server-components-vs-ssr  
https://www.thearmchaircritic.org/mansplainings/react-server-components-vs-server-side-rendering  
https://github.com/mochang2/development-diary/issues/8

## TL;DR

> RSC **run only on the server and have zero impact on bundle-size**. Their code is never downloaded to clients, helping to reduce bundle sizes and improve startup time.
> RSC **can access server-side data sources**, such as databases, files systems, or (micro)services.
> RSC **seamlessly integrate with RCC**. RSC can load data on the server and pass it as props to RCC, allowing the client to handle rendering the interactive parts of a page.
> RSC **can dynamically choose which RCC to render**, allowing clients to download just the minimal amount of code necessary to render a page.
> RSC **preserve client state when reloaded**. This means that client state, focus, and even ongoing animations aren’t disrupted or reset when a Server Component tree is refetched.
> RSC **are rendered progressively and incrementally stream rendered units of the UI to the client**. Combined with Suspense, this allows developers to craft intentional loading states and quickly show important content while waiting for the remainder of a page to load.
> Developers can also **share code between the server and client**, allowing a single component to be used to render a static version of some content on the server on one route and an editable version of that content on the client in a different route.

## 1. 개요

![react-tree](https://github.com/mochang2/development-diary/assets/63287638/8842418f-14fc-4863-870a-511a24024d3c)

위 그림은 React 팀이 최종적으로 그리고 있는 구조라고 한다.  
현재는 `props.children` 기법을 활용하지 않으면 RSC는 RCC 하위에 rendering될 수 없으며, 유저 인터랙션이 포함된 내용은 클라이언트 컴포넌트에서만 작성이 가능하다.  
개인적인 생각으로는 최종 목표가 달성되면 `props.children` 없이, 즉 RCC에서 RSC를 직접 import함으로써 RCC에 RSC를 둘 수 있는 거는 아닌지...

RSC를 알기 전에 먼저 RSC 설명에 자주 사용되는 용어들부터 정리하고자 한다.

### serialization

"컴포넌트가 serialize된다"는 표현은 **RSC에서 컴포넌트의 상태와 로직이 서버에서 클라이언트로 전송되고, 클라이언트에서 재사용될 수 있도록 변환되는 과정**을 의미한다.  
React 컴포넌트는 일반적으로 클라이언트 측에서 JS로 실행되기 위해 구성된다.  
그러나 RSC는 서버 측에서 실행되는 동안 컴포넌트의 상태와 로직을 클라이언트로 전송할 수 있어야 한다.  
이를 위해 컴포넌트는 일종의 serialization 과정을 거쳐야 한다.

컴포넌트의 serialization은 컴포넌트 인스턴스를 일련의 데이터 형식(일종의 JSON 형태, 아래에서 자세히 다룸)으로 변환하는 과정이다.  
이 데이터 형식은 클라이언트로 전송된 후, 클라이언트에서 다시 deserialization하여 클라이언트 측에서 실행 가능한 형태로 재구성한다.  
이렇게 함으로써 서버에서 생성된 컴포넌트의 상태와 로직을 클라이언트에서 동일하게 재사용할 수 있다.

_참고_  
RSC가 동작이 가능해지기 위해서는 모든 컴포넌트가 serializable해야 한다.  
이 때문에 RSC에서는 이벤트 핸들링 내용을 props로 전달할 수가 없다.  
RSC가 일반 HTML로 변경되는 것, RCC를 module reference로 표현하는 것 모두 seriazation의 일종으로 볼 수 있다.

### streaming

RSC의 streaming은 **컴포넌트의 rendering을 단계적으로 처리하고 전송하는 방식**을 의미한다.  
또 다른 표현으로 **컴포넌트의 일부가 rendering되는 동안 이벤트나 데이터를 통해 서버와 클라이언트 간에 실시간으로 통신하는 방식**을 의미한다.  
RSC는 컴포넌트를 여러 단계로 나누어 서버에서 클라이언트로 전송할 수 있다.  
이를 통해 초기 요청 시에 필요한 최소한의 데이터를 먼저 전송하고, 이후 추가적인 데이터를 streaming 방식으로 전송하여 컴포넌트를 완전히 rendering할 수 있다.

streaming 방식으로 RSC를 요청하면, 서버는 클라이언트로부터 추가 데이터를 요청하는 청크(request chunk)를 받을 수 있다.  
이후 서버는 해당 요청 청크에 필요한 데이터를 응답으로 보내고, 클라이언트는 이를 받아 컴포넌트를 계속해서 rendering하고 업데이트한다.  
이 과정을 반복하여 컴포넌트의 완전한 rendering과 상호작용을 구현할 수 있다.

_참고) vs code splitting_  
RSC의 streaming: 컴포넌트의 rendering과 데이터 전송을 단계적으로 처리하는 방식.  
code splitting: 일반적으로 웹 애플리케이션에서 번들된 JS 파일은 초기에 모두 다운로드되어야 함. 이때 번들된 JS 코드를 작은 청크로 분할하여 필요한 부분만 로드하는 기술.

### RSC

RSC는 한 마디로 React 애플리케이션을 rendering할 때 서버의 도움을 같이 받는 거라고 할 수 있다.
'엥 이게 무슨 소리지?' 싶으면 기존 React 애플리케이션을 생각해보자.

- 사용자가 페이지를 방문한다.
- 서버로부터 애플리케이션 코드를 받는다. 여러 가지 asset들도 다운로드한다.
- 클라이언트에서 React element tree를 만든다.
- DOM을 rendering하고 이벤트를 다는 등의 작업을 한다.
- state가 변화하면 re-rendering한다.

RSC는 이 작업을 서버와 분담해서 처리한다.  
서버 측에서 할 수 있는 일을(예를 들면 data fetching) 처리하고 나머지는 브라우저에게 넘겨준다.  
즉, RSC를 사용하면 서버와 클라이언트는 React element tree를 서로 협력하여 rendering 할 수 있다.

### 장점

- 서버-클라이언트 상호작용이 가능하다.
  - 서버와 클라이언트 간에 컴포넌트의 상태와 로직을 공유하여 실시간 상호작용을 처리할 수 있다.
- data를 더 빠르게 가져올 수 있다.
  - 서버는 API 엔드포인트를 통하지 않고 하는 데이터를 직접 가져올 수 있다. 이는 일반적으로 데이터 소스와 더 밀접하게 연결되어 있어 브라우저보다 더 빠르게 데이터를 가져올 수 있다.

```js
// Note.server.js

import db from 'db.server';

function Note({ id }) {
  const note = db.notes.get(id);
  return <NoteWithMarkdown note={note} />;
}
```

- 가벼워진다.
  - 모든 노드 모듈을 브라우저에 다운로드할 필요가 없어진다. 즉, rendering에 필요하던 별도의 JS 코드를 브라우저에 다운로드할 필요가 없어진다.
  - 또한 API 요청 시 "processed"된 데이터만 받아 정말 *rendering*만 할 수 있다.
  - 기존의 SSR에서도 클라이언트로 전송되는 HTML과 JS 번들에는 페이지에서 사용되는 모든 컴포넌트의 코드가 포함되어 있다. 따라서 초기에 전송되는 데이터의 크기는 상대적으로 크다. 하지만 RSC에서는 컴포넌트의 최소한의 JS 코드만을 전달하고, 컴포넌트의 초기 상태와 상호작용을 위해 필요한 데이터만 전송한다. 이렇게 하면 초기에 전달되는 데이터의 크기가 기존의 번들 크기보다 작아질 수 있다.
- 자동 code splitting이 가능하다.
  - 기존에 dynamic import 식으로 진행한 code splitting을 사용하면 두 가지 한계가 있다. 하나는 개발자가 직접 `React.lazy`를 일일이 추가해줘야 됐다는 것이고, 다른 하나는(RSC에서는) 선택한 컴포넌트가 로드하기 시작하는 시점을 지연시켜 JS 코드를 덜 다운로드하는 이점을 상쇄한다.
  - RSC에서 이 한계를 두 가지 방식으로 다룬다. 첫 번째는 RSC에서 import하는 모든 RCC를 code splitting 가능 지점으로 간주하는 것이다. 두 번째는 개발자가 어떤 컴포넌트가 먼저 rendering되는지 직접 선택하게끔 해서 좀 더 일찍 다운로드 받게 하는 것이다.
- waterfall fetching을 줄일 수 있다.
  - 아래 코드에서 note가 null이고 `else` 내부에서 또다시 data fetching을 해야 하는 컴포넌트가 rendering되어야 한다고 하자. 그러면 `Note` 컴포넌트에서 data fetching이 끝나기 전까지 `SomeComponent`는 data fetching을 할 수 없고 이는 사용자 경험을 해칠 수 있다.
  - 하지만 이때 `Note`를 RSC로 바꾼 뒤 `note` 데이터를 직접 db에서 접근해서 가져온다면 `SomeComponent`를 (서버에서) rendering하며 data fetching이 가능하다.
  - 만약 data fetching이 지연된다면, 클라이언트는 기다리는 동안 아무런 컨텐츠를 표시하지 않을 수 있다. 즉, 서버의 응답 속도에 따라 초기 로딩 속도가 영향을 받을 수 있다. 그러나 RSC는 지연 데이터 액세스를 통해 서버에서 데이터를 가져오므로 일부 컨텐츠는 서버에서 도착하는 대로 클라이언트에게 보여질 수 있다. 예를 들어, 초기 데이터를 받지 못한 상태에서 빈 화면이나 로딩 스피너를 표시하고, 데이터가 서버에서 도착하면 해당 부분을 업데이트(streaming 방식)하여 컨텐츠를 표시할 수 있다.

```js
// Note.js

function Note(props) {
  const [note, setNote] = useState(null);
  useEffect(() => {
    // NOTE: loads *after* rendering, triggering waterfalls in children
    fetchNote(props.id).then((noteData) => {
      setNote(noteData);
    });
  }, [props.id]);

  if (note === null) {
    return 'Loading';
  } else {
    return <SomeComponent />;
  }
}

// 아래와 같이 바뀌면 no waterfalls 가능
// Note.server.js - Server Component

function Note(props) {
  // NOTE: loads *during* render, w low-latency data access on the server
  const note = db.notes.get(props.id);
  if (note == null) {
    // handle missing note
  }
  return (/* render note here... */);
}
```

- 함수 composition, 추상화로 인한 추가 비용을 피할 수 있다.

  - [컴파일에 대한 자세한 지식을 모르므로 참고만](https://github.com/reactjs/rfcs/blob/bf51f8755ddb38d92e23ad415fc4e3c02b95b331/text/0000-server-components.md). JS는 AOT(Ahead-of-Time) 컴파일이 불가능한 언어이지만, 적어도 RSC를 이용하면 추상화에 따른 비용을 서버가 부담하게 할 수 있다.

### 단점

공식 문서에서 얘기하는 단점은 새로운 기술 도입에 따른 러닝 커브 정도밖에 없다.  
내가 쓰면서 느낀 바가 있으면 추가하겠다.

## 2. RSC rendering의 life cycle

우선 서버가 rendering 요청을 받는다.  
참고로 최상단의 root 엘리먼트는 RSC이며, 다른 RSC 또는 RCC를 rendering할 수 있다.

이후 서버는 RSC를 `div`, `p`와 같은 html 코드로 변환하고, RCC를 일종의 JSON 형태의 placeholder로 대체해서 클라이언트에 전달한다.  
클라이언트는 서버에서 받은 내용들을 deserialize해서 해석해야 한다.
아래 사진은 서버에서 받은 데이터를 토대로 최종적으로 클라이언트가 만들 React tree이다.

![re-constructed react tree](https://github.com/mochang2/development-diary/assets/63287638/c5381e8d-33b8-431c-bb73-140902d76ce6)

다만 이때의 JSON은 React element의 특성상 일반적인 `JSON.stringify()`함수를 통해 serialize할 수 없다.

```js
<div>oh my</div>;
// > React.createElement("div", { title: "oh my" })
// {
//   $$typeof: Symbol(react.element),
//   type: "div",
//   props: { title: "oh my" },
//   ...
// }

function MyComponent({ children }) {
  return <div>{children}</div>;
}
// > React.createElement(MyComponent, { children: "oh my" });
// {
//   $$typeof: Symbol(react.element),
//   type: MyComponent  // reference to the MyComponent function
//   props: { children: "oh my" },
//   ...
// }
```

`<div>oh my</div>`와 같은 기본 HTML은 JSON으로 처리가 가능해서 특별히 처리할 것이 없다.  
따라서 RSC는 props와 함께 호출한 뒤 그 결과를 JSON으로 만들어서 React element tree를 만들 수 있다.  
다시 말하지만 **RSC는 HTML 태그로 바꾼 뒤 JSON으로 전달할 수 있다**.

하지만 React element(`MyComponent`)는 단순히 string으로 표현할 수 있는 형태가 아니기 때문에(특히 `type: MyComponent`) 바로 JSON으로 serialize하기 까다롭다.  
그래서 React 팀은 serializable한 reference를 만들기 위해 "module reference"라는 것을 도입하여, React element의 serialize는 `ReactFlilghtServer.js` 내에 `resolveModelToJSON()` 함수를 이용한다.  
RCC seririalized 내용을 보면 아래와 같이 생겼다.  
참고로 RCC에 대한 참조를 serialize할 수 있는 "module reference"는 `react-server-dom-webpack`에서 이루어진다.

```js
{
  $$typeof: Symbol(react.element),
  // The type field  now has a reference object,
  // instead of the actual component function
  type: {
    $$typeof: Symbol(react.module.reference),
    name: "default",
    filename: "./src/ClientComponent.client.js"
  },
  props: { children: "oh my" },
}
```

최종적으로 클라이언트는 전달받은 JSON을 deserialize해서 placeholder 자리(즉, "module reference"인 element를 만날 때마다)에 RCC를 채우고 최종 결과를 rendering한다.  
사용자 인터렉션이 발생하는 등의 이유로 re-rendering이 필요하다면 RSC 형식의 새 컨텐츠를 서버에서 만들어서 브라우저에게 보내주고, 브라우저는 reconciliation을 수행한다.  
이 덕분에 RCC가 가지고 있던 모든 state와 event handler는 그대로 유지된다.

## 3. 새로운 wire foramt

Client에서 HTML을 직접 파싱해서 React element tree를 만드는 것보다 새로운 wire format을 토대로 React element tree를 만드는 게 tree 구성에 비용이 덜 든다고 한다.

```jsx
// Tweets.server.js
import { fetch } from 'react-fetch' // React's Suspense-aware fetch()
import Tweet from './Tweet.client'
export default function Tweets() {
  const tweets = fetch(`/tweets`).json()
  return (
    <ul>
      {tweets.slice(0, 2).map((tweet) => (
        <li>
          <Tweet tweet={tweet} />
        </li>
      ))}
    </ul>
  )
}

// Tweet.client.js
export default function Tweet({ tweet }) {
  return <div onClick={() => alert(`Written by ${tweet.username}`)}>{tweet.body}</div>
}

// OuterServerComponent.server.js
export default function OuterServerComponent() {
  return (
    <ClientComponent>
      <ServerComponent />
      <Suspense fallback={'Loading tweets...'}>
        <Tweets />
      </Suspense>
    </ClientComponent>
  )
}
```

위 컴포넌트는 아래와 같은 형태로 변경된다.

```json
{
  "M1": {
    "id": "./src/ClientComponent.client.js",
    "chunks": ["client1"],
    "name": ""
  },
  "S2": "react.suspense",
  "J0": [
    "$",
    "@1",
    null,
    {
      "children": [
        ["$", "span", null, { "children": "Hello from server land" }],
        ["$", "$2", null, { "fallback": "Loading tweets...", "children": "@3" }]
      ]
    }
  ],
  "M4": {
    "id": "./src/Tweet.client.js",
    "chunks": ["client8"],
    "name": ""
  },
  "J3": [
    "$",
    "ul",
    null,
    {
      "children": [
        ["$", "li", null, { "children": ["$", "@4", null, { "tweet": {} }] }],
        ["$", "li", null, { "children": ["$", "@4", null, { "tweet": {} }] }]
      ]
    }
  ]
}
```

위 구조를 전부 알 필요는 없이 간단히만 설명하자면 `M`으로 시작하는 것은 클라이언트 번들에서 컴포넌트 함수를 조회하는데 필요한 정보와 클라이언트 컴포넌트 module reference를 정의 한다.  
`J`는 실제 React element tree를 의미하고 `@1`을 통해 `M`에서 정의된 RCC를 의미한다.

## 4. RSC vs SSR(Server Side Rendering)

사실 둘을 비교한다는 것 자체가 말이 안 된다.  
code splitting이랑 tree shaking을 비교한다는 느낌이랄까...?

Dan Abramov의 RSC와 Next.js의 다른 점에 대한 좋은 설명을 요약하면 다음과 같다.

> RSC 코드는 절대 클라이언트에게 전달되지 않는다. 많은 React를 사용한 SSR의 구현은 JS 번들을 통해 클라이언트로 컴포넌트 코드가 보내지게 된다. 이로 인해 상호작용이 지연될 수 있다.
> RSC를 사용하면 트리의 어느 곳에서나 백엔드에 접근할 수 있다. Next.js를 사용한다면, 최상위 페이지에서만 가능한 `getServerProps()`를 통해 백엔드에 접근하는 것에 익숙할 것이다. 하지만, 임의 npm 컴포넌트는 이런 동작이 불가능하다.
> 트리 내부에서 클라이언트 측의 상태(state)를 유지하면서 서버 컴포넌트를 다시 가져올 수 있다. 이는 주요 전송 메커니즘이 HTML보다 훨씬 풍부하기 때문이다. 따라서, 내부 상태(e.g 검색 입력 텍스트, 포커스, 텍스트 선택)를 없애지 않고 서버에서 rendering 한 부분(e.g 검색 결과 목록)을 다시 가져올 수 있게 한다.

| X                        | RSC                                                                                                                                       | SSR                                                                                        |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| 사용 목적                | - 서버로부터 다운로드해야 하는 코드 양을 줄이고 싶을 때<br/>- data fetching의 속도를 높이고 싶을 때<br/>- 자동 code splitting이 필요할 때 | - JS가 실행되기(hydration) 전에 사용자에게 무언가를 보여주고 싶을 때<br/>- SEO가 필요할 때 |
| output                   | stream 가능한 형태의 JSON                                                                                                                 | HTML                                                                                       |
| output이 만들어지는 시점 | 클라이언트가 RSC와 관련된 요청을 보낼 때마다 매번                                                                                         | 클라이언트가 페이지를 요청하는 시점에 1번만                                                |
| 컴포넌트 rendering       | 서버, 클라이언트에서 나눠서                                                                                                               | 클라이언트에서만                                                                           |

SSR은 서버에서 JS를 이용해 HTML을 채워넣고 이를 클라이언트에 전달한다.  
하지만 SSR을 사용해도 사용자 인터렉션을 받기 위한 이벤트 핸들링을 추가하고 이에 대해 처리하는 것은 클라이언트에서 진행해야 되기 때문에(hydration 단계) 클라이언트는 **JS 코드를 서버로부터 다운로드**해야 한다.  
또한 일반적으로 SSR은 초기 페이지 rendering 속도 향상에 사용되므로 **hydration 이후에는 다시 사용할 수 없다**.

반면 RSC는 컴포넌트를 **다시 가져올 수 있다**.  
또한 새 데이터가 있을 때 re-rendering되는 컴포넌트가 서버에서 실행된 후에 전달되므로 **클라이언트가 다운받아야 하는 코드의 양을 줄일 수 있다**.

Server에서 동작한다는 점, 클라이언트에서의 성능이 개선된다는 점에서 둘이 같은 개념이라는 생각이 들 수 있다.  
하지만 명확히 둘은 구분되며, 서로 상반되는 개념도, 충분 조건의 개념도 아니다.  
상호 보완할 수 있는 개념이다.  
SSR과 RSC를 같이 사용한다면, SSR를 통해 초기 rendering 속도가 빨라지며 serialized stream 형태를 이용해 빠른 re-rendering도 가능하다.  
SSR이 다른 데이터를 받아오는(fetch) 메커니즘과 함께 사용되는 방식과 비슷하게, RSC가 만든 RCC를 SSR 한다.  
이때 역시 클라이언트가 다운로드해야 할 JS 코드 크기가 작아진다.
