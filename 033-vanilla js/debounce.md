## debounce

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
  let inDebounce;
  const { leading, trailing } = options;

  return function () {
    // inDebounce 값이 변하기 전에 미리 저장하기 위해 사용
    // 첫 이벤트를 무조건 발생시킬지 여부
    let callNow = leading && !inDebounce;

    // leading이 아닌 경우에만 wait 이후 callback을 실행
    const later = () => {
      inDebounce = null;
      if (!leading) {
        callback.apply(this, arguments);
      }
    };

    if (trailing) {
      if (inDebounce) {
        clearTimeout(inDebounce);
      }
      inDebounce = setTimeout(later, time);
    }

    if (callNow) {
      callback.apply(this, arguments);
    }
  };
};

// 사용
const debounced = debounce(() => console.log(123), 500);
```

_참고) `throttle`_  
`debounce`는 주어진 인터벌마다 실행되는 것이 아니라, 주어진 time 이내에 연속으로 이벤트가 발생할 경우, 더 이상 이벤트가 발생하지 않을 때까지 일단 대기하도록 하는 함수이다.  
검색어 자동 완성과 같은 기능을 할 때 주로 사용된다.

반면 `throttle`은 여러 번 발생하는 이벤트를 일정 시간 동안 한 번만 실행되도록 만드는 개념이다.  
이벤트가 꾸준히 발생되며 주어진 인터벌마다 주기적으로 이벤트가 발생하게 하고 싶을 때 `throttle`을 사용한다.  
무한 스크롤에서 주로 사용된다.
