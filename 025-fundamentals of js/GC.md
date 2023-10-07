## GC(garbage collection)

출처
https://blog.sessionstack.com/how-javascript-works-memory-management-how-to-handle-4-common-memory-leaks-3f28b94cfbec  
https://developer.mozilla.org/ko/docs/Web/JavaScript/Memory_Management

<img src="https://user-images.githubusercontent.com/63287638/178136632-c33e7ae6-c245-4cd9-a1c9-63a6c434d678.png" alt="memory lifecycle" width="50%" />

위 사진에서 볼 수 있듯이 메모리는 크게 3가지의 life cycle을 가지고 있다.  
할당한 뒤에 사용되고, 사용되지 않으면 해제된다.  
C와 같은 저레벨 언어에서는 allocate와 release를 malloc(calloc 등)과 free를 통해서 개발자가 직접 명령해야 하지만 JS, JAVA, Python 등은 별도의 Garbage Collector가 동작해서 메모리 관리를 해준다.

```javascript
let n = 374; // allocates memory for a number

let s = 'sessionstack'; // allocates memory for a string

let o = {
  // allocates memory for an object and its contained values
  a: 1,
  b: null,
};

let a = [1, null, 'str']; // (like object) allocates memory for the array

function f(a) {
  // allocates memory for a function
  return a + 3;
}

// 이러한 allocate 이후에 해당 변수나 함수 등을 사용하면 use 단계이다.
// 하지만 JS에서 release를 위한 문법이 별도로 존재하지 않는다.
```

### GC 전략 1: reference counting

reference 되는 횟수가 0인 메모리는 release하는 전략이다.  
언뜻 보기에는 문제가 없어보이는 전략 같지만, 상호 참조와 같은 상황에서는 쓸데없는 메모리가 낭비되는 것을 막지 못한다는 단점이 있다.  
간단한 코드 예제로 아래와 같은 상황이 있다.

```javascript
function f() {
  let o1 = {};
  let o2 = {};
  o1.p = o2; // o1 references o2
  o2.p = o1; // o2 references o1. This creates a cycle.
}
```

### GC 전략 2: mark and sweep algorithm

사용되는 메모리는 **mark** 한 뒤에 mark되지 않은 메모리들을 **sweep** 하는 알고리즘이다.  
GC의 root(웹은 window, node는 global)를 지정한 뒤 해당 루트를 타고, 타고 내려갔을 때 참조되지 않는 메모리를 release하는 방법이다.  
하지만 이 방법에도 문제점은 존재한다.  
'언제' Garbage Collector가 동작해야 하는지 모른다는 것이다.  
일반적으로 대부분의 Garbage Collector는 allocate한 뒤에 동작하는데, 만약 한번에 엄청 많은 allocation이 일어난 직후 더이상의 allocation이 일어나지 않는다면 메모리가 정리되지 않게 된다.  
하지만 이는 극단적인 경우이므로 일반적으로 자주 발생하지는 않는다.

이를 해결하는 방법이 수동 메모리 release인데, 위에서 얘기했듯이 JS에서는 제공하지 않는다.  
Node에서는 디버깅을 위한 추가 옵션과 도구를 제공한다. [참고](https://developer.mozilla.org/ko/docs/Web/JavaScript/Memory_Management#node.js)

### GC 전략 3: mark and sweep algorithm 최적화 기법

- generational collection(세대별 수집) – 객체를 '새로운 객체’와 '오래된 객체’로 나눈다. 객체 상당수는 생성 이후 제 역할을 빠르게 수행해 금방 쓸모가 없어지는데, 이런 객체를 '새로운 객체’로 구분한다. 가비지 컬렉터는 새로운 객체를 엄격한 기준에 따라 메모리에서 제거한다. 일정 시간 이상 동안 살아남은 객체는 '오래된 객체’로 분류하고, 가비지 컬렉터가 덜 감시한다.
- incremental collection(점진적 수집) – 방문해야 할 객체가 많다면 모든 객체를 한 번에 방문하고 mark 하는데 상당한 시간이 소모된다. 자바스크립트 엔진은 이런 현상을 개선하기 위해 가비지 컬렉션을 여러 부분으로 분리한 다음, 각 부분을 별도로 수행한다. 작업을 분리하고, 변경 사항을 추적하는 데 추가 작업이 필요하긴 하지만, 긴 지연을 짧은 지연 여러 개로 분산시킬 수 있다는 장점이 있다.
- idle-time collection(유휴 시간 수집) – 가비지 컬렉터가 실행에 주는 영향을 최소화하기 위해 CPU가 유휴 상태일 때에만 가비지 컬렉션을 실행한다.

### memory leak이 일어날 수 있는 경우

**global variable**

```javascript
function f() {
  variable = 'text text';
  // == window.variable 또는 global.variable = 'text text'
  // == this.variable = 'text text'
  // 이러한 전역 오염을 막기 위해서는 var, let, const 반드시 사용
}
```

**필요 없는 Timer나 callback**

```javascript
let serverData = loadData();

setInterval(function () {
  let what = document.getElementById('what');
  if (what) {
    what.innerHTML = serverData;
  }
}, 5000);
```

위 코드는 'what'이라는 id를 가진 element가 삭제될 때 더이상 필요하지 않는 타이머인데, 이를 삭제해주는 코드가 존재하지 않는다.  
아래 코드는 이러한 처리가 잘 되어있는 예시이다.

```javascript
let element = document.getElementById('launch-button');
let counter = 0;

function onClick(event) {
  counter++;
  element.innerHtml = 'text ' + counter;
}
element.addEventListener('click', onClick);

// Do stuff

element.removeEventListener('click', onClick); // onClick 함수에 대한 참조를 제거
element.parentNode.removeChild(element);

// element가 유효한 범위를 벗어나게 되면,
// element와 onClick은 GC에 의해 release됨
```

_+) 추가 사항_  
위 예시에서 `removeEventListener`를 굳이 실행시켜주지 않아도 (모던 브라우저에서는) 무한정 heap이 늘어나지 않는다. ~미친 성능의 GC~  
(`removeChild` 말고 `element.remove()`를 써도 마찬가지이다)  
https://www.tutorialspoint.com/if-a-dom-element-is-removed-are-its-listeners-also-removed-from-memory-in-javascript 와 https://yung-developer.tistory.com/82 를 참고했을 때, 그리고 실제로 개발자 도구를 이용해본 결과 GC가 알아서 event listener를 remove 해준다는 것을 알았다.  
다만 GC가 언제 동작할지 예측이 불가능하고(2019년 이후로 명시적으로 GC를 호출하는 것이 프로그래밍적으로 가능하지 않다고 함), 일정 기간 동안은 메모리 누수가 확실하게 발생한다.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>test</title>
    <link rel="stylesheet" href="./style.css" />
  </head>
  <body>
    <button id="btn1">btn1</button>
    <button id="btn2">btn2</button>
    <script>
      'use strict';

      let btn1 = document.getElementById('btn1');
      let btn2 = document.getElementById('btn2');

      btn1.addEventListener('click', function () {
        console.log('btn1 clicked');
        btn1.remove();
        // btn1 = null
      });

      btn2.addEventListener('click', function () {
        console.log(btn1);
        btn1.dispatchEvent(new Event('click')); // #btn1 클릭 이벤트 발생
      });
    </script>
  </body>
</html>
```

위 코드에서 `btn1 = null`을 명시적으로 해주지 않으면 GC가 동작할 때까지 `btn1`의 event listener는 사라지지 않는다.  
따라서 `btn2`를 눌렀을 때 지속적으로 `console.log('btn1 clicked')`가 동작한다.  
그래서 위와 같은 코드만으로는 GC의 동작을 확인하기 힘들어 프로그래머스 과제관에서 풀었던 '고양이 사진첩 애플리케이션 만들기'의 코드를 이용해 테스트해봤다.

```javascript
import { IMAGE_URL } from '../lib/constants.js';

class ImageViewer {
  constructor({ app, filePath }) {
    const image = this.createImage(filePath);
    const content = this.createContent(image);
    const modal = this.createModal(content);

    modal.addEventListener('click', (event) => {
      if (event.target.matches('.Modal')) {
        app.removeChild(modal);
      }
    });
    modal.addEventListener('keydown', (event) => {
      if (event.key === 'Escape' || event.key === 'Esc') {
        app.removeChild(modal);
      }
    });

    app.append(modal); // modal을 dom에 추가
  }

  createImage(filePath) {
    const image = document.createElement('img');
    image.setAttribute('src', `${IMAGE_URL}${filePath}`);

    return image;
  }

  createContent(image) {
    const content = document.createElement('div');
    content.append(image);

    return content;
  }

  createModal(content) {
    const modal = document.createElement('div');
    modal.setAttribute('class', 'Modal');
    modal.append(content);

    return modal;
  }
}
```

위 코드는 `app.js`에서 어떠한 동작을 하면 `new ImageViewer({ app, filePath })`가 동작하여 modal이라는 div가 DOM에 생성된다.  
그리고 esc를 누르거나 modal의 외부를 누르면 modal이라는 div가 DOM에서 사라진다.

아래 사진은 `new ImageViewer({ app, filePath })`를 수백 번 호출한 뒤 개발자 도구에서 확인한 memory 정보이다.  
별도의 `removeEventListener`가 존재하지 않지만 결과를 보면 꾸준히 'JS Heap, Nodes, Listeners'가 증가하다가 GC가 잘 처리해주면서 어느 순간 감소하는 것을 알 수 있다.  
추가적으로 `element.remove()`와 같이 DOM에서 element 지우는 API를 호출해도 바로 메모리에서 사라지지 않고 GC가 처리할 때까지 메모리에 상주한다는 것을 알 수 있다.  
따라서 최적화가 너무 중요하다면 DOM에서 지움과 동시에 `null`로 초기화해주고, `removeEventListener`도 명시적으로 호출해줘야 한다.

![network](https://user-images.githubusercontent.com/63287638/187958527-b7f9f296-4584-44b2-9107-8404d2e6f4d7.png)

**closure**

```javascript
let theThing = null;

let replaceThing = function () {
  let originalThing = theThing;

  let unused = function () {
    // theThing을 참조하는 originalThing이 아래에서 할당되므로 더이상 null이 아님
    // 사용되지 않는 함수지만 memory release되지 않음
    if (originalThing) {
      console.log('hi');
    }
  };

  theThing = {
    // replaceThing이 아래에서 호출되면서 더이상 null이 아님.
    longStr: new Array(1000000).join('*'),
    someMethod: function () {
      console.log('message');
    },
  };
};

setInterval(replaceThing, 1000);
```

**DOM 객체 참조**

```javascript
let elements = {
  button: document.getElementById('button'),
  image: document.getElementById('image'),
};

function doStuff() {
  elements.image.src = 'http://example.com/image_name.png';
}

function removeImage() {
  // image를 지워도 button이 남아있기 때문에 elements는 메모리에 남아 있음
  document.body.removeChild(document.getElementById('image'));
}
```

위 예시는 극단적인 예시같지만, 만약 어떠한 변수가 html 테이블에서 td를 참조하고 있다고 가정할 때는 문제가 심각해진다.  
만약 테이블 자체를 삭제해도 td를 참조하고 있는 변수가 GC에 의해 release되지 않으면 테이블 전체가 메모리에 남아있게 된다.

### WeakRef

참고: https://blog.shiren.dev/2021-08-30/

ECMAScript 2021에 나온 최신 문법이다.  
[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakRef)에 따르면

> WeakRef 개체에는 대상 또는 참조라고 하는 개체에 대한 약한 참조가 포함되어 있습니다.
> 개체에 대한 약한 참조는 개체가 가비지 수집기에 의해 회수되는 것을 막지 않는 참조입니다. 반대로 일반(또는 강력한) 참조는 객체를 메모리에 보관합니다.
> 개체가 더 이상 강력한 참조를 갖지 않으면 JavaScript 엔진의 가비지 컬렉터가 개체를 삭제하고 해당 메모리를 회수할 수 있습니다.
> 그렇게 되면 더 이상 약한 참조에서 객체를 얻을 수 없습니다.

기존에 JS는 강한 참조밖에 없었기 때문에, 아무리 메모리를 많이 잡아먹어도 어떤 객체나 데이터가 참조되고 있었다면 해당 객체나 데이터는 GC의 대상이 되지 않았다.  
그리고 이러한 현상이 메모리 누수로 이어졌다.  
`WeakRef`는 새로운 참조 방식으로 '알고 있지만 GC의 대상이 되는' 참조이다.  
**객체가 약한 참조로만 참조되고 있다면 아무도 모르는 것과 동일하게 언젠가 사라질 수 있다**

```javascript
const map = new Map();

const obj = { data: new Array(10000).join('*') };

map.set('someData', obj);

setInterval(() => {
  console.log(map.get('someData').data);
}, 1000);
```

위 코드에서 `obj`가 `map`에 저장되어 있고, `setInterval`에 의해 `map`이 지속적으로 불리기 때문에 콜백이 실행될 때마다 데이터의 존재가 보장된다.  
반면 `WeakRef`를 사용하는 아래 코드는 그렇지 않다.

```javascript
const map = new Map();

const obj = { data: new Array(10000).join('*') };

map.set('someData', new WeakRef(obj));

setInterval(() => {
  console.log(map.get('someData').deref().data); // 참조하는 대상이 GC에 의해 회수됐다면 deref()는 undefined를 반환
}, 1000);
```

일정 시간이 지나면 GC에 의해 `obj`의 메모리가 회수된다.  
따라서 `sestInterval`의 콜백은 객체의 존재를 보장받지 못하기 때문에 콜백에서 에러가 발생한다.

### Finalizers

`WeakRef`에서 *있을 수도 있고, 없을 수도 있음*이라는 개념을 사용하기 위해서는 해당 객체가 언제 없어졌는지도 알 필요가 있다.  
`Finalizers`는 이벤트 기반으로 옵저버 패턴 또는 구독 패턴과 비슷한 패턴을 사용하고 있다.

```javascript
const weakObj = {};
const gcCallback = (value) => console.log(value);

const finalizer = new FinalizationRegistry(gcCallback);
finalizer.register(weakObj, 'GC 당한 weakObj', weakObj);
// 인자: 관심 객체, GC 되었을 때 해야할 작업 callback, 관심 객체에 대해 더이상 관심이 없어질 때 finalizer에 전달할 토큰(일반적으로 해당 객체 그 자체)

// 세 번째 인자인 토큰을 통해 unregister 가능
finalizer.unregister(weakObj);
```
