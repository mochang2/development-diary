## 0. 공부하게 된 계기

electron 개발 중 main process 리팩토링 작업을 진행했다.  
공통 부분 모듈화, 별도의 기능 추상화 등의 작업을 진행하던 중 `export default class Class`로 선언한 `Class`를 다른 파일에서 `require`로 가져오지 못해서 에러가 났고, 그것 때문에 조금 헤맨 게 있었다.  
왜 안됐을까 찾아보다가 예전에 공부했던 거 다시 상기시킬 겸, 그리고 그때 정리하지 못했던 것도 같이 정리할 겸 써본다.

JS 모듈의 역사는 [여기](https://github.com/mochang2/development-diary/blob/main/026-FE%20development%20environment.md#module)서 정리했으니 CJS, ESM의 다른 점 위주로 정리하고자 한다.

참고  
https://velog.io/@kakasoo/ESM%EA%B3%BC-CommonJS%EC%9D%98-%EC%B0%A8%EC%9D%B4  
https://ui.toast.com/weekly-pick/ko_20180716  
https://toss.tech/article/commonjs-esm-exports-field  
https://yceffort.kr/2020/08/commonjs-esmodules
https://ui.toast.com/weekly-pick/ko_20180402

## 1. 모듈 종속성 분석

### CJS

CJS는 종속성 그래프를 그리지 않고 그 순간에 필요한 모듈과 그 아래의 의존성을 불러오고 인스턴스화되어 한꺼번에 모든 평가가 이루어진다.

```js
// counter.js
const count = 5;

module.exports = { count };
```

```js
// index.js
const { count } = require('./counter');

console.log(count);
```

`index.js`에서 첫 번째 줄이 실행되면 `counter` 모듈을 로드한 뒤, `count`가 5라는 값으로 평가된다.  
이후 5가 콘솔에 출력된다.

이러한 이유 때문에 CJS는 코드 중간에 언제든 다른 모듈을 불러올 수 있다(동적 모듈이라고 표현하기도 한다).  
아래처럼 말이다.

```js
// some code
{
  // ...
  const { a } = require('./A.js');
}
```

### ESM

반면 ESM는 단순히 필요한 모듈을 가져오는 것만으로 그치지 않는다.  
인터프리터는 모듈이 실행되어야 할 코드의 순서와 함께 어떠한 종속성을 갖는지 이해하기 위해 종속성 그래프를 그린다.  
인터프리터는 진입점(entry point)에서부터 필요한 모든 코드가 탐색되고 평가될 때까지 `import / export`를 DFS 방식으로 찾아나간다.

```js
// AB.js
export function a() {}

export function b() {}
```

```js
// CD.js
import { a } from './AB';

export function c() {
  a();
}

export function d() {}
```

```js
// index.js
import { c, d } from './CD';
```

위 코드 예시를 보고 아래와 같은 그림을 그리는 것이라고 생각하면 된다.

![esm dependency tree](https://github.com/mochang2/development-diary/assets/63287638/21856a15-0ec9-44d8-b494-0485f4e547ff)

구체적으로 다음과 같은 세 과정을 거치며 독립적으로 수행된다(이 때문에 ESM을 비동기식이라고 말하는 것이다).

1. 생성(또는 파싱): `import / export`를 재귀적으로 찾아가며 모든 파일의 내용을 적재한다.

```text
main.js -> a.js -> b.js // 차례대로 모두 방문한다.
main.js -> b.js // b는 이미 방문한 지점이므로 무시된다.
b.js -> a.js // a는 이미 방문한 지점이므로 무시된다.
```

2. 인스턴스화: 각 파일에서 export된 참조를 메모리에 유지하고, 각 종속성 관계를 추적한다.

```text
b = { loaded : <uninitalized> a : <uninitalized> } // a file을 참조하고 있다.
a = { loaded : <uninitalized> b : <uninitalized> }
main
```

JS 엔진은 모듈 환경 레코드를 생성하고, 이를 통해 모듈 레코드의 변수를 관리한다.  
그다음 모든 export에 대해 속성을 찾는다.  
모듈 환경 레코드는 각 export와 연관된 속성을 추적한다.  
이때까지 값이 정해지지는 않는다.

3. 평가: 코드를 평가한다.

위 과정까지는 모듈 간 링크를 만든 것일 뿐, 실질적인 코드가 있는 것이 아니다.  
실제로 코드가 동작할 수 있도록 코드를 평가한다(간단하게 값을 할당하는 거라고 생각하면 편하다).  
이 과정은 DFS의 반대인, 아래에서 위로 올라가 마지막 파일인 entry point를 평가한다.

## 2. 장단점

### 동적 로딩(yes CJS, no ESM)

모듈의 경로가 동적으로 결정되는 경우가 있을 수 있다.

```js
const path = makeDynamicPath();
const module = require(path);
```

위와 같은 방식으로는 가능하지만, 아래와 같은 방식으로는 불가능한 표현이다.

```js
const path = makeDynamicPath();
import module from path;

// 그치만 아래는 가능하다
import(path).then(
  // ...
)
```

### 기본 옵션(yes CJS, no ESM)

ESM은 자바스크립트의 많은 부분에 변경이 필요하다.  
ESM은 일단 기본 값으로 `use strict`가 설정되어 있어야하고, `this`는 global object를 참조하지 않고, 스코프는 다르게 작동 되는 등등 변화가 많다.
이것이 브라우저에서 조차 `<script>`가 기본적으로 `type="module"` 없이는 ESM으로 동작하지 않는 이유다.

_cf) 기본 값을 CJS에서 ESM으로 바꾸는 것은 호환성을 해치는 문제가 된다(node의 대안으로 주목받고 있는 deno는 ESM을 기본값으로 사용하지만, 결과적으로 모든 생태계를 처음부터 다시 설계해야 했다)._

### 순환 참조 감지(yes ESM, no CJS)

ESM은 CJS와 달리 서로에 대한 완전한 참조를 가지게 된다.  
반면 CJS는 서로에 참조에 대해 시간 차가 발생한다.

예를 들어 a가 b를 부른 후 b가 a를 부르는 구조가 되어 있을 때,  
a는 b를 가지고 있지만 a가 가진 b에는 a가 아직 존재하지 않는다.

```text
a = { b : { } } // a가 가지고 있는 종속성
b = { a : { b } } // b가 가지고 있는 종속성
```

하지만 ESM은 종속성 트리를 만들기 때문에 순환 참조를 감지하고 이를 사전에 방지할 수 있다.

### 트리 쉐이킹(yes ESM, no CJS)

트리 쉐이킹이란 사용하지 않는 코드를 제거하는 방식으로 웹 페이지 성능을 올릴 수 있는 한 가지 방법이다.  
ESM에서 모듈 종속성을 분석할 수 있기 때문에 사용하지 않는 코드를 사전에(빌드 시) 감지하고 모듈에서 이를 제거할 수 있다(ESM은 정적 모듈이기 때문이다).

## 3. 참고

ESM와 CJS는 서로를 호출할 수 있다.  
다만 제약이 있다.

1. ESM에서는 `require()`를 사용할 수는 없다. 오로지 `import`만 가능하다.
2. CJS도 마찬가지로 `import`를 사용할 수는 없다.
3. ESM에서 CJS를 `import`하여 사용할 수 있다. 그러나 오로지 default import만 가능하다. 그러나 CJS가 named export를 사용하고 있다면 named import 인 `import { shuffle } from 'lodash'`와 같은 것은 불가능하다.
4. ESM을 CJS에서 `require()`로 가져올 수는 있다. 그러나 이는 별로 권장되지 않는다. 그 이유는 이를 사용하기 위해서는 더 많은 boilerplate가 필요하고, 최악의 경우 Webpack이나 Rollup 같은 번들러도 필요하다. ESM이 `require()`에서 어떻게 동작해야 하는지 모르기 때문이다.
5. CJS는 기본값으로 지정되어 있다. 따라서 ESM 모드를 사용하기 위해서는 opt-in해야 한다. `.js`를 `.mjs`로 바꾸거나, `package.json`에 `"type": "module"` 옵션을 넣는 방법이 있다.
