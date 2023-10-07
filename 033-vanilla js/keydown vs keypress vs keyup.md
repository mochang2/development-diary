## keydown vs keypress vs keyup

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
});
```

`keydown`과 `keypress`는 사용자가 해당 키를 누르고 있는 동안 여러 번 발생하지만,  
`keyup`은 키에서 손을 뗄 때 한 번만 발생한다.

_참고_  
2023년 현재 `keydown`은 deprecated된 상태이다.
