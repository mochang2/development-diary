## 0. 공부하게 된 계기

회사 업무는 아니다.  
다만 회사에서 Next.js를 쓰는 가장 큰 이유가 SSR 때문이라고 한다.  
깨비팀에서 내가 만든 React.js를 보면 첫 로딩 시간이 너무 느리긴 했다.  
그 당시에는 중요한 이슈가 아니었어서 스치듯이 문서만 본 적이 있는데, 이번 기회에 확실히 정리하고자 한다.  
결론부터 말하자면 둘 중 하나가 탁월히 좋다는 것이 아니다.  
각각의 장단점이 있기 때문에 프로젝트에 맞는 렌더링을 선택하는 것이 맞다고 생각한다.  
참고한 블로그는 https://oneroomtable.tistory.com/entry/%EC%84%9C%EB%B2%84-%EC%82%AC%EC%9D%B4%EB%93%9C-%EB%A0%8C%EB%8D%94%EB%A7%81%EA%B3%BC-%ED%81%B4%EB%9D%BC%EC%9D%B4%EC%96%B8%ED%8A%B8-%EC%82%AC%EC%9D%B4%EB%93%9C-%EB%A0%8C%EB%8D%94%EB%A7%81%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80%EC%9A%94 와 https://velog.io/@ash3767/%EC%84%9C%EB%B2%84%EC%82%AC%EC%9D%B4%EB%93%9C-%EB%A0%8C%EB%8D%94%EB%A7%81-%ED%81%B4%EB%9D%BC%EC%9D%B4%EC%96%B8%ED%8A%B8-%EC%82%AC%EC%9D%B4%EB%93%9C-%EB%A0%8C%EB%8D%94%EB%A7%81 이다.

## 1. 개념 및 특징

우선 렌더링이란 서버에 요청해서 받은 내용을 브라우저 화면에 표시하는 것이다.  
SSR / CSR은 서버에서 렌더링을 하냐, 클라이언트에서 렌더링 하냐의 차이가 있다.

### SSR

SSR은 Server Side Rendering의 약어로, 사용자가 페이지를 이동할 때마다 새로운 페이지를 요청한다.  
사용자가 받을 템플릿(html 등)은 서버에서 렌더링하고 완성된 페이지 형태로 응답을 준다.  
특히 단순한 텍스트와 정적인 이미지만 표시하던 과거에 SSR은 큰 문제가 없었다.  
하지만 현재의 복잡한 웹 사이트의 경우 사용자가 특정 기능을 실행하기 위해 단 한번의 클릭을 했어도 사이트 전체가 다시 로딩되어야 하는 문제에 생긴다.  
장단점은 다음과 같다

- 장점
  - 검색엔진 최적화(SEO) 가능
  - 처음 로딩 속도가 빠름
- 단점
  - 페이지 이동시 화면이 깜빡거림
  - 사이트 상호 작용의 부족
  - 서버 측 부하 발생

내가 장고로 개발을 할 때 이런 방식이었다.

### CSR

CSR은 Client Side Rendering의 약어로, 브라우저에서 첫 요청 시 한 페이지만 불러온다.  
일반적으로 브라우저에서 자바스크립트를 사용하여 콘텐츠를 렌더링하는 것을 의미한다.  
_SPA와 항상 같이 나오는 개념이라고 생각해도 된다._  
React.js와 같은 경우 index.js(또는 ts)에서 모든 페이지 내용을 가지고 있고, 사용자는 index.js만 받아서 화면에 표시한다.  
장단점은 다음과 같다

- 장점
  - 처음 로딩 이후 사용자의 행동에 따라 필요한 부분만 다시 읽어들이기 때문에(특히 React.js가 변화된 부분만 빠르게 캐치하는 엄청난 알고리즘이 있다고 한다) SSR보다 빠른 응답을 기대 가능
  - 웹 애플리케이션에 좋음
  - 사이트 상호 작용 활발
- 단점
  - 검색엔진 최적화(SEO, Search Engine Optimization)가 어려움
  - 초기 로딩에 많은 시간이 걸림
  - 대부분의 경우 추가적인 라이브러리를 필요로 함

## 2. 리액트 프레임워크

Remix, Next.js, Gatsby 세 개 다 써보지 않아서 정확한 비교는 내가 정리하는 것보다는 https://www.youtube.com/watch?v=RP8nvTeurbQ 여길 보는 게 나을 것 같다.  
다만 이러한 프레임워크를 사용하는 이유는 CSR / SSR을 넘어서 그냥 React로만 프로젝트를 구성하는 것보다 다양한 이유가 있기 때문이다.  
CSR, SSR에 대한 것만 정리하자면 세 가지 프레임워크 모두 React의 CSR을 극복할 수 있다.  
Remix는 SSR만 제공하는 것 같다.  
Next.js는 CSR, SSR을 모두 제공할 수 있다.  
Gatsby는 서버 없이, 오로지 정적 사이트 생성을 위해 사용하는 프레임워크이다.

## 3. Next

[공식문서](https://nextjs.org/learn/foundations/about-nextjs?utm_source=next-site&utm_medium=nav-cta&utm_campaign=next-website) 와 [여기](https://kyounghwan01.github.io/blog/React/next/basic/)를 참고했다.

#### 주요 기능

~SSR과 SEO는 생략~

- **pre-fetching**: 백그라운드에서 페이지를 미리 가져오는 것을 말함.
- **pre-rendering**: 미리 서버측에서 렌더링을 완료함.
- hot reloading: 개발 중 저장되는 코드는 자동으로 새로고침됨.
- code splitting: 본인이 원하는 페이지에서 원하는 js와 library를 렌더링. js 로딩 시간을 개선해주는 기능.
- typescript 간단히 사용 가능: webpack, babel 등을 건드릴 필요가 없이 `yarn add typescript @types/node @types/react`로 설치 가능.
- 글로벌 스타일 정의. 다만 app.ts(tsx, js, jsx)에서만 선언이 가능함.
- sass 간단히 사용 가능: config 파일 정의 없이 `yarn add -D sass`로만으로 scss 파일 사용 가능.
- **automatic routing / dynamic routing**: pages 디렉터리 아래에 있는 파일은 해당 파일 이름으로 라우팅됨. 예를 들어 page.ts라는 파일이 있다면 `domain:\[port\]/page`로 라우팅됨. 파일 이름을 \[id\].ts라고 지었다면 `domain:\[port\]/id`로 접근이 가능하며 id는 동적으로 변화가 가능함. 해당 컴포넌트(정확하게는 페이지)에서는 `useRouter`를 이용한 뒤 `router.query`를 통해 id를 얻을 수 있음.

```typescript
// [id].ts
import { useEffect } from "react";
import { useRouter } from "next/router";

export default SomePage = () => {
  const router = useRouter();
  useEffect(() => {
    console.log(router.query.id);
  });
};
```

#### routing

https://stackoverflow.com/questions/65086108/next-js-link-vs-router-push-vs-a-tag 를 참고했다.  
Next에서는 라우팅을 하는 방법이 _\<Link\>_ 를 이용한 방법, _useRouter().push()(이하 router.push)_ 를 이용한 방법, _\<a\>_ 를 이용한 방법이 있다.  
하지만 \<a\>를 이용한 라우팅은 보통 사용되지 않는데, 두 가지 단점이 있기 때문이다.

1. \<a\>를 사용하면 처음 페이지에 진입시 번들 파일을 받고, \<a\>에 의해 라우팅되면 다시 번들 파일을 받기 때문이다. 즉, 렌더링 속도가 느려진다.
2. redux로 전역 state를 관리할 때 store의 값이 증발된다. 스택 오버플로우에 따르면 `<a> tag without using next/link's <Link> creates a standard hyperlink which directs end user to the url as a new page` 라고 한다.

_router.push_ 와 _\<Link\>_ 의 차이는 주로 **SEO** 와 관련되어 있다.  
router.push는 window.location과 비슷하게 동작하여 \<a\>를 생성하지 않는다.  
이 말은 연관된 링크가 크롤러에 의해 수집되지 않아 SEO에 불리하다.  
반면, \<Link\>는 \<a\>를 생성하여 연관된 링크가 크롤러에 의해 수집이 될 수 있다.  
스택 오버플로우에 따르면 `Endusers will still navigate with without reloading the page, creating the behavior of a Single Page App` 라고 한다.

_cf) 같은 page에서 다른 컴포넌트로 router 이동 시 history가 쌓이지 않기 때문에 뒤로가기 버튼을 누르면 이전 화면이 뜨지 않고 이전전 page 화면이 뜬다._  
이를 해결하고 싶다면 [next.js에서 쿼리스트링으로 같은 page 내에서 window.history 쌓기](https://kyounghwan01.github.io/blog/React/next/same-page-stack-history/#%E1%84%80%E1%85%A2%E1%84%8B%E1%85%AD)를 참고.

#### pre-fetching

기본값은 `true`로 `<Link />` 가 렌더링될 때 `href` 옵션으로 연결되어 있는 모든 항목이 미리 로드되는 것을 말하며 production 레벨에서만 이루어진다.

#### pre-rendering

공식 홈페이지에 있는 그림을 통해 보면 한 눈에 알 수 있다.

![pre](https://user-images.githubusercontent.com/63287638/170853036-87c96189-9898-40ad-8d24-63b06449f794.png)  
![non pre](https://user-images.githubusercontent.com/63287638/170853038-f846d041-e514-4265-9494-eb019f31149c.png)

pre-rendering의 방식에는 두 가지가 존재한다.

1. static generation
   build time에 HTML을 만드는 메소드로, 한 번 만들어진 뒤에는 매 request 마다 해당 HTML이 재사용된다.  
   ![static](https://user-images.githubusercontent.com/63287638/170853165-089df46b-1bcd-41b7-adfe-ea4f5a786647.png)

2. server-side rendering
   매 request 마다 HTML을 만드는 메소드로, static generation과 달리 해당 HTML이 재사용되지 않는다.  
   ![ssr](https://user-images.githubusercontent.com/63287638/170853169-8e4df98e-550b-4ae2-85ef-257b5a793958.png)

#### 기본 구조

루트 디렉터리 안에는 component, pages, global 등의 디렉터리가 위치된다.  
해당 디렉터리 안에 들어가지 않는 파일에는 index.ts, \_app.ts, \_document.ts가 있다.  
다만 index.ts가 반드시 존재하는 것은 아니다.  
index.ts는 domain:\[port\]/ 와 같이 루트 페이지에 접근할 때, 상황에 따라(로그인 여부 등) 화면을 보여주거나 redirect 시키는 등의 기능을 주로 넣어두는 파일이다.

```typescript
// _app.ts 예시
function App({ Component, pageProps }) {
  ~~~
  return <Component {...pageProps} />
}
```

- \_app.ts
  - 이 곳에서 렌더링하는 값은 모든 페이지에 영향을 준다.
  - 페이지를 열 때 가장 먼저 실행되는 공통 레이아웃으로, 내부에 컴포넌트들을 실행한다.
  - 내부에 컴포넌트가 있다면 전부 실행하고 html의 body로 구성한다.
  - 인자로 `Component`와 `pageProps`가 있다.
    - `Component`는 요청한 페이지이다. 'GET /' 을 보냈다면, `Component`에는 `/pages/index.ts` 파일이 props로 전달된다.
    - `pageProps`는 getInitialProps를 통해 내려 받는 props들을 말하며 자세한 설명은 [아래](https://github.com/mochang2/development-diary/edit/main/007-C,S%20Side%20Rendering.md#getinitialprops)에 있다.
  - 이 파일 이후에 \_document.ts가 실행된다.
  - `console.log` 실행시 서버, 클라이언트 모두에서 콘솔 결과를 볼 수 있다.

```typescript
// _document.ts 예시
import Document, { Html, Head, Main, NextScript } from "next/document";
export default class MyDocument extends Document {
  render() {
    return (
      <Html>
        <Head>
          // 모든페이지에 아래 메타(웹 타이틀, ga 같은 것)가 head에 들어감
          // 루트파일이기에 가능한 적은 코드만 넣음
          <meta ~ />
        </Head>
        <body>
          <Main />
        </body>
        <NextScript />
      </Html>
    );
  }
}
```

- \_document.ts
  - meta 태그를 정의하거나, 전체 페이지에 관려하는 컴포넌트이다.
  - `console.log` 실행시 서버에서만 보인다.
  - `componentDidMount` 같은 훅도 실행되지 않으며 static한 상황(로직)에만 사용된다.
  - material-ui나 styled component를 사용하기 위해서는 이 파일을 고쳐야 하는데, 이에 대한 내용은 [아래](https://github.com/mochang2/development-diary/edit/main/007-C,S%20Side%20Rendering.md#_documentts)에 나와 있다.

#### \_document.ts

공식문서에서 각 사용법에 대해 나와 있으므로 해당 내용을 참고해도 된다.

**material-ui(mui) 사용법**

1. `yarn add @emotion/react @emotion/styled @mui/icons-material @mui/material @mui/styles` 실행
2. 아래와 같이 변경(서버에서 받아온 html, css와 클라이언트가 렌더링한 html, css가 다르면 next에서 wraning을 띄우므로, 서버단에서 mui를 지원함으로써 서버와 클라이언트간 간극을 맞추기 위해)

```typescript
// document.ts
import React from "react";
import Document, { Html, Head, Main, NextScript } from "next/document";
import { ServerStyleSheets } from "@mui/styles";

export default class MyDocument extends Document {
  static getInitialProps = async (ctx) => {
    const materialSheets = new ServerStyleSheets();
    const originalRenderPage = ctx.renderPage;

    ctx.renderPage = () =>
      originalRenderPage({
        enhanceApp: (App) => (props) =>
          materialSheets.collect(<App {...props} />),
      });

    const initialProps = await Document.getInitialProps(ctx);
    return {
      ...initialProps,
      styles: <>{initialProps.styles}</>,
    };
  };
}

// app.ts
import type { AppProps } from "next/app";
import CssBaseline from "@mui/material/CssBaseline";

const App = ({ Component, pageProps }: AppProps) => {
  return (
    <>
      <CssBaseline />
      <Component {...pageProps} />
    </>
  );
};

export default App;
```

**styled component 사용법**

1. `yarn add -D babel-plugin-styled-components` 실행
2. .babelrc 수정

```
{
  "presets": ["next/babel"],
  "plugins": [
    [
      "babel-plugin-styled-components",
      { "fileName": true, "displayName": true, "pure": true, "ssr": true } // 옵션보고 설정
    ]
  ]
}
```

3. \_document.ts 변경

```typescript
// ~~
import { ServerStyleSheet } from "styled-components";
// ~~

export default class MyDocument extends Document {
  static getInitialProps = async (ctx) => {
    const sheet = new ServerStyleSheet();
    // ~~
      ctx.renderPage = () =>
      originalRenderPage({
        enhanceApp: App => props =>
          sheet.collectStyles(materialSheets.collect(<App {...props} />))
      });

      // ~~~
      return {
      ...initialProps,
      styles: (
        <>
          {initialProps.styles}
          {sheet.getStyleElement()}
        </>
      )
    // ~~~
}
```

#### getInitialProps

Next v9 이상에서는 `getInitialProps` 대신 `getStaticProps, getStaticPaths, getServerSideProps`을 사용하도록 가이드하지만 편의상 `getInitialProps`라고 사용했다.  
react, vue같은 Client Side Rendering (CSR)의 경우는 useEffect, created 함수를 이용하여 data fetching을 한다.  
서버사이드에서 실행하는 Next에서는 getInitialProps를 이용하여 data fetching 작업을 한다.

### 장점

1. 속도가 빨라진다. 서버는 data fetching을, 브라우저는 렌더링만 함으로써 연산이 분산되기 때문에 속도가 빨라진다.
2. 함수형 컴포넌트로 코딩할 경우 data fetching, 렌더링이 분리되어 있으므로 로직 파악이 쉽다.

### 주의사항

**1. 하나의 페이지에서는 하나의 getInitialProps가 실행된다**.
next에서는 \_app -> page component 순으로 페이지가 렌더링된다.  
만약 \_app에 `getInitialProps`를 정의했다면, 하위 컴포넌트에서는 `getInitialProps`가 실행되지 않는다.  
하위 컴포넌트에서도 `getInitialProps` 값을 반영하기 위해서는 아래와 같이 변경이 필요하다.

```typescript
// app.ts
function MyApp({ Component, pageProps }) {
  return <Component ponent {...pageProps} />;
}

MyApp.getInitialProps = async ({ Component, ctx }) => {
  let pageProps = {};
  // 하위 컴포넌트에 getInitialProps가 있다면 추가 (각 개별 컴포넌트에서 사용할 값 추가)
  if (Component.getInitialProps) {
    pageProps = await Component.getInitialProps(ctx);
  }

  // _app에서 props 추가 (모든 컴포넌트에서 공통적으로 사용할 값 추가)
  pageProps = { ...pageProps, extra: { id: 1234, property: "what" } };

  return { pageProps };
};

export default MyApp;
```

**2. getInitialProps는 서버에서 실행된다**
브라우저 api(setTimeout, window.\~, document.\~)는 해당 함수 내에서 실행되지 않는다.

**getStaticProps**  
`getStaticProps`라는 함수를 export 하면 Next는 `getStaticProps`에 의해 return되는 props를 build 타임에 pre-render한다.  
해당 값은 빌드 시 고정되며, 빌드 이후에는 값 변경이 불가능하다.  
다음과 같은 상황에서 보통 사용된다.

- The data required to render the page is available at build time ahead of a user’s request
- The data comes from a headless CMS
- The page must be pre-rendered (for SEO) and be very fast — getStaticProps generates HTML and JSON files, both of which can be cached by a CDN for performance
- The data can be publicly cached (not user-specific). This condition can be bypassed in certain specific situation by using a Middleware to rewrite the path.

```javascript
// code example

// posts will be populated at build time by getStaticProps()
function Blog({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li>{post.title}</li>
      ))}
    </ul>
  );
}

// This function gets called at build time on server-side.
// It won't be called on client-side, so you can even do direct database queries.
export async function getStaticProps() {
  const res = await fetch("https://.../posts"); // any data fetching library. ex) swr
  const posts = await res.json();

  return {
    props: {
      posts,
    },
  };
}

export default Blog;
```

**getStaticPaths**  
dynamic routing을 사용하는 페이지에서 `getStaticProps`라는 함수를 export 하면 Next는 빌드시 return된 모든 path를 정적으로 pre-render한다.  
이곳에서 정의하지 않는 하위 경로는 접근해도 페이지가 뜨지 않는다.  
다음과 같은 상황에서 보통 사용된다.

- The data comes from a headless CMS
- The data comes from a database
- The data comes from the filesystem
- The data can be publicly cached (not user-specific)
- The page must be pre-rendered (for SEO) and be very fast — getStaticProps generates HTML and JSON files, both of which can be cached by a CDN for performance

```javascript
// code example
export async function getStaticPaths() {
  return {
    paths: [
      { params: { ... } }
    ],
    fallback: true // false or 'blocking'
  };
}
```

**getServerSideProps**  
`getServerSideProps`라는 함수를 export 하면 Next는 `getServerSideProps`에 의해 return되는 props를 build 타임에 pre-render한다.  
데이터를 미리 렌더링할 필요가 없는 경우나 데이터가 자주 변경되는 경우에는 클라이언트 측에서 데이터를 가져오는(useEffect 사용) 것을 고려해야 한다.  
다음과 같은 상황에서만 사용된다.

```
You should use getServerSideProps only if you need to render a page
whose data must be fetched at request time.
This could be due to the nature of the data or properties of the request
(such as authorization headers or geo location).
Pages using getServerSideProps will be server side rendered at request time and
only be cached if cache-control headers are configured.

If you do not need to render the data during the request, then you should consider fetching data on the client side or getStaticProps.
```

```javascript
// code example
function Page({ data }) {
  // Render data...
}

// This gets called on every request
export async function getServerSideProps() {
  const res = await fetch(`https://.../data`);
  const data = await res.json();

  // Pass data to the page via props
  return { props: { data } };
}

export default Page;

// 캐시하는 방법
export async function getServerSideProps({ req, res }) {
  res.setHeader(
    "Cache-Control",
    "public, s-maxage=10, stale-while-revalidate=59"
  );

  return {
    props: {},
  };
}
```

#### life cycle

1. nextJs 서버가 GET 요청을 받는다.
2. GET 요청에 맞는 pages를 찾는다.
3. \_app.ts의 `getInitialProps`가 있다면 실행한다.
4. route에 맞는 페이지 Component의 `getInitialProps`가 있다면 실행하고, pageProps들을 받아온다.
5. \_document.ts의 `getInitialProps`가 있다면 실행하고, pageProps들을 받아온다.
6. 모든 props들을 구성하고, \_app.ts -> page Component 순서로 렌더링한다.
7. 모든 Content를 구성하고 \_document.tsx를 실행하여 html 형태로 출력한다.

## 4. Remix, Gatsby는 나중에 사용하게 되면 작성할 예정
