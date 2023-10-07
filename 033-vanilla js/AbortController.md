## AbortController

하나 이상의 웹 요청을 취소할 수 있게 해주는 인터페이스이다.  
이 API가 나오기 전까지는 요청을 보낸 뒤 해당 요청에 대한 응답이 필요없다면 그냥 받지 않고 무시했다고 한다.  
하지만 네트워크 비용이 크거나 [카카오 블로그 리뷰](https://tech.kakao.com/2023/03/27/2023-new-krew-onboarding-fe/)에 나오는 내용처럼 비동기 처리 과정에서 과거에 보낸 요청이 최근에 보낸 요청보다 늦게 도착해, 오히려 과거에 보낸 요청에 대한 응답이 최신 요청에 대한 응답을 이용한 렌더링을 덮어버리는 문제가 발생하는 경우가 존재한다.

23년 4월 현재 [MDN](https://developer.mozilla.org/ko/docs/Web/API/AbortController)에 따르면 실험적인 기능이라는데...

![abortcotroller compatibility](https://user-images.githubusercontent.com/63287638/230041439-e22b0994-199b-45c6-9293-952818e9dc43.PNG)

다 제공된다??  
어쨌든 다 제공되니 앞으로 더 발전하고 기능이 추가될 수 있는 유용한 API라고 생각한다.

다음과 같은 property와 method를 제공한다.

- `AbortController.signal`: DOM 요청과 통신하거나 취소하는데 사용되는 AbortSignal 객체 인터페이스를 반환.
- `AbortController.abort()`: DOM 요청이 완료되기 전에 취소. `fetch 요청`, 모든 응답 Body 소비, 스트림을 취소 가능.

아래 코드와 같이 사용할 수 있다.

```js
const controller = new AbortController();

setTimeout(() => {
  controller.abort();
}, 2000);

const URL = 'https://httpbin.org/delay/5'; // 5초 뒤에 응답을 내려주는 사이트
fetch(URL, { signal: controller.signal }) // fetch는 2번째 인자로 init을 받고, init의 객체로서 signal을 던질 수 있다.
  .then((res) => {
    console.log(`Received: ${res.status}`);
  })
  .catch((err) => {
    if (err.name === 'AbortError') {
      console.error('Aborted: ', err);
      return;
    }

    throw err;
  });
```

위 코드는 5초 뒤에 응답을 받는 사이트에 요청을 보내지만, 2초 뒤에 controller가 abort 시그널을 발생시키기 때문에 `catch`안에 `console.error`가 실행된다.  
그리고 `AbortController.abort()`는 `AbortError`라는 에러를 던지기 때문에 별도의 에러 처리 로직을 가질 수 있다.  
참고로 위와 같은 비동기 처리에 대한 timeout은 [AbortSignal.timeout()](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal/timeout)란 static method를 사용하는 것이 더 편하다.

React는 다음과 같이 사용할 수 있다.  
보통 `useEffect` 훅에서 side-effect를 처리하므로 `useEffect`에서 사용하는 예시를 들었다.  
cleanup을 사용한 것은 컴포넌트가 unmount될 때 abort 시그널을 날려야 되기 때문이다.

```js
function Component() {
  // ...

  useEffect(() => {
    const abortController = new AbortController();
    const URL = 'https://httpbin.org/delay/5';

    axios
      .get(URL, { signal: abortController.signal })
      .then(() => {})
      .catch(() => {});

    return () => {
      abortController.abort();
    };
  }, []);

  // ...
}
```
