## childNodes vs children

`childNodes`은 `Node`의 property이다.  
따라서 HTML tag, text, comment 등이 모두 `childNodes`에 해당한다.

반면 `children`은 `Element`의 property이다.  
따라서 HTML tag만이 `children`이다.

```js
const element = document.createElement('div');
element.textContent = 'foo';

element.childNodes.length === 1; // 'foo'라는 text Node.
element.children.length === 0; // Element 존재하지 않음.
```
