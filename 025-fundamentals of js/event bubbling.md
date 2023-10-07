## Event Bubbling

이벤트 버블링은 특정 화면 요소에서 이벤트가 발생했을 때 해당 이벤트가 더 상위의 화면 요소들로 전달되어 가는 특성을 의미한다.  
아래와 같이 코드를 짰을 경우, `div3`를 눌렀을 때, 'click `div3`' -> 'click `div2`' -> 'click `div1`' (-> `document.addEventListener`가 있다면 마지막에 실행) 순으로 alert가 실행된다.

_참고)_  
이벤트 발생 시 event capturing -> target -> event bubbling의 순서대로 실행된다.  
이때 이벤트 핸들러는 (별다른 옵션을 주지 않는다면 기본적으로)event bubbling 단계에서 시작되기 때문이다.

```html
<!DOCTYPE html>

<html>
  <head>
    <title>test</title>
    <meta charset="utf-8" />
    <link rel="stylesheet" type="text/css" href="./test.css" />
    <script defer src="./test.js" type="text/javascript"></script>
    <!-- defer 옵션이 붙은 이유는 아래 6번째 소제목에서 설명할 예정 -->
  </head>
  <body>
    <div id="div1">
      <div id="div2">
        <div id="div3"></div>
      </div>
    </div>
  </body>
</html>
```

```css
body {
  width: 100%;
  height: 500px;
}

div {
  display: flex;
  justify-content: center;
  align-items: center;
}

#div1 {
  width: 100%;
  height: 100%;
  background-color: aquamarine;
}

#div2 {
  width: 50%;
  height: 50%;
  background-color: beige;
}

#div3 {
  width: 25%;
  height: 25%;
  background-color: goldenrod;
}
```

![event test code](https://user-images.githubusercontent.com/63287638/178144554-11968076-1eaa-47db-a43b-518bcbc48bd5.png)

```javascript
// querySelector로 가져온 뒤 forEach를 돌리면 되는데 귀찮아서 일일이 다 씀...
const div3 = document.getElementById('div3');
div3.addEventListener('click', (e) => {
  alert('click div3');
});

const div2 = document.getElementById('div2');
div2.addEventListener('click', (e) => {
  alert('click div2');
});

const div1 = document.getElementById('div1');
div1.addEventListener('click', (e) => {
  alert('click div1');
});
```

### stopPropagation

이벤트가 전파되는 것을 막는 것이다.  
아래 [이벤트 캡처링](https://github.com/mochang2/development-diary/edit/main/025-fundamentals%20of%20js.md#event-capturing)에서도 동일한 원리로 동작한다.

```javascript
const div3 = document.getElementById('div3');
div3.addEventListener('click', (e) => {
  // e.stopPropagation(); // 여기에 쓰면 click div3까지만 실행된다
  alert('click div3');
});

const div2 = document.getElementById('div2');
div2.addEventListener('click', (e) => {
  // e.stopPropagation(); // 여기에 쓰면 click div2까지만 실행된다
  alert('click div2');
});

const div1 = document.getElementById('div1');
div1.addEventListener('click', (e) => {
  alert('click div1');
});
```

### event capturing

이벤트가 전파되는데, bubbling과는 반대 방향으로 전파된다.  
즉, 부모의 이벤트가 먼저 실행되는 것이다.  
아래와 같은 코드면 (`document.addEventListener {capture: true}`가 있다면 처음에 실행 ->) `click div1` -> `click div2` -> `click div3` 순으로 실행된다.

```javascript
const div3 = document.getElementById('div3');
div3.addEventListener(
  'click',
  (e) => {
    alert('click div3');
  },
  { capture: true }
);

const div2 = document.getElementById('div2');
div2.addEventListener(
  'click',
  (e) => {
    alert('click div2');
  },
  { capture: true } // 호기심에 이 부분을 빼보니 div2의 이벤트가 가장 늦게 실행됐다
);

const div1 = document.getElementById('div1');
div1.addEventListener(
  'click',
  (e) => {
    alert('click div1');
  },
  { capture: true }
);
```

### event delegation

하위 요소에 각각 이벤트를 붙이지 않고 상위 요소에서 하위 요소의 이벤트들을 제어하는 방식이다.  
focus와 같이 이벤트 전파가 되지 않거나 `event.stopPropagation`을 사용하지 않았을 때 사용 가능하다.  
또한 동적으로 element가 새로 생성될 때, 해당 객체에 대한 이벤트를 자동으로 할당할 수 있어서 유용하게 사용할 수 있는 기술이다.

_출처: https://joshua1988.github.io/web-development/javascript/event-propagation-delegation/_

```html
<h1>오늘의 할 일</h1>
<ul class="itemList">
  <li>
    <input type="checkbox" id="item1" />
    <label for="item1">이벤트 버블링 학습</label>
  </li>
  <li>
    <input type="checkbox" id="item2" />
    <label for="item2">이벤트 캡쳐 학습</label>
  </li>
</ul>
```

```javascript
// input에 이벤트 추가
var inputs = document.querySelectorAll('input');
inputs.forEach(function (input) {
  input.addEventListener('click', function (event) {
    alert('clicked');
  });
});

// 새로운 element를 추가하는 코드
var itemList = document.querySelector('.itemList');

var li = document.createElement('li');
var input = document.createElement('input');
var label = document.createElement('label');
var labelText = document.createTextNode('이벤트 위임 학습');

input.setAttribute('type', 'checkbox');
input.setAttribute('id', 'item3');
label.setAttribute('for', 'item3');
label.appendChild(labelText);
li.appendChild(input);
li.appendChild(label);
itemList.appendChild(li);
```

위에 코드(`addEventListener` 코드)까지는 새로운 TODO가 생기지 않다면 잘 동작할 것이다.  
하지만 새로운 TODO가 추가될 경우(`addEventListener` 아래 코드) 원하는 click 이벤트가 잘 동작하지 않는다.  
이럴 때 event delegation을 이용할 수 있는데, `.itemList` 자체에 이벤트를 다는 것이다.

```javascript
// 위에 쓴 JS 코드 다 지우고
var itemList = document.querySelector('.itemList');
itemList.addEventListener('click', function (event) {
  alert('clicked');
});
```
