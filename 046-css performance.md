## 0. 공부하게 된 계기

`link` 태그의 `preload`, `preconnect`, `prefetch`가 '무엇'인지 공부하다가 외부 리소스 가져오는 부분이라는 것을 알게 돼서 정리하고자 한다.  
우선 한 가지 확실히 할 것은 모든 것에는 일장일단이 있다.  
초기 렌더링 속도를 향상시키느냐, 초기 렌더링 후의 업데이트 속도를 향상시키느냐에 대해 결정해서 옵션을 적용해야 한다.

CSS 성능 향상을 위한 모든 방법은 아니므로 혹시 최적화가 필요하다면 아래 블로그들을 참고해서 적용해보고 여기에 더 정리해보겠다.

https://yceffort.kr/2021/03/improve-css-performance  
https://pustelto.com/blog/optimizing-css-for-faster-page-loads/  
https://kinsta.com/blog/optimize-css/

참고 및 사진 출처  
https://beomy.github.io/tech/browser/critical-rendering-path/#%EB%A6%AC%EC%86%8C%EC%8A%A4-%EC%9A%B0%EC%84%A0%EC%88%9C%EC%9C%84-%EC%A7%80%EC%A0%95  
https://beomy.github.io/tech/browser/preload-preconnect-prefetch/

## 1. Critical Rendering Path

Critical Rendering Path란 브라우저가 페이지의 초기 출력을 위해 실행해야 하는 순서를 말한다.  
여기서 순서는 (DOM 트리 구축 - CSSOM 트리 구축 - JS 실행) - 렌더 트리 구축 - 레이아웃 - 페인트 - 컴포지트를 말한다.  

아래 사진은 Critical Rendering Path 타임스탬프이다.

![critical rendering path](https://user-images.githubusercontent.com/63287638/230037470-5a444796-4beb-457a-9057-534149165a02.png)

_domLoading_: 전체 프로세스의 시작 타임스탬프. 브라우저가 처음 수신한 HTML 문서 바이트의 파싱을 시작하려고 할 때의 시점.
_domInteractive_: 브라우저가 파싱을 완료한 시점. 모든 HTML 및 DOM 생성이 완료됨.
**domContentLoaded**: DOM이 준비되고 JS 실행을 차단하는 스타일시트가 없는 시점. DOM과 CSSOM이 모두 준비된 상태로 렌더 트리를 생성할 수 있는 시점. 많은 JS 프레임워크는 자체 로직을 실행하기 전에 이 타임스탬프를 기다림. 이 타임스탬프를 통해 실행이 얼마나 오래 걸렸는지 추적할 수 있음.
_domComplete_: 페이지의 모든 리소스(이미지 등) 다운로드가 완료된 시점.
**loadEvent**: 페이지 로드의 마지막 단계로, 브라우저가 추가 애플리케이션 로직을 트리거 할 수 있는 onload 이벤트를 발생시킴.

## 2. Critical Rendering Path 최적화

기본적으로 JS, CSS, 이미지, 폰트 등은 이러한 Critical Rendering Path를 차단하는 리소스이다.  
JS처럼 `defer` / `async` / `type="module"` 속성이나 이미지처럼 lazy loading이 가능한 방법을 통해 초기 렌더링이 느린 문제를 해결할 수 있다.  
반면 CSS는 rel, media 등의 property를 통해 해결할 수 있다.  
HTTP/2의 병렬 요청 기능이 이를 해소해줄 수 있긴 하지만 여전히 고려해야 될 점이 존재할 수 있기 때문에 자세히 정리하고자 한다.  

### `media` property

페이지가 인쇄될 때나 대형 모니터에 출력하는 경우 등 몇 가지 특수한 경우에만 사용되는 CSS가 있다면, 해당 CSS가 렌더링을 차단하지 않는 것이 좋다.  
이 경우에 미디어 유형과 미디어 쿼리를 사용면 CSS 리소스를 렌더링 비차단 리소스로 표시할 수 있다.

```html
<link href="style.css" rel="stylesheet" />
<!-- 1 -->
<link href="style.css" rel="stylesheet" media="all" />
<!-- 2 -->
<link href="print.css" rel="stylesheet" media="print" />
<!-- 3 -->
<link href="portrait.css" rel="stylesheet" media="orientation:landscape" />
<!-- 4 -->
<link href="other.css" rel="stylesheet" media="min-width: 40em" />
<!-- 5 -->
```

위 `media` property는 아래와 같은 결과를 발생시킨다.

1: 항상 렌더링 차단.  
2: 항상 렌더링 차단.  
3: 콘텐츠가 인쇄될 때만 적용되어, 처음 로드될 때는 렌더링이 차단되지 않음.  
4: 기기 방향이 가로일 때만 렌더링 차단.  
5: 기기의 너비 조건이 일치하면 렌더링 차단.

### code splitting

(특히 SPA 라이브러리, 프레임워크를 사용한다면 편히 적용 가능)  
컴포넌트에 따라 CSS를 분리하고, 해당 컴포넌트에서만 가져오게끔 설정하면 초기에 모든 CSS를 가져오느라 느려지는 것을 해소할 수 있다.

### `rel` property

브라우저에 전송되는 모든 리소스는 똑같이 중요한 것은 아니다.  
브라우저는 가장 중요한 리소스(스크립트나 이미지보다 CSS 우선)를 우선 로드하기 위해 가장 중요하다 생각되는 리소스를 추측하여 먼저 로드한다.  
하지만 이런 방법이 항상 정답이 아닐 수 있기 때문에, preload, preconnect, prefetch를 이용해서 브라우저에게 리소스의 우선순위를 전달 해 줄 수 있다.

먼저 간단하게 정리하자면 다음과 같다.

1\) 현재 페이지에서 반드시 사용되는 리소스는 preload.  
2\) 외부 도메인의 리소스는 preconnect.  
3\) 미래에 사용되는 리소스는 prefetch.

#### `preload`

특정 리소스를 미리 로드하면 현재 페이지에 중요하다고 확신하기 때문에 브라우저가 검색할 때보다 더 빨리 가져오고 싶다고 알리는 기능이다.  
다만 CSS를 미리 가져와도 DOM이 없으므로 바로 적용할 수 없다.

> This(resources) ensures they are available earlier and are less likely to block the page's render, improving performance. We preload our CSS and JavaScript files so they will be available as soon as they are required for the rendering of the page later on.

이외에도 캐싱 기능을 지원한다.
[MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/rel/preload)에서는 용량이 클수록 효과가 좋아서 다음 자원들에서 효과적이라고 이야기한다.

- Resources that are pointed to from inside CSS, like fonts or images.
- Resources that JavaScript can request, like JSON, imported scripts, or web workers.
- Larger images and video files.

`preload`는 `<link rel="preload" as="...">`와 같이 사용한다.  
참고로 `as`에 script, style 등 리소스의 유형을 브라우저에게 알려주지 않으면 브라우저는 사용하지 않는다.

#### `preconnect`

현재 페이지에서 외부 도메인의 리소스를 참고하는 것을 브라우저에게 알려 미리 외부 도메인과 연결을 설정할 수 있게 한다.  
`preconnect`를 사용하면 브라우저가 사이트에 필요한 연결을 미리 예상할 수 있게 되어 브라우저는 필요한 소켓을 미리 설정할 수 있다.  
따라서 DNS, TCP, TLS 왕복에 필요한 시간을 절약할 수 있게 된다.  
`preconnect`는 외부 도메인과 연결을 구축하기 때문에 많은 CPU 시간을 차지할 수 있다.  
보안 연결의 경우 더 많은 시간을 차지할 수 있다.  
10초 이내로 브라우저가 닫힌다면, 이전의 모든 연결 작업은 낭비되는 것이기 때문에 브라우저가 빨리 닫힐 수 있는 페이지에서는 `preconnect`를 사용하지 않는 것이 좋다.

#### `prefetch`

미래에 사용될 것이라고 예상되는 리소스들에 대해 사용된다.  
브라우저는 미래에 사용될 리소스들을 가져와 캐시에 저장한다.  
`prefetch`는 사용자가 다음에 할 행동을 미리 준비하는데 적합한 기능이다.  
예를 들어, 결과 목록에서 첫 번째 제품 상세 페이지를 가져오거나 콘텐츠의 다음 페이지를 가져올 때 사용할 수 있다.  
다만 `prefetch`는 재귀적으로 동작하지 않는다.  
이 말은 HTML 파일을 prefetch한다고 해당 HTML 파일이 사용하는 리소스들도 전부 `prefetch`되지 않는다는 의미이다.

