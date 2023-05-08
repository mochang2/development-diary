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

참고 및 출처 
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
https://ko.legacy.reactjs.org/docs/render-props.html  
https://flowergeoji.me/react/react-pattern-control-props/

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
function Button({ onClick, className = '', children, isClose }: TProps) {
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
function BootstrapButton({ onClick, className = '', children }: TProps) {
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
function CloseButton({ onClick, className = '' }: TProps) {
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

예시로 거대한 게시판 목록 페이지 하나를 만들고 역할과 책임에 따라 분리해보겠다.  
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

class Storage {
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

export default Storage;
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
// src/routes/App.tsx

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
// src/components/post-list/PostListHeader.tsx

type TProps = {
  style?: React.CSSProperties;
  headers: string[];
}

function PostListHeader({ headers, style }: TProps) {
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
// src/components/post-list/PostListBody.tsx

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
// src/components/post-list/PostListRow.tsx

type TProps = {
  style?: React.CSSProperties;
  cells: React.ReactNode[];
}

function PostListRow({ cells, style }: TProps) {
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
// src/components/TableHeader.tsx

type TProps = {
  style?: React.CSSProperties;
  headers: string[];
}

function TableHeader({ headers, style }: TProps) {
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
// src/components/TableRow.tsx

type TProps = {
  style?: React.CSSProperties;
  cells: React.ReactNode[];
}

function TableRow({ cells, style }: TProps) {
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
// src/components/TableRow.tsx

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

export default TableHeader;
```

### 2) 디자인 시스템을 이용한다.

_사진 및 디자인 출처_  
[리메인](https://www.remain.co.kr)

디자인 시스템은 디자인 원칙과 규격, 재사용할 수 있는 UI 패턴과 컴포넌트, 코드를 포괄하는 시스템을 말한다.  
컴포넌트가 잘 정의되어 있지 않으면 재사용이 불가능한 것처럼, 디자이너들도 매번 버튼을 새로 만들거나, 일관되지 않은 디자인을 만들거나, 유지 보수에 불편함을 겪는다고 한다.  

예를 들어 통일성 없이 만들어진 input과 button이 있다.  
처음 선언했던 곳에서는 각자의 역할을 잘 했고, 검색 컴포넌트를 만드는 과정에서 두 컴포넌트를 합치고 싶다고 하자.  
하지만 `font-size`나 `padding`, `height`(아래 사진에는 안 나와있지만 `box-sizing`)의 차이 때문에 새로운 컴포넌트를 또 만들어야 한다.

![reason to need design system](https://user-images.githubusercontent.com/63287638/236710883-2720290e-2dd1-4159-a24b-f03dacaf10bb.png)

이러한 문제에 대해 디자인 시스템을 해결책을 제시한다.  
잘 정의된 디자인 시스템은 UI/UX 가이드라인을 제공하고 재사용 가능한 디자인을 만든다.  
자세한 내용은 [토스 디자인 시스템](https://www.youtube.com/watch?v=LmLchZ4tCXc)을 참고한다.

#### 코드 예시(SCSS)

디자인 쪽은 문외한이라서 리메인에서 제공하는 [컴포넌트 코어 시스템](https://www.remain.co.kr/page/designsystem/component-core.php)을 참고했다.  
(브랜드, 시스템 색상 시스템도 예시로 제공할 수 있겠지만) 위 사진에서 제기한 height 문제를 해결해보고자 한다.

디자인 시스템이 없는 코드를 먼저 보자.  

```tsx
// src/components/home/Button.tsx

export default function Button({onClick, text}: TProps) {
  return (
    <button onClick={onClick} className="search-from__login-button">
      {text}
    </button>
  )
}
```

```tsx
// src/components/post/Button.tsx

export default function Button({onClick, children}: TProps) {
  return (
    <button onClick={onClick} className="search-from__submit-button">
      {children}
    </button>
  )
}
```

사실 위 두 가지 컴포넌트에서 다른 것은 `className`밖에 없다. `text` prop은 `string`일 것이고 이는 `chilren` 타입에 포함된다.  
UI와 관련된 부분이 통일되지 않아서, 컴포넌트를 분리해도 특정 도메인과 엮여있어서 재사용하지 못할 가능성이 크다.  
또한 위 사진처럼 `height`가 달라 `Button` 컴포넌트를 다른 `Input` 컴포넌트와 합쳐서 `Search`와 같은 조금 더 큰 컴포넌트를 만들지 못할 수도 있다.

하지만 디자인 시스템이 갖춰져 있다면 초기 설계 비용이 크지만 아래와 같이 재사용 가능한 컴포넌트를 만들 수 있다.  
참고로 JS in CSS가 아니라서 utility-first css 방법론을 이용해서 컴포넌트 이름 자체로 역할을 명시(`Button` -> `WarningButton`)하지 않고 `className`으로 버튼의 역할을 암시한다.  
(JSS in CSS를 사용하면 className을 직접 지정하지 않을 수 있다)

```scss
// src/assets/styles/presets.scss

* {
  margin: 0;
  padding: 0;

  box-sizing: border-box;
}

```

```scss
// src/assets/styles/common/boxes.scss

$PAD: 4;
$HEIGHT: 4;

@mixin box-xxxs {
  height: #{$HEIGHT * 6}px;
}

@mixin box-xxs {
  padding: 0 #{$PAD * 1}px;

  height: #{$HEIGHT * 7}px;
}

@mixin box-xs {
  padding: 0 #{$PAD * 2}px;

  height: #{$HEIGHT * 8}px;
}

// ...
```

```scss
// src/assets/styles/utility-first.scss

@import "./common/boxes";

// button
button {
  &.xxxs {
    @include box-xxxs;

    font-size: 10px;
  }

  &.xxs {
    @include box-xxs;

    font-size: 12px;
  }

  &.xs {
    @include box-xs;

    font-size: 12px;
  }
}
```

```tsx
// src/components/Button.tsx

export default function Button({onClick, children, ...props}: TProps) {
  return (
    <button onClick={onClick} {...props}>
      {children}
    </button>
  )
}
```

~그냥 `button` 태그 쓰는 거랑 큰 차이가 없지만 예시니까 그러려니 하자~  
이제 `Button` 컴포넌트는 도메인과 분리돼서 사용할 수 있다.  
`props`에 `className`을 전달함으로써 `height`를 조정할 수 있고, 여기저기서 사용 가능한 `Input` 컴포넌트와 합쳐 새로운 `Search` 컴포넌트를 만들 수 있다.  
새로운 컴포넌트나 스타일을 선언하지 않고도 말이다.  
만약 특정 도메인에서 `font-size`만 달라져야 한다고 해도 `style` props를 전달해줘서 inline으로 이를 설정할 수도 있다.

#### 코드 예시(Storybook)

Storybook은 개발자, 디자이너 및 프로젝트 관리자에게 UI 개발과 관리를 보다 효율적으로 처리할 수 있는 도구이다.  
Storybook을 사용하면 다음과 같은 장점이 있다.

1. 디자이너와 협업하기 용이하다. Storybook은 UI 컴포넌트의 디자인과 동작을 독립적으로 테스트하고 문서화할 수 있으며, 디자이너와 개발자 간에 UI의 외관과 기능을 공유하기 쉽다.
2. 재사용 가능한 컴포넌트를 만들 수 있다. Storybook을 사용하여 UI 컴포넌트를 작성하고 문서화하면, 재사용 가능한 UI 컴포넌트 라이브러리를 만들 수 있다. 이는 프로젝트에서 일관성 있는 UI를 제공하고 코드의 재사용성을 높이는 데 도움이 된다.
3. UI 컴포넌트를 테스트하고 디버그하기 쉽다. Storybook은 UI 컴포넌트를 독립적으로 테스트하고, 개발자가 UI 컴포넌트를 디버그하고 수정할 수 있도록 도와준다.
4. 개발 속도를 높일 수 있다(다만 모든 구조가 그렇듯이 프로젝트에 자리 잡기 전에는 오히려 속도가 느리다). Storybook은 UI 컴포넌트를 독립적으로 개발하고 테스트할 수 있으므로, 개발자는 더 빠르게 개발을 진행할 수 있다.

```js
// src/components/Task.stories.js
// 단순 홈페이지 예시

import Task from './Task';

export default {
  component: Task,
  title: 'Task',
};

const Template = (args) => <Task {...args} />;

export const Default = Template.bind({});
Default.args = {
  task: {
    id: '1',
    title: 'Test Task',
    state: 'TASK_INBOX',
    updatedAt: new Date(2021, 0, 1, 9, 0),
  },
};

export const Pinned = Template.bind({});
Pinned.args = {
  task: {
    ...Default.args.task,
    state: 'TASK_PINNED',
  },
};

export const Archived = Template.bind({});
Archived.args = {
  task: {
    ...Default.args.task,
    state: 'TASK_ARCHIVED',
  },
};
```

TODO: 당장에 디자이너와 일할 일이 없어서... 내가 직접 사용할 일이 생기면 불편했던 점, 개선됐으면 하는 점, 잘 사용하는 방법을 추가할 예정이다. 

### 3) 패턴을 사용한다.

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
2. 구조에 대한 이해가 쉬워진다. 기능과 UI가 명확히 분리되기 때문이다.
3. 테스트하기 쉽다.

##### 코드 예시

엄청 간단한 예시이다.  
아래와 같이 게시글 목록을 가져와서 보여주는 컴포넌트가 있다고 하자.

```tsx
// src/pages/PostList.tsx

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

export default PostList
```

위 컴포넌트는 복잡한 구조가 없어서 큰 문제가 없지만 삭제 버튼, 조건부 렌더링 등이 추가되면 하나의 컴포넌트가 담당하는 기능이 점점 커지게 된다.  
데이터 목록을 받아 뿌려주는 부분이 많다면 위 `PostList`를 아래와 같이 `PostConatiner`, `ListPresentational`로 분리할 수 있을 것이다.

```tsx
// src/components/post-list/PostContainer.tsx

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

export default PostContainer;
```

```tsx
// src/components/ListPresentational.tsx

function ListPresentational({ data, ...props }: TProps) {
  return (
    <ul>
      {data.map(datum) => (
        <li key={datum.id}>{datum.text}</li>
      )}
    </ul>
  )
}

export default ListPresentational
```

#### Custom Hook

hook은 React 16.8부터 도입된 개념이다.  
`useState`, `useEffect`가 대표적인 예시이다.  
이 내용은 [여기](https://github.com/mochang2/development-diary/blob/main/030-react.md)에 정리했었으니까 자세한 내용을 알고 싶다면 참고하자.

React에서 기본적으로 제공하는 hook 이외에도 사용자가 공통된 코드를 추상화할 수 있는 방법이 custom hook을 이용하는 것이다.  
(Vue의 composition이나 class 컴포넌트의 HoC하고 비슷하다)

Presentational - Container 패턴하고 비슷한 장점을 가진다.

##### 코드 예시

```tsx
// src/hooks/post-list/use-post.ts

function usePost() {
  const [posts, setPosts] = useState<Post[]>([]);

  useEffect(() => {
    async function getPosts() {
      const { data } = await request(URL);

      setPosts(data);
    }

    getPosts();
  }, []);

  return {
    posts,
  };
}

export default usePost;
```

```tsx
// src/pages/PostList.tsx

function PostList() {
  const {posts} = usePost();

  return (
    <ul>
      {posts.map(post) => (
        <li key={post.id}>{post.text}</li>
      )}
    </ul>
  );
}

export default PostList
```

#### Compound

도입 이유(비슷한 UI를 해결하기 위해서 조건부 렌더링을 이용해 하나의 컴포넌트에서 처리하다가 결국 괴물 컴포넌트가 됨)부터 결과까지 잘 말해준 [카카오 엔터테인먼트 기술 블로그](https://fe-developers.kakaoent.com/2022/220731-composition-component/)를 참고했다.

다음과 같은 특징이 있다.

- 장점
  - API 복잡도가 낮다. 비슷한 UI / 기능을 가졌을 때 손쉽게 재사용 가능하다.
  - 사용자가 Compound 패턴으로 선언된 컴포넌트를 자세히 몰라도 사용하기 쉽다.
  - 불필요한 props drilling을 없앤다. 보통 내부적으로 Context API를 사용하기 때문이다.
- 단점
  - 자유도가 커서 처음 컴포넌트를 설계했을 때와 다르게 사용할 수 있다.

API 복잡도가 낮다는 것은 하나의 거대한 부모 컴포넌트에 모든 `props`을 때려넣고 자식 컴포넌트에 꽂아주는 방법보다 각각의 `props`이 각각의 서브컴포넌트에 붙어있다는 뜻이다.

```tsx
function Component() {
  // ...

  return (
    <Counter
      label="label"
      max={10}
      iconDecrement="minus"
      iconIncrement="plus"
      onChange={handleChangeCounter}
    />
  );
}
```

위 방법은 API 복잡도가 높다.  
Compound 패턴을 적용하면 아래와 같이 사용이 가능하다.

```tsx
function Component() {
  // ...

  return (
    <Counter onChange={handleChangeCounter}>
      <Counter.Decrement icon={'minus'} />
      <Counter.Label>Counter</Counter.Label>
      <Counter.Count max={10} />
      <Counter.Increment icon={'plus'} />
    </Counter>
  );
}
```

##### 코드 예시

dimmed 처리 여부, arrow가 있는 버튼 존재 여부 등에 따라 동작이나 UI가 달라지는 bottom sheet가 있다고 하자.  
`BottomSheet1`은 dimmed 有, arrow 버튼 有  
`BottomSheet2`은 dimmed 無, arrow 버튼 有  
`BottomSheet3`은 dimmed 無, arrow 버튼 無 라고 하겠다.

```tsx
// src/components/__compounds__/BottomSheet.tsx

const Store = createContext<boolean | null>(null);

function BottomSheet({ children }: BottomSheetProps) {
  const [dimmed, setDimmed] = useState<boolean>(true);

  const handleDimmed = () => {
    setDimmed((dimmed) => !dimmed);
  };

  return (
    <Store.Provider value={{ dimmed, handleDimmed }}>{children}</Store.Provider>
  );
}

function Dimmed() {
  const { dimmed, handleDimmed } = useContext(Store);

  if (!dimmed) {
    return <></>;
  }

  return (
    <div>
      dimmed 됐다고 가정 <button onClick={handleDimmed}>버튼</button>
    </div>
  );
}

function Title({ children }: TitleProps) {
  return <h1>{children}</h1>;
}

function Description({ children }: DescriptionProps) {
  return <div>{children}</div>;
}

function Button({ children, hasArrow }: ButtonProps) {
  return (
    <>
      {hasArrow ? '->' : ''}
      <button>{children}</button>
    </>
  );
}

BottomSheet.Dimmed = Dimmed;
BottomSheet.Title = Title;
BottomSheet.Description = Description;
BottomSheet.Button = Button;

export default BottomSheet;
```

```tsx
// src/components/BottomSheet1.tsx

function BottomSheet1({ title, description, buttonTexts }: TProps) {
  // dimmed, arrow 버튼 모두 존재
  return (
    <BottomSheet>
      <BottomSheet.Dimmed />
      <BottomSheet.Title>{title}</BottomSheet.Title>
      <BottomSheet.Description>{description}</BottomSheet.Description>
      {buttonTexts.map((text) => (
        <BottomSheet.Button hasArrow={text % 2 === 0} key={text}>
          {text}
        </BottomSheet.Button>
      ))}
    </BottomSheet>
  );
}

export default BottomSheet1;
```

```tsx
// src/components/BottomSheet2.tsx

function BottomSheet2({ title, description, buttonTexts }: TProps) {
  // dimmed 미존재, arrow 버튼 존재
  return (
    <BottomSheet>
      <BottomSheet.Title>{title}</BottomSheet.Title>
      <BottomSheet.Description>{description}</BottomSheet.Description>
      {buttonTexts.map((text) => (
        <BottomSheet.Button hasArrow={text % 2 === 0} key={text}>
          {text}
        </BottomSheet.Button>
      ))}
    </BottomSheet>
  );
}

export default BottomSheet2;
```

```tsx
// src/components/BottomSheet3.tsx

function BottomSheet3({ title, description, buttonTexts }: TProps) {
  // dimmed 미존재, arrow 버튼 존재
  return (
    <BottomSheet>
      <BottomSheet.Title>{title}</BottomSheet.Title>
      <BottomSheet.Description>{description}</BottomSheet.Description>
      {buttonTexts.map((text) => (
        <BottomSheet.Button hasArrow={false} key={text}>
          {text}
        </BottomSheet.Button>
      ))}
    </BottomSheet>
  );
}

export default BottomSheet3;
```

#### Render Props

[React 레거시 공식문서](https://ko.legacy.reactjs.org/docs/render-props.html)에 따르면 이 패턴도 대부분 custom hook으로 대체되었다고 한다.  
custom hook은 로직을 추상화하고 재사용 가능한 함수로 만들어 컴포넌트 간에 상태와 동작을 공유할 수 있도록 도와주기 때문이다.  
그냥 Render Props 패턴이 왜 사용됐는지 정도만 알아보자.

##### 코드 예시

마우스 움직임을 트래킹해야 되는 컴포넌트가 있다고 하자.  
아래와 같이 만들 수 있을 것이다.

```tsx
// src/pages/MouseTracker.tsx

function MouseTracker() {
  const [location, setLocation] = useState<Axis>({ x: 0, y: 0 });

  function handleMouseMove(event) {
    setLocation({
      x: event.clientX,
      y: event.clientY,
    });
  }

  return (
    <div style={{ height: '100vh' }} onMouseMove={handleMouseMove}>
      <p>
        마우스 위치 - x: ({location.x}, y: {location.y})
      </p>
    </div>
  );
}

export default MouseTracker;
```

만약 다른 페이지에서 `<p>` 태그 이외에 내용을 넣고 마우스를 트래킹하는 로직만을 재사용하는 방법은 필요하다고 하자.  
아래처럼 `<p>` 태그 위치에 별도의 컴포넌트를 선언해서 렌더링하는 것도 하나의 방법일 것이다.

```tsx
// src/pages/MouseTracker2.tsx

function MouseTracker2() {
  const [location, setLocation] = useState<Axis>({ x: 0, y: 0 });

  function handleMouseMove(event: MouseEvent) {
    setLocation({
      x: event.clientX,
      y: event.clientY,
    });
  }

  return (
    <div style={{ height: '100vh' }} onMouseMove={handleMouseMove}>
      {/* 원래는 <p>태그였음 */}
      <Component mouse={location} />
    </div>
  );
}

export default MouseTracker2;
```

하지만 이렇게 하면 마우스 트래킹 로직을 사용할 때마다 매번 새로운 컴포넌트를 선언해야 한다.  
이때 사용할 수 있는 것이 Render Props 패턴이다.  
이 패턴은 로직을 사용하는 쪽에서 렌더링 결과물도 결정해준다.

```tsx
// src/components/Mouse.tsx

// 반드시 'render'이라는 이름이 아니어도 됨
function Mouse({ render }: TProps) {
  const [position, setPosition] = useState<Axis>({ x: 0, y: 0 });

  const handleMouseMove = (event: MouseEvent) => {
    setPosition({ x: event.clientX, y: event.clientY });
  };

  return <div onMouseMove={handleMouseMove}>{render(position)}</div>;
}

export default Mouse;
```

```tsx
// src/pages/MouseTracker.tsx

function MouseTracker() {
  return (
    <div>
      <Mouse
        render={(position) => (
          <p>
            마우스 위치 - x: {position.x}, y: {position.y}
          </p>
        )}
      />
    </div>
  );
}

export default MouseTracker;
```

이 내용을 Custom API로도 바꿔보겠다.

```tsx
// src/hooks/use-mouse.ts

function useMouse() {
  const [position, setPosition] = useState<Axis>({ x: 0, y: 0 });

  const handleMouseMove = (event: MouseEvent) => {
    setPosition({ x: event.clientX, y: event.clientY });
  };

  useEffect(() => {
    window.addEventListener('mousemove', handleMouseMove);

    return () => {
      window.removeEventListener('mousemove', handleMouseMove);
    };
  }, []);

  return position;
}

export default useMouse;
```

```tsx
// src/pages/MouseTracker.tsx

function MouseTracker() {
  const position = useMouse();

  return (
    <div>
      <p>
        마우스 위치 - x: {position.x}, y: {position.y}
      </p>
    </div>
  );
}

export default MouseTracker;
```

취향 차이일 수도 있겠다만 나는 custom hook을 사용하는 것이 더 깔끔한 것 같다.  
그리고 무엇보다 Render Props보다 각각의 컴포넌트가 하는 역할이 명확히 구분되어서 더 나은 것 같다.

#### Control Props

`props`를 통해 외부에서 컴포넌트 상태를 제어할 수 있도록 만드는 방법이다.  
간단히 `button` 태그의 `onClick`을 외부에서 내려주는 것이라고 생각할 수 있다.

```tsx
function Button({ children, hasArrow, onClick }: TProps) {
  return (
    <>
      {hasArrow ? '->' : ''}
      <button onClick={onClick}>{children}</button>
    </>
  );
}
```

Control Props 패턴은 외부에서 `props`가 존재하면 해당 값을 사용하고, 존재하지 않으면 내부의 state로 컴포넌트의 상태를 관리할 수 있도록 할 수 있다.  
또한 custom hook을 사용해 제어 컴포넌트를 간결하게 만들 수도 있다.

Control Props 패턴은 두 가지 장점이 있다.

1. 사용자한테 좀 더 유연성과 제어권을 줄 수 있다(IOC, Inversion Of Control).
2. 상태를 한곳에서 관리하는 SSOT(Single Source Of Truth)가 가능해진다.

##### 코드 예시

`value`와 `onChange`를 `props`로 전달 받는 `Counter` 컴포넌트가 존재한다고 하자.  
`Counter`는 `value`를 전달 받으면 `state`를 관리하지 않고(uncontrolled)(상위 컴포넌트에서 관리), 전달 받지 않으면 스스로 `state`를 관리한다(controlled).

```tsx
type TProps = {
  value?: number;
  onChange: (next: number) => void;
}

function Counter({ value: countProp, onChange }: TProps) {
  const [count, setCount] = useState(countProp ?? 0);

  function onClickUp() {
    if (value === undefined) {
      const nextCount = count + 1;
      setCount(nextCount);
      onChange(nextCount);
    } else {
      onChange(countProp + 1);
    }
  }

  function onClickDown() {
    if (value === undefined) {
      const nextCount = count - 1;
      setCount(nextCount);
      onChange(nextCount);
    } else {
      onChange(countProp - 1);
    }
  }

  return (
    <div>
      <span>{countProp ?? count}</span>
      <button onClick={onClickUp}>up</button>
      <button onClick={onClickDown}>down</button>
    </div>
  );
}
```

hook을 선언하면 `value === undefined` 확인 로직을 custom hook에서 처리하고 `ControlledCounter`를 더 간결하게 만들 수 있다.

```ts
interface Arguments<T extends any> {
  valueProp?: T;
  defaultValue: T;
}

type Return<T extends any> = [T | undefined, Dispatch<SetStateAction<T>>];

function useControlled<T extends any>({
  valueProp,
  defaultValue,
}: Arguments<T>): Return<T> {
  const { current: isControlled } = useRef(valueProp !== undefined);
  const [state, setState] = useState<T>(defaultValue);

  // controlled이면 valueProp 사용, uncontrolled면 state사용
  const value = isControlled ? valueProp : state;

  // uncontrolled이면 setState로 내부 상태 변경
  // controlled이면 외부에서 상태 값을 넣어줄 것이므로 아무것도 하지 않음
  const setValue: Dispatch<SetStateAction<T>> = useCallback(
    (newState: T | ((prev: T) => T)) => {
      !isControlled && setState(newState);
    },
    [isControlled]
  );

  return [value, setValue];
}
```

```tsx
type TProps = {
  value?: number;
  onChange: (next: number) => void;
}

function Counter({ value: countProp, onChange }: TProps) {
  // count는 number | undefined
  const [count, setCount] = useControlled<number>({
    valueProp: countProp,
    defaultValue: 0,
  });

  function onClickUp() {
    setCount((count) => (count ? count + 1 : 1));
    onChange(count ? count + 1 : 1);
  }

  function onClickDown() {
    setCount((count) => (count ? count - 1 : -1));
    onChange(count ? count - 1 : -1);
  }

  return (
    <div>
      <span>{count}</span>
      <button onClick={onClickUp}>up</button>
      <button onClick={onClickDown}>down</button>
    </div>
  );
}
```

#### Props Getters

getter의 return값을 spread 연산자로 컴포넌트에게 뿌려주는 패턴이다.  
유연하고 유저가 원한다면 props를 오버로드할 수 있지만 변화 추적이 힘들다.

Control Props 패턴처럼 자식 컴포넌트가 부모 컴포넌트에게 데이터나 콜백을 전달할 수 있는 패턴이다.

##### 코드 예시

먼저 `props.children`을 통해 별도의 컴포넌트 없이 자식 컴포넌트에게 `props`를 전달하는 방법을 알아야 한다.

```tsx
// src/components/Toggle.tsx

function Toggle({ children }: TProps) {
  const [isOn, setIsOn] = useState(false);
  const togglerProps = {
    onClick: toggle,
  };

  function toggle() {
    setIsOn((isOn) => !isOn);
  }

  return children({ isOn, togglerProps });
}

export default Toggle;
```

```tsx
// src/App.tsx

function App() {
  return (
    <div className="App">
      <Toggle>
        {({ isOn, togglerProps }) => {
          return (
            <>
              <div>토글: {isOn ? 'ON' : 'OFF'}</div>
              <button {...togglerProps}>click</button>
            </>
          );
        }}
      </Toggle>
    </div>
  );
}

export default App;
```

`button`을 클릭할 때마다 토글이 된다.  
다만 아직까지는 자식 컴포넌트에서 부모 컴포넌트로 무언가를 전달할 수 있는 상황이 아니다.

위에서 `togglerProps`를 일반 객체가 아니라 콜백으로 변경한다면 `Toggle.props.children`에서 `Toggle`에게 데이터를 전달할 수 있다.  
추가적으로 커링 기법을 이용하는 `compose`라는 유틸 함수가 추가된다면 로직을 더 간결하게 표현할 수도 있다.

```ts
// src/utils/functions.ts

type AnyFunction = (x: any) => any;

export function compose(...fns: ((x: any) => any)[]): AnyFunction {
  return (x: any) => {
    return fns.reduce((acc, fn) => fn(acc), x);
  };
}
```

```tsx
// src/components/Toggle.tsx

type ChildrenProps = {
  isOn: boolean;
  getTogglerProps: ({ onClick }: { onClick: AnyFunction }) => any;
}

type TProps = {
  children: ({ isOn, getTogglerProps }: ChildrenProps) => JSX.Element;
}

function Toggle({ children }: TProps) {
  const [isOn, setIsOn] = useState(false);

  function getTogglerProps({ onClick }: { onClick: AnyFunction }) {
    return {
      onClick: compose(toggle, onClick),
    };

    // 또는
    // return {
    //   onClick: () => {
    //     toggle()
    //     onClick()
    //   }
    // }
  }

  function toggle() {
    setIsOn((isOn) => !isOn);
  }

  return children({ isOn, getTogglerProps });
}
```

```tsx
// src/App.tsx

function App() {
  return (
    <div className="App">
      <Toggle>
        {({ isOn, getTogglerProps }) => {
          return (
            <>
              <div>토글: {isOn ? 'ON' : 'OFF'}</div>
              <button
                {...getTogglerProps({
                  onClick: () => {
                    alert('1234');
                  },
                })}
              >
                click
              </button>
            </>
          );
        }}
      </Toggle>
    </div>
  );
}
```

#### State Reducer

위에 나온 모든 패턴 중에 가장 자유도(유연성)가 높고 제어의 정도가 낮다.  
모든 내부 action들을 외부에서 접근가능하며 오버라이드할 수 있다.  
이 말은 반대로 사용자가 책임지고 이해해야 하는 부분이 많고 컴포넌트 내부 로직에 대한 이해도가 필요하다는 말이다.

~이 내용은 좀 뇌피셜이다.~  
React의 [useReducer](https://react.dev/reference/react/useReducer) 훅을 이용한 것에서 시작한 말인 것 같다.  
`useReducer`는 dispatch를 이용해 상태 업데이트 로직을 컴포넌트 밖에서 수행하도록 선언할 수 있는 훅이다.  
아래와 같이 사용 가능하다.

```ts
// src/reducers/count-reducer.ts

interface CountState {
  count: number;
}

export const ACTION = {
  INCREMENT: 'INCREMENT',
  DECREMENT: 'DECREMENT',
};

type CountAction = (typeof ACTION)[keyof typeof ACTION];

function countReducer(state: CountState, action: { type: CountAction }) {
  switch (action.type) {
    case ACTION.INCREMENT:
      return { count: state.count + 1 };
    case ACTION.DECREMENT:
      return { count: state.count - 1 };
    default:
      throw new Error('올바르지 않은 reducer action 타입');
  }
}

export default counterReducer;
```

```tsx
// src/pages/Counter.tsx

function Counter() {
  const [{ count }, dispatch] = useReducer(countReducer, { count: 0 });

  return (
    <div className="App">
      <span>count: {count}</span>
      <button onClick={() => dispatch({ type: ACTION.INCREMENT })}>+1</button>
      <button onClick={() => dispatch({ type: ACTION.DECREMENT })}>-1</button>
    </div>
  );
}

export default Counter;
```

##### 코드 예시

toggle 예시를 보자.  
toggle이 자주 사용돼서 toggle에 대한 로직을 custom hook으로 뺀 상황이다.  
`Switch`는 on/off slide가 되는 컴포넌트라고 가정하겠다.  
`Toggle` 컴포넌트는 `useToggle` hook을 이용한 단순한 UI이다.

```ts
// src/hooks/use-toggle.ts

function useToggle() {
  const [on, setOn] = useState<boolean>(false);

  const toggle = () => setOnState((o) => !o);
  const setOn = () => setOnState(true);
  const setOff = () => setOnState(false);

  return { on, toggle, setOn, setOff };
}
```

```tsx
// src/components/Toggle.tsx

// 이렇게 사용되고 있었음
function Toggle() {
  const { on, toggle, setOn, setOff } = useToggle();

  return (
    <div>
      <button onClick={setOff}>Switch Off</button>
      <button onClick={setOn}>Switch On</button>
      <Switch on={on} onClick={toggle} />
    </div>
  );
}
```

만약 toggle은 연속 최대 4번만 가능하고, 이후에 toggle을 하고 싶다면 reset 버튼을 눌러야지만 가능하게 해야 한다는 요구사항이 생기면 어떻게 할 수 있을까?  
쉽게 아래와 같이 요구 사항을 반영할 수 있다.

```tsx
// src/components/LimitedToggle.tsx

function LimitedToggle({ clickCount }: TProps) {
  const [clicksSinceReset, setClicksSinceReset] = useState(0);
  const clickAvailable = clicksSinceReset < clickCount;

  const { on, toggle, setOn, setOff } = useToggle();

  function handleClick() {
    if (clickAvaiable) {
      toggle();
      setClicksSinceReset((count) => count + 1);
    }
  }

  return (
    <div>
      <button onClick={setOff}>Switch Off</button>
      <button onClick={setOn}>Switch On</button>
      <Switch on={on} onClick={handleClick} />
      {tooManyClicks ? (
        <button onClick={() => setClicksSinceReset(0)}>Reset</button>
      ) : null}
    </div>
  );
}
```

만약 이렇게 toggle 행위에 제한이 걸린 `~~Toggle` 컴포넌트가 많이 생기면 어떻게 될까?  
해당 컴포넌트들이 위 `LimitedToggle`처럼 `clickAvailable` 등의 변수를 선언하고 로직을 컴포넌트 안에서 다뤄야만 할까?  
이에 대한 해답을 제시해주는 것이 State Reducer 패턴이 될 수 있다.  
default reducer를 선언하지만 컴포넌트에 따라 이를 오버라이드하고, 업데이트 로직을 `useToggle` 안으로 밀어넣을 수 있다.

```ts
// src/hooks/use-toggle.ts

export const ACTION = {
  TOGGLE: 'TOGGLE',
  ON: 'ON',
  OFF: 'OFF',
};

type ToggleAction = (typeof ACTION)[keyof typeof ACTION];

export function toggleReducer(
  state: { on: boolean },
  action: { type: ToggleAction }
) {
  switch (action.type) {
    case ACTION.TOGGLE: {
      return { on: !state.on };
    }
    case ACTION.ON: {
      return { on: true };
    }
    case ACTION.OFF: {
      return { on: false };
    }
    default: {
      throw new Error('올바르지 않은 reducer action 타입');
    }
  }
}

function useToggle({ reducer = toggleReducer } = {}) {
  const [{ on }, dispatch] = useReducer(reducer, { on: false });

  const toggle = () => dispatch({ type: ACTION.TOGGLE });
  const setOn = () => dispatch({ type: ACTION.ON });
  const setOff = () => dispatch({ type: ACTION.OFF });

  return { on, toggle, setOn, setOff };
}

export default useToggle;
```

```tsx
// src/components/LimitedToggle.tsx

function LimitedToggle() {
  const [clicksSinceReset, setClicksSinceReset] = useState(0);
  const tooManyClicks = clicksSinceReset >= 4;

  const { on, toggle, setOn, setOff } = useToggle({
    reducer(currentState, action) {
      const changes = toggleReducer(currentState, action);

      return tooManyClicks && action.type === ACTION.TOGGLE
        ? { ...changes, on: currentState.on }
        : changes;
    },
  });

  function handleClick() {
    toggle();
    setClicksSinceReset((count) => count + 1);
  }

  return (
    <div>
      <button onClick={setOff}>Switch Off</button>
      <button onClick={setOn}>Switch On</button>
      <Switch onClick={handleClick} on={on} />
      {tooManyClicks ? (
        <button onClick={() => setClicksSinceReset(0)}>Reset</button>
      ) : null}
    </div>
  );
}
```

---

(작성 중)

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
