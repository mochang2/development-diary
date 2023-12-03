## 0. 공부하게 된 계기

딱히 큰 이유는 없다.  
그저 react의 대안책이라 불릴 수 있는 vue나 svelte가 굳이 단방향 데이터 바인딩을 사용하지 않은 이유에 대해 궁금했다.

참고로 데이터가 변경됐음을 감지하는 방법이 여러 가지가 있지만, 각각의 라이브러리나 프레임워크는 각자의 장점을 최대화할 수 있는 방법을 통해 변화를 감지한다.  
"아주 간단히" 만들기 위해서라면 단순히 `MutationObserver`를 사용할 수 있을 것이다.

참고  
https://velog.io/@sunaaank/data-binding  
https://adjh54.tistory.com/49  
https://oliviakim.tistory.com/91  
https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Proxy  
https://ko.vuejs.org/guide/extras/reactivity-in-depth.html#how-reactivity-works-in-vue  
https://ko.vuejs.org/guide/extras/rendering-mechanism.html  
https://pinokio0702.tistory.com/363  
https://medium.com/@youyuxi/re-performance-261023557027

## 1. 데이터 바인딩

두 데이터 혹은 정보의 소스를 일치시키는 기법으로, 화면에 보이는 데이터와 브라우저 메모리에 있는 데이터(자바스크립트 객체)를 일치시키는 것을 말한다.  
MVC 모델에서 model과 view를 서로 묶어 model과 view의 "자동 동기화" 시키기 라고 이해할 수 있다.  
예를 들어서 HTML에서 서버 혹은 스크립트상에서 받아온 데이터를 화면상에 그려주고 있다고 가정을 했을 때, 해당 값이 변경이 될 경우 다시 HTML 상에 데이터(값)를 변경된 값에 따라서 맞추어 주는 동작을 '데이터 바인딩'이라고 한다.

## 2. 단방향 데이터 바인딩

JS(model) -> HTML(view) 한 방향으로만 데이터를 동기화하는 것을 말한다.  
일반적으로 데이터 모델에서 UI로의 전달이나 UI에서 데이터 모델로의 역전파가 없는 경우에 해당한다.  
JS에서 변경된 데이터가 화면에 반영되지만, 화면에서 직접 해당 값을 바꾸도록 하려면 사용자(개발자)가 동기화 로직을 작성해줘야 한다.  
React는 virtual DOM을 통해 단방향 데이터 바인딩을 구현했다.

```jsx
const Component = () => {
  const [text, setText] = useState('');

  function handleText(event) {
    setText(event.target.value);
  }

  return <input type="text" value={text} onChange={handleText} />;
};
```

(양방향 데이터 바인딩과 비교할 때)

- 장점
  - 모든 JS 코드가 데이터에 집중되며 일관된 데이터 로직을 가짐
  - 데이터의 변화에 따른 성능 저하 없이 DOM 객체 갱신이 가능
- 단점
  - HTML -> JS로의 데이터 동기화에 대한 로직을 매번 직접 작성해야 하기 때문에 코드 양이 많아짐

## 3. 양방향 데이터 바인딩

JS(model) -> HTML(view), HTML(view) -> JS(model) 양방향으로 데이터를 동기화하는 것을 말한다.  
사용자가 UI를 통해 데이터를 수정하면 데이터 모델이 자동으로 업데이트되고, 반대로 데이터 모델이 변경되면 UI가 업데이트되는 것을 의미한다.  
단방향 데이터 바인딩과 달리 화면을 통해 직접 값을 바꿀 수 있기 때문에 모든 동기화 로직을 사용자(개발자)가 작성할 필요는 없다.
Vue는 반응적 시스템과 virtual DOM을 통해 양방향 데이터 바인딩을 구현했다.

```vue
<!-- 예제 코드는 Vue가 더 길어 보이지만... 바인딩해야 할 데이터가 많아진다면 Vue가 훨씬 짧다 -->
<template>
  <input v-model="text" />
</template>

<script setup>
export default {
  setup() {
    const text = ref('');

    return {
      text,
    };
  },
};
</script>
```

(단방향 데이터 바인딩과 비교할 때)

- 장점
  - 동기화 로직을 직접 작성할 필요가 없기 때문에 코드 양이 줄어듦
- 단점
  - 변화에 따라 DOM 객체 전체를 렌더링해주거나 데이터를 바꿔주므로 성능이 상대적으로 감소되는 경우가 있음(대부분의 인기 있는 프레임워크들은 이를 극복하는 각자의 방식이 있음)
  - 변경 사항 추적이 어려울 수 있음(이는 익숙하지 않은 사람에게 해당하는 문제라고 생각함)

## 4. Vue 성능

~예전에 React를 직접 만든 것만큼 상세히 다루지는 않을 예정이다.~

똑같은 어플리케이션을 React와 Vue로 각각 만들 때 Vue의 성능이 미세하지만 더 뛰어나다고 한다(계속 변화해서 현재도 사실인지는 모르겠다).  
실제로 특정 거래소 사이트들은 이러한 이유로 Vue로 만들기도 한다.  
Vue가 성능을 최적화하는 데는 크게 세 가지 방법이 있다.

1. 반응적 시스템(Reactivity System)

Vue는 데이터 변화를 감지하기 위한 반응적 시스템을 내장하고 있다.  
이를 통해 특정 데이터의 변경에 대한 응답이 가능하며, 세밀한 제어가 가능하다.

Vue2에서는 브라우저 지원 제한으로 인해 단독으로 getter/setter를 사용했지만, Vue3에서는 Proxy를 이용해서 이를 구현했다.  
아래는 공식 문서에서 제공한 대략적인 수도 코드이다.

```js
function reactive(obj) {
  return new Proxy(obj, {
    get(target, key) {
      track(target, key);
      return target[key];
    },
    set(target, key, value) {
      target[key] = value;
      trigger(target, key);
    },
  });
}

function ref(value) {
  const refObject = {
    get value() {
      track(refObject, 'value');
      return value;
    },
    set value(newValue) {
      value = newValue;
      trigger(refObject, 'value');
    },
  };
  return refObject;
}
```

`track`는 현재 실행 중인 이팩트가 있는지 확인한다.  
존재하는 경우, 추적 중인 속성에 대한 구독자 이팩트(`Set`에 저장됨)를 조회하고 이팩트를 `Set`에 추가한다.  
이팩트 구독은 전역 `WeakMap<target, Map<key, Set<effect>>>` 데이터 구조에 저장된다.  
`trigger`는 속성에 대한 구독자 이팩트를 다시 조회한 후 호출한다.

```js
let activeEffect

function track(target, key) {
  if (activeEffect) {
    const effects = getSubscribersForProperty(target, key) // 속성에 대한 구독 이팩트 `Set`이 발견되지 않은 경우(처음 추적 시) 생성
    effects.add(activeEffect)
  }

function trigger(target, key) {
  const effects = getSubscribersForProperty(target, key)) // 속성에 대한 구독 이팩트 `Set`이 발견되지 않은 경우(처음 추적 시) 생성
  effects.forEach((effect) => effect())
}
```

2. Virtual DOM

Virtual DOM은 특정 언어, 프레임워크에 종속된 개념이 아니고 패턴에 가깝다.  
[virtual DOM](https://github.com/mochang2/development-diary/blob/main/029-virtual%20DOM.md)에서 더 자세한 설명이 나와 있다.

(알고리즘이 정확히 어떻게 다른지는 모르겠지만)  
Vue가 React와 다른 점이 있다면, 컴포넌트 re-rendering하는 방법이다.  
React에서는 메모이제이션이나 `props.children`과 같은 방법이 없다면, 부모 컴포넌트의 `state`나 `props`가 변경되면 하위 모든 컴포넌트에서 re-rendering이 발생한다.  
하지만 Vue는 그 변경된 값이 부모 컴포넌트로부터 `props`로 전달받지 않은 값이라면, 하위 컴포넌트에서는 re-rendering이 발생하지 않는다.

3. SFC(Single-File Components)

SFC는 관심사가 같은 코드들을 하나의 파일로 관리, scoped 스타일 외에도 **성능**과 관련해서 이점이 있다.  
vue compiler를 이용하여 Vue는 빌드 시에 여러 가지 최적화가 가능하다.

- pre-compilation: Vue 컴포넌트의 템플릿은 런타임에서 처리되는데, 개발 중에는 이 템플릿이 문자열로 유지되지만, 프로덕션 환경에서는 이를 미리 컴파일하여 JS 렌더링 함수로 변환된다. 이로써 템플릿을 파싱하고 렌더링 함수로 변환하는 과정이 최적화되어 초기 렌더링 성능이 향상된다.
- static node hoisting: 템플릿에서 반복되는 정적인 요소들은 렌더링 함수에서 미리 추출되어, 런타임에 중복 계산을 피하고 성능을 향상시킨다.
- inline rendering functions: 템플릿에서 자주 사용되는 함수는 미리 인라인되어 최적화되어, 런타임에서의 함수 호출 오버헤드를 줄인다.
- automatic vendor prefixing: 스타일에 적용된 속성들은 빌드 시 자동으로 벤더 프리픽스가 적용되어 여러 브라우저에서의 일관된 렌더링을 지원한다.
- CSS node separation and minification: 빌드 시에는 스타일 코드를 분리하고 최소화하여 불필요한 공백 및 주석을 제거하고, CSS 파일을 압축하여 전송 크기를 줄인다.
- code splitting: 초기 로딩 시 필요한 최소한의 코드만 다운로드하여 로딩 속도를 향상시킨다.
