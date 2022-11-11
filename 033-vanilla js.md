## 0. 공부하게 된 계기

vanilla.js로 개발할 일이 좀 적다보니 새로 얻는 개념에 대해 기록해야 조금 더 머리에 오래 남을 것 같아서 정리한다.
~원래는 [vanilla.js](https://github.com/mochang2/development-diary/blob/main/026-FE%20development%20environment.md)에 있는 내용인데 분리했다.~

### 목차

[innerHTML](https://github.com/mochang2/development-diary/blob/main/033-vanilla%20js.md#1-innerhtml)  
[insertAdjacentHTML](https://github.com/mochang2/development-diary/blob/main/033-vanilla%20js.md#2-insertadjacenthtml)  
[cloneNode](https://github.com/mochang2/development-diary/blob/main/033-vanilla%20js.md#3-clonenode)  
[HTMLCollection vs NodeList](https://github.com/mochang2/development-diary/blob/main/033-vanilla%20js.md#4-htmlcollection-vs-nodelist)  
[Element.closest()](https://github.com/mochang2/development-diary/blob/main/033-vanilla%20js.md#5-elementclosest)  
[keydown vs keypress vs keyup](https://github.com/mochang2/development-diary/blob/main/033-vanilla%20js.md#6-keydown-vs-keypress-vs-keyup)  
[removeEventListener](https://github.com/mochang2/development-diary/blob/main/033-vanilla%20js.md#7-removeeventlistener)  
[append vs appendChild](https://github.com/mochang2/development-diary/blob/main/033-vanilla%20js.md#8-parentnodeappend-vs-parentnodeappendchild)  
[history](https://github.com/mochang2/development-diary/blob/main/033-vanilla%20js.md#9-history)  
[data-\* 속성](https://github.com/mochang2/development-diary/blob/main/033-vanilla%20js.md#10-data--%EC%86%8D%EC%84%B1)  
[modulo](https://github.com/mochang2/development-diary/blob/main/033-vanilla%20js.md#11-modulo)  
[debounce](https://github.com/mochang2/development-diary/blob/main/033-vanilla%20js.md#12-debounce)

## 1. innerHTML

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

## 2. insertAdjacentHTML

DOM 요소를 삽입할 때 쓰인다.  
[MDN](https://developer.mozilla.org/ko/docs/Web/API/Element/insertAdjacentHTML)과 https://dev.to/jeannienguyen/insertadjacenthtml-vs-innerhtml-4epd 에 따르면

> `insertAdjacentHTML` 메서드는 HTML or XML 같은 특정 텍스트를 파싱하고, 특정 위치에 DOM tree 안에 원하는 node들을 추가한다. 이미 사용중인 element 는 다시 파싱하지 않는다. 그러므로 element 안에 존재하는 element를 건드리지 않는다. `innerHtml`보다 작업이 덜 드므로 빠르다.

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

## 3. cloneNode

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

## 4. HTMLCollection vs NodeList

참고: https://yung-developer.tistory.com/m/79 , https://stackoverflow.com/questions/32486199/whats-the-difference-between-live-and-not-live-collection-in-javascript-selecto

이 둘을 비교하기 전에 **live dom collection**과 **non-live dom collection**을 먼저 비교할 필요가 있다.

- live dom collection
  - DOM에 변화가 생길 때 collection에도 변화가 생기는 것을 말한다.
  - node에 변화가 생길 때 그 내용도 변한다.
  - `document.getElementsByClassName()`, `document.getElementsByTagName()`, `document.getElementsByName()` 등이 이에 속하는 DOM API이다.
- non-live dom collection
  - DOM에 변화가 생겨도 collection에 해당 변화가 반영이 되지 않는 것을 말한다.
  - request가 발생할 때마다만 변화가 compute 된다.
  - 이 경우는 live dom collection과 달리 DOM의 각 수정(내용, 속성, 클래스)이 컬렉션의 각 요소를 re-evaluate해야 하기 때문에 비용이 많이 든다. 최대 O(the umber of all elements in the DOM \* the number of active `querySelectorAll()` collections)의 시간 복잡도를 가진다.
  - `document.querySelectorAll()` 등이 이에 속하는 DOM API이다.

HTMLCollection은 live dom collection 객체이다.

```html
<!DOCTYPE html>
<html lang="kr">
  <head>
    <meta charset="UTF-8" />
    <title>test</title>
  </head>
  <body>
    <div id="app">
      <div class="greeting">Hello</div>
    </div>
  </body>
  <script>
    const app = document.getElementById('app')
    const greeting = document.getElementsByClassName('greeting')
    console.log(greeting, greeting.length) // HTMLCollection [div.greeting] 1
    app.insertAdjacentHTML('beforeend', '<div class="greeting">Hello2</div>')
    console.log(greeting, greeting.length) // HTMLCollection(2) [div.greeting, div.greeting] 2
  </script>
</html>
```

처음에 'greeting'이라는 className을 가진 element는 하나밖에 없어 첫 번째 `console.log`에서는 길이를 1로 출력한다.  
하지만 `insertAdjacentHTML`을 실행한 뒤에는 'greeting'을 재선언하거나 재할당하지 않았음에도 `console.log`에서 길이가 2로 변경된다.  
이는 HTMLCollection이 live dom collection 객체이기 때문이다.

반면 NodeList는 non-live dom collection 객체이다.
아래와 같은 경우는 `console.log`의 결과가 달라지지 않는다.

```html
<!DOCTYPE html>
<html lang="kr">
  <head>
    <meta charset="UTF-8" />
    <title>test</title>
  </head>
  <body>
    <div id="app">
      <div class="greeting">Hello</div>
    </div>
  </body>
  <script>
    const app = document.getElementById('app')
    const greeting = document.querySelectorAll('.greeting')
    console.log(greeting, greeting.length) // NodeList [div.greeting] 1
    app.insertAdjacentHTML('beforeend', '<div class="greeting">Hello2</div>')
    console.log(greeting, greeting.length) // NodeList [div.greeting] 1
  </script>
</html>
```

하지만 *주의할 점*은 모든 NodeList 객체가 non-live dom collection인 것은 아니다.  
`childNodes`라는 property를 통해 반환하는 NodeList 객체는 live dom collection이다.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>test</title>
  </head>
  <body id="app">
    <ul id="students">
      <li class="frontend">s1</li>
      <li class="frontend">s2</li>
    </ul>
  </body>
  <script>
    const $students = document.getElementById('students')
    const childNodes = $students.childNodes

    console.log(childNodes instanceof NodeList) // true
    console.log(childNodes)
    // NodeList(5) [text, li.frontend, text, li.frontend, text]
    // childNodes는 요소노드 뿐만아니라 공백 텍스트 노드(엔터 키)도 포함되어 있다.

    for (let i = 0; i < childNodes.length; i++) {
      // removeChild 메서드가 호출될 때마다 NodeList live 객체인 childNodes가 실시간으로 변경된다.
      // 따라서 첫 번째, 세 번째, 다섯 번째 요소만 삭제된다.
      $students.removeChild(childNodes[i])
    }

    console.log(childNodes) // NodeList(2) [li.frontend, li.frontend]
  </script>
</html>
```

추가 사항으로, NodeList와 HTMLCollection은 유사 배열 객체이지만 `Array.prototype`에 있는 배열의 메서드를 대부분 사용하지 못 한다.  
다만 NodeList는 예외적으로 `forEach` 메서드를 사용할 수 있다.

_`querySelectorAll` 참고 사항_  
ES6부터 추가된 `Array.from()` 메서드를 사용하면 HTMLCollection이나 NodeList를 배열로 반환할 수 있다.  
이를 통해 형제 요소 중 특정 property를 가진 element를 특정할 수 있다.  
아래는 코드 예시이다.

```javascript
// suggestions 중에서 Suggestion__item--selected 라는 클래스를 가진 요소의 index

const suggestions = document.querySelectorAll('div.Suggestion li')
const suggestedIndex = Array.from(suggestions).indexOf(
  document.querySelector('.Suggestion__item--selected')
)
```

## 5. Element.closest()

참고: https://developer.mozilla.org/ko/docs/Web/API/Element/closest

Element의 closest() 메서드는 주어진 CSS 선택자(`querySelector`와 같은 방식)와 일치하는 요소를 찾을 때까지, **자기 자신을 포함**해 위쪽(부모 방향, 문서 루트까지)으로 문서 트리를 순회한다.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>test</title>
  </head>
  <body>
    <article>
      <div id="div-01">
        Here is div-01
        <div id="div-02">
          Here is div-02
          <div id="div-03">Here is div-03</div>
        </div>
      </div>
    </article>
    <script>
      const el = document.getElementById('div-03')

      // ID가 "div-02"인 가장 가까운 조상
      console.log(el.closest('#div-02')) // <div id="div-02">

      // div 안에 놓인 div인 가장 가까운 조상
      console.log(el.closest('div div')) // <div id="div-03">

      // div면서 article을 부모로 둔 가장 가까운 조상
      console.log(el.closest('article > div')) // <div id="div-01">

      // div가 아닌 가장 가까운 조상
      console.log(el.closest(':not(div)')) // <article>
    </script>
  </body>
</html>
```

### event delegation

`closest`가 유용하게 쓰이는 방법 중 하나는 event delegation을 이용할 때이다.  
event bubbling의 개념이 이용되며 `event.stopPropagation()`이나 bubbling이 되지 않는 'focus'와 같은 이벤트에서는 사용 불가능한 기술이다.

event delegation이란 비슷한 방식으로 여러 요소를 다뤄야 할 때 사용된다.  
이벤트 위임을 사용하면 요소마다 핸들러를 할당하지 않고, 요소의 공통 조상에 이벤트 핸들러를 단 하나만 할당해도 여러 요소를 한꺼번에 다룰 수 있다.

event delegation의 장점은 다음과 같다.

1. 동적인 element(이벤트 등으로 중간에 생성되는 element들)에 대한 이벤트 처리가 수월하다. 상위 엘리먼트에서만 이벤트 리스너를 관리하기 때문에 하위 엘리먼트는 자유롭게 추가 삭제할 수 있다.
2. 이벤트 핸들러 관리가 쉽다. 동일한 이벤트에 대해 한 곳에서 관리하기 때문에 각각의 엘리먼트를 여러 곳에 등록하여 관리하는 것보다 관리가 수월하다.
3. 메모리 사용량이 줄어든다. 동적으로 추가되는 이벤트가 없어지기 때문이다.
4. 메모리 누수 가능성도 줄어든다. 등록 핸들러 자체가 줄어들기 때문에 메모리 누수 가능성도 줄어든다.

```javascript
const parent = document.getElementById('parent') // 상위 element

parent.addEventListener('click', (e) => {
  const child = e.target.closest('#child') //

  if (!child) {
    return
  }

  // something
})
```

위와 같은 방식으로 동작시키면 부모에서 한 번의 이벤트 등록으로 모든 자식 요소들에게 동일한 이벤트가 동작하도록 할 수 있다.

### event.target vs event currentTarget

- `target`: event를 trigger한 element 자체. event가 bubble될 때면 root element.
- `currentTarget`: event listener가 달려있는 element.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>test</title>
    <link rel="stylesheet" href="./style.css" />
  </head>
  <body>
    <div class="modal">
      <div class="content">
        <div>공간공간</div>
      </div>
    </div>
    <script>
      const modal = document.querySelector('div.modal')

      modal.addEventListener('click', (e) => {
        console.log(e.target) // 가장 안에 있는 div를 클릭하면 <div>공간공간</div>. div.modal을 클릭하면 <div class="modal">...</div>
        console.log(e.currentTarget) // 항상 <div class="modal">...</div>
      })
    </script>
  </body>
</html>
```

## 6. keydown vs keypress vs keyup

이벤트가 발생하는 순서는 `keydown` -> `keypress` -> `keyup`이다.  
`keydown`은 사용자가 키를 눌렀을 때 trigger되고, `keyup`은 사용자가 키에서 손을 뗄 때 trigger된다.  
`keypress`는 그 중간에서 발생하는 event이다.

`keydown`과 `keyup`은 물리적인 키들을 다루지만 `keypress` 그렇지 않다.  
무슨 뜻이냐면 delete, arrows, esc, ctrl, alt, shift 등은 `keypress`가 감지할 수 없다는 의미이다.  
그래서 esc 키를 감지하고 싶다면 아래와 같은 방식으로 사용해야 된다.

```javascript
document.addEventListener('keydown', (e) => {
  if (e.key === 'Escape') {
    // do something
  }
})
```

`keydown`과 `keypress`는 사용자가 해당 키를 누르고 있는 동안 여러 번 발생하지만,  
`keyup`은 키에서 손을 뗄 때 한 번만 발생한다.

## 7. removeEventListener

element에 부여된 listener를 1번만 발동시키기 위한 방법은 크게 3가지가 있지만 가장 추천되는 방법은 1번이다.

1. 이름 있는 함수 사용

```javascript
function onClickFunction() {
  element.removeEventListener('click', onClickFunction)
  console.log('event has been removed')
}

element.addEventListener('click', onClickFunction)
```

2. once 옵션 사용하기

```javascript
element.addEventListener(
  'click',
  () => {
    console.log('event has been removed')
  },
  { once: true }
)
```

위 방법은 간편하지만 조건에 따라 listener를 삭제하고 싶을 때엔 적합한 방법이 아니다.

3. `arguments.callee`

```javascript
element.addEventListener('click', function () {
  element.removeEventListener('click', arguments.callee)
})
```

`arguments.callee`는 현재 실행 중인 함수를 참조할 수 있는 속성이다.  
익명 함수에서 함수를 참조할 때 사용한다.
이때 콜백 함수는 반드시 `function` 키워드로 작성해야 하는데, arrow function은 arguments의 바인딩이 일어나지 않기 때문이다.

다만, `arguments.callee`는 ES5 이후에서는 strict mode에서 사용이 금지되었다.

## 8. parentNode.append vs parentNode.appendChild

둘다 parent element에 child Element를 추가하는 메서드이다.

차이점은 다음과 같다.

- append
  - Node object뿐만 아니라 text도 child로 추가할 수 있다.
  - 한 번에 여러 개의 child를 추가할 수 있다.
  - return값이 없다.
- appendChild
  - Node object만 child로 추가할 수 있다.
  - 한 번에 하나의 child만 추가할 수 있다.
  - 추가된 child를 반환한다.

```javascript
// append 예시
const div = document.createElement('div')
const div2 = document.createElement('div')
document.body.append(div, 'hello', div, div2)

/*
<body> 
  hello
  <div></div>
  <div></div>
</body>
*/

// appendChild 예시
const div = document.createElement('div')
console.log(document.body.appendChild(div)) // <div></div>

/*
<body>
  <div></div>
</body>
*/
```

## 9. history

vanilla.js에서도 `react-router`(`react-router-dom`, `react-router-native`) 와 같은, SPA에서의 router를 구현할 수 있다.  
우선 필요한 DOM 개념 2가지를 알아야 한다.

**`CustomEvent()`**

새로운 CustomEvent를 생성하는 생성자로 다음과 같이 사용된다.

`new CustomEvent(typeArg, options)`

- `typeArg`: 이벤트의 이름을 나타내는 문자열
- `options`: 다음을 포함하는 객체
  - `detail`: 이벤트 내에 될 이벤트의 세부 정보를 나타내는 값(객체), 기본값은 null
  - `Event()` 생성자 옵션에 지정할 수 있는 모든 속성

**BrowserRouter vs HashRouter**

- BrowserRouter
  - HTML5의 history API를 활용해서 업데이트를 한다.
  - 서버에 있는 데이터들을 스크립트에 의해 가공처리한 후 생성되어 전달되는 동적인 페이지에 적합하다.
  - 해당 라우터를 이용해서 이동한 경로에서 새로 고침을 하면 에러가 나므로 추가적인 세팅이 필요하다.
- HashRouter
  - URL의 hash를 이용한 라우터로, 주소에 '#'이 붙는다.
  - 서버가 주소의 이동을 모른다. 따라서 정적인 페이지에 주로 사용된다.
  - `www.domain.com/#/path`로 요청하면 실제로는 `www.domain.com`에 대한 요청이 간다.
  - 주소 이동 후, 새로고침해도 에러가 발생하지 않는다.
  - 검색엔진이 읽지 못한다.

SPA와 같은 웹 페이지를 만들기 위한 것이 목적이므로 BrowserRouter를 사용해야 한다.  
따라서 `history.pushState()`를 이용한다.

### 코드 예시

`ROUTE_CHANGE_EVENT`는 (custom)event의 `typeArg`이다.

```javascript
// // app.js
const app = document.querySelector('#app')
const route = () => {
  const { pathname } = location

  app.innerHTML = ''

  if (pathname === 'product-list') {
    new ProductList() // app에 ProductList 렌더링
  } else if (pathname === 'cart') {
    // ...
  } // ...
  // 제대로된 path가 아니면 오류 페이지를 렌더링
}

window.addEventListener(ROUTE_CHANGE_EVENT, () => {
  route() // route가 바뀔 때마다 불릴 수 있도록 설정
})
window.addEventListener('popstate', () => {
  // window.onpopstate. 앞으로 가기, 뒤로 가기 처리
  route()
})

// // utils.js
// url을 바꿔야 되는 상황이 있을 때 해당 함수 호출
const changeRoute = (url, params) => {
  history.pushState(null, '', url)
  window.dispatchEvent(new CustomEvent(ROUTE_CHANGE_EVENT, params))
}
```

_참고) https://woong-jae.com/javascript/220325-spa-from-scratch-2 는 path를 key, callback을 value로 둬서 사용한 예시이다._

## 10. data-\* 속성

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

### JS에서의 접근

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

### CSS에서의 접근

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

## 11. modulo

아래는 파이썬 코드의 나머지 연산이다.

```python
print(-5 % 10) # 5
```

하지만 JS는 음수의 나머지 연산이 일반적이지 않다.

```javascript
console.log(-5 % 10) // -5
```

index에 대한 나머지 연산이 필요한 경우, 예를 들면 -1번째 인덱스를 원하면 object.length - 1번째 인덱스를 가리켜야 하는 경우 위와 같은 연산은 문제가 된다.  
[스택오버플로우](https://stackoverflow.com/questions/4467539/javascript-modulo-gives-a-negative-result-for-negative-numbers)의 해설에 따르면 다음과 같은 custom modulo 연산을 만들어야 한다.

```javascript
// prototype chaining. 보통 더 느림
Number.prototype.mod = function (n) {
  return ((this % n) + n) % n
}

// 아래 방법 추천
function mod(n, m) {
  return ((n % m) + m) % m
}
```

## 12. debounce

`lodash`에서 많이 사용하는 함수 중 하나가 `debounce`이다.  
이벤트 호출을 지연시키거나 한 번에 처리하게끔 해서 API의 호출을 최소화하기 위해 많이 사용된다.

`lodash`에서는 `leading`과 `trailing`이라는 옵션을 받는다.

- `leading`
  - `true`면 첫 이벤트 발생시 그 이벤트는 일단 반영되고, 이후의 연속하는 이벤트는 묶어서 처리한다.
  - 기본값은 `false`이다.
- `trailing`
  - `true`면 연속된 이벤트의 마지막 이벤트가 발생 후 wait으로 대입된 밀리세컨트 만큼이 지난 후 마지막 이벤트 하나가 반영된다.
  - 기본값은 `true`이다.

아래는 vanilla.js로 구현한 debounce이다.  
클로저 개념을 사용해서 구현한다.

```javascript
// arguments는 함수 선언시 기본적으로 전달되는 매개변수들을 의미하는 keyword

/**
 * @param {function} callback
 * @param {number} time - trailing milliseconds
 * @param {object} options
 * @return {function}
 */
const debounce = (callback, time, options) => {
  let inDebounce
  const { leading, trailing } = options

  return function () {
    // inDebounce 값이 변하기 전에 미리 저장하기 위해 사용
    // 첫 이벤트를 무조건 발생시킬지 여부
    let callNow = leading && !inDebounce

    // leading이 아닌 경우에만 wait 이후 callback을 실행
    const later = () => {
      inDebounce = null
      if (!leading) {
        callback.apply(this, arguments)
      }
    }

    if (trailing) {
      if (inDebounce) {
        clearTimeout(inDebounce)
      }
      inDebounce = setTimeout(later, time)
    }

    if (callNow) {
      callback.apply(this, arguments)
    }
  }
}

// 사용
const debounced = debounce(() => console.log(123), 500)
```

_참고) `throttle`_  
`debounce`는 주어진 인터벌마다 실행되는 것이 아니라, 주어진 time 이내에 연속으로 이벤트가 발생할 경우, 더 이상 이벤트가 발생하지 않을 때까지 일단 대기하도록 하는 함수이다.  
검색어 자동 완성과 같은 기능을 할 때 주로 사용된다.

반면 `throttle`은 여러 번 발생하는 이벤트를 일정 시간 동안 한 번만 실행되도록 만드는 개념이다.  
이벤트가 꾸준히 발생되며 주어진 인터벌마다 주기적으로 이벤트가 발생하게 하고 싶을 때 `throttle`을 사용한다.  
무한 스크롤에서 주로 사용된다.

## css 성능 향상 방법

https://yceffort.kr/2021/03/improve-css-performance
