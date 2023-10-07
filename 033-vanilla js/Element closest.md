## Element.closest()

참고  
https://developer.mozilla.org/ko/docs/Web/API/Element/closest

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
      const el = document.getElementById('div-03');

      // ID가 "div-02"인 가장 가까운 조상
      console.log(el.closest('#div-02')); // <div id="div-02">

      // div 안에 놓인 div인 가장 가까운 조상
      console.log(el.closest('div div')); // <div id="div-03">

      // div면서 article을 부모로 둔 가장 가까운 조상
      console.log(el.closest('article > div')); // <div id="div-01">

      // div가 아닌 가장 가까운 조상
      console.log(el.closest(':not(div)')); // <article>
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
const parent = document.getElementById('parent'); // 상위 element

parent.addEventListener('click', (e) => {
  const child = e.target.closest('#child'); //

  if (!child) {
    return;
  }

  // something
});
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
      const modal = document.querySelector('div.modal');

      modal.addEventListener('click', (e) => {
        console.log(e.target); // 가장 안에 있는 div를 클릭하면 <div>공간공간</div>. div.modal을 클릭하면 <div class="modal">...</div>
        console.log(e.currentTarget); // 항상 <div class="modal">...</div>
      });
    </script>
  </body>
</html>
```
