## 0. 공부하게 된 이유

퍼블리싱의 영역이라고 볼 수도 있지만, 개인적으로 UI적 요소도 프론트엔드 개발의 중요한 요소라고 생각한다.  
디자이너가 요구한 내용을 최대한 반영하기 위해서 css에 대한 기본 소양이 필요하다고 느꼈다.  
React에는 css를 적용하는 다양한 방법이 있고, 특히 CSS-in-JS 방법에는 tailwindcss, emotion, styled-jsx 등이 있다고 한다.  
https://www.freecodecamp.org/news/how-to-style-react-apps-with-css/ 를 토대로 정리해봤다.

## 1. inline-styling

inline 요소는 말 그대로 jsx(또는 tsx) 파일 안에 css적 요소를 선언하는 것이다.  
stylesheet에 선언한 style 보다 우선순위가 높다. 즉, 다른 stylesheet에 쓴 내용이 'override' 될 수 있다는 의미이다.

```jsx
export default function App() {
  return (
    <section
      style={{
        fontFamily: '-apple-system',
        fontSize: '1rem',
        fontWeight: 1.5,
        lineHeight: 1.5,
        color: '#292b2c',
        backgroundColor: '#fff',
        padding: '0 2em',
      }}
    >
      <div
        style={{
          textAlign: 'center',
          maxWidth: '950px',
          margin: '0 auto',
          border: '1px solid #e6e6e6',
          padding: '40px 25px',
          marginTop: '50px',
        }}
      >
        <div>
          <p
            style={{
              lineHeight: 1.5,
              fontWeight: 300,
              marginBottom: '25px',
              fontSize: '1.375rem',
            }}
          >
            contents
          </p>
        </div>
        <p
          style={{
            marginBottom: '0',
            fontWeight: 600,
            fontSize: '1rem',
          }}
        >
          contents
        </p>
      </div>
    </section>
  )
}
```

위 코드 예시에서 볼 수 있듯이 가장 큰 단점은 작은 컴포넌트임에도 return 부분이 매우 길어져서 코드 가독성이 좋지 않다.  
아래 코드는 이를 극복하기 위한 방법이다.

```jsx
const styles = {
  section: {
    fontFamily: '-apple-system',
    fontSize: '1rem',
    fontWeight: 1.5,
    lineHeight: 1.5,
    color: '#292b2c',
    backgroundColor: '#fff',
    padding: '0 2em',
  },
  wrapper: {
    textAlign: 'center',
    maxWidth: '950px',
    margin: '0 auto',
    border: '1px solid #e6e6e6',
    padding: '40px 25px',
    marginTop: '50px',
  },
  quote: {
    lineHeight: 1.5,
    fontWeight: 300,
    marginBottom: '25px',
    fontSize: '1.375rem',
  },
}

export default function App() {
  return (
    <section style={styles.section}>
      <div style={styles.wrapper}>
        <div>
          <p style={styles.quote}>content</p>
        </div>
      </div>
    </section>
  )
}
```

이전보다 코드 가독성은 좋아졌지만 단순히 css stylesheet를 쓰는 것과 비교했을 때 얻을 수 있는 큰 장점은 없다.

- 장점
  - 생산성이 높다.
  - prototype을 만들기에 편하다.
  - 우선순위가 높게 css를 적용할 수 있다.
- 단점
  - pseudo-class(:hover 등), pseudo-elements(::first-line) 또는 animation을 사용할 수 없다.
  - 가독성이 좋지 않다.
  - 확장성이 좋지 않다.

## 2. plain css(또는 scss)

일반적인 html에 css 적용하는 것과 크게 다르지 않다.  
class나 id name을 지정해줘서 스타일을 적용한다.  
scss에 보다 자세한 내용은 [여기](https://github.com/mochang2/development-diary/blob/main/021-styled%20sheet.md)에 정리했다.

```jsx
import './styles.css'

export default function App() {
  return (
    <section className="testimonial">
      <div className="testimonial-wrapper">
        <div className="testimonial-quote">
          <p>content</p>
        </div>
      </div>
    </section>
  )
}
```

- 장점
  - modern css의 모든 기능을 사용할 수 있다.
  - jsx에 style과 관련된 모든 코드를 없앨 수 있다.
- 단점
  - scope이 없어서 모든 component에 스타일이 적용된다.
  - 위 단점 때문에 변수 이름들(또는 class name)이 길어진다.
  - css의 경우 sass 등과 비교했을 때 typing이나 boilerplate가 더 많이 필요하다.

_cf) 보일러플레이트 또는 보일러플레이트 코드: 최소한의 변경으로 여러 곳에서 재사용되며, 반복적으로 비슷한 형태를 띄는 코드를 말함_

## 3. css modules

[2](https://github.com/mochang2/development-diary/edit/main/022-React%20css.md#2-plain-css%EB%98%90%EB%8A%94-scss)와 크게 다르지 않은 방법이지만, stylesheet를 module(object 형식)처럼 import 해와 inline-styling처럼 사용할 수 있다.  
다만 모든 css 파일의 확장자는 .module.css로 변경해야 한다.  
CRA로 react 앱을 시작하면 기본적으로 사용할 수 있는 방법이다.

```css
body {
  font-family: -apple-system, system-ui, BlinkMacSystemFont, 'Segoe UI', Roboto,
    'Helvetica Neue', Arial, sans-serif;
  margin: 0;
  font-size: 1rem;
  font-weight: 1.5;
  line-height: 1.5;
  color: #292b2c;
  background-color: #fff;
}

/* styles skipped */

.testimonial-name span {
  font-weight: 400;
}
```

```jsx
import styles from './styles.module.css'

export default function App() {
  return (
    <section className={styles.testimonial}>
      <div className={styles['testimonial-wrapper']}>
        <div>
          <p className={styles['testimonial-quote']}>content</p>
        </div>
      </div>
    </section>
  )
}
```

위 방식으로 만들어진 html은 아래와 같은 형태이다.  
이를 통해 알 수 있듯이 이 방법은 global scope 문제를 해결할 수 있다.

```html
<p class="_styles__testimonial-name_309571057">content</p>
```

- 장점
  - style이 해당 컴포넌트에만 적용된다(unique한 class name이 생겨 style 간에 충돌이 날 일이 없다).
  - CRA 프로젝트에서 별다른 setup이 필요없다.
  - sass(scss)도 이와 같은 방법으로 사용할 수 있다.
- 단점
  - class name 추론이 어렵다.

## 4. css in js

- 장점
  - style이 해당 컴포넌트에만 적용된다(unique한 class name이 생겨 style 간에 충돌이 날 일이 없다).
  - css가 js object가 되기 때문에 export, reuse, extend가 가능하다.
- 단점
  - third-party js library를 추가적으로 설치해야 해서 프로젝트가 무거워질 수 있다.

다양한 방법으로 구현이 가능하지만 많이 사용되는 tailwind css / styled component / emotion 3가지에 관해 간단한 차이점 위주로 정리해보고자 한다.  
추가적으로 전부 dev dependency에 설치할 줄 알았지만 tailwind css는 dependency에 설치하라고 공식문서에 나와 있었다.
~공식문서 잘 보자~.

#### tailwind css

tailwind css는 utility-first css 프레임워크이다.  
utility-first css라고 하면 생소하겠지만 bootstrap이 이 개념을 기반으로 제작되었다.  
따라서 사용방법도 bootstrap과 매우 유사하다.  
각 class가 담당할 스타일을 미리 정의하고 필요한 class들을 조합해서 적용하는 방법으로 사용한다.

`.{property}{side}-{size}`와 같이 class name을 짓는다.  
`npm install -D tailwindcss` 또는 `yarn add -D tailwindcss`로 설치가 가능하다.

// TODO: tailwind.config.js 변경이나 react 프로젝트에 적용하기 등은 나중에 직접 사용하게 되면 추가
tailwind.config.js 관련해서 알고 싶다면 [여기](https://so-so.dev/web/tailwindcss-w-twin-macro-emotion/)를 볼 것.

아래는 tailwind css를 적용한 코드의 예시이다.

```jsx
const Navbar = () => {
  return (
    <nav className="h-1/6">
      <ul className="flex">
        <li className="mr-9 hover:underline underline-offset-8">
          <a className="color-black">Home</a>
        </li>
        <li className="mr-9 hover:underline underline-offset-8">
          <a className="color-pink">Search</a>
        </li>
        <li className="mr-9 hover:underline underline-offset-8">
          <a className="color-blue">Profile</a>
        </li>
      </ul>
    </nav>
  )
}
```

보다시피 가독성이 굉장히 좋지 않다.  
이를 해결하기 위해 [tailwind-styled-components](https://www.npmjs.com/package/tailwind-styled-components)라는 모듈을 이용해  
styled-components의 장점과 같이 섞어서 사용하는 방법도 있다.

#### styled component

`npm install styled-components` 또는 `yarn add styled-components`로 설치가 가능하다.

아래는 예제코드이다.

```tsx
import styled from 'styled-components'

const GridContent = styled.div<{ active: boolean }>`
  display: flex;
  justify-content: space-between;
  background-color: ${({ active }) => (active ? 'black' : '#f1f1f1')};
`

const Title = styled.div`
  color: #202020;
  & > h2 {
    display: inline-block;
    margin-left: 15px;
  }
`

// 기본적인 html 태그뿐만 아니라 본인이 만든 component도 styled 안에 인자로 들어갈 수 있음
const NewsAgency = styled(GridContent)`
  flex-direction: column;
  padding: 10px 0;
`

const NonNewsAgency = styled(GridContent)`
  padding: 20px 16px;
`

export default function Component() {
  const active = true

  return (
    <>
      <Title>제목</Title>
      <NewsAgency active={active}>
        {' '}
        // props로 active를 넘겨줌
        {/* some jsx code */}
      </NewsAgency>
      <NonNewsAgency active={active}>{/* some jsx code */}</NonNewsAgency>
    </>
  )
}
```

주의할 것은 styled component의 변수명이 **PascalCase** 가 아니면 리액트에서 인식을 못 한다.  
아래는 가변 스타일링으로 여러 css 속성을 변경하고 싶다면 사용할 수 있는 방법이다.

```jsx
import styled, { css } from 'styled-components'

const CustomButton = styled.button`
  line-height: 120%;
  border: 1px dotted #fff;

  ${(props) =>
    props.primary &&
    css`
      color: white;
      background: navy;
      border-color: navy;
    `}
`

export default function Component(props) {
  return <CustomButton {...props}>content</CustomButton> // 넘겨야 할 props가 많다면 spread operator 사용할 수 있음
}
```

_+) 추가사항 1_  
`styled-components` 5.1부터 [transient props](https://styled-components.com/docs/api#transient-props)가 생겼다.  
컴포넌트에 스타일 구성 요소가 (props로서) react element로 전달되거나 DOM 요소로 렌더링되는 것을 방지하려면 props 앞에 달러 기호($)를 붙여 임시 소품으로 바꿀 수 있다.

```jsx
// 아래 예시에서 $draggable은 DOM에는 전달되지 않는 반면,
// draggable은 DOM에 전달된다.

const Comp = styled.div`
  color: ${(props) => props.$draggable || 'black'};
`

render(
  <Comp $draggable="red" draggable="true">
    Drag me!
  </Comp>
)
```

_+) 추가사항 2_  
`styled-components`에 global style, typescript, media query 적용하는 방법: https://velog.io/@hwang-eunji/styled-component-typescript

#### emotion

단순한 예제 코드가 아닌 react 프로젝트에 적용하기를 정확히 알고 싶다면 [여기](https://www.howdy-mj.me/css/emotion.js-intro/)를 참고.  
emotion은 **framework agnostic**(프레임워크를 사용하지 않는 것)과 **react** 두 가지 방식이 사용된다.  
전자는 `@emotion/css` 모듈을 설치함으로써, 후자는 `@emotion/react` 모듈을 설치함으로써 사용할 수 있다.

아래는 예제코드이다.

```jsx
/** @jsxImportSource @emotion/react */ // 이 주석이 없으면 에러가 난다고 한다
import { css, jsx } from '@emotion/react'

const divStyle = css`
  background-color: hotpink;
  font-size: 24px;
  border-radius: 4px;
  padding: 32px;
  text-align: center;
  &:hover {
    color: white;
  }
`

export default function component() {
  return <div css={divStyle}>content</div> // css props를 넘겨주지 않고 styled component처럼 사용할 수도 있다
}
```

예제코드에서 알 수 있듯이 컴포넌트 하나하나가 정확히 어떤 html 태그인지 알 수 있다.  
styled component도 babel 설정을 바꾸면 이를 지원해준다고 한다.  
개인적인 생각이지만 html 태그를 정확히 알지 못해도 props를 덜 전달하는 것이 코드가 깔끔해질 것 같다.  
emotion의 또다른 장점은 css modules나 styled components와 같이 className을 임의로 생성해줌으로써 class name이 겹칠 일이 없다.  
임의로 생성되는 이름을 바꾸고 싶다면 `.babelrc` 안에 설정을 바꾸면 된다(자세한 내용은 위에 적어놓은 블로그를 통해).

#### 추가사항. styled component vs emotion

https://velog.io/@bepyan/styled-components-%EA%B3%BC-emotion-%EB%8F%84%EB%8C%80%EC%B2%B4-%EC%B0%A8%EC%9D%B4%EA%B0%80-%EB%AD%94%EA%B0%80 이 velog를 참고했다.

styled component와 emotion은 sass 문법을 사용하며, 전반적인 스타일 기능이 동일하다.

- Attaching Props
- Media Queries
- Global Styles
- Nested Selectors
- Server Side Rendering
- Theming Support
- Composition

둘 모두 위 기능들을 제공한다.

### 차이점 1. 번들 사이즈

[번들 사이즈를 알아볼 수 있는 사이트](https://bundlephobia.com/)를 참고하면 최신 라이브러리 번들 사이즈를 알 수 있다.

<img src="https://user-images.githubusercontent.com/63287638/178094724-dedcd99a-e201-41e9-b521-630f3209387b.png" alt="emotion style" width="600" height="auto" />
<br/>
<br/>

<img src="https://user-images.githubusercontent.com/63287638/178094726-3c2d5250-2edf-44fc-a096-3d4cfbe8bdea.png" alt="styled component" width="600" height="auto" />
<br/>
<br/>

react에서 사용하면 @emotion/react 도 설치한다지만, 단순하게 styled component 패키지만을 비교해보면 @emotion/style이 훨씬 가볍다.

### 차이점 2. 속도

일반적으로는 근소하게 emotion이 더 빠르다고 한다.  
다만 라이브러리 버전마다 속도 차이가 있기 때문에 항상이라고는 말할 수 없다.
