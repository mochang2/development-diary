## requestAnimationFrame

참고  
https://inpa.tistory.com/entry/%F0%9F%8C%90-requestAnimationFrame-%EA%B0%80%EC%9D%B4%EB%93%9C

CSS로도 화려한 애니메이션(`animation`, `transition`, `transform`)은 충분히 구현할 수 있다.  
하지만 사용자의 특정 인터렉션, 예를 들면 어떤 element를 클릭한다든지, scroll 이벤트가 발생한다든지에 따라 애니메이션을 실행하고 싶다면 JS를 이용해야만 한다.  
다만 JS만으로 스타일을 변화시키는 것은 CSS만 이용하는 것보다 성능이 좋지 않다.  
`requestAnimationFrame`은 이러한 상황에서 사용 가능한 **최적화 기법**이다.

### 사용법

```ts
const callback = (timestamp: DOMHighResTimeStamp) => {
  // 인자 참고: https://developer.mozilla.org/en-US/docs/Web/API/DOMHighResTimeStamp
  // do something
};

window.requestAnimationFrame(callback);
// callback: 다음 repaint를 위한 애니메이션을 업데이트할 때 호출할 함수
```

### 주사율

모니터에는 fps(frame per second)라는 단위가 있다.  
1초당 모니터의 출력 빈도 즉, 몇 프레임을 보여주는지에 대한 내용이다.  
보통 사람은 1초에 60번 출력해야 즉, 60fps(또는 60hz)에 맞게 렌더링해야 장면이 부드럽게 넘어간다고 느낀다.

![fps](https://github.com/mochang2/development-diary/assets/63287638/7fc4e949-e0e4-4ed7-a468-b8d3541a6040)

JS로 자연스러운 애니메이션을 구현하기 위해 초당 60개의 프레임을 렌더링해야 한다는 말은 대략 16.6ms마다 함수를 호출해야 한다는 것이다.

### timer API를 이용한 애니메이션 구현

```js
const naturalFps = 1000 / 60;

const animate = () => {
  // height 등을 변경함
};

setInterval(animate, naturalFps);

// 또는

const animate2 = () => {
  // height 등을 변경함

  setTimeout(animate2, naturalFps);
};

setTImeout(animate2, naturalFps);
```

timer API, 그 중에서 특히 `setTimeout`의 가장 큰 문제는 16.6ms마다 실행이 보장되지 않는다는 것이다.  
JS에서 timer API는 비동기로 동작하기 때문에 콜스택이 비어져 있을 때만 실행된다.  
만약 `naturalFps`(16.6ms) 만큼의 시간이 지난 뒤 `animate2`를 실행하고 싶었지만 메인 스레드가 다른 업무를 하느라 콜스택이 비어져 있지 않다면 `animate2`의 실행은 그만큼 뒤로 밀리고, 이후의 `setTimeout`에 대한 호출도 그 밀린 시간 만큼 뒤늦게 실행된다.  
이는 프레임 드랍을 야기한다.  
`setInterval`는 위와 같은 이유로 콜백의 실행 시간이 밀리면 내부적으로 다음 호출에 대해 조정하는 로직이 포함되어 있다.  
하지만 여전히 프레임 드랍을 보장하지는 못한다.  
아래 사진과 같은 상황이 발생할 수 있기 때문이다.

![timer API frame drop](https://github.com/mochang2/development-diary/assets/63287638/82c5a520-e2f7-4d11-b696-12c19cec145d)

JS에 의해 reflow가 일어나는데, 그러면 프레임이 생성되지 못하고 누락된다.

### requestAnimationFrame을 이용한 애니메이션 구현

`requestAnimationFrame` 함수는 시스템이 프레임을 그릴 준비가 되면 애니메이션 프레임을 호출하여 웹 페이지를 보다 원활하고 효율적으로 생성할 수 있도록 해준다.  
실제 화면이 갱신되어 표시되는 주기에 따라 함수를 호출하기 때문에 JS가 프레임 시작할 때 실행되도록 보장해 timer API에서 발생할 수 있는 프레임 드랍을 방지한다.

![raf not frame drop](https://github.com/mochang2/development-diary/assets/63287638/dba157aa-7140-40b9-b07e-649dcc3e3a88)

```html
<h2>setInterval vs rAF 애니메이션 부드러움 차이</h2>
<div class="ex interval"></div>
<span>setInterval</span>

<hr />

<div class="ex request"></div>
<span>requestAnimationFrame</span>

<br />
<button class="btn-start" onClick="onClick()">테스트 시작</button>
```

```css
body {
  background: #333;
  color: #ccc;
}

.ex {
  height: 20px;
  width: 10px;
  background: #fcdba1;
}
```

```js
const intervalEl = document.querySelector('.interval');
const requestEl = document.querySelector('.request');

let intervalWidth = 10;
let requestWidth = 10;

function requestRender() {
  requestEl.style.width = `${requestWidth}px`;
  requestWidth += 10;
  const id = requestAnimationFrame(requestRender);
  if (requestWidth > window.innerWidth) cancelAnimationFrame(id);
}

function intervalRender() {
  const id = setInterval(() => {
    intervalEl.style.width = `${intervalWidth}px`;
    intervalWidth += 10;

    if (intervalWidth > window.innerWidth) clearInterval(id);
  }, 1000 / 60);
}

function onClick() {
  intervalWidth = 10;
  requestWidth = 10;

  requestRender();
  intervalRender();
}
```

위 코드를 실행시켜보면 `setInterval`로 실행한 bar는 width 변화가 끊기는 것이 눈에 보이지만 `requestAnimationFrame`으로 실행한 bar는 width 변화가 끊기는 것이 눈에 띄지 않는다.

### requestAnimationFrame 장점

timer API와 비교할 때 최적화 관점에서 크게 3가지 장점이 있다.

1. 백그라운드 동작 중지

timer API와 다르게, 페이지가 비활성화 상태일 때(브라우저의 다른 탭을 보거나 브라우저가 최소화되어 있을 때) `requestAnimationFrame`에 의한 화면 그리기 작업도 일시 중지되면서 CPU 리소스를 낭비하지 않는다.

2. 디스플레이 주사율에 맞게 호출 횟수 조정

60hz 주사율 모니터일 경우 16.6ms마다 모니터에 렌더링하는 게 자연스럽다.  
하지만 만약 모니터의 주사율이 144hz라면 16.6ms는 적합한 시간이 아니다.  
timer API는 콜백 함수 호출 간격을 모니터 주사율에 따라 동적으로 변화시킬 수 없지만, `requestAnimationFrame`은 모니터의 주사율을 그대로 따른다.

3. 별도의 큐에서 처리

- Task queue: 이벤트 콜백 함수, timer API 콜백 함수 담당
- Micro task queue: `Promise` 객체, `Mutation Observer` 객체 담당(가장 우선순위가 높음)
- Animation frames: 브라우저 렌더링 엔진이 다음 프레임을 그리기 전에 실행해아 하는 콜백 담당

`requestAnimationFrame`은 별도의 큐(Animation frames)에서 처리되기 때문에 실행이 뒤처지는 등의 현상이 감소할 수 있다.  
단, 항상은 아니고 CPU나 GPU 사용량에 따라 콜백 함수 실행이 뒤로 밀릴 수는 있다.
