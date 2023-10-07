## innerHTML

참고  
https://velog.io/@1106laura/insertAdjacentHTML

DOM 요소 내부의 내용을 변경하는데 사용된다.

`innerHTML`은 요소 내에 포함된 HTML을 가져오고, 문자열처럼 '='이나 '+=' 연산자로 해당 내용을 추가하거나 변경할 수 있도록 해준다.  
`innerHTML`이 실행되면 해당 요소 내의 DOM Tree가 초기화되고 대입해준 값으로 대체된다.

아래와 같은 방법으로 사용된다.

```javascript
const body = document.querySelector('body');
body.innerHTML = '';
```

다만 *변경*이 아닌 *삽입*만 필요한 상황이라면 아래의 `insertAdjacentHTML`이 더 효율적이다.

```javascript
const foodArray = ['김밥', '방어', '사과', '오렌지'];
const FOOD_TEMPLATE = (food) => '<div class="list_food">' + food + '</div>';
foodArray.forEach((food) => (body.innerHTML += FOOD_TEMPLATE(food)));
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
body.innerHTML = '<div class="list_food">김밥</div>';
body.innerHTML =
  '<div class="list_food">김밥</div><div class="list_food">방어</div>';
body.innerHTML =
  '<div class="list_food">김밥</div><div class="list_food">방어</div><div class="list_food">사과</div>';
body.innerHTML =
  '<div class="list_food">김밥</div><div class="list_food">방어</div><div class="list_food">사과</div><div class="list_food">오렌지</div>';
```

위 방법을 보완하기 위해 `forEach` 대신 `reduce`를 사용할 수 있지만 그럼에도 더 빠른 아래 방법이 있다.
