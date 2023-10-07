## HTMLCollection vs NodeList

참고  
https://yung-developer.tistory.com/m/79  
https://stackoverflow.com/questions/32486199/whats-the-difference-between-live-and-not-live-collection-in-javascript-selecto

이 둘을 비교하기 전에 **live dom collection**과 **non-live dom collection**을 먼저 비교할 필요가 있다.

- live dom collection
  - DOM에 변화가 생길 때 collection에도 변화가 생기는 것을 말한다.
  - node에 변화가 생길 때 그 내용도 변한다.
  - `document.getElementsByClassName()`, `document.getElementsByTagName()`, `document.getElementsByName()` 등이 이에 속하는 DOM API이다.
- non-live dom collection
  - DOM에 변화가 생겨도 collection에 해당 변화가 반영이 되지 않는 것을 말한다.
  - request가 발생할 때마다만 변화가 compute 된다.
  - 이 경우는 live dom collection과 달리 DOM의 각 수정(내용, 속성, 클래스)이 컬렉션의 각 요소를 re-evaluate해야 하기 때문에 비용이 많이 든다. 최대 O(the number of all elements in the DOM \* the number of active `querySelectorAll()` collections)의 시간 복잡도를 가진다.
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
    const app = document.getElementById('app');
    const greeting = document.getElementsByClassName('greeting');
    console.log(greeting, greeting.length); // HTMLCollection [div.greeting] 1
    app.insertAdjacentHTML('beforeend', '<div class="greeting">Hello2</div>');
    console.log(greeting, greeting.length); // HTMLCollection(2) [div.greeting, div.greeting] 2
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
    const app = document.getElementById('app');
    const greeting = document.querySelectorAll('.greeting');
    console.log(greeting, greeting.length); // NodeList [div.greeting] 1
    app.insertAdjacentHTML('beforeend', '<div class="greeting">Hello2</div>');
    console.log(greeting, greeting.length); // NodeList [div.greeting] 1
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
    const $students = document.getElementById('students');
    const childNodes = $students.childNodes;

    console.log(childNodes instanceof NodeList); // true
    console.log(childNodes);
    // NodeList(5) [text, li.frontend, text, li.frontend, text]
    // childNodes는 요소노드 뿐만아니라 공백 텍스트 노드(엔터 키)도 포함되어 있다.

    for (let i = 0; i < childNodes.length; i++) {
      // removeChild 메서드가 호출될 때마다 NodeList live 객체인 childNodes가 실시간으로 변경된다.
      // 따라서 첫 번째, 세 번째, 다섯 번째 요소만 삭제된다.
      $students.removeChild(childNodes[i]);
    }

    console.log(childNodes); // NodeList(2) [li.frontend, li.frontend]
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

const suggestions = document.querySelectorAll('div.Suggestion li');
const suggestedIndex = Array.from(suggestions).indexOf(
  document.querySelector('.Suggestion__item--selected')
);
```
