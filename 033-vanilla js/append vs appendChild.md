## parentNode.append vs parentNode.appendChild

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
const div = document.createElement('div');
const div2 = document.createElement('div');
document.body.append(div, 'hello', div, div2);

/*
<body> 
  hello
  <div></div>
  <div></div>
</body>
*/

// appendChild 예시
const div = document.createElement('div');
console.log(document.body.appendChild(div)); // <div></div>

/*
<body>
  <div></div>
</body>
*/
```
