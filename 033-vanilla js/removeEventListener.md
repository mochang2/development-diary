## removeEventListener

element에 부여된 listener를 1번만 발동시키기 위한 방법은 크게 3가지가 있지만 가장 추천되는 방법은 1번이다.

1. 이름 있는 함수 사용

```javascript
function onClickFunction() {
  element.removeEventListener('click', onClickFunction);
  console.log('event has been removed');
}

element.addEventListener('click', onClickFunction);
```

2. once 옵션 사용하기

```javascript
element.addEventListener(
  'click',
  () => {
    console.log('event has been removed');
  },
  { once: true }
);
```

위 방법은 간편하지만 조건에 따라 listener를 삭제하고 싶을 때엔 적합한 방법이 아니다.

3. `arguments.callee`

```javascript
element.addEventListener('click', function () {
  element.removeEventListener('click', arguments.callee);
});
```

`arguments.callee`는 현재 실행 중인 함수를 참조할 수 있는 속성이다.  
익명 함수에서 함수를 참조할 때 사용한다.
이때 콜백 함수는 반드시 `function` 키워드로 작성해야 하는데, arrow function은 arguments의 바인딩이 일어나지 않기 때문이다.

다만, `arguments.callee`는 ES5 이후에서는 strict mode에서 사용이 금지되었다.
