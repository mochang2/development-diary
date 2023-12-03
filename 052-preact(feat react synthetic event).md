## 0. 공부하게 된 계기

[Preact로 번들 사이즈 대폭 줄이기](https://helloinyong.tistory.com/355)라는 글을 보고 Preact가 뭐지...? 싶어서 한 번 읽어봤다.

~참고로 2023.10.17 기준으로 작성 중인 이 글은 정화히 말하자만 Preact를 다루는 것은 아니고 React의 synthetic event와 관련된 내용이다...ㅎ~

참고  
https://preactjs.com/guide/v10/differences-to-react/  
https://handhand.tistory.com/287  
https://medium.com/hcleedev/web-react%EC%9D%98-event-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EB%82%B4%EB%B6%80-%EA%B5%AC%ED%98%84-%EC%9E%90%EC%84%B8%ED%9E%88-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0-react-v18-2-0-39d59ab45bec  
https://react.dev/reference/react-dom/components/common

## 1. vs React

아래는 공식 문서에서 가져와 번역한 내용이다.

> Preact는 React를 보강하려고 만든 것이 아니다. 그 둘 사이에는 명확한 차이점이 있다. 대부분 그 차이점은 사소하고 `preact/compat`(React와 100% 호환성을 가지기 위해 사용하는 "thin"한 layer)를 사용해 제거할 수 있다.
> Preact가 React의 모든 특징을 사용하려 하지 않는 이유는 조금 더 작고 집중적으로 유지하기 위해서이다(그렇지 않으면 이미 매우 복잡하고 잘 설계된 코드베이스인 리액트 프로젝트에 단순히 최적화를 제출하는 것이 더 합리적일 것이다).

> React와 비교할 때 가장 큰 차이점은 Preact는 "synthetic event" 시스템을 사용하지 않고 번들 크기와 최적화를 달성했다는 점이다. Preact는 브라우저 표준인 `addEventListener`를 이용해서 이벤트 등록을 했고, 이는 바닐라 JS에서 사용하는 것과 똑같은 이벤트명을 사용할 수 있다는 뜻이다.
> 표준 브라우저 이벤트는 React에서 이벤트가 작동하는 방식과 매우 유사하게 작동하며 약간의 작은 차이점이 있다. Preact는:
>
> -   이벤트가 `<Portal>` 컴포넌트를 통해 bubble up 되지 않음.
> -   `onChagne` 대신 `onInput`을 사용해야 함(단, `preact/compat`을 사용하지 않는다는 조건 하에).
> -   `onDoubleClick` 대신 `onDblClick`을 사용해야 함(단, `preact/compat`을 사용하지 않는다는 조건 하에).
> -   "x" 버튼이 IE11에서는 `onInput`을 작동시키지 않기 때문에 `<input type="search">`에 대해서는 `onSearch`를 사용해야 함.

Preact가 이런 장점이 있으면 왜 React는 안 쓰지? 싶었는데 이 궁금증은 ChatGPT가 해결해줬다 ㅎ.  
하지만 React는 이미 큰 커뮤니티와 방대한 코드베이스에서 Synthetic Event를 사용하고 있기 때문에 큰 변화를 가하려면 많은 노력과 호환성 문제가 발생할 것이다.  
또한 React의 Synthetic Event는 특정한 사용 사례와 생태계에 특화되어 있으며, Synthetic Event를 버리고 실제 DOM 이벤트 위임 방식으로 전환하기 어려울 것이다.  
반면 Preact는 처음부터 이러한 방식으로 설계되어 React보다 경량하게 유지되고 있다.

### Synthetic Event(합성 이벤트)

가장 큰 차이점이 synthetic event를 사용하지 않는다는 점인데, 그게 무엇인지 React에서는 왜 사용했던 건지 짚고 넘어가보자.  
참고로 아래 React 코드들은 v18.2 기준이다.

#### React Root(Native Event Listener)

React 17부터 CRA를 하면 최상단 `index.js`에는 아래와 같은 코드가 생성된다.

```js
import React from "react";
import ReactDOM from "react-dom/client";
import "./index.css";
import App from "./App";

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(
    <React.StrictMode>
        <App />
    </React.StrictMode>
);

React.createElement();
```

이와 관련된 설명을 [(legacy)리액트 공식 문서](https://legacy.reactjs.org/blog/2020/10/20/react-v17.html)에서 확인할 수 있다.

![react root](https://github.com/mochang2/development-diary/assets/63287638/4e9565a2-961e-46e4-a69c-0021940b2681)

기존에는 `document`가 native event listener였지만, React 17부터는 `document`가 아닌 `root`가 native event listener이다.  
이 말은 `<button onClick={() => console.log('btn')}>`을 `root.addEventListener('click', () => console.log('btn'))`로 변형한다는 뜻이 아니다.  
Native Event('click' 등)을 listen하고 발생하는 이벤트에 따라 '알맞는 타겟의 이벤트 핸들러를 실행시킨다'는 의미이다.

이전까지 `document`에서 event delegation 패턴을 사용하던 부분을 바꾼 이유는 **React와 Non-React 코드 베이스를 합쳐서 하나의 어플리케이션을 만들 때** `event.stopPropagation()` 등이 원하는 대로 동작하지 않을 수 있기 때문이다.  
그래서 과거에는 각각의 React 앱을 `iframe`으로 감싸는 방식을 사용했다고 한다.

아래 코드를 보면 `createRoot`의 동작 과정 이해에 도움이 될 것이다.

```ts
// https://github.com/facebook/react/blob/v18.2.0/packages/react-dom/src/client/ReactDOM.js
export function createRoot(
  container: Element | Document | DocumentFragment,
  options?: CreateRootOptions,
): RootType {
  // 생략 ...

  const rootContainerElement: Document | Element | DocumentFragment =
    container.nodeType === COMMENT_NODE
      ? (container.parentNode: any)
      : container;
  listenToAllSupportedEvents(rootContainerElement);

  return new ReactDOMRoot(root);
}


// https://github.com/facebook/react/blob/v18.2.0/packages/react-dom/src/events/DOMPluginEventSystem.js
// allNativeEvents.forEach 를 통해 모든 종류의 Native Event를 root에 붙이는 작업을 진행한다.
// listenToNativeEvent 는 내부에서 rootContainerElement 에다가 addEventListener 를 이용해 Listener를 붙이는 역할을 한다.
export function listenToAllSupportedEvents(rootContainerElement: EventTarget) {
  if (!(rootContainerElement: any)[listeningMarker]) {
    (rootContainerElement: any)[listeningMarker] = true;
    allNativeEvents.forEach(domEventName => {
      // We handle selectionchange separately because it
      // doesn't bubble and needs to be on the document.
      if (domEventName !== 'selectionchange') {
        if (!nonDelegatedEvents.has(domEventName)) {
          listenToNativeEvent(domEventName, false, rootContainerElement);
        }
        listenToNativeEvent(domEventName, true, rootContainerElement);
      }
    });

    // 생략...
  }
}
```

#### Native Event to Synthetic Event

native event를 받기 위한 listener는 생성됐으니, native event 발생 시 이를 React에서 처리하기 위한 과정이 필요하다.  
이 과정에서 native event를 `SyntheticEvent` 타입으로 감싸주는 과정을 거친다.

```ts
declare class SyntheticEvent<+T: EventTarget = EventTarget, +E: Event = Event> {
    bubbles: boolean;
    cancelable: boolean;
    +currentTarget: T;
    // ...
    nativeEvent: E;
    preventDefault(): void;
    stopPropagation(): void;
    +target: EventTarget;
    // ...
    persist(): void;
}

// https://github.com/facebook/react/blob/v18.2.0/packages/react-dom/src/events/DOMEventProperties.js
export const topLevelEventsToReactNames: Map<
  DOMEventName,
  string | null,
> = new Map();

const simpleEventPluginEvents = [
  'abort',
  'auxClick',
  'cancel',
  'canPlay',
  'canPlayThrough',
  'click',
  // ...
  'touchMove',
  'waiting',
  'wheel',
];

// ...

function registerSimpleEvent(domEventName, reactName) {
  topLevelEventsToReactNames.set(domEventName, reactName);
  registerTwoPhaseEvent(reactName, [domEventName]);
}

// click 이벤트 이름 -> onClick(props)으로 변경하는 작업을 한다.
export function registerSimpleEvents() {
  for (let i = 0; i < simpleEventPluginEvents.length; i++) {
    const eventName = ((simpleEventPluginEvents[i]: any): string);
    const domEventName = ((eventName.toLowerCase(): any): DOMEventName);
    const capitalizedEvent = eventName[0].toUpperCase() + eventName.slice(1);
    registerSimpleEvent(domEventName, 'on' + capitalizedEvent);
  }

  // 생략 ...
}


// https://github.com/facebook/react/blob/v18.2.0/packages/react-dom/src/events/EventRegistry.js
// bubble 단계와 capture 단계 이벤트 리스너 props 이름을 등록한다.
export function registerTwoPhaseEvent(
  registrationName: string,
  dependencies: Array<DOMEventName>,
): void {
  registerDirectEvent(registrationName, dependencies);
  registerDirectEvent(registrationName + 'Capture', dependencies);
}

export function registerDirectEvent(
  registrationName: string,
  dependencies: Array<DOMEventName>,
) {

  registrationNameDependencies[registrationName] = dependencies;

  // 생략 ...

  for (let i = 0; i < dependencies.length; i++) {
    allNativeEvents.add(dependencies[i]);
  }
}
```

여기까지의 작업을 요약하면 다음과 같다.

1. 앱이 실행될 때 미리 준비해뒀던 native event 목록을 이용해, `click` 을 `onClick` 으로 바꿔서 저장한다.
2. native event가 들어올 때 `SyntheticEvent`로 변환해줄 메서드, 즉 생성자도 미리 선언한다.
3. 비로소 `createRoot`가 실행되면 그때 `root`에 native event 목록을 참고해 들어올 수 있는 event에 대해 `addEventListener`로 등록한다.

그렇다면 React에서 이렇게 복잡한 작업을 하면서까지 native event를 synthetic event로 변환하는 이유는 뭘까?  
이유는 이벤트 처리에 대한 크로스 브라우징 이슈를 해결하고 개발자에게 일관된 경험을 제공하기 위해서이다.  
브라우저 간 이벤트 처리에는 다음과 같은 부분에서 차이점이 있다.

-   event bubbling과 cpaturing: 브라우저마다 동작 방식이 다를 수 있다.
-   event target과 currentTarget: 이벤트가 발생한 요소(이벤트 타겟)와 현재 이벤트가 처리되고 있는 요소(현재 타겟) 사이의 차이가 브라우저마다 다를 수 있다.
-   property: 브라우저마다 이벤트 객체의 속성 및 메서드 이름, 지원하는 이벤트 유형 등이 조금씩 다를 수 있다.

React의 synthetic event 시스템은 이러한 차이점을 추상화하고, 개발자가 브라우저별로 다른 코드를 작성하지 않도록 돕는다.  
그러나 synthetic event를 사용하면 이벤트 핸들러 호출 및 관리, 이벤트 객체 생성 및 관리 등으로 인해 React에서는 약간의 오버헤드가 발생할 수 있다.

_참고) synthetic instance pool(지금은 사라진 기능)_  
React 17 이전에는 `SyntheticEvent` 사용에 따른 메모리 최적화를 위해 `Event pooling`이라는 것을 사용했다.  
React는 synthetic event 사용 때문에 이벤트가 발생할 때마다 인스턴스를 생성해야 하고, 필요 없어진 경우에는 GC에 의한 삭제가 필요하다.  
이 경우 이벤트가 많은 경우 메모리에, GC가 수행될 경우 CPU에 부담이 가기 때문에 인스턴스를 재활용하는 "synthetic instance pool"을 사용하려 했지만 아래와 같은 비동기 콜백을 다룰 때 문제가 생겼다.  
React가 특정 이벤트의 핸들러를 실행시킨 뒤 다시 pool에 넣기 위해 event의 모든 속성을 `null`로 초기화시켰는데

```jsx
const handleState = (event) => {
    this.setState({
        name: event.target.value,
    });
};
```

위와 같은 경우 이러한 초기화 과정 때문에 문제가 생겼다고 한다.  
그래서 아래와 같이 해결했다고 한다.

```jsx
const handleState = (event) => {
    event.persist();
    this.setState({
        name: event.target.value,
    });
};

// 또는

const handleState = (event) => {
    const name = event.targe.value;
    this.setState({
        name,
    });
};
```

#### Event Dispatch

```ts
// https://github.com/facebook/react/blob/v18.2.0/packages/react-dom/src/events/DOMPluginEventSystem.js
function addTrappedEventListener(
  targetContainer: EventTarget,
  domEventName: DOMEventName,
  eventSystemFlags: EventSystemFlags,
  isCapturePhaseListener: boolean,
  isDeferredListenerForLegacyFBSupport?: boolean,
) {
  let listener = createEventListenerWrapperWithPriority(
    targetContainer,
    domEventName,
    eventSystemFlags,
  );
  // ... 중략 ...
  } else {
    if (isPassiveListener !== undefined) {
      unsubscribeListener = addEventBubbleListenerWithPassiveFlag(
        targetContainer,
        domEventName,
        listener,
        isPassiveListener,
      );
    } else {
      unsubscribeListener = addEventBubbleListener(
        targetContainer,
        domEventName,
        listener,
      );
    }
  }
}

// https://github.com/facebook/react/blob/v18.2.0/packages/react-dom/src/events/ReactDOMEventListener.js
export function createEventListenerWrapperWithPriority(
  targetContainer: EventTarget,
  domEventName: DOMEventName,
  eventSystemFlags: EventSystemFlags,
): Function {
  const eventPriority = getEventPriority(domEventName);
  let listenerWrapper;
  switch (eventPriority) {
    case DiscreteEventPriority:
      listenerWrapper = dispatchDiscreteEvent;
      break;
    case ContinuousEventPriority:
      listenerWrapper = dispatchContinuousEvent;
      break;
    case DefaultEventPriority:
    default:
      listenerWrapper = dispatchEvent;
      break;
  }
  return listenerWrapper.bind(
    null,
    domEventName,
    eventSystemFlags,
    targetContainer,
  );
}
```

나머지 코드들은 너무 세세한 구현 사항이라 생략했다.  
간단하게 요약하면 다음과 같다.

1. native event가 발생했음을 `root`가 인지하고 현재 target에 붙어있는 이벤트에 대한 핸들러부터 `root`에 있는 관련 핸들러까지 모두 모아서 전달하며 `dispatchEvent`가 호출된다.
2. 받아온 naitive event를 분류해서 적절한 `SyntheticEvent`로 추출하고, target에 대한 핸들러를 뽑아서 `dispatchQueue`에 넣는다.
3. `dispatchQueue`에 들어있는 이벤드 핸들러들을 순서대로 쭉 실행한다.

#### Preact Event 처리

~아직 세부 구현사항까지 파헤쳐본 것은 아니라서 코드 구현단에서 차이점이 뭐냐고 물으면 모르겠다... 나중에 기회가 되면(? 안 되면 안 하고...) 더 자세히 추가할 수 있도록~

-   Preact는 Synthetic Event 대신에 (virtual DOM을 이용하는 것이 아닌) 실제 DOM 이벤트 위임을 사용하여 이러한 오버헤드를 줄이고 더 가벼운 구현을 제공한다.

```text
실제 DOM 이벤트 위임: Preact는 실제 DOM 이벤트 위임을 사용하여 이벤트 처리를 관리. 이것은 브라우저에서 이벤트가 DOM 요소를 통해 버블링하거나 캡처링하는 방식을 활용하여 크로스 브라우징 문제를 자동으로 처리함. Preact는 DOM 요소에 직접 이벤트를 등록하는 대신, 더 효율적인 이벤트 위임을 통해 이벤트를 처리하므로 오버헤드가 적음.
가벼운 이벤트 시스템: Preact의 이벤트 시스템은 React의 Synthetic Event보다 더 경량임. Synthetic Event는 브라우저 이벤트를 래핑하고 여러 메서드 및 속성을 추가하는 작업이 필요한 반면, Preact의 이벤트 처리는 더 단순함.
```
