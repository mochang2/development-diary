## insertAdjacentHTML

DOM 요소를 삽입할 때 쓰인다.  
[MDN](https://developer.mozilla.org/ko/docs/Web/API/Element/insertAdjacentHTML)과 https://dev.to/jeannienguyen/insertadjacenthtml-vs-innerhtml-4epd 에 따르면

> `insertAdjacentHTML` 메서드는 HTML or XML 같은 특정 텍스트를 파싱하고, 특정 위치에 DOM tree 안에 원하는 node들을 추가한다. 이미 사용중인 element 는 다시 파싱하지 않는다. 그러므로 element 안에 존재하는 element를 건드리지 않는다. `innerHtml`보다 작업이 덜 드므로 빠르다.

> `insertAdjacentHTML` 메서드는 호출된 element를 reparse하지 않으므로 요소를 손상시키지 않는다. `insertAdjacentHTML`는 element를 연속적으로 serialize하고 reparse하지 않기 때문에 콘텐츠가 많을 때마다 추가 속도가 느려지는 `innerHtml`보다 훨씬 빠르다.

`element.insertAdjacentHTML(position, text)`와 같은 방식으로 사용된다.
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
const foodArray = ['김밥', '방어', '사과', '오렌지'];
const FOOD_TEMPLATE = (food) => '<div class="list_food">' + food + '</div>';
foodArray.forEach((food) =>
  body.insertAdjacentHTML('afterbegin', FOOD_TEMPLATE(food))
); // 또는 'beforeend'
```
