## 0. 공부하게 된 계기

[motion](https://github.com/mochang2/motion) (노션 클린 코딩 프로젝트)을 진행하기 전 컨벤션을 정하다가 CSS에도 컨벤션이 존재한다는 것을 알았다.  
가장 많이 사용되는 OOCSS, BEM, SMACSS를 정리한 뒤 프로젝트에 사용할 컨벤션을 정하고자 공부를 시작했다.

참고 및 사진 출처  
https://github.com/stubbornella/oocss/wiki  
https://www.smashingmagazine.com/2011/12/an-introduction-to-object-oriented-css-oocss/  
https://jmjmjm.tistory.com/44  
https://dev.to/nouran96/oocss-methodology-92d  
https://wit.nts-corp.com/2015/04/16/3538
https://blog.illunex.com/css-%EB%B0%A9%EB%B2%95%EB%A1%A0-oocss/  
http://smacss.com/  
https://getbem.com/introduction/  
https://en.bem.info/methodology/  
https://nykim.work/15

## 1. 공통점

OOCSS, BEM, SMACSS는 같은 지향점을 가지고 있다.

- 코드의 재사용성을 높이기.
- 쉬운 유지보수.
- 확장 가능성.
- class 이름만으로도 무슨 의미인지 예측 가능하도록 하기.

## 2. OOCSS(Object Oriented CSS)

약어에서 알 수 있듯이 CSS를 모듈 방식으로 작성하여 중복을 줄이는 방법론이다.  
CSS object는 추상화될 수 있는 반복되는 패턴을 의미한다.

OOCSS의 주요 개념은 다음 두 가지이다.

- separate structure and skin
- separate container and content

### Separate Structure and Skin

structure은 `width`, `height`, `margin`, `padding` 등 invisible style을 의미한다.  
skin은 `color`, `font` 등 visible style을 의미한다.

한 웹 사이트에서는 대체적으로 한, 두 가지의 색으로 통일하다 보면 다음과 같이 똑같은 skin 스타일 코드가 반복적으로 사용되는 상황이 생기곤 한다.

```css
#button {
  width: 200px;
  height: 50px;
  padding: 10px;
  border: solid 1px #ccc;
  background: linear-gradient(#ccc, #222);
  box-shadow: rgba(0, 0, 0, 0.5) 2px 2px 5px;
}

#box {
  width: 400px;
  overflow: hidden;
  border: solid 1px #ccc;
  background: linear-gradient(#ccc, #222);
  box-shadow: rgba(0, 0, 0, 0.5) 2px 2px 5px;
}

#widget {
  width: 500px;
  min-height: 200px;
  overflow: auto;
  border: solid 1px #ccc;
  background: linear-gradient(#ccc, #222);
  box-shadow: rgba(0, 0, 0, 0.5) 2px 2px 5px;
}
```

이러한 코드가 존재할 때 다음과 같이 바꿔준다.

```css
.button {
  width: 200px;
  height: 50px;
}

.box {
  width: 400px;
  overflow: hidden;
}

.widget {
  width: 500px;
  min-height: 200px;
  overflow: auto;
}

.skin {
  border: solid 1px #ccc;
  background: linear-gradient(#ccc, #222);
  box-shadow: rgba(0, 0, 0, 0.5) 2px 2px 5px;
}
```

더 적은 코드로 이루어지고 재사용성이 늘어난다.

(다른 예시)HTML에서는 다음과 같이 사용할 수도 있다.

```html
<a href="#" class="btn-base cart">장바구니</a>
<a href="#" class="btn-base buy">구매</a>
```

### Separate Container and Content

container는 무엇인가를 포함한 것, 그리고 content는 이미지, 문단, 박스 등 container 안에 포함되어 있는 요소들을 이야기한다.  
CSS의 space selector나 '>' selector 보다는 content 스타일을 container로부터 분리하는 것을 추구한다.

```css
#sidebar {
  /*...*/
}

#sidebar .list {
  /*...*/
}

#sidebar .list .list-header {
  /*...*/
}

#sidebar .list .list-body {
  /*...*/
}
```

위 코드를 아래와 같이 바꿀 수 있다.

```css
.sidebar {
  /*...*/
}

.list {
  /*...*/
}

.list-header {
  /*...*/
}

.list-body {
  /*...*/
}
```

### 장단점

- 장점
  - CSS 코드 양을 줄일 수 있음. 따라서 브라우저 렌더링 시 스타일 시트 다운로드 속도를 증가시킴으로써 성능 향상을 기대할 수 있음.
  - 렌더링 엔진이 이미 계산된 값을 사용하려기 때문에(캐싱 등) CSS rule이 재사용될수록 성능 향상을 기대할 수 있음.
  - 재사용이 가능하므로 유지 보수성이 높음.
- 단점
  - HTML 코드가 복잡하여 가독성이 떨어질 수 있음.
  - OOP가 그러하듯, 크기가 작은 프로젝트에서 사용한다면 overkill일 수 있음.
  - 공통된 클래스가 많기 때문에 멀티 클래스(`<div class="aa bb cc">`와 같은 것)를 자주 사용하는데, 멀티 클래스가 많아질수록 유지 보수가 오히려 힘들 수 있음.

### 가이드라인

- space나 '>' 등과 같은 descendent selector 사용을 피하라.
- id 속성을 사용하지 말라는 의미는 아니다. 다만 id는 너무 강력하기 때문에 스타일을 위해서 사용한다기 보다, linking이나 js hook을 위해서 사용하라.
- `div.header` 등 HTML tag를 selector 앞에 추가하는 것을 자제하라.
- `!important` 사용을 자제하라.
- css grid를 사용하라.
- content는 '%' 등이 아닌 'px'로 크기를 지정하라.

### base classes

[wiki](https://github.com/stubbornella/oocss/wiki)에서 사용하는 base classess 들의 용도를 정리하려고 추가했다.

| property                                       | description                                                                                          |
| ---------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| `media`                                        | wrapper for the media object.                                                                        |
| `img`, `imgExt`                                | media object의 child node. 보통 link, image, flash의 wrapper로 사용됨.                               |
| `bd`                                           | media object에서 가장 중요. 반드시 필요하고 안에 무엇이든 들어갈 수 있음.                            |
| `data`                                         | wrapper for the data table                                                                           |
| `txtL`, `txtR`, `txtC`, `txtT`, `txtM`, `txtB` | `<table>`, `<tr>`, `<td>`, `<th>` 등에 적용할 수 있는 속성. 각각 좌, 우, 중간, 위, 중간, 아래를 의미 |
| `line`                                         | 가로 줄, 행                                                                                          |
| `unit`                                         | `line`은 한 section(column)으로 나눈 단위(base class).                                               |
| `sizeXofY`                                     | `unit`의 width를 분수로 표현한 것. extends `unit`.                                                   |
| `lastUnit`                                     | 각 line의 마지막 child에게 적용되는 속성. extends `unit`.                                            |
| `mod`                                          | 모든 container 모듈의 base class. 안에 무엇이든 들어갈 수 있음. 내부가 transparent함.                |
| `complex`                                      | 내부가 transparent함. `mod`나 `inner`만으로 구분짓기 어려운 내부를 구현. extends `mod`.              |
| `pop`                                          | backdrop같은 것처럼 외부가 transparent함. extends `mod`.                                             |
| `tl`, `tr`, `bl`, `br`                         | 블록의 왼쪽 상단 등 모서리를 복합적으로 표시하는 데 사용되는 표현 요소.                              |

~막상 정리하고 나니까 그닥 뭐 별거 없는거 같기도...~

## 3. BEM(Block Element Modifier)

block / element / modifier 로 나누는 개발 접근법이다.  
UI를 독립된 여러 개의 블록으로 분리하자는 목표를 가지고 있다.

전반적으로 다음과 같은 특징이 있다.

- tag selector나 id는 사용하지 않고 class만 사용한다.
- 모든 이름을 간단하고 직관적으로 만듦으로써 selector만으로도 무엇을 하는지 유추 가능하게 class 이름을 작성해야 한다.
- `.block__element--modifier` 형식을 가진다.
- 이름을 연결할 때는 `block-name`처럼 하이픈 하나만 사용한다.
- '어떻게 보이는가'가 아닌 '어떤 목적인가'에 초점을 맞춘다. 예를 들어 경고를 표시할 때 클래스 이름을 `red`가 아닌 `error`로 작성한다.

### Block

![BEM block](https://user-images.githubusercontent.com/63287638/197430432-5441b0f5-716a-41d4-a23e-15e7bb447551.jpg)

block은 문단 전체에 적용된 element, 또는 element를 담고 있는 container이다.
조금 구체적으로 설명하자면, 재사용 가능한 기능적으로 독립적인 페이지 컴포넌트(A functionally independent page component that can be reused)이다(react스러운 느낌이 남).  
`header`, `footer`, `sidebar`, `content` 등이 각각의 block이며 참고로 block은 다른 block을 감쌀 수 있다.

### Element

![BEM element](https://user-images.githubusercontent.com/63287638/197430434-66e0e711-fb9e-4c24-bdd8-186adaf7c34e.jpg)

element는 block을 구성하며 block 안에 특정 기능을 수행하는 컴포넌트를 의미한다.
block은 독립적인 형태인 반면, element는 의존적인 형태다.  
element는 자신이 속한 block 내에서만 의미를 가진다.

```html
<form class="search-form">
  <input class="search-form__input" />
  <button class="search-form__button">Search</button>
</form>
```

위 예시에서 `.search-form`은 block이고, `.search-form__input`과 `.search-form__button`은 element이다.
`.search-form`은 여러 페이지에서 반복적으로 사용될 수 있다.  
하지만 내부의 `input`과 `button`은 `.search-form` 안에서만 의미가 있는 element이다.

### Modifier

![BEM modifier](https://user-images.githubusercontent.com/63287638/197433121-d436534e-0076-47c3-82b7-d5e4d198a4e1.png)

modifier는 block이나 element의 속성을 담당한다.  
block 또는 element의 외관이나 상태를 변화시켜 다르게 동작하는 block이나 element를 만들 때 사용하면 된다.

```html
<ul class="tab">
  <li class="tab__item tab__item--focused">탭 01</li>
  <li class="tab__item">탭 02</li>
  <li class="tab__item">탭 03</li>
</ul>
```

위 코드에서 `--focused`가 modifier다.

아래처럼 `key-value` 타입도 '-'을 사용해서 표시 가능하다.

```html
<!-- color-gray, theme-normal이 key-value 타입 -->
<div class="column">
  <strong class="title">일반 로그인</strong>
  <form class="form-login form-login--theme-normal">
    <input type="text" class="form-login__id" />
    <input type="password" class="form-login__password" />
  </form>
</div>

<div class="column">
  <strong class="title title--color-gray">VIP 로그인 (준비중)</strong>
  <form class="form-login form-login--theme-special form-login--disabled">
    <input type="text" class="form-login__id" />
    <input type="password" class="form-login__password" />
  </form>
</div>
```

### 장단점

- 장점
  - 직관적인 클래스 명으로 마크업 구조를 직접 보지 않아도 구조의 파악이 쉬움.
- 단점
  - 클래스명이 상대적으로 길어질 수밖에 없는 구조이기 때문에 코드가 길어지고 복잡해짐.
  - 기존 마크업에서 새롭게 기능이 추가되었을 경우 클래스명 재수정이 불편.

## 4. SMACSS(Scalable and Modulalr Architecture for CSS)

CSS에 대한 확장형 모듈식 구조의 형태로 5개의 구분된 카테고리로 CSS 코딩 기법 제시하는 방법이다.  
어떤 카테고리에 스타일이 속하는지 결정하는데 고민과 숙고가 요구된다.

해당 카테고리들은 다음과 같다.

- base
  - base 규칙은 기본값이다. base는 해당 엘리먼트가 페이지 어디에 존재하든 기본적으로 가질 값을 나타낸다.
- layout
  - page를 section으로 나눈다. 하나 이상의 module을 포함하고 있다.
- module
  - module은 재사용 가능한 부분을 말한다.
- state
  - layout이나 module이 어떤 상태인지 묘사하는데 사용된다(`is-hidden`, `is-active` 등).
- theme
  - state과 같이 layout이나 module에서 어떻게 보일지를 나타낸다.

### Base

```css
body,
p,
h1,
h2,
h3,
h4,
h5,
h6,
ul,
ol,
li,
dl,
dt,
dd,
table,
th,
td,
form,
fieldset,
legend,
input,
textarea,
button,
select {
  margin: 0;
  padding: 0;
}
body,
input,
textarea,
select,
button,
table {
  font-size: 14px;
  line-height: 1.25;
}
body {
  position: relative;
  background-color: #fff;
  color: #000;
}
img,
fieldset {
  border: 0;
}
ul,
ol {
  list-style: none;
}
table {
  border-collapse: collapse;
}
em,
address {
  font-style: normal;
}
a {
  color: inherit;
  text-decoration: none;
}
```

위와 같이 속성 선택자, 가상 선택자, 자식 선택자 등을 사용할 순 있지만 거의 대부분 single element selector(`span`, `body` 등)을 사용한다.  
CSS 재설정(reset)은 기본 여백, 패딩 및 기타 속성을 제거하거나 재설정하도록 설계된 기본 스타일 세트다.

다음과 같은 특징이 있다.

- class나 id는 전혀 사용하지 않는다.
- `!important`를 사용하지 않는다.

### Layout

```css
#article {
  width: 80%;
  float: left;
}

#sidebar {
  width: 20%;
  float: right;
}

.l-fixed #article {
  width: 600px;
}

.l-fixed #sidebar {
  width: 200px;
}
```

layout style은 재사용 여부에 따라 major / minor 스타일로 나뉜다.  
`header`, `footer`, `aside`, `container`, `content`와 같은 것을 위해 id를 사용할 수도 있다(그리고 SMACSS에서 유일하다).

다음과 같은 특징이 있다.

- id, class를 활용하여 오직 하나의 selector만 사용한다(다만 id를 사용할 때는 꼭 필요한지 고민하자). 예를 들어 유저에 따른 페이지 설정을 조정한다면 id가 필요할 수도 있다.
- 접두사로 `l-`을 사용한다.

### Module

버튼, 배너, 아이콘, 박스 등 페이지에서 재사용 가능한 요소들을 이야기한다.

다음과 같은 특징이 있다.

- 재사용을 위해 id, element selector를 사용하지 않는다. 오직 class로만 지정한다.
- 반드시는 아니지만 `module-`이라는 접두사를 붙인다.

같은 module이 다른 section이 사용되는데 약간 차이가 있을 때 부모에 스타일을 지정하면 `!important`를 사용하거나 많은 selector를 사용해야 되는 문제가 있다.  
subclass를 활용하면 이를 방지할 수 있다.  
공식 문서에 나온 예시 상황이다.

> Expanding on our example pod, we have an input with two different widths. Throughout the site, the input has a label beside it and therefore the field should only be half the width. In the sidebar, however, the field would be too small so we increase it to 100% and have the label on top. All looks well and good. Now, we need to add a new component to our page. It uses most of the same styling as a .pod and so we re-use that class. However, this pod is special and has a constrained width no matter where it is on the site. It is a little different, though, and needs a width of 180px.

```css
.pod {
  width: 100%;
}
.pod input[type='text'] {
  width: 50%;
}
#sidebar .pod input[type='text'] {
  width: 100%;
}

.pod-callout {
  width: 200px;
}
#sidebar .pod-callout input[type='text'],
.pod-callout input[type='text'] {
  width: 180px;
}
```

위 방식은 너무 많은 selector가 사용된다.  
`.pod-constrained`와 `.pod-callout`이라는 subclass를 활용하면 아래와 같이 selector를 줄일 수 있다.

```css
.pod {
  width: 100%;
}
.pod input[type='text'] {
  width: 50%;
}
.pod-constrained input[type='text'] {
  width: 100%;
}

.pod-callout {
  width: 200px;
}
.pod-callout input[type='text'] {
  width: 180px;
}
```

```html
<!-- HTML에서 사용 예시 -->
<div class="pod pod-constrained">...</div>
<div class="pod pod-callout">...</div>
```

### state

```css
.tab {
  background-color: purple;
  color: white;
}

.is-tab-active {
  background-color: white;
  color: black;
}
```

다음과 같은 특징이 있다.

- `!important`를 사용해도 되며 이 방식이 추천된다.
- single class selector를 이용해야 한다.
- state가 module에 의존적인 경우가 있다. 이러한 경우 state의 class 이름을 `is-tab-active`와 같이 module을 포함한다.
- class 이름으로 `is-`라는 접두사를 가진다.

### Theme

```css
/* in module-name.css */
.mod {
  border: 1px solid;
}

/* in theme.css */
.mod {
  border-color: blue;
}
```

자주 사용되지는 않는다.  
전반적인 look and feel을 정의한다.  
사용자가 선택 가능하도록 스타일을 재선언하여 사용한다.  
`background`, `color`, `border` 등을 불변하는 스타일과 분리한다.

- `theme-`이라는 접두사를 가지거나 `theme/`과 같은 디렉토리로 계층을 분리한다.
- 나라마다 다른 언어를 사용해야 하는 경우 각 locale을 제공해야 하며 한글이나 한자는 크기가 작으면 사용자 경험이 좋지 않다. 폰트와 같은 경우는 `f-`라는 접두사를 붙일 수 있다. 웹 사이트는 보통 3~6개 정도의 폰트 크기만 가지고 있는게 좋다. 그 이상 존재하면 유지 보수가 힘들다.

### 장단점

- 장점
  - 클래스명을 통한 예측 용이성.
  - 재사용을 통한 코드의 간결화.
  - 확장의 용이성.
- 단점
  - 카테고리를 나누는 기준이 불분명.
  - 코드를 나누어서 작성하기 때문에 오히려 CSS가 복잡해질 수 있음.

### 추가 사항

[공식 문서](http://smacss.com/)에서 저자가 추가한 본인의 좋은 습관(?)이다.

1. CSS 특성마다 그루핑한다. 가독성이 더 좋아진다.

```css
.class {
  display: flex;
  justify-content: space-between;
  align-items: center;

  border: 1px solid black;

  position: relative;
  top: 0;
}
```

2. color 관련 속성은 `#fff;`처럼 3digit으로 표현 가능하면 3digit으로 표현해라. 그리고 `hex`나 `rgb`는 사용하지 않는다. 왜냐면 `#`속성이 더 짧기 때문이다.

## 5. 결론

motion 프로젝트에서 OOCSS와 BEM을 합쳐서 사용할 예정이다.  
(개인적으로 BEM은 `styled-component`와 같은 느낌이 나서 react스럽다라고 느껴졌다)

내가 생각한 notion은 다음과 같은 특징이 있다.

1. 페이지는 결국 하나다.
2. sidebar, nav, content는 컴포넌트를 선언해서 반복하여 사용한다.

```css
.d-flex {
  display: flex;
  justify-content: center;
  align-items: center;
}

.m-0 {
  margin: 0;
}

.f-16 {
  font-size: 16px;
}
```

위와 같은 코드들은 모든 컴포넌트에서 사용할 수 있을 만한 속성들이다.  
반복적인 요소들이기 때문에 따로 선언할 예정이다.  
다만 이 때문에 overkill이 나지 않도록 3번 이상 반복적인 요소들에 대해서만 추가할 예정이다.

특정 컴포넌트들은 어떤 컴포넌트 안에서만 의미를 갖는 경우가 있다.  
예를 들면 상단 아이콘, private page list 등이다.  
class 이름이 길어지는 단점이 생길 수 있지만 해당 컴포넌트가 하는 기능의 명확성을 표시하기 위해 사용할 예정이다.
