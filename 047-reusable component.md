## 0. 공부하게 된 계기

동아리 면접 때 거짓말치다가 걸렸다...

현재 첫 회사에 입사해 본격적으로 일한지 2개월이 되지 않았다.  
지금까지 맡은 업무는 사장된 Angular.js를 React.js로 전환하는 일이 전부이다.  
그래서 설계 등에 대해 고민할 것이 적었다.  
공통적으로 사용되는 메서드는 HoC로 분리하든가 util 함수를 선언하든가, 하나의 메서드가 너무 크면 분리하든가, 변수명이 명확하지 않으면 변경하거나, 레거시 코드 제거하는 일 정도는 했다.  
하지만 좋은 컴포넌트는 무엇인가에 대한 고민은 많이 하지 않았다.  
(현재 테스트 코드가 없기 때문에...)컴포넌트를 분리하거나 구조를 변경하는 순간 영향도가 급격히 증가하기 때문에 랜딩 프로젝트로 진행하는 현재로서는 해당 부분을 생각하기까지에 부담스러운 부분이 있었다.

그런데 이 부분에 대해 고민했다고 말해서 이러쿵 저러쿵 하다가... "재사용 가능한 컴포넌트가 무엇일까"라는 말에 말문이 막혔다.  
FE 개발자로서... 이런 부분에 대해 대답을 못 한다면 공부해야겠지??

참고  
http://www.ktword.co.kr/test/view/view.php?m_temp1=2837  
http://wiki.hash.kr/index.php/%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8  
https://as-you-say.tistory.com/221  
https://jbee.io/web/components-should-be-flexible/  
https://velog.io/@jay/%EB%A6%AC%EC%97%91%ED%8A%B8-%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-%EC%84%A4%EA%B3%84#container-presentational-components  
https://velog.io/@dnr6054/%EC%9C%A0%EC%9A%A9%ED%95%9C-%EB%A6%AC%EC%95%A1%ED%8A%B8-%ED%8C%A8%ED%84%B4-5%EA%B0%80%EC%A7%80  
https://fe-developers.kakaoent.com/2022/220731-composition-component/  
https://javascript.plainenglish.io/5-react-design-patterns-you-should-know-629030e2e2c7  
https://tecoble.techcourse.co.kr/post/2021-04-26-presentational-and-container/  
https://www.turing.com/blog/custom-react-js-hooks-how-to-use/

## 1. 컴포넌트란

React의 장점 중 하나로 꼽히는 것이 컴포넌트 기반으로 개발한다는 것이다.  
여기서 말하는 *컴포넌트*의 정의부터 되짚어보자.  
[해시넷](http://wiki.hash.kr/index.php/%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8)에 따르면

> 컴포넌트(component)란 여러 개의 프로그램 함수들을 모아 하나의 특정한 기능을 수행할 수 있도록 구성한 작은 기능적 단위를 말한다. 컴포넌트를 이용하면 소프트웨어 개발을 마치 레고(Lego) 블록을 쌓듯이 조립식으로 쉽게 할 수 있다. 모듈(module)이라고도 한다. 컴포넌트는 프로그램의 한 부분을 의미하며 재사용이 가능한 최소 단위를 말한다.

여기서 집중해야 할 것은 **재사용 가능한 최소 단위**라는 것이다.  
이 정의에 입각해 규칙을 세우고 개발을 진행해야 CBD(Component Based Development, 재사용 가능한 컴포넌트의 개발 또는 상용 컴포넌트를 조합하여 하나의 새로운 응용 프로그램을 만드는 소프트웨어 개발 방법론)에 맞는 개발을 진행할 수 있다.

## 2. 좋은 컴포넌트란

좋은 컴포넌트, 즉 잘 짠 컴포넌트는 **재사용 가능한** 컴포넌트라는 결론을 얻었다(사실 정의에 입각했을 뿐이다).  
재사용 가능한 컴포넌트는 **어떤 맥락(사용하고 있는 페이지나 컴포넌트)에서 사용되든 동일한 역할을 하고 일관된 UI를 렌더링한다.**  
재사용 가능한 컴포넌트는 외부와 인터페이스(React에서는 `props`로 볼 수 있다)를 통해 소통한다.  
이 인터페이스는 해당 컴포넌트를 사용하는 쪽을 위한 것이며, 컴포넌트를 사용하는 쪽에서는 인터페이스를 보고 어떻게 동작할지 예상하다.  
어떠한 곳에서 사용되든, 동일한 형태의 버튼을 렌더링 한다든가(버튼 margin, 위치 등은 외부에서 결정한다), 목록을 렌더링 한다든가(목록에 넣어줄 데이터는 외부에서 주입한다)... 토글 / API 요청 등등 본인이 해야 하는 역할을 컴포넌트의 인터페이스 변경 없이 사용 가능하다.

여러 글들을 읽어본 결과, 재사용성이 높은 컴포넌트는 "추상화"를 통해 역할과 책임에 따라 분리된다.  
역할과 책임에 따른 컴포넌트 분리는 두 가지 추가적인 이점을 가져다준다.
첫 번째는 컴포넌트를 테스트하기 쉽게 만든다.  
(TODO: 책임이 적어지기 때문에. 테스트할 범위가 좁아지고, 테스트 코드가 통과하기 위한 억지 모킹이 줄어들 수 있을 것이라 기대한다. 이 내용은 사이드 프로젝트를 끝내고 다시 후기를 작성하러 와보자)
두 번째는 컴포넌트를 변경에 유연하도록 만든다.  
어느 곳에서든 쓸 수 있도록 비즈니스 로직이 분리되고, 일반적인 인터페이스로 디자인되기 때문이다.

### 여기서 한 가지 짚고 넘어가자면

프로젝트에서 무조건 재사용성 높은 컴포넌트 선언이 1순위가 아니다.  
얻는 게 있으면 잃는 게 있다는 것 반드시 기억하자.
'인터페이스 변경 없이 사용 가능하게 하기' 또는 '추가적인 컴포넌트 선언을 없애기'를 1순위로 하면 컴포넌트 그리고 컴포넌트의 인터페이스가 너무 비대해질 수 있다.

다음과 같이 부트스트랩 모양의 버튼, 닫기 모양의 버튼 이렇게 두 가지 종류의 버튼이 있다고 하자.

![bootstrap buttons](https://user-images.githubusercontent.com/63287638/234282991-3a48bc2b-8620-4f2d-aa6d-05a45d4d2e8c.png)

![X button](https://user-images.githubusercontent.com/63287638/234282726-725c426e-9d55-4ad2-b8b6-fddea3b1abb1.png)

'컴포넌트를 2개로 따로 선언하는 것이 재사용성을 해치니까 나는 1개로 선언해야지! 이 프로젝트에 버튼 컴포넌트는 무조건 하나야'라는 생각으로 다음과 같이 선언할 수 있다.

```ts
function Button({ onClick, className = '', children, isClose }: Props) {
  return isClose ? (
    <button
      onClick={onClick}
      className={`button-base ${className}`}
      style={{ margin: 3 }}
    >
      X
    </button>
  ) : (
    <button
      onClick={onClick}
      className={`button-base ${className}`}
      style={{ margin: 4 }}
    >
      {children}
    </button>
  );
}
```

취향 차이일 수 있다.  
나는! 개인적으로! 역할, UI가 명확히 구분되고 다르다면 아래와 같이 두 가지 컴포넌트로 분리하는 것이 낫다고 생각한다...  
`isClose`나 인터페이스나 삼항 연산자 문법이 필요 없기 때문이다.

```tsx
function BootstrapButton({ onClick, className = '', children }: Props) {
  return (
    <button
      onClick={onClick}
      className={`button-base ${className}`}
      style={{ margin: 4 }}
    >
      {children}
    </button>
  );
}
```

```tsx
function CloseButton({ onClick, className = '' }: Props) {
  return (
    <button
      onClick={onClick}
      className={`button-base ${className}`}
      style={{ margin: 3 }}
    >
      X
    </button>
  );
}
```

## 3. 재사용 가능한 컴포넌트 만드는 방법

그렇다면 재사용 가능한 컴포넌트는 어떻게 만들까?

우선 컴포넌트를 역할과 책임에 따라 분리한다(다만 분리만 한다고 재사용 가능한 것은 아니다. 분리는 밑작업일 뿐이다).  
컴포넌트 하나가 비대해지면 재사용성이 떨어지며 가독성도 해친다.  
일반적으로 역할과 책임에 따라 분리하면 컴포넌트는 **비즈니스 로직이 없거나 상태값이 없도록 분리된다.**  
하나의 페이지를 만든다면 페이지 자체는 결과적으로 하나의 거대한 컴포넌트가 되겠지만 그 구성 요소는 작은 컴포넌트들이다.

컴포넌트는 보통 세 가지 역할과 책임 중에 하나를 맡는다.

1. 외부(API, Storage, 부모 컴포넌트)로부터 주입된 데이터를 관리한다.
2. 데이터를 UI로 표현한다.
3. 사용자로부터 인터랙션을 받는다.

<br />

예시로 거대한 게시판 목록 페이지를 하나를 만들고 역할과 책임에 따라 분리해보겠다.  
해당 어플리케이션의 table header는 text만 들어오고 `font-weight: 600`으로 고정되어 있다.  
또한 table body는 text도 `React.ReactNode`도 들어올 수 있고, `font-weight: 300`으로 고정되어 있다.  
간단히 만들기 위해서 페이지네이션은 없다고 가정하겠다.

우선 아래 파일들은 컴포넌트 재사용을 위한 내용과 상관 없이 공통적으로 사용될 파일들이다.  
스타일 파일은 ~귀찮아서~ 여러 파일로 분리 안 하고 적당히 모양만 내도록 짰으니까 대충 넘어가자.

```json
// public/result.json

{
  "posts": [
    {
      "id": 1,
      "title": "제목입니다1",
      "author": "user1"
    },
    {
      "id": 2,
      "title": "제목입니다2",
      "author": "user2"
    },
    {
      "id": 3,
      "title": "제목입니다3",
      "author": "user3"
    },
    {
      "id": 4,
      "title": "제목입니다4",
      "author": "user4"
    },
    {
      "id": 5,
      "title": "제목입니다5",
      "author": "user5"
    },
    {
      "id": 6,
      "title": "제목입니다6",
      "author": "user6"
    }
  ]
}
```

```ts
// src/api/fetch.ts

const request = async (url: string) => {
  try {
    const data = await fetch(url);

    if (!data.ok) {
      throw new Error('데이터 fetch 에러 발생');
    }

    return await data.json();
  } catch (error) {
    // 무시
  }
};

export default request;
```

```ts
// src/store/storage.ts

export default class Storage {
  static #instance: Storage;

  constructor() {
    if (Storage.#instance) {
      return Storage.#instance;
    }

    Storage.#instance = this;
  }

  get(key: string) {
    return localStorage.getItem(key) || '저장된 데이터가 없습니다';
  }

  set(key: string, value: string) {
    localStorage.setItem(key, value);
  }
}
```

```ts
// src/types.d.ts

declare module 'types' {
  interface Post {
    id: number;
    title: string;
    author: string;
  }
}
```

```css
/* src/style.css */

button {
  background-color: #0a0a23;
  color: #fff;
  border: none;
  border-radius: 10px;
  padding: 4px 8px;
}

button:hover {
  outline-color: transparent;
  outline-style: solid;
  box-shadow: 0 0 0 4px #5a01a7;
  transition: 0.7s;
}

thead {
  font-weight: 600;
}

tbody {
  font-weight: 300;
}

.warning {
  color: red;
}

.post-list .row {
  display: flex;
}

.post-list .cell {
  flex: 1 0 250px;
}
```

아무런 분리를 시도하지 않은 `App.tsx`는 다음과 같이 생겼다.

```tsx
// src/App.tsx

import { useState, useEffect } from 'react';
import Storage from './store/storage';
import request from './api/fetch';
import { Post } from 'types';

function App() {
  const HEADERS = ['NO', '제목', '작성자', '삭제'];
  const [posts, setPosts] = useState<Post[]>([]);

  useEffect(() => {
    new Storage().set('USER_ID', 'user1');
  }, []);

  useEffect(() => {
    async function getPosts() {
      const { posts }: { posts: Post[] } = await request('result.json');

      if (posts) {
        setPosts(posts);
      }
    }

    getPosts();
  }, []);

  function onClickCreationButton() {
    alert('게시글 등록 페이지로 이동합니다.');
  }

  function onClickDeleteButton(id: number) {
    alert('게시글을 삭제합니다.'); // API 요청 가정

    setPosts((posts) => posts.filter((post) => post.id !== id));
  }

  function getUserId() {
    return new Storage().get('USER_ID');
  }

  return (
    <div className="App">
      <div className="post-list">
        <button onClick={onClickCreationButton}>게시글 추가</button>
        <table>
          <thead>
            <tr className="row">
              {HEADERS.map((header) => (
                <td key={header} className="cell">
                  {header}
                </td>
              ))}
            </tr>
          </thead>
          <tbody>
            {posts.map(({ id, title, author }) => (
              <tr key={id} className="row">
                <td className="cell">{id}</td>
                <td className="cell">{title}</td>
                <td className="cell">{author}</td>
                <td className="cell">
                  {getUserId() === author ? (
                    <button onClick={() => onClickDeleteButton(id)}>
                      삭제
                    </button>
                  ) : (
                    '삭제 불가능한 게시글입니다.'
                  )}
                </td>
              </tr>
            ))}
          </tbody>
          {posts.length === 0 && (
            <h1 className="warning">게시글이 없습니다.</h1>
          )}
        </table>
      </div>
    </div>
  );
}

export default App;
```

혼자 API 요청 없이 렌더링할 수 있는 부분 렌더링, API 요청, API 요청 후에 렌더링할 수 있는 부분 렌더링, 클릭 인터랙션을 받은 후 삭제 API 요청 등 많은 부분을 담당한다.  
애플리케이션이 작다면 이러한 페이지는 크게 문제가 되지 않을 수 있다.  
하지만 표를 그리는 페이지가 많다면 반복되는 코드가 많이 생길 것이다.  
또한 "삭제" 버튼 클릭 시 `tbody` 내부만 re-rendering되면 되지만, 불필요하게 `thead`도 re-rendering된다.
테스트 코드를 짤 때도 헤더만 제대로 렌더링되는지를 확인하고 싶은데 불필요한 API 모킹이 필요하게 된다.

이제 위 컴포넌트를 비즈니스 로직이 없거나 상태값이 없도록 분리해보자.

1. `App`: 애플리케이션 최상단에서 라우팅 처리
2. `PostList`: 게시글 목록 최상단 컴포넌트로 '게시글 추가' 버튼, `PostTableHeader`와 `PostTableBody` 렌더링
3. `PostTableHeader`: 게시글 목록 헤더를 나타내는 컴포넌트로 `headers`(text 배열), `style`(thead의 스타일)을 `props`로 받음
4. `PostTableBody`: 일종의 container 컴포넌트. API를 요청해 posts 데이터를 받아오고 `PostListRow`를 렌더링
5. `PostListRow`: 일종의 presentational 컴포넌트. `PostTableBody`에게 받은 데이터를 렌더링

위와 같이 총 5가지 컴포넌트로 분리할 수 있겠다.  
현재 `Button` 컴포넌트는 디자인이 하나밖에 없어서 분리하지 않았다.  
그리고 props가 없다면 `memo`로 감싸면 좋겠지만 현재는 성능 관련된 내용을 이야기하는 것이 아니니 pass.

```tsx
// src/App.tsx

import { useEffect } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import PostList from './pages/PostList';
import Storage from './store/storage';

function App() {
  useEffect(() => {
    new Storage().set('USER_ID', 'user1');
  }, []);

  return (
    <div className="App">
      <BrowserRouter>
        <Routes>
          <Route path="/" element={<PostList />} />
        </Routes>
      </BrowserRouter>
    </div>
  );
}

export default App;
```

```tsx
// src/pages/PostList.tsx

import PostListHeader from '../components/PostListHeader';
import PostListBody from '../components/PostListBody';

function PostList() {
  const HEADERS = ['NO', '제목', '작성자', '삭제'];

  function onClickCreationButton() {
    alert('게시글 등록 페이지로 이동합니다.');
  }

  return (
    <div className="post-list">
      <button onClick={onClickCreationButton}>게시글 추가</button>
      <table>
        <PostListHeader headers={HEADERS} />
        <PostListBody />
      </table>
    </div>
  );
}

export default PostList;
```

```tsx
// src/components/PostListHeader.tsx

interface Props {
  style?: React.CSSProperties;
  headers: string[];
}

function PostListHeader({ headers, style }: Props) {
  return (
    <thead style={{ ...style }}>
      <tr className="row">
        {headers.map((header) => (
          <td key={header} className="cell">
            {header}
          </td>
        ))}
      </tr>
    </thead>
  );
}

export default PostListHeader;
```

```tsx
// src/components/PostListBody.tsx

import { useState, useEffect } from 'react';
import PostListRow from '../components/PostListRow';
import request from '../api/fetch';
import Storage from '../store/storage';
import { Post } from 'types';

function PostListBody() {
  const [posts, setPosts] = useState<Post[]>([]);

  useEffect(() => {
    async function getPosts() {
      const { posts }: { posts: Post[] } = await request('result.json');

      if (posts) {
        setPosts(posts);
      }
    }

    getPosts();
  }, []);

  function onClickDeleteButton(id: number) {
    alert('게시글을 삭제합니다.');

    setPosts((posts) => posts.filter((post) => post.id !== id));
  }

  function getUserId() {
    return new Storage().get('USER_ID');
  }

  return (
    <>
      <tbody>
        {posts.map(({ id, title, author }) => (
          <PostListRow
            key={id}
            cells={[
              id,
              title,
              author,
              getUserId() === author ? (
                <button onClick={() => onClickDeleteButton(id)}>삭제</button>
              ) : (
                '삭제 불가능한 게시글입니다.'
              ),
            ]}
          />
        ))}
      </tbody>
      {posts.length === 0 && <h1 className="warning">게시글이 없습니다.</h1>}
    </>
  );
}

export default PostListBody;
```

```tsx
// src/components/PostListRow.tsx

interface Props {
  style?: React.CSSProperties;
  cells: React.ReactNode[];
}

function PostListRow({ cells, style }: Props) {
  return (
    <tr className="row" style={{ ...style }}>
      {cells.map((cell) => (
        <td className="cell">{cell}</td>
      ))}
    </tr>
  );
}

export default PostListRow;
```

### 1) 일반적인 인터페이스로 디자인한다.

위 예시에서 분리한 컴포넌트들은 쉽게 재사용될 수 _없다_.  
특정 도메인(여기서는 게시글)에 얽혀있기 때문이다.  
페이지 자체인 `PostList.tsx`나 API를 요청해야 하는(props로 API URL을 넘겨준다면 가능하겠지만 내 생각에는 굳이...? 재사용할 수 있을까? 싶긴 하다) `PostListBody.tsx`는 도메인에서 분리하기 힘들 것 같다.  
하지만 `PostListHeader.tsx`나 `PostListRow.tsx`는 넘겨받은 데이터를 뿌려주기만 하는 presentational 컴포넌트이므로 충분히 도메인을 제거할 수 있겠다.

```tsx
interface Props {
  style?: React.CSSProperties;
  headers: string[];
}

function TableHeader({ headers, style }: Props) {
  return (
    <thead style={{ ...style }}>
      <tr className="row">
        {headers.map((header) => (
          <td key={header} className="cell">
            {header}
          </td>
        ))}
      </tr>
    </thead>
  );
}

export default TableHeader;
```

```tsx
interface Props {
  style?: React.CSSProperties;
  cells: React.ReactNode[];
}

function TableRow({ cells, style }: Props) {
  return (
    <tr className="row" style={{ ...style }}>
      {cells.map((cell) => (
        <td className="cell">{cell}</td>
      ))}
    </tr>
  );
}

export default TableRow;
```

위와 같이 말이다.  
이제 `TableHeader`, `TableRow` 두 컴포넌트는 테이블을 만드는 여러 페이지에서 재사용할 수 있다.

하지만 (애플리케이션마다 다르겠지만) 위와 같이 변경한 것이 끝이 아닐 수 있다.  
`TableHeader`나 `TableRow`는 `style`만이 아니라 각각 `thead`, `tr` 태그가 받을 수 있는 모든 `props`를 받고 싶게 변경할 수도 있다.  
그러면 `TableHeader`는 아래와 같이 변경할 수도 있다(`TableRow`는 비슷하므로 생략).

```tsx
import { HTMLAttributes } from 'react';

interface Props extends HTMLAttributes<HTMLTableCellElement> {
  style?: React.CSSProperties;
  headers: string[];
}

function TableHeader({ headers, style, ...props }: Props) {
  return (
    <thead {...props}>
      <tr className="row">
        {headers.map((header) => (
          <td key={header} className="cell">
            {header}
          </td>
        ))}
      </tr>
    </thead>
  );
}
```

### 2) 디자인 시스템을 이용한다.

### 3) 디자인 패턴을 사용한다.

#### Presentational - Container

처음 이 패턴을 소개한 Dan Abramov는 2019년 기준으로 현재는 이 패턴을 사용하지 말라고 [언급](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)했다.

> Update from 2019: I wrote this article a long time ago and my views have since evolved. In particular, I don’t suggest splitting your components like this anymore. If you find it natural in your codebase, this pattern can be handy. But I’ve seen it enforced without any necessity and with almost dogmatic fervor far too many times. The main reason I found it useful was because it let me separate complex stateful logic from other aspects of the component. Hooks let me do the same thing without an arbitrary division. This text is left intact for historical reasons but don’t take it too seriously.

hook 개념이 나오고 이 패턴은 deprecated 된 느낌이다.  
그래도 가장 유명하고 오래된 패턴이니까 간단히만 짚고 넘어가겠다.

React 컴포넌트는 상태, DOM, 이벤트 등을 모두 관리할 수 있지만 코드 재사용하는 방법이 없었다.  
그래서 컴포넌트 내에서도 로직과 view를 분리하기 위함으로써 의존도를 낮추는 방법으로 등장한 것이 Presentational - Container 패턴이다.

대체적으로 presentational 컴포넌트는 stateless하고 container 컴포넌트는 stateful하다.

이 패턴은 다음과 같은 장점이 있다.

1. 재사용성을 높일 수 있다. 로직이 포함되어 있지 않은 presentational 컴포넌트는 그저 받아온 정보를 화면에 표현할 뿐이므로 다양한 container 컴포넌트와 조합하여 재사용할 수 있다.
2. 구조에 대한 이해가 쉬워진다. 기능과 UI가 명확히 분리되므로

##### 코드 예시

엄청 간단한 예시이다.  
아래와 같이 게시글 목록을 가져와서 보여주는 컴포넌트가 있다고 하자.

```tsx
function PostList() {
  const [posts, setPosts] = useState<Post[]>([]);

  useEffect(() => {
    async function getPosts() {
      const { data } = await request(URL);

      setPosts(data);
    }

    getPosts();
  }, []);

  return (
    <ul>
      {posts.map(post) => (
        <li key={post.id}>{post.text}</li>
      )}
    </ul>
  );
}
```

위 컴포넌트는 복잡한 구조가 없어서 큰 문제가 없지만 삭제 버튼, 조건부 렌더링 등이 추가되면 하나의 컴포넌트가 담당하는 기능이 점점 커지게 된다.  
데이터 목록을 받아 뿌려주는 부분이 많다면 위 `PostList`를 아래와 같이 `PostConatiner`, `ListPresentational`로 분리할 수 있을 것이다.

```tsx
function PostContainer() {
  const [posts, setPosts] = useState<Post[]>([]);

  useEffect(() => {
    async function getPosts() {
      const { data } = await request(URL);

      setPosts(data);
    }

    getPosts();
  }, []);

  return <ListPresentational data={posts} />;
}
```

```tsx
function ListPresentational({ data, ...props }: Props) {
  return (
    <ul>
      {data.map(datum) => (
        <li key={datum.id}>{datum.text}</li>
      )}
    </ul>
  )
}
```

#### Custom Hook

#### Compound

#### Render Props

#### Control Props

#### Props Getters

#### State Reducer

---

(FE conf) 컴포넌트 다시 생각하기 요약  
컴포넌트: props + hooks에게 의존성을 주입 받음.  
그런한 것 중.

1. 스타일(코드 안)
2. UI 조작에 필요한 커스텀 로직 or 사이드 이펙트(코드 안)
3. store / theme 등 전역 상태(코드 안)
4. remote data schema(API 서버에서 내려주는 데이터의 모양)(코드 밖)

만약 Article 컴포넌트에 새로운 정보가 추가된다면 => remote data에 의해 컴포넌트의 props를 수정 => 렌더링 부분 수정 => ArticleList의 props 수정 => PageArticleList의 useEffect 훅 수정(props drilling을 사용하지 않더라도 페이지 기반 라우팅을 한다면 결국 root component에 의존함)

의존성 정리 방법

1. 비슷한 관심사라면 가까운 곳에(co-locate). (CSS in JS / custom hook / store에서 식별자 값을 통해 해당 데이터만 custom hook <- 2번 내용)
2. 데이터를 ID 기반으로 정리(데이터 정규화)
3. 의존한다면 그대로 드러내기(props 이름을 일반화시키는 것과 반대)
4. UI가 비슷한지에 상관 없이 모델 기준으로 컴포넌트 분리하기
