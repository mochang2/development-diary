## Async vs Defer

출처  
https://jae04099.tistory.com/entry/HTML-script-%ED%83%9C%EA%B7%B8%EB%8A%94-%EC%96%B4%EB%94%94%EC%97%90-%EC%9C%84%EC%B9%98-%ED%95%B4%EC%95%BC-%ED%95%A0%EA%B9%8C

[event bubbling](https://github.com/mochang2/development-diary/main/025-fundamentals%20of%20js/event%20bubbling.md)에서 예시로 쓴 코드에서처럼 HTML에서 script 모듈을 가져올 때 defer 옵션을 주지 않으면 JS에서 `getElementById` 등의 DOM과 관련된 상호작용하는 코드가 제대로 동작하지 않는다.  
div1, div2, div3가 `undefined` 객체라고 뜨는데 이는 HTML에서 JS를 실행시키는 시점과 관련이 있다.

웹에서 파싱이란 네트워크로 받은 HTML과 CSS 파일을 토큰화시키고 Parse Tree를 생성하는 것이다.  
이 Parse Tree를 DOM 트리로 만들어 렌더링하는 것이다.  
HTML을 화면에 보이기 위해 파싱하는 도중에 파싱을 멈추는 경우가 있는데, 바로 script 태그를 만날 때다.

script 태그는 실행 전에 다운로드 과정을 거친다.  
다음은 script 태그의 위치에 따른 구분이다.

### head 태그 내부

script 파일이 너무 크다면 HTML을 파싱하다가 사용자에게 빈 화면을 보여줄 수 있다.  
또한, script 내에서 DOM 요소를 조작한다면 존재하지 않는 DOM 요소에 접근하려는 시도이므로 제대로 된 접근이 되지 않을 수도 있다.

```html
<html>
  <head>
    <script src="~~"></script>
  </head>
  <body>
    <!-- ... -->
  </body>
</html>
```

![head 내부](https://user-images.githubusercontent.com/63287638/178145304-29b634ad-d2dd-4db1-9c52-df30b05b505a.png)

### body 태그 마지막

head 태그에 넣은 방법보다 HTML 콘텐츠를 빠르게 보여줄 수 있다.  
하지만 웹이 자바스크립트에 의존적이라면 사용자 경험을 안 좋게 만들 수 있다.

```html
<html>
  <head>
    <!-- ... -->
  </head>
  <body>
    <!-- ... -->
    <script src="~~"></script>
  </body>
</html>
```

![body의 마지막](https://user-images.githubusercontent.com/63287638/178145324-9301a9f4-8184-4ed7-8dae-5a5012de7b24.png)

### async

head 태그 내부에서만 사용 가능한 옵션이다.  
자바스크립트에 의존적인 웹을 좀 더 빨리 실행시킬 수 있다.  
하지만 HTML 파싱이 자바스크립트 파일을 실행시키는 동안 멈추게 돼 사용자 경험이 좋지 않을 수 있다.  
또한 script 태그가 여러 개일 경우, 순서에 상관 없이 먼저 다운로드 받아지는 script를 실행시키기 때문에 해당 프로젝트가 script 파일의 실행 순서에 영향을 받는다면 문제가 될 수 있다.

```html
<html>
  <head>
    <script async src="~~"></script>
  </head>
  <body>
    <!-- ... -->
  </body>
</html>
```

![async](https://user-images.githubusercontent.com/63287638/178145367-efe0f799-618f-402d-96cc-8c80b35f798d.png)

### defer

head 태그 내부에서만 사용 가능한 옵션이다.  
async처럼 script 다운을 HTML을 파싱할 때 진행하지만, 다른 점은 실행을 HTML 파싱이 전부 완료된 뒤에 한다는 것이다.  
여러 개의 script 태그를 이용해도 미리 다운을 다 받고 HTML도 끝까지 파싱시킨 후에 실행되기 때문에 async의 단점을 보완할 수 있다.

```html
<html>
  <head>
    <script defer src="~~"></script>
  </head>
  <body>
    <!-- ... -->
  </body>
</html>
```

![defer](https://user-images.githubusercontent.com/63287638/178145371-3b64a83a-f709-43b2-96b7-c6ee5a33798c.png)
