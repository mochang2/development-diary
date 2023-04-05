## 0. 공부하게 된 이유

서로 다른 css 파일에 선언했지만 class name이 같아서 예상치 못한 스타일이 적용된 적이 있다.  
이를 피하기 위해 class name을 엄청 길게 지었던 기억이 있다.  
이 단점들을 극복하며 또 다른 장점도 갖고 있는 scss / sass를 알아보고 싶어졌다.

## 1. 개념

https://cocoon1787.tistory.com/843 를 참고했다.  
css는 복잡한 언어는 아니지만 작업이 크고 고도화 될수록 유지보수에 어려움이 생긴다.

- css(Cascading Styled Sheets): 종속형 시트
- sass(Syntactically Awesome Style Sheets): 문법적으로 훌륭한 스타일 시트
- scss(Sassy CSS): 문법적으로 멋진(sassy) css

해석에서 엿볼 수 있듯이 css를 보완하기 위해 나온 것이 sass와 scss이다.  
추가적으로, sass v3에서 scss가 생겨났다. 즉, scss가 sass보다 뒤에 나왔으며, scss가 좀 더 넓은 범용성과 css의 호환성을 가지고 있다.  
공식 레퍼런스에서도 sass보다 scss를 사용하기를 권유하고 있다.

### 예시

sass와 scss의 가장 큰 차이점은 들여 쓰기+줄 바꿈 형식이냐, 중괄호+세미콜론 형식이냐 이다.  
scss는 css와 비슷해 보이지만 [아래](https://github.com/mochang2/development-diary/edit/main/021-React%20css.md#sass-scss%EC%9D%98-%EA%B8%B0%EB%8A%A5)에 추가된 기능을 보면 다른 점을 알 수 있다.

```scss
// scss
.div-block {
  color: rgb(250, 222, 120);
  border-radius: 2px;
}
```

```sass
// sass
.div-block
  color: rgb(250, 222, 120)
  border-radius: 2px
```

## 2. 기능

sass와 scss에는 css에 없는 추가 기능이 있다.  
sass와 scss는 비슷하므로 아래에 코드 예제에서는 css와 scss를 비교하겠다.

1. 주석  
   css에서는 주석을 `/* */`로만 사용할 수 있었다.  
   scss에서는 한 줄 주석인 `//`을 지원한다.  
   다만 컴파일 했을 때 css 주석인 `/* */`은 남아있지만, 한 줄 주석인 `//`은 사라진다.

2. 변수 사용  
   자주 사용하는 색이나 폰트 등등을 변수로 지정해 재사용할 수 있다.

```css
/* css */
body {
  font-size: 16rem;
  font-family: Helvetica, sans-serif;
  color: #000;
}
```

```scss
// scss
$primary-font: Helvetica, sans-serif;
$primary-color: #000;

body {
  font-size: 16rem;
  font-family: $primary-font;
  color: $primary-color;
}
```

3. 중첩 구문(nesting)  
   c, c++, c#, java 등에서 사용하는 중첩 구문을 통해 코드의 양을 줄이고 가독성을 높일 수 있다.  
   nesting을 위해 scss에는 본인은 지정하는 선택자(selector) `&`이 존재한다.

```css
/* css */
nav ul {
  list-style: none;
  /* ... */
}
nav li {
  color: #000;
  /* .... */
}
nav li:last-child {
  text-decoration: underline;
  /* ..... */
}
```

```scss
// scss
nav {
  ul {
    list-style: none;
    /* ... */
  }
  li {
    color: #000;
    /* .... */

    &:last-child {
      text-decoration: underline;
      /* ..... */
    }
  }
}
```

4. 모듈화  
   `@use`나 `@import`를 사용하여 파일을 기능별로 분할하고 다른 파일에 선언된 내용을 가져다 쓸 수 있다.

```css
/* css */
body {
  font-size: 16rem;
  font-family: Helvetica, sans-serif;
  color: #000;
}

input {
  backgroud-color: #fff;
  padding: 4%;
}
```

```scss
// scss
// _main.scss
$primary-font: Helvetica, sans-serif;
$primary-color: #000;

body {
  font-size: 16rem;
  font-family: $primary-font;
  color: $primary-color;
}

// _component.scss
@use 'main';

.mail-box {
  font-size: 100%;
  font-family: main.$primary-font;
}
```

5. 믹스인(mixins)  
   재사용성에 도움이 되는 속성이다.  
   함수처럼 default parameter를 지정할 수 있고, parameter를 받아서 속성을 부여할 수 있다.

```scss
@mixin center($size: 12px, $color: #000) {
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: $size;
    color: $color;
    @content;
}

.container {
    @include center($color: green);
    .child {
        @include center(16px) {
            position: relative;
            top: 0;
        }
    }
}
```

위 SCSS 코드는 아래와 같이 컴파일 됨.

```css
.container {
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 12px;
    color: green;
}

.container .child {
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 16px;
    color: #000;
    position: relative;
    top: 0;
}
```

6. 확장 및 상속
   사용법은 mixin과 약간 차이가 있지만 원하는 바는 비슷하다.  
   코드의 양을 줄이고 재사용성을 높이고자 할 때 사용할 수 있는 기능이다.

```css
/* css */
.message,
.success,
.error,
.warning {
  border: 1px solid #ccc;
  padding: 10px;
  color: #333;
}

.success {
  border-color: green;
}

.error {
  border-color: red;
}

.warning {
  border-color: yellow;
}
```

```scss
// scss
/* This CSS will print because %message-shared is extended. */
%message-shared {
  border: 1px solid #ccc;
  padding: 10px;
  color: #333;
}

/* This CSS won't print because %equal-heights is never extended. */
%equal-heights {
  display: flex;
  flex-wrap: wrap;
}

.success {
  @extend %message-shared;
  border-color: green;
}

.error {
  @extend %message-shared;
  border-color: red;
}

.warning {
  @extend %message-shared;
  border-color: yellow;
}
```

<hr />

_아래에서부터는 scss에 대한 예제만 존재한다._

7. 연산자  
   div, sin, cos, tan, random, max, min 등 일반적인 프로그래밍 언어에서 사용할 수 있는 수학 모듈을 사용할 수 있다.

```scss
// scss
@use 'sass:math';

span {
  width: math.div(100px, 200px) * 100%;
}
```

8. 조건문

```scss
// if(함수)
$default-width: 400px;

.info {
  width: $default-width;
  background-color: if($default-width > 500, #fff, #5f5f5f);
}

// @if(조건문)
$default-width: 400px;

//공통 CSS

.info {
  width: $default-width;
  @if ($default-width > 500) {
    // ()은 생략 가능, 조건식 안에 and, or, not 등을 사용할 수 있음
    background-color: coral;
  } @else {
    background-color: orange;
  }
}
```

9. 반복문  
   일전에 반응형 웹을 만들기 위해 `@media screen and (max-width: xx px)`을 수십 번씩 사용했는데, 훨씬 간편한 방법으로 만들 수 있는 기능이다.

```scss
// @for(반복문)
// through: 시작부터 종료조건까지 반복
@for $변수 from 시작 through 종료 {
  /* 반복내용 */
}
//1~3까지 반복
@for $i from 1 through 3 {
  .info:nth-child(3n + #{$i}) {
    width: 50px * $i;
  }
}

// to: 시작부터 종료조건 직전까지 반복
@for $변수 from 시작 to 종료 {
  /* 반복내용 */
}
//1~2까지 반복
@for $i from 1 to 3 {
  .info:nth-child(3n + #{$i}) {
    width: 50px * $i;
  }
}

// @while 은 @if(조건문)과 문법이 동일하므로 생략.

// @each(js의 map과 유사)
.list-fruits {
  @each $fruit in apple, banana, mango, orange {
    li.#{$fruit} {
      background: url('../img/#{fruit}.png');
    }
  }
}
// 또는 리스트를 변수로 저장해서 사용 가능
$fruits: apple, banana, mango, orange;
.list_fruits {
  @each $fruit in $fruits {
    li.#{$fruit} {
      background: url('../img/#{fruit}.png');
    }
  }
}
```

## 3. 컴파일

scss를 사용하면서 느꼈던 불편함 중에 하나는 스타일 반영 속도가 느리다는 것이다.  
처음에는 렌더링에 의한 문제인 줄 알고 최적화 방법이 있나 찾아봤지만, 컴파일에 의한 문제였다.

sass(scss)는 css pre-processor(전처리기)라고도 불리는 css 스크립팅 확장 언어이기 때문에  
컴파일러를 통해서 일반 css 문법으로 바꾼 뒤 적용하는 과정이 필요하다.

순서도로 표현하자면 다음과 같다.  
![image](https://user-images.githubusercontent.com/63287638/172085533-fbca2fd4-9f2c-4cf2-b050-a5b70d36f008.png)

_cf) sass(scss) 컴파일 방식에 대해서 궁금하다면 [Ruby를 통한 scss 컴파일 방법](https://www.biew.co.kr/entry/Sass%E3%86%8DSCSS-%EC%86%94%EC%A7%81%ED%95%9C-%EC%9E%A5%E3%86%8D%EB%8B%A8%EC%A0%90-%EC%86%8C%EA%B0%9C-%EB%B0%8F-%EC%84%A4%EC%B9%98%EB%B0%A9%EB%B2%95)을 참고하면 도움이 될 것 같다._

## 4. React에서 scss 사용

1. `npm install --dev sass` 또는 `yarn add -D sass`을 실행한다.  
   `node-sass`는 scss를 다룰 수 있는 node program인데 현재는 `sass`에 있는 dart-sass라는 패키지 사용이 권장된다.

2. .css 파일을 .scss로 파일명 변경한다.

이렇게 해도 scss가 적용이 되지 않는다면(프레임워크를 사용한다면 이미 아래는 적용되어 있을 가능성이 있다) webpack을 변경해야 한다.  
자세한 설정은 [webpack 공식 사이트](https://webpack.js.org/loaders/sass-loader/)를 보면 알 수 있다.

3. `npm install --dev sass-loader` 또는 `yarn add -D sass-loader`을 실행한다.  
   `sass-loader`는 webpack에 필요한 loader이다.

4. `webpack.config.js`를 수정한다.

```javascript
module.exports = {
module: {
    rules: [{
      ...
    }, {
      test: /\.s[ac]ss$/i, // 여기 부분
      use: [
        "style-loader",
        "css-loader",
        {
          loader: "sass-loader",
          options: {
            // Prefer `dart-sass`
            implementation: require("sass"),
          },
        },
      ]
    }]
  },
}
```

fibers라는 패키지를 설치하면 Dart Sass의 컴파일 속도를 2배로 올릴 수 있다.  
node는 기본적으로 비동기 방식을 사용하지만 동기방식으로 전환하는 방식으로 동작한다.  
webpack은 기본적으로 이 패키지가 설치된 것을 전제로 동작하기 때문에 특별한 설정은 필요없다.  
만일 이 패키지가 동작하는 것을 원하지 않는다면 아래처럼 수정한다.

```javascript
// ...
        {
          loader: "sass-loader",
          options: {
            implementation: require("sass"),
            sassOptions: {
              fiber: false,
            },
          },
        },
// ...
```

5. 만약 production 버전과 dev 버전을 구분해서 사용하고 있다면, build할 때 scss가 적용이 되지 않을 수도 있다.  
   이떄는 [여기](https://medium.com/@jsh901220/react%EC%97%90-scss-%EC%A0%81%EC%9A%A9%EB%B0%8F-%EA%B8%B0%EB%B3%B8-%EC%82%AC%EC%9A%A9%EB%B2%95-1-c7bd2895f5a6)를 참고해서 적용하면 된다.
