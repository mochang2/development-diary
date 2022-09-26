## 0. 공부하게 된 계기

React? Vue? 아무리 개발을 해봐도 근본을 모르면 말짱 도루묵인 것 같다.  
~과제 테스트 통과 못하는 건 덤~  
우테캠으로 과제 전형을 보고, vanilla js로 [virtual DOM을 비슷하게 따라 만드는 강의](https://junilhwang.github.io/TIL/Javascript/Design/Vanilla-JS-Virtual-DOM/)가 있다는 것을 알고 공부해야겠다 마음을 먹었다.

## 1. babel

JS는 ES6에서 JS 표준 모듈 문법이 정의되었음에도 ES6 문법을 구형 브라우저에서 사용하지 못해 SystemJS 같은 또 다른 라이브러리에 의존했어야 했다.

[공식문서](https://babeljs.io/docs/en/)에 따르면

> Babel is a JavaScript compiler.

바벨은 자바스크립트 컴파일러이다.  
JS 자체는 인터프리터 언어이지만 JS를 또 다른 JS 결과물로 만들어줄 필요가 있고, 바벨은 이러한 행위를 하기 때문에 *소스 대 소스 컴파일러 즉, 트랜스파일러*라고 불린다.  
따라서 바벨이 하는 가장 중요한 일은 **ES6 이후의 문법들이 구형 브라우저에서도 돌아갈 수 있도록 컴파일**하는 것이다.

이외에도 babel은 polyfill, source code transformation, type annotation 등의 기능을 제공한다.

바벨은 다음과 같은 3단계를 거쳐서 컴파일이 진행된다.

1. 파싱
2. 변환
3. 출력

이때 1, 3 단계는 바벨 코어가 진행하지만 2단계는 플러그인이 처리한다.

### Custom Plugin 만들기

`npm install @babel/core @babel/cli`로 필요한 모듈을 먼저 설치한다.

```javascript
// my-babel-plugin.js

module.exports = function myBabelPlugin({ types }) {
  return {
    visitor: {
      VariableDeclaration(path) {
        console.log('VariableDeclaration() kind:', path.node.kind) // const

        if (path.node.kind === 'const') {
          path.node.kind = 'var'
        }
      },
    },
  }
}
```

이후 `npx babel [변환할 파일.js] --plugins ./my-babel-plugin.js` 실행한다.

위 코드는 ES6의 `const` 키워드를 `var`로 바꾸는 작업을 한다.  
바벨 플러그인은 반드시 객체 안에 `visitor`가 있어야 되고 그 안에 객체 메서드가 존재한다.  
파싱 결과물(AST)에서 해당하는 타입의 노드가 생성되면 같은 이름의 메서드가 호출된다.  
많이 나오는 예시로는 다음과 같다.

- Program : 루트의 타입
- VariableDeclaration : 변수 타입
- FunctionDeclaration : 함수 타입
- Identifier : 개발자가 만든 값(변수, 함수 등)
- BinaryExpression : 사칙연산
- Literal : 문자열

더 많은 내용은 [AST explorer](https://astexplorer.net/)에서 확인 가능하다.

### 이미 만들어진 설정 사용하기

`.babelrc` 또는 `babel.config.js(on)`, `package.json` 등에 설정을 추가할 수 있다.  
다만 `package.json`에는 보통 잘 안 두고 설정을 분리하는 편이다.  
해당 설정 파일들에 사용할 babel plugin들을 모아놓는다.

`npm install @babel/plugin-transform-block-scoping @babel/plugin-transform-arrow-functions @babel/plugin-transform-strict-mode`로 모듈을 설치하고 다음과 같이 파일을 추가한다.

```javascript
// babel.config.js

module.exports = {
  plugins: [
    '@babel/plugin-transform-block-scoping',
    '@babel/plugin-transform-arrow-functions',
    '@babel/plugin-transform-strict-mode',
  ],
}
```

다만 실제로는 이렇게 custom plugin을 만들거나 모든 plugin을 별도로 설치하지 않고 미리 만들어놓은 preset을 사용한다.  
preset은 플러그인들을 모아놓은 것을 말한다.  
아래와 같이 사용할 수 있다.

```javascript
// babel.config.js

module.exports = {
  presets: ['@babel/preset-env'],
}
```

해당 preset이 하는 자세한 역할은 [공식 문서](https://babeljs.io/docs/en/babel-preset-env)를 참조.

### 웹팩과 사용

~웹팩은 [아래](https://github.com/mochang2/development-diary/blob/main/026-vanilla%20js.md#polyfill)를 참조.~

실무에서는 바벨을 직접 사용하지 않고 보통 웹팩에 `babel-loader`를 통해 사용한다.  
아래 인용은 [공식 문서](https://webpack.js.org/loaders/babel-loader/)의 내용이다.

> babel-loader exposes a loader-builder utility that allows users to add custom handling of Babel's configuration for each file that it processes.

`babel-loader`의 정의가 명확히 나오지 않아서 해당 인용문을 토대로 `babel-loader`를 필자의 방식대로 정의했다.  
webpack은 loader를 통해 `test`에 해당하는 파일명을 가진 파일들을 전처리를 할 수 있다.  
따라서 `babel-loader`는 `js` 파일들에 대해 babel 설정(`babel.config.js`)을 적용하여 컴파일할 수 있도록 도와주는 놈(?) 정도로 정의할 수 있을 것 같다.

사용은 `npm install babel-loader`를 한 뒤 아래와 같이 설정하면 된다.

```javascript
// webpack.config.js

module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.js$/,
        loader: 'babel-loader',
        exclude: /node_modules/,
      },
    ],
  },
}
```

### polyfill

babel을 사용한다고 반드시 JS 최신 함수를 사용할 수 있는 것은 아니다.  
예를 들어 `let`, `const`나 arrow function, block scoping 등은 구식 문법으로 변경할 수 있지만 `new Promise()`와 같은 문법은 ie 등에서 사용할 수 없다.  
이러한 문법은 polyfill을 사용해서 해결해야 한다.

babel은 문법을 변환해주는 역할만 할 뿐이고 실제로 최신 문법의 코드들이 브라우저에서 동작할 수 있도록 각 함수를 검사해서 object의 proptotype에 추가하는 역할을 한다.  
즉, babel은 컴파일 타임에, babel-polyfill은 런타임에 실행된다.

[모던 JS 튜토리얼](https://ko.javascript.info/polyfills)에 따르면

> 명세서엔 새로운 문법이나 기존에 없던 내장 함수에 대한 정의가 추가되곤 합니다. 새로운 문법을 사용해 코드를 작성하면 트랜스파일러는 이를 구 표준을 준수하는 코드로 변경해줍니다. 반면, 새롭게 표준에 추가된 함수는 명세서 내 정의를 읽고 이에 맞게 직접 함수를 구현해야 사용할 수 있습니다. 자바스크립트는 매우 동적인 언어라서 원하기만 하면 어떤 함수라도 스크립트에 추가할 수 있습니다. 물론 기존 함수를 수정하는 것도 가능합니다. 개발자는 스크립트에 새로운 함수를 추가하거나 수정해서 스크립트가 최신 표준을 준수 할 수 있게 작업할 수 있습니다.
> 이렇게 변경된 표준을 준수할 수 있게 기존 함수의 동작 방식을 수정하거나, 새롭게 구현한 함수의 스크립트를 "폴리필(polyfill)"이라 부릅니다. 폴리필(polyfill)은 말 그대로 구현이 누락된 새로운 기능을 메꿔주는(fill in) 역할을 합니다.

[core js](https://github.com/zloirock/core-js)는 다양한 폴리필을 제공해주며,  
[polyfill.io](https://polyfill.io/v3/)는 기능이나 사용자의 브라우저에 따라 폴리필 스크립트를 제공해주는 서비스이다.

바벨이 특정한 대상 환경을 지원하기 위해 어떤 JS polyfill을 사용해야 할지 결정하려면 [여기](https://kangax.github.io/compat-table/es6)를 참조하자.

### type annotation

참고: https://ui.toast.com/weekly-pick/ko_20181220

JS나 TS에서 타입 **표기**에 대해 도와준다.  
착각하기 쉬운 부분이지만, 타입 **체킹**을 도와주는 것이 아니다.  
[Flow preset](https://babeljs.io/docs/en/babel-preset-flow), [Typescript preset](https://babeljs.io/docs/en/babel-preset-typescript) 등을 통해 사용 가능하다.

flow preset은

`npm install --save-dev @babel/preset-flow`

```javascript
// flow 자체가 JS에 대한 static type checker이다.

// in
// @flow
function square(n: number): number {
  return n * n
}

// out
function square(n) {
  return n * n
}
```

typescript preset은

`npm install --save-dev @babel/preset-typescript`

```typescript
// in
function Greeter(greeting: string) {
  this.greeting = greeting
}

// out(ts 부분을 완전히 지워버림)
function Greeter(greeting) {
  this.greeting = greeting
}
```

'typescript는 `tsc` 명령어로 ES5로 변환시켜주는데 굳이 tsc와 babel을 같이 사용할 필요가 있는가?' 싶었다.  
(참고로, tsc와 babel을 동시에 사용한다면 컴파일러의 순서는 TS -> (ts-node 등 tsc) -> JS -> (babel) -> JS이다)  
하지만 동시에 사용하면 두 가지 이점이 있다.

1. 컴파일이 빠르다. 바벨의 캐싱과 단일 파일 방출 설계(?) 덕분에 더욱 빠른 컴파일 속도를 제공한다.
2. 원할 때만 타입 체킹할 수 있다. babel 컴파일러는 ts를 완전히 지워버리기 때문에 한참 코딩하고 타입 체킹 없이 컴파일을 빠르게 해야 할 때 유용하다.

### 궁금했던 점

**Q. React.createElement vs jsx**

_참고: https://www.daleseo.com/react-jsx/_

먼저 간단히 jsx가 무엇인지 정리하자면 Javascript XML의 약자로 HTML이 아니고, JS를 확장하여 XML처럼 사용할 수 있도록 도와주는 문법이다.  
단순히 템플릿 언어(A templating language basically is a language which allows defining placeholders that should later on be replaced for the purpose of implementing designs)라고 생각할 수 있지만 jsx는 JS의 모든 기능이 포함되어 있다.
React element를 생성하는데 사용되며 vue나 svelte에서도 사용할 수 있다.

[공식문서](https://ko.reactjs.org/docs/jsx-in-depth.html)에 따르면 jsx는 사실 `React.createElement`의 syntax sugar일 뿐이다.

```jsx
<MyButton color="blue" shadowSize={2}>
  Click Me
</MyButton>
```

위 코드는 아래와 같이 컴파일된다.

```javascript
React.createElement(MyButton, { color: 'blue', shadowSize: 2 }, 'Click Me')
// 참고로 컴포넌트 이름은 반드시 대문자로 시작해야 함
// 소문자로 시작하면 html태그로 인식하여
// <component /> => React.createElement('component') 로 트랜스파일되는 반면
// <Component /> => React.createElement(Component)로 트랜스파일됨
// 단, dot notation을 사용하면 <obj.component /> => React.createElement(obj.component)처럼 사용 가능

// 이후 `ReactDOM.render(변수명, 배치할 곳)`을 통해 화면에 렌더링 됨
```

이 모든 과정을 도와주는 것이 babel이다.

```jsx
import React from 'react'

const Box = () => {
  return <div>i'm box</div>
}
```

react 17 버전 이전 함수 컴포넌트에서는 위와 같이 `React`를 사용하지 않는데도 반드시 React를 import해야 된다.  
이는 JSX를 transpile하기 위해선 `react`를 import함으로써 `React.createElement`를 사용해야 하기 때문이다.

하지만 react 17 버전 이후부터는 새로운 JSX transform인 React 패키지 자체의 `_jsxRuntime` 함수를 불러와 이러한 작업을 가능하게 해준다.

## 2. Webpack

Webpack은 module bundler이다.

### module

_참고: [js module](https://blog.bitsrc.io/javascript-require-vs-import-47827a361b77)_

Webpack을 온전히 이해하기 위해서는 JS에서 module이 무엇인지 먼저 알 필요가 있다.  
참고로 JS는 모듈이 없는 상태로 세상에 나타났고(이전에는 `<script>` 태그 이용), 모듈 개념은 node.js가 만들어지고 서버에서 JS를 사용할 수 있게 되면서 나온 개념이다.  
**module**이란 프로그램을 구성하는 내부의 코드가 기능별로 나뉘어져 있는 형태로 다음과 같은 장점이 있다.

- 유지보수성: 기능들이 모듈화가 잘 되어 있다면, 의존성을 그만큼 줄일 수 있기 때문에 어떤 기능을 개선하거나 수정할 때 훨씬 편하다.
- 네임스페이스: 모듈로 분리하여 모듈만의 네임스페이스를 가지게 할 수 있다.
- 재사용성: 똑같은 코드를 반복하지 않고 모듈을 분리시켜서 필요할 때마다 사용할 수 있다.

JS에서 모듈의 개념이 나오기 전에는 아래와 같은 방식으로 사용했다.

```html
<script src="jquery.js"></script>
<script src="tweenmax.js"></script>
<!-- 위 두 js를 사용해 내 코드 작성-->
<script>
  window.$
  window.TweenMax
</script>
```

위와 같은 방식을 사용하면 변수명이 겹칠 경우 오류가 발생하며, 필요가 없는 코드들도 전부 가져오게 되는 문제가 있었다.  
만약 `jquery.js`와 `tweenmax.js`에서 `a`라는 변수를 각각 사용하고 있었다면, 나중에 로드된 모듈이 먼저 로드된 모듈의 변수를 재정의했다.

이러한 문제를 해결하기 위해 나온 방식이 4가지가 있다.

#### 1) commonJS

Node.js에서 채택한 방식으로 지금도 node.js를 사용하면 바벨 없이는 `require()`과 `module.exports`를 사용해야 한다.  
동기적으로 모듈을 불러오기 때문에 보통 서버사이드에서 사용한다.  
사용하는 문법은 아래와 같다.

```javascript
// 내보내는 부분
function a() {
  console.log('Hello Wolrd')
}

module.exports = {
  a,
}
// 또는
module.exports = a

// 사용하는 부분
const { a } = require('./xxx')
// 또는
const a = require('./xxx')
```

#### 2) AMD(Asynchronous Module Definition)

FE에서 모듈 개념을 친숙하게 만들기 위해 도입된 개념이다.  
서버 프로그래밍의 특성상 commonJS는 파일에서 코드를 동적으로 불러오면 용량의 제한과 방식의 제한이 없는 반면 브라우저는 JS 파일을 저장하지 못하고 매번 불러와야 했기 때문에 로딩과 비동기를 고려해야 했다.  
commonJS처럼 동기적으로 모듈을 가져오면 필요한 모듈을 전부 다운로드할 때까지 아무것도 할 수 없는 상태가 되기 때문에 생긴 개념이다(commmonJS와 함께 논의하다 합의점을 이루지 못하고 독립한 그룹으로, commonJS가 JS를 브라우저 밖으로 꺼낸 개념이라면, AMD는 브라우저에 중점을 뒀음).  
또한 commonJS는 tree shaking(import 되었지만 실제로 사용되지 않은 코드를 분석하고 삭제하는 코드 최적화 기술)이 어려운 데다 순환 참조에 취약했다.  
AMD는 bundler가 필요 없으며 모든 dependency가 동적으로 resolve된다.  
사용하는 문법은 아래와 같다.

```javascript
// 내보내는 부분
// 종속성을 갖는 모듈인 'package/lib'를 모듈 선언부의 첫 번째 파라미터에 넣으면, 'package/lib'은 콜백 함수의 lib 파라미터 안에 담긴다
define(['package/lib'], function (lib) {
  // 로드된 종속 모듈을 아래와 같이 사용할 수 있다
  function foo() {
    lib.log('lib')
  }

  return {
    foobar: foo,
  }
})

// 사용하는 부분
require(['package/myModule'], function (myModule) {
  myModule.foobar()
})
```

#### 3) UMD(Universal Module Definition)

commonJS와 AMD로 나뉜 모듈 구현방식을 하나로 합치기 위한 방식이다.  
사실 그래서 UMD는 모듈 사용 방식이라기 보다는 디자인 패턴에 가깝다고 한다.  
Webpack이나 RollUp같은 몇몇 JS 번들러들은 ES6 방식으로 모듈 로드에 실패했을 때 대안책으로 UMD 패턴으로 로드하는 방식을 사용한다.  
UMD는 [공식 UMD 소스코드](https://github.com/umdjs/umd/blob/master/templates/returnExports.js)를 보면 아래와 같이 IIFE를 사용한다.

```javascript
;(function (root, factory) {
  if (typeof define === 'function' && define.amd) {
    // AMD. Register as an anonymous module.
    define([], factory)
  } else if (typeof module === 'object' && module.exports) {
    // Node. Does not work with strict CommonJS, but
    // only CommonJS-like environments that support module.exports,
    // like Node.
    module.exports = factory()
  } else {
    // Browser globals (root is window)
    root.returnExports = factory()
  }
})(typeof self !== 'undefined' ? self : this, function () {
  // Just return a value to define the module export.
  // This example returns an object, but the module
  // can return a function as the exported value.
  return {}
})
```

#### 4) ECMAScript2015(ES6)

JS 공식 모듈 시스템이다.  
`import`와 `export`라는 키워드를 사용한다.

```html
<script type="module" src="./index.js"></script>
```

위와 같이 선언하면 일반적인 JS 파일과 다른 세 가지 특징을 지닌다.

1. `import` 혹은 `export` 구문을 사용할 수 있다.
2. 기본적으로 strict mode로 동작한다.
3. 모듈의 가장 바깥쪽에서 선언된 이름은 전역 스코프가 아니라 모듈 스코프에서 선언된다. (`a.js`에서 선언한 `const foo = 'bar'`를 `b.js`에서 별다른 `import` 없이는 사용하지 못한다)

(3번은 HTML 파일에서 `<script type="module" src="./a.js"></script>` 하지 않았다는 가정이다)

가장 큰 장점은 모듈을 비동기적으로 불러오며 빌드 타임에 정적 분석이 가능하여 tree shaking이 쉽다는 것이다.  
또한 commonJS와는 다르게 실제 객체/함수를 바인딩하기 때문에 순환 참조 관리도 편하다.  
모든 브라우저가 지원하는 것은 아니며 ~(RIP IE...)~ Node.js에서도 아직 commonJS가 공식적으로 사용되기 때문에 Babel의 `@babel/plugin-transform-modules-commonjs`를 통해 변환시켜야 한다.

### bundle

_참고: https://www.freecodecamp.org/news/javascript-modules-part-2-module-bundling-5020383cf306/ , https://snipcart.com/blog/javascript-module-bundler_

module을 사용하는 것까지는 좋았으나 브라우저 특성상 JS 파일을 동시에 여러 개 호출하면 속도 문제가 발생하거나 특정 JS 파일의 로딩이 지연되면 전체가 늦어지는 문제가 있었다.  
이러한 문제를 해결하기 위해 등장한 것이 bundle이다.

> As a result, each of those files has to be included in your main HTML file in a `<script>` tag, which is then loaded by the browser when a user visits your home page. Having separate `<script>` tags for each file means that the browser has to load each file individually: one… by… one.

module이 기능에 관한 코드가 모여있는 각각의 파일이었다면 bundle은 모듈들의 의존성을 안전하게 유지시키면서 하나의 파일로 만드는 과정이다.  
즉, 서로 참조관계를 가지고 있는 모듈들을 모아서 하나의 파일로 묶는 것이라고 할 수 있다.

이게 바로 모듈 번들러의 탄생 배경이다.  
모듈 번들러란 JS 모듈을 브라우저에서 실행할 수 있는 단일 JS 파일로 번들링하는데 사용되는 프론트엔드 개발 도구이다.  
_모듈 로더(JS 모듈을 런타임에 로드할 수 있게 하는 구현체로 RequireJS나 네이티브 브라우저 등이 포함)와 유사한 부분이 있지만, 모듈 번들러(컴파일 시간에 빌드 산출물을 만들어서 하나의 js파일을 산출)는 코드를 프로덕션 환경에서 사용할 수 있도록 준비하는 데 더 큰 목적이 있음._

번들링 과정은 두 가지 과정(**dependency graph generation, eventual bundling**)으로 진행된다.

1. dependency graph generation은 모듈 간의 관계 지도를 만드는 것이다. 이 과정을 위해서는 `index.js`와 같은 entry file이 필요하며 이 entry file을 바탕으로 각각의 파일에 unique ID를 부여한 뒤 dependency order를 만든다. 이 과정을 통해 naming conflict을 해결하며 사용되지 않은 파일을 찾아 제거한다.
2. 이후 bundler는 브라우저가 처리할 수 있는 static asset을 만든다. 이 과정을 *Packing*이라 하며, bundler는 dependency graph를 활용하여 여러 코드 파일을 통합하고 필요한 함수 및 `module.exports` 개체를 삽입하고 브라우저가 성공적으로 로드할 수 있는 단일 실행 파일 bundle을 반환한다.

bundler마다 약간의 차이가 있을 수는 있지만, 이러한 과정을 통해 bundling을 진행하면 다음과 같은 장점이 있다.

1. 모든 모듈을 로드하기 위해 검색하는 시간을 단축 가능하다.
2. 사용하지 않는 코드(공백, 주석, 줄바꿈 문자 등)와 파일을 제거함으로써 브라우저가 리소스를 더 빠르게 불러올 수 있도록 도와준다. 렌더 트리를 그리기 위해 파싱하는 시점도 빨라질 수 있다.

### 그래서 webpack이 뭔데, 뭐가 좋은데

- 많은 서드 파티를 필요로 하는 복잡한 애플리케이션이라면 Webpack이 가장 적합하다고 한다.

1. **code splitting**

특정 상황에서만 필요한 코드 블록이 있는 web application의 경우 전체 코드 베이스를 하나의 대용량 번들 파일에 넣는 것은 효율적이지 않다.  
이 경우 code splitting은 요청 시에만 로드할 수 있는 bundle chunk로 코드를 쪼갬으로써 'big up-front payload' 문제를 방지할 수 있다.  
(RollUp이나 Parcel도 code splitting을 지원하며 더 빠르지만 안정성 측면에서는 Webpack이 더 좋다고 함)

2. **리소스 및 애셋**

리소스(CSS)나 애셋(Image, Font 등)들도 JS 코드로 변환하고 이를 분석해서 bundling하는 방식을 제공한다. 다만 이 때문에 다른 bundler에 비해 설정할 게 많고 복잡하다.

#### 로더(Loader)

**로더는 Node.js에서 실행된다.**  
그래서 설정 파일과 같은 경우 commonJS의 모듈 방식인 `require`을 이용한다.

```javascript
import SomeImage from 'assets/image/some-image.png'
import SomeFont from 'assets/font/some-font.otf'
import './main.css'
```

위와 같이 이미지, 폰트, CSS를 모듈로 `import`하는 코드를 본 적이 있을 것이다.  
이는 웹팩이 정적인 애셋들을 JS의 모듈처럼 사용할 수 있게 도와주는 기능이다.

위와 같이 사용할 수 있도록 `webpack.config.js` 설정은 다음과 같이 변경해야 한다.

우선 `npm install webpack webpack-cli css-loader style-loader file-loader`(자주 사용되는 로더 예시) 한 뒤 아래와 같이 설정 파일을 변경한다.

```javascript
// webpack.config.js
const path = require('path')

module.exports = {
  mode: 'development', // production, none이 존재.
  entry: {
    main: './src/app.js', // 번들을 여러 개로 나눌 경우 entry를 분리할 수 있음.
  },
  output: {
    filename: '[name].js', // entry의 key 값이 [name]에 해당.
    path: path.resolve('./dist'),
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'], // 순서가 중요. 뒤에 있는 loader부터 실행.
      },
      {
        test: /\.(png|jpg|gif|svg)$/,
        loader: 'file-loader',
        options: {
          publicPath: './dist/',
          // index.html가 src 내부에 위치하지 않는다면 필요한 설정. 빌드 산출물 경로(정적 파일 접근을 위한 경로).
          name: '[name].[ext]?[hash]',
          // 캐시 무력화를 위해 해시값 사용. 다른 사진이 같은 이름이 되는 것을 방지하기 위해 사용.
        },
      },
    ],
  },
}
```

참고로 `rules` 안에 `use`와 `loader`는 하는 역할이 똑같은데 2개 이상의 로더를 사용하고 싶다면 `use`를 사용해야 한다.

inline으로 사용하는 방법도 있는데 보통은 잘 사용되지 않는 것 같다.
아래는 `css-loader`를 inline으로 사용하는 방법이다.

```javascript
import Styles from 'style-loader!css-loader?modules!./styles.css'
```

위 `webpack.config.js`를 설정한 뒤 `webpack` 명령어를 입력하면 아래와 같은 결과를 얻을 수 있다.  
안에 내용이 난독화되지 않은 건 별도의 설정 없이 `development`로 빌드해서 그렇다.

![webpack result1](https://user-images.githubusercontent.com/63287638/192182948-8d444f1a-618d-46e2-9800-26afbc791d4e.jpg)

![webpack result2](https://user-images.githubusercontent.com/63287638/192182949-895c8232-a55d-48ee-ab34-55e02315f694.jpg)

##### file-loader vs url-loader

`file-loader`는 배포 폴더에 해당 파일을 옮기는 정도의 역할이다.  
`url-loader` 또한 사진과 같은 정적 파일을 모듈처럼 사용하게 해주는 로더이지만 다른 결과물을 낸다.  
파일을 base64(인코딩 시 33% 정도 크기가 커지는, binary 값을 text로 표현할 수 있게 해주는 인코딩) URL로 변환하는 처리를 한다.

사용하는 이미지가 많은데, 각각의 이미지의 크기가 작다면 네트워크 리소스를 여러 번 사용하는 것 보다 [Data URI Scheme](https://en.wikipedia.org/wiki/Data_URI_scheme) 방식이 나을 수 있다.  
다만 번들의 크기가 커지므로 상황에 따라 각종 설정을 성능이 좋은 방향으로 바꿔야 한다.

설정 파일은 다음과 같이 사용할 수 있다.

```javascript
// webpack.config.js

module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif|svg)$/,
        loader: 'url-loader',
        options: {
          publicPath: './dist/',
          name: '[name].[ext]?[hash]',
          limit: 5000, // byte 단위. 5kb 미만의 파일은 data url로 처리. 이외의 옵션은 동일.
        },
      },
    ],
  },
}
```

빌드를 하면 다음과 같은 결과물을 낸다.  
중간에 `data:image/png;base64, ...`와 같은 형식을 볼 수 있다.

```javascript
// ./dist/main.js

'use strict'
eval(
  '__webpack_require__.r(__webpack_exports__);\n/* harmony default export */ __webpack_exports__["default"] = ("data:image/png;base64, (인코딩 내용은 생략) # sourceURL=webpack:///./src/images/times-circle.png?'
)
```

_cf) 로더의 또다른 역할_

로더는 파일에 대한 전처리도 가능하다.  
보통 TS를 JS로 변환할 때 많이 사용된다고 한다.

```javascript
// myloader.js

module.exports = function myWebpackLoader(content) {
  const newContent = _.cloneDeep(content)
  // 뭔가 하는 동작
  return newContent
}
```

```javascript
// webpack.config.js
const path = require('path')

module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.js$/,
        use: [path.resolve('./myloader.js')], // 커스텀 로더 적용.
      },
    ],
  },
}
```

#### 플러그인(Plugin)

로더는 모듈마다 실행한 것에 비해 플러그인은 번들에 대해서 1번만 실행한다.  
class 생성자 또는 function 생성자로 만들 수 있다.  
다음은 custom plugin 예시이다.

```javascript
// my-plugin.js
class MyPlugin {
  apply(compiler) {
    compiler.hooks.done.tap('My Plugin', (stats) => {
      console.log('\n12313123\n')
    })
  }
}

// webpack.config.js
module.exports = {
  // ...
  plugins: [new MyPlugin()],
}
```

다음은 자주 사용하는 플러그인이다.

##### BannerPlugin

번들링된 결과물 상단에 빌드 결과물 추가한다.

```javascript
// webpack.config.js
const webpack = require('webpack')

module.exports = {
  // ...
  plugins: [
    new webpack.BannerPlugin({
      banner: `
        Build Date: ${new Date().toLocaleString()} 
      `,
    }), // 빌드 타임 등 정보를 추가할 수 있음.
  ],
}
```

위와 같은 코드를 추가하면 빌드 결과물 상단에 `Build Date: ~~`가 추가된다.

##### DefinePlugin

빌드 타임에 결정되는 환경 변수를 어플리케이션에 주입한다.

```javascript
// webpack.config.js
const webpack = require('webpack')

module.exports = {
  // ...
  plugins: [
    new webpack.DefinePlugin({
      TWO: '1+1', // 2
      NOT_TWO: JSON.stringify('1 + 1'), // 1 + 1
    }),
  ],
}
```

아무런 인자도 지정하지 않으면 기본적으로 `NODE_ENV`만 들어가 있다.

##### HtmlTemplatePlugin

HTML 자체를 빌드 과정에 추가하여 HTML을 동적으로 만들 수 있다.

```javascript
// webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  // ...
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html', // html 파일 위치 명시
      templateParameters: {
        env: process.env.NODE_ENV === 'production' ? '' : '(개발용)', // html 파일 내부에서 ejs 문법으로 접근 가능한 파라미터
      },
      minify:
        process.env.NODE_ENV === 'production'
          ? {
              collapseWhitespace: true, // 빈칸 제거
              removeComments: true, // 주석 제거
            }
          : false,
      hash: true, // 정적 파일을 불러올때 쿼리문자열에 웹팩 해쉬값을 추가
    }),
  ],
}
```

##### CleanWebpackPlugin

빌드할 때마다 `dist`(또는 `output`) 폴더를 삭제한다.
`clean-webpack-plugin` 모듈을 설치하면 사용 가능하다.

##### MiniCssExtractPlugin

스타일 시트를 번들에서 별도로 분리한다.  
브라우저는 하나의 큰 파일을 내려받는 것보다 여러 개의 작은 파일을 동시에 내려받는 것이 빠르므로 번들이 커질 때 사용하기 좋은 플러그인이다.  
개발 환경에서는 모듈로서 처리해도 상관없지만 배포 환경에서는 분리하는 것이 일반적으로 더 효과적이다.

```javascript
// webpack.config.js
const MiniCssExtractPlugin = require('mini-css-extract-plugin')

module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          process.env.NODE_ENV === 'production'
            ? MiniCssExtractPlugin.loader // 프로덕션 환경
            : 'style-loader', // 개발 환경
          'css-loader',
        ],
      },
    ],
  },
  plugins: [
    ...(process.env.NODE_ENV === 'production'
      ? [new MiniCssExtractPlugin({ filename: `[name].css` })]
      : []), // 배포 환경일 때만 분리하기 위함. 스프레드 연산자 사용
  ],
}
```

3. **HMR(Hot Module Replacement)**

새로고침 없이 런타임에 브라우저의 모듈을 업데이트할 수 있는 기능이다. 개발할 때 코드를 저장하면 화면이 깜빡이면서 화면 전체가 reloading되는 것을 방지한다는 말이다. Webpack은 기본적으로 해당 옵션이 활성화된 `webpack-dev-server`(Webpack 자체 웹 서버)만 설치하면 되지만, RollUp과 Parcel을 별도의 dependency와 설정을 추가해주거나 특정 상황에서는 잘 동작하지 않는 경우를 보인다고 한다.

### 다른 모듈 번들러와 비교

_참고: https://betterprogramming.pub/the-battle-of-bundlers-6333a4e3eda9_

**Browserify**

commonJS 즉, Node.js와 똑같은 방식의 module bundler이다.  
Webpack은 모든 기능을 포함해 구현하고 최적화하는 반면 Browerify는 핵심적 요소만 충실히 구현하고 다른 기능이 필요할 땐 외부에서 구현한 모듈을 조합해 사용하므로 속도 측면에서 우수하다.

**RequireJS**

AMD API 명세를 구현한 구현체이다.  
[사용할 일은 없을 듯](https://d2.naver.com/helloworld/591319)

**RollUp**

- 최소한의 서드파티로 라이브러리를 만들고 싶을 때 가장 적합하다고 한다.

Webpack가 가장 큰 차이점이자 장점은 ES6 모듈 형식으로 빌드 결과물을 출력한다는 것이다. 로더가 ES6 모듈을 따르기 때문인데, 이를 라이브러리나 패키지 개발에 활용할 수 있다.  
이 때문에 code splitting에서 entry point가 달라서 중복해서 번들될 수 있는 부분을 알아내고 독립된 모듈로 분리할 수 있다.

**Parcel**

- 복잡한 설정을 피하고 비교적 간단한 애플리케이션을 만들 때 적합하다고 한다.

별도의 설정이 없이 동작 가능하다는 *zero config*가 가장 두드러진 특징이다. 설치만 하면 설정 파일 없이 빌드 명령어를 입력해서 사용할 수 있다. Webpack과 달리 JS 엔트리 포인트를 지정해주는 것이 아니라 애플리케이션 진입을 위한 HTML 파일을 자체적으로 읽을 수 있다.
RollUp, Webpack과 비교했을 때 Dead code elimination(==tree shaking) 면에서 가장 우수하다.  
Module transformation(JS 외의 파일을 만나면 dependency graph에 추가하고 bundle 작업하는 것)이 가장 똑똑하다. RollUp이나 Webpack은 파일 타입을 명시한 뒤 변환하고 설치하고 설정(`specify file types to transform, install and configure`)해야 하지만 Parcel은 built-in support가 된다.

### bundler 단점

_참고: https://yozm.wishket.com/magazine/detail/1261/_

번들러가 기존 모듈 방식의 문제점을 해결할 수 있었지만, 속도가 문제로 제기됐다.  
기존에는 그냥 js를 작성하면 바로 브라우저에서 실행할 수 있었지만, 이제 모든 파일을 하나로 만드는 작업이 선행되어야 한다.  
즉, 수정할 때마다 매번 새롭게 빌드가 필요했고 심지어 빌드 속도가 느렸다.  
이러한 문제를 해결하며 나온 것이 esbuild라는 빌드 도구가 나왔다.  
esbuild는 편의성 문제로 사용되지는 않았으나 2020년 snowpack이 나오면서, 그리고 조금더 개선한 vite가 나오면서 안정적으로 사용되고 있다.

~아직 사용해보지 않았으니 나중에 사용하게 되면 추가해보자~

### 궁금했던 점

**Q. ES6 문법은 자동으로 번들링 해주는건가?**

이게 무슨 궁금증인가 싶을텐데, React를 사용할 때 bundler 설정을 따로 해준 적이 없는데 자동으로 chunk된 JS들이 로딩되는 것을 크롬 network 탭에서 목격했다.  
[공식문서](https://reactjs.org/docs/create-a-new-react-app.html)에 따르면

> Under the hood, it(Create React App) uses Babel and webpack, but you don’t need to know anything about them.

이란다.  
자동으로 생성되는 것을 막거나 override 하고 싶다면 [여기](https://blog.devgenius.io/how-to-create-a-react-app-without-using-create-react-app-c004a62b52fc)나 [여기](https://marmelab.com/blog/2021/07/22/cra-webpack-no-eject.html)를 참고하면 될 듯 싶다.

아래 사진은 CRA로 생성했던 내 프로젝트의 build 산출물이다(따로 웹팩을 건든적이 없다)

![cra 프로젝트 build 산출물](https://user-images.githubusercontent.com/63287638/183119085-0ddc744b-66fb-4d94-9a45-94be6a548f3a.png)

## 3. DOM 관련 내용

### data-\* 속성

HTML5부터 추가된 개념으로 '사용자 지정 데이터 특성'이라는 특성 클래스를 형성할 수 있다.  
해당 특성을 이용해 page나 application에 private한 custom data를 저장하거나, HTML element에 custom data attirbute를 embed할 수 있다.  
이를 통해 BE에 Ajax 호출 없이 데이터를 보관할 수 있는 방법으로 사용할 수도 있다(다만 크롤링되어야 하거나 secret한 내용을 담기에는 적합하지 않은 속성이다).  
하나의 element에 여러 개의 data-\* 속성이 포함될 수 있다.

다음과 같은 두 가지 규칙을 지켜야 된다.

1. attribute 이름에 문자, 숫자, 대시(-), 점(.), 콜론(:), 언더스코어(\_)는 사용 가능하지만 대문자를 포함할 수 없다.
2. attribute 이름에 접두사 "data-" 뒤에 하나 이상의 문자가 있어야 한다.
3. attribute 이름이 대소문자 관계없이 'xml'로 시작하면 안 된다.
4. attribute 값은 어떠한 string도 가능하다.

```html
<ul>
  <li data-animal-type="bird">Owl</li>
  <li data-animal-type="fish">Salmon</li>
  <li data-animal-type="spider">Tarantula</li>
</ul>
```

html에서는 위와 같은 형식이지만 dataset은 JS이기 때문에 attribute명은 camelCase로 변환된다.  
즉, html과 dataset 간 `data-create-date -> (dateset) createDate`, `(dataset) dataset.monthSalary = '500 -> data-month-salary="500"`로 변환되는 것이다.

#### JS에서의 접근

JS에서 dataset을 get/set 하는 방법은 두 가지가 있다.  
`getAttribute`/`setAttribute`를 이용하는 방법과 DOM 객체의 property를 이용하는 방법이다.

```javascript
const div = document.getElementsByTagName('div')[0]
// const div = document.querySelector('div[data-auto="true"]')

div.setAttribute('data-auto', true)
console.log(div.dataset.auto) // typeof div.dataset.auto === 'string'

div.dataset.size = 'big'
console.log(div.getAttribute('data-size'))
```

#### CSS에서의 접근

위 예시에서 `querySelector`의 예시에서 나왔듯이 data-\* 속성은 html의 속성이기 때문에 css에서도 속성 선택자를 통해 선별할 수 있다.

```css
/* https://developer.mozilla.org/ko/docs/Learn/HTML/Howto/Use_data_attributes 예시 */

article[data-columns='3'] {
  width: 400px;
}
article[data-columns='4'] {
  width: 600px;
}
```
