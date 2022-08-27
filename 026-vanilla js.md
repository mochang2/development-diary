## 0. 공부하게 된 계기

React? Vue? 아무리 개발을 해봐도 근본을 모르면 말짱 도루묵인 것 같다.  
~과제 테스트 통과 못하는 건 덤~  
우테캠으로 과제 전형을 보고, vanilla js로 [virtual DOM을 비슷하게 따라 만드는 강의](https://junilhwang.github.io/TIL/Javascript/Design/Vanilla-JS-Virtual-DOM/)가 있다는 것을 알고 공부해야겠다 마음을 먹었다.

## 1. babel

ES6에서 JS 표준 모듈 문법이 정의되었음에도 ES6 문법을 구형 브라우저에서 사용하지 못해 SystemJS 같은 또 다른 라이브러리에 의존했어야 함.  
트랜스파일러, 즉 한 번 컴파일하면 구형 브라우저에서도 동작하는 JS 코드가 나오게 만드는 도구가 필요했음.

js 컴파일러. 소스 코드를 웹 브라우저가 처리할 수 있는 JS 버전으로 변환. 새로운 ESNext 문법을 사용하여 개발할 때, 지원하지 않는 브라우저에서 오류가 나지 않도록 하기 위해.
.babelrc

- polyfill
  => 개발자가 특정 기능이 지원되지 않는 브라우저를 위해 사용할 수 있는 코드 조각이나 플러그인.

### 궁금했던 점

**Q. React.createchild vs jsx**

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
// 이후 `ReactDOM.render(변수명, 배치할 곳)`을 통해 화면에 렌더링 됨
```

이 모든 과정을 도와주는 것이 babel이다.  
그리고 아래와 같이 컴포넌트 내부에서 `React`를 사용하지 않는데도 굳이 import해야 하는 이유가 jsx 변환을 위해서이다.

```jsx
import React from 'react'

const Box = () => {
  return <div>i'm box</div>
}
```

## 2. Webpack

Webpack은 module bundler이다.

### module

_참고: [js module](https://blog.bitsrc.io/javascript-require-vs-import-47827a361b77)_

Webpack을 온전히 이해하기 위해서는 JS에서 module이 무엇인지 먼저 알 필요가 있다.  
참고로 JS는 모듈이 없는 상태로 세상에 나타났고(해봐야 `<script>` 태그 이용), 모듈 개념은 node.js가 만들어지고 서버에서 JS를 사용할 수 있게 되면서 나온 개념이다.  
**module**이란 프로그램을 구성하는 내부의 코드가 기능별로 나뉘어져 있는 형태로 다음과 같은 장점이 있다.

- 유지보수성: 기능들이 모듈화가 잘 되어 있다면, 의존성을 그만큼 줄일 수 있기 때문에 어떤 기능을 개선하거나 수정할 때 훨씬 편하다.
- 네임스페이스: 모듈로 분리하여 모듈만의 네임스페이스를 가지게 할 수 있다.
- 재사용성: 똑같은 코드를 반복하지 않고 모듈을 분리시켜서 필요할 때마다 사용할 수 있다.

JS에서 모듈의 개념이 나오기 전에는 아래와 같은 방식으로 사용했다.

```html
<script src="jquery.js"></script>
<script src="tweenmax.js"></script>
<!-- 그걸 사용해 내 코드 작성-->
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
HTML에서 JS코드를 사용할 때 이 문법을 사용하면 `<script type="module" src="index.mjs">` 식으로 가져올 수 있다.  
`import`와 `export`라는 키워드를 사용한다.  
가장 큰 장점은 모듈을 비동기적으로 불러오며 빌드 타임에 정적 분석이 가능하여 tree shaking이 쉽다.  
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
_모듈 로더(JS 모듈을 런타임에 로드할 수 있게 하는 구현체로 RequireJS나 네이티브 브라우저 등이 포함)와 유사한 부분이 있지만, 모듈 번들러(컴파일 시간에 빌드 산출물을 만들어서 하나의 js파일을 산축)는 코드를 프로덕션 환경에서 사용할 수 있도록 준비하는 데 더 큰 목적이 있음._

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

## 3. Node.js

기존의 JS는 HTML에 독립적으로 실행할 수 없는 프로그래밍 언어였다(`<script>` 태그 사용했어야 함).  
하지만 Node.js가 등장하면서 JS를 HTML에 독립적으로 실행할 수 있게 됐다.  
물론 그 이전에도 JS를 HTML에 독립적으로 실행할 수 있도록 만드는 시도가 있었으나 엔진(Chrome V8 JavaScript Engine 이전) 속도 문제로 실패했었다.  
[Node.js 공식 사이트](https://nodejs.org/)에 따르면 Node.js는

> Node.js는 Chrome V8 JavaScript 엔진으로 빌드된 JavaScript 런타임입니다.\
> Node.js는 이벤트 기반, Non-blocking I/O 모델을 사용해 가볍고 효율적입니다.\
> 이는 (I/O를 직접 수행하지 않으므로) 사용자가 프로세스의 교착상태에 대해 걱정할 필요가 없다는 말이기도 합니다.\
> Node.js의 패키지 생태계인 npm은 세계에서 가장 큰 오픈 소스 라이브러리 생태계이기도 합니다.\
> Node.js는 이벤트 루프를 시작하는 호출이 없으며, 각 연결에서 콜백이 실행되는데 실행할 작업이 없다면 Node.js는 대기합니다.
> Node.js는 스레드를 사용하지 않도록 설계되었지만 멀티 코어 환경의 장점을 얻지 못하는 것은 아닙니다. 'child_process.fork()' API를 사용해 자식 프로세스를 생성할 수 있습니다.

_cf) 런타임: 특정 언어로 만든 프로그램들을 실행할 수 있게 해주는 가상 머신의 상태. 다른 런타임으로는 웹 브라우저(크롬, 사파리 등)이 있음._

공식 사이트의 설명대로 Node.js는 서버의 역할도 수행할 수 있는 JS 런타임이다.  
Node.js는 서버 실행을 위해 필요한 http/https/http2 모듈을 제공한다.

이러한 특성 때문에 Node.js는 다음과 같은 개발에 용이하다.

- 정적 파일 서버
- 웹 응용프로그래밍
- 메시지 미들웨어
- HTML5 멀티 플레이어 게임용 서버

### 1) NVM(Node Version Manager)

Node.js의 버전을 관리하는 도구이다.  
협업, 여러 가지 프로젝트를 동시에 진행해야 할 때와 라이브러리 버전 호환 문제에 유용하게 사용할 수 있다.  
파이썬으로 개발할 때 virtualenv를 사용하는 이유와 같다.

Node.js를 PC에 설치하면 `node -v`로 버전을 확인할 수 있다.  
지금 사용중인 Node.js 버전이 필요한 버전과 맞지 않으면 [NVM Github](https://github.com/coreybutler/nvm-windows/)에 가서 설치할 수 있다.

이후 `nvm install <버전>`으로 원하는 버전을 설치할 수 있다.  
`nvm list`를 입력하면 설치되어 사용할 수 있는 Node.js 버전이 조회되며  
`nvm use <버전>`으로 사용할 버전을 선택할 수 있다.  
마지막으로 다시 `node -v`를 입력하면 버전이 바뀐 것을 확인할 수 있다.

### 2) NPM(Node Pacakge Module)

세상에서 가장 큰 소프트웨어 레포지토리이다.  
jQuery, react, vue 등의 다양한 오픈 소스들도 올라가 있고, 사설 모듈을 올려서 회사 내에서만 사용할 수도 있다.

[NPM 공식문서](https://docs.npmjs.com/about-npm)에 따르면 다음과 같은 특징이 있다.

> Use the website to discover packages, set up profiles, and manage other aspects of your npm experience. For example, you can set up organizations to manage access to public or private packages.  
> The CLI runs from a terminal, and is how most developers interact with npm.  
> The registry is a large public database of JavaScript software and the meta-information surrounding it.

### `yarn`

_참고: https://developer0809.tistory.com/128_

[위키피디아](<https://ko.wikipedia.org/wiki/Yarn_(%ED%8C%A8%ED%82%A4%EC%A7%80_%EA%B4%80%EB%A6%AC%EC%9E%90)>)에 따르면 yarn은 Node.js 런타임 환경을 위해 페이스북이 개발한 소프트웨어 패키지 시스템이다.

다음은 npm과 비교했을 때의 yarn의 장점이 있다.

- yarn은 로컬 캐시로부터 패키지를 설치할 수 있다.
- yarn은 데이터 무결성 보장을 위해 체크섬을 사용하는 반면 npm은 SHA-512를 사용하여 다운로드된 패키지의 데이터 무결성을 검사한다.
- yarn은 병렬로 패키지를 설치하는 반면, npm은 순차적으로 설치해서 일반적으로 yarn의 다운로드 속도가 빠르다.
- yarn은 yarn.lock 또는 package.json 파일에 있는 파일만 설치한다. 반면에 npm은 다른 패키지를 즉시 포함시킬 수 있는 코드를 자동으로 실행하므로, 보안 시스템에 여러 가지 취약성이 발생한다. 따라서 yarn이 npm보다 보안이 강화된 것으로 간주된다.

_cf) 명령어_
동일한 명령어: `npm(yarn) init`, `npm(yarn) run`, `npm(yarn) test`, `npm(yarn) login`, `npm(yarn) logout`, `npm(yarn) link`, `npm(yarn) publish`, `npm(yarn) cache clean`

다른 명령어:

| npm                                  | yarn                           |
| ------------------------------------ | ------------------------------ |
| npm install                          | yarn(yarn install)             |
| npm install \<package\>              | yarn add \<package\>           |
| npm install --save-dev \<package\>   | yarn add --dev \<package\>     |
| npm uninstall \<package\>            | yarn remove \<package\>        |
| npm uninstall --save-dev \<package\> | yarn remove \<package\>        |
| npm update                           | yarn upgrade                   |
| npm update \<package\>               | yarn upgrade \<package\>       |
| npm install --global \<package\>     | yarn global add \<package\>    |
| npm uninstall --global \<package\>   | yarn global remove \<package\> |

### `package.json`

npm을 통해 설치된 패키지 목록을 관리하고 프로젝트의 정보 및 기타 실행 스크립트를 작성하는 파일이다.  
프로젝트를 처음 시작할 때, `npm init` 명령어를 통해 설치할 수도 있고, 직접 editor에서 생성해서 작성할 수도 있다.  
이후 `npm install` 또는 `npm install package.json` 또는 `yarn` 또는 `yarn install` 등으로 관련 모듈들을 설치할 수 있다.  
package.json은 다음과 같이 구성되어 있다.

```json
{
  "name": "project",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "axios": "^0.27.2",
    "cookie-parser": "~1.4.4",
    "react": "0.10.0"
  },
  "devDependencies": {
    "supertest": "^4.0.2"
  }
}
```

devDependencies는 개발할 때만 사용되고 배포할 때는 포함되지 않는 패키지들을 적는 곳으로 `-D` 옵션을 추가해서 설치한다.

참고로 모듈 버전은 일반적으로 "x.x.x"로 구성되는데, 첫 번째 자리 수는 대대적인 변화가 있을 경우, 두 번째 자리 수는 버그 픽스 등 적당한 변화가 있을 경우, 세 번째 자리 수는 아주 미묘한 변화가 있을 경우 1씩 올려서 사용한다.  
모듈 버전을 관리할 때 "x.x.x"는 세 자리 수 모두 똑같은 버전을 설치하라는 의미이다.  
`~`는 두 번째 자리 수까지 똑같은 버전을 설치하라는 의미이다. 예를 들어 `~0.0.1`은 `>=0.0.1 <0.1.0`을 의미한다.  
`^`은 첫 번째 자리 수까지 똑같은 버전을 설치하라는 의미이다. 예를 들어 `^1.0.2`은 `>=1.0.2 <2.0`을 의미한다.

### `package.json` vs `package-lock.json`(`yarn.lock`)

_참고: https://velog.io/@songyouhyun/Package.json%EA%B3%BC-Package-lock.json%EC%9D%98-%EC%B0%A8%EC%9D%B4 , https://hyunjun19.github.io/2018/03/23/package-lock-why-need/_

[geeks for geeks](https://www.geeksforgeeks.org/difference-between-package-json-and-package-lock-json-files/)에 나온 내용을 먼저 표로 정리해보면 다음과 같다.

| package.json                                                                         | pacakge-lock.json                                                                                                                             |
| ------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------- |
| It contains basic information about the project.                                     | It describes the exact tree that was generated to allow subsequent installs to have the identical tree.                                       |
| It is mandatory for every project.                                                   | It is automatically generated for those operations where npm modifies either node_modules tree or package.json(install or up 등이 발생할 때). |
| It records important metadata about the project.                                     | It allows future devs to install the same dependencies in the project.                                                                        |
| It contains information such as name, description, author, script, and dependencies. | It contains the name, dependencies, and locked version of the project.                                                                        |

위 표에서 나온 것처럼 package-lock.json은 모듈의 '정확한 버전 정보'를 가지고 있기 때문에 package-lock.json이 존재하면 `npm install(yarn)`은 package.json을 이용해서 node_modules를 생성하지 않고 package-lock.json을 이용해서 node_modules를 생성한다.

같은 `package.json`을 사용해도 서로 다른 `node_modules`를 생성하는 경우가 발생합니다.

1. npm 버전이 다를 때
2. 버전을 명시하지 않고 version range를 사용할 때
3. 내가 사용하는 패키지를 의존하고 있는 패키지가 새로운 버전으로 배포되었을 때

와 같은 경우입니다

2번과 같은 문제를 해결하기 위해 처음부터 정확한 정보를 package.json에 명시하면 되지 않겠냐고 생각하겠지만,  
`package-lock.json`을 별도로 사용하면 프로젝트 패키지의 중요한 버그 수정이 이루어질 때마다 프로젝트의 `package.json`에 적혀 있는 버전을 항상 수정하지 않고 version range(^ 또는 `)로 해결할 수 있다.

### `node_modules/.package-lock.json`

`npm install`로 dependencies에 있는 패키지들을 설치하니 `.package-lock.json`이라는 파일이 있었다.  
[공식문서](https://docs.npmjs.com/cli/v8/configuring-npm/package-lock-json)를 확인해보니 `node_modules` 폴더를 반복적으로 처리하지 않기 위해 v7부터 추가된 '숨겨진' lock 파일이라고 한다.

이 파일은 트리에 대한 정보를 포함하며, 다음 조건이 충족될 경우 `node_modules` 계층을 읽는 대신에 사용된다.

- 참조하는 모든 패키지 폴더가 `node_modules` 계층에 있고,
- `node_modules` 계층 구조에 lock 파일이 추적하지 않는 패키지 폴더가 없으며,
- 파일의 수정 시간이 최신이어야 한다. 즉, 이 숨겨진 lock 파일은 패키지 트리의 최신 업데이트의 일부로 생성된 경우에만 관련이 있다.

세 번재 조건에 대한 부연 설명이다.  
직접 수동으로 `node_modules/foo/lib/bar.js`라는 파일을 추가하면 `node_modules/foo`의 수정된 시간이 이 변경 사항을 반영하지 않으므로 `node_modules/.package-lock.json`에서 파일을 삭제해야 된다는 말이다.

## 4. DOM

### innerHTML

참고: https://velog.io/@1106laura/insertAdjacentHTML

DOM 요소 내부의 내용을 변경하는데 사용된다.

`innerHTML`은 요소 내에 포함된 HTML을 가져오고, 문자열처럼 '='이나 '+=' 연산자로 해당 내용을 추가하거나 변경할 수 있도록 해준다.  
`innerHTML`이 실행되면 해당 요소 내의 DOM Tree가 초기화되고 대입해준 값으로 대체된다.

아래와 같은 방법으로 사용된다.

```javascript
const body = document.querySelector('body')
body.innerHTML = ''
```

다만 *변경*이 아닌 *삽입*만 필요한 상황이라면 아래의 `insertAdjacentHTML`이 더 효율적이다.

```javascript
const foodArray = ['김밥', '방어', '사과', '오렌지']
const FOOD_TEMPLATE = (food) => '<div class="list_food">' + food + '</div>'
foodArray.forEach((food) => (body.innerHTML += FOOD_TEMPLATE(food)))
```

위 코드를 실행되면 아래와 같은 html 결과가 생성될 것이다.

```html
<div class="list_food">김밥</div>
<div class="list_food">방어</div>
<div class="list_food">사과</div>
<div class="list_food">오렌지</div>
```

다만 이는 효율적인 코드가 아니다.  
아래와 같은 연산이 반복되어 일어나며 반복적으로 리렌더링 되기 때문이다.

```javascript
body.innerHTML = '<div class="list_food">김밥</div>'
body.innerHTML =
  '<div class="list_food">김밥</div><div class="list_food">방어</div>'
body.innerHTML =
  '<div class="list_food">김밥</div><div class="list_food">방어</div><div class="list_food">사과</div>'
body.innerHTML =
  '<div class="list_food">김밥</div><div class="list_food">방어</div><div class="list_food">사과</div><div class="list_food">오렌지</div>'
```

위 방법을 보완하기 위해 `forEach` 대신 `reduce`를 사용할 수 있지만 그럼에도 더 빠른 아래 방법이 있다.

### insertAdjacentHTML

DOM 요소를 삽입할 때 쓰인다.  
[MDN](https://developer.mozilla.org/ko/docs/Web/API/Element/insertAdjacentHTML)과 https://dev.to/jeannienguyen/insertadjacenthtml-vs-innerhtml-4epd 에 따르면

> `insertAdjacentHTML` 메서드는 HTML or XML 같은 특정 텍스트를 파싱하고, 특정 위치에 DOM tree 안에 원하는 node들을 추가 한다. 이미 사용중인 element 는 다시 파싱하지 않는다. 그러므로 element 안에 존재하는 element를 건드리지 않는다. `innerHtml`보다 작업이 덜 드므로 빠르다.

> `insertAdjacentHTML` 메서드는 호출된 element를 reparse하지 않으므로 요소를 손상시키지 않는다. `insertAdjacentHTML`는 element를 연속적으로 serialize하고 reparse하지 않기 때문에 콘텐츠가 많을 때마다 추가 속도가 느려지는 `innerHtml`보다 훨씬 빠르다.

`$element.insertAdjacentHTML(position, text)`와 같은 방식으로 사용된다.
position은 `beforebegin`, `afterbegin`, `beforeend`, `afterend`만 가능하다.
text는 HTML 또는 XML 형태의 문자열을 의미한다.

```html
<!-- beforebegin 형제 요소로, 앞에 -->
<body>
  <!-- afterbegin 자식 요소로, 맨앞에 -->
  <!-- insert here something -->
  <!-- beforeend 자식 요소로, 맨뒤에-->
</body>
<!-- afterend 형제 요소로 뒤에-->
```

위 body 안에 무언가 element를 삽입하고 싶다면 아래와 같이 사용하면 된다.

```javascript
const foodArray = ['김밥', '방어', '사과', '오렌지']
const FOOD_TEMPLATE = (food) => '<div class="list_food">' + food + '</div>'
foodArray.forEach((food) =>
  body.insertAdjacentHTML('afterbegin', FOOD_TEMPLATE(food))
) // 또는 'beforeend'
```

### cloneNode

`Node.cloneNode()` 메서드는 이 메서드를 호출한 Node의 복제된 Node를 반환한다.

`const dupNode = node.cloneNode(option)`과 같은 문법으로 사용된다.

- node: 복제되어야 할 node
- dupNode: 복제된 새로운 node
- option?: node의 children까지 복제할지, 해당 node만 복제할지 여부. default: false

```javascript
const test = document.getElementById('cloneTest')
// test 변수에 복제 할 노드를 지정

const testClone1 = test.cloneNode()
const testClone2 = test.cloneNode()
const testClone3 = test.cloneNode()
// 복사할 개수만큼 복제변수를 생성

body.appendChild(testClone1)
body.appendChild(testClone1)
body.appendChild(testClone1)
```

이 메서드를 사용할 때 주의할 점은 duplicated element ID를 생성할 수 있다는 점이므로, `id` property를 부여한 node라면 지양하는 게 좋을 거 같다.
