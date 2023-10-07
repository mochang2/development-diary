## IntersectionObserver

Intersection Observer API는 target이 현재 viewport에 보여지는지 여부에 따라 비동기적으로 callback을 실행해줄 수 있는 API이다.  
정확히는 target이 viewport 또는 특정 element(`root` 아래 `option` 참고)와 교차할 때나 observer가 최초로 타겟을 observe하도록 요청받을 때 callback이 호출된다.  
특히 다음과 같은 상황에서 유용하게 사용될 수 있다.

- 스크롤에 이동에 따른 이미지 lazy loading
- 무한 스크롤
- 광고 노출 횟수 추정
- 애니메이션을 실행할지 여부 판단

```ts
const callback = (
  entries: IntersectionObserverEntry[],
  observer: IntersectionObserver
) => {
  // entry의 property
  // boundingClientRect: 관찰 대상의 사각형 정보
  // intersectionRect: 관찰 대상의 교차한 영역 정보
  // intersectionRatio: 관찰 대상의 교차한 영역 백분율(intersectionRect 영역에서 boundingClientRect 영역까지 비율, Number)
  // isIntersecting: 관찰 대상의 교차 상태
  // rootBounds: 지정한 루트 요소의 사각형 정보
  // target: 관찰 대상 요소
  // time: 변경이 발생한 시간 정보
};

const option = {
  root: null,
  // target의 가시성을 확인할 때 사용되는 element. null일 경우 브라우저의 뷰포트. 기본값은 null
  rootMargin: '200px 100px',
  // root가 가진 margin. 단위를 반드시 입력해야 함. 기본값은 0
  threshold: 0.1,
  // 콜백이 실행될 target의 가시성이 얼마일 때 callback이 실행되는지를 나타냄. 현재 옵션에서는 target의 가시성이 10%일 때 callback이 실행됨. 배열로 주면 각 배열의 요소마다 실행됨. 기본값은 [0]
};

const observer = new IntersectionObserver(callback, option);
const main = document.querySelector('main');
observer.observe(main); // target에 대한 관측을 시작
observer.unobserve(main); // target에 대한 관측을 중지
observer.disconnect(); // 인스턴스가 관찰하는 모든 요소의 관찰을 중지
```

### 기존에 무한 스크롤 구현하던 방식

~문법 틀렸을 수도 있음... 그냥 대충 아래와 같이 쓴다는 뜻~

```js
const handleScroll = () => {
  const { scrollTop, offsetHeight } = document.documentElement; // cause reflow
  if (window.innerHeight + scrollTop >= offsetHeight) {
    // fetch or appear ...
  }
};

// 이벤트 달아야 되면
window.addEventListener('scroll', handleScroll);

// 이벤트 해지해야 하면
window.removeEventListener('scroll', handleScroll);
```

위와 같은 방식으로 `scroll` 이벤트 리스너를 단 뒤, 스크롤 높이와 offset 등을 비교해서 viewport에 나타나면 API 호출 등과 같은 특정 조치를 취했다.  
하지만 이 방법에는 아주 큰 문제가 있다.  
바로 스크롤할 때마다 이벤트 리스너가 발생해서 엄청 잦은 계산(`scrollTop`, `offsetHeight`에 대한 참조에 의한 reflow)이 발생한다.  
이는 당연히 **성능 저하**로 이어질 수밖에 없다.

위와 같은 문제를 해결하기 위해 `throttle` 등을 도입할 수 있지만 사용자 디바이스마다, 네트워크 상황마다 적정한 시간을 설정하기 쉽지 않다.  
그리고 [setTimeout API를 이용하는 throttle은 콜스택에 따라 예상치 못하게 동작할 수도 있다](https://tech.kakaoenterprise.com/149).

또다른 해결책으로 브라우저가 렌더링하는 빈도인 60pfs(정확히는 브라우저가 조절함)에 맞춰서 실행되는 `requestAnimationFrame` API를 사용하는 방법도 있다.

```js
const throttleByAnimationFrame = (handler) =>
  function (this, ...args) {
    window.requestAnimationFrame(() => {
      handler.apply(this, args);
    });
  };
```

다만 이 방법은 잘못하면 무한 재귀 호출이 발생할 수 있으며, 브라우저 및 디바이스 성능에 의존적이다.  
또한 별도의 함수가 추가되기 때문에 _덜 우아한_ 방법인 것 같은 느낌도...?

### 대안책

`IntersectionObserver`는 이에 대한 우아한 해결책이다.  
앞서 이야기했듯이 비동기로 동작하여 스레드 블로킹을 방지하며 브라우저에서 직접 최적의 방식으로 교차 영역을 계산하고 처리하기 때문에 성능상 이점이 있다.  
무엇보다 대상 요소가 나타나거나 사라지는 경우를 파악하기 위한 별도의 함수 선언이 필요하지 않다.

```js
const callback = (entries, observer) => {
  entries.forEach((entry) => {
    if (entry.isIntersecting) {
      // fetch or appear ...
    }
  });
};

// 이벤트 달아야 되면
const observer = new IntersectionObserver(callback, options);
observer.observe(ref.current);

// 이벤트 해지해야 한다면
observer.disconnect();
```
