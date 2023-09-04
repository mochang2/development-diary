## 0. 공부하게 된 계기

이번 페이지는 다른 페이지들과는 조금 다른 맛?이다.  
[슬랙봇 만든 후기](https://github.com/mochang2/development-diary/blob/main/017-slack%20bot.md)를 적었던 거 같이 개발하면서 새로 공부하고 고민이 됐던 부분 정리하려고 한다.

## 1. 개요

기본적으로 내가 만든 바텀 시트는 다음과 같다.  
![bottom sheet layout](https://github.com/mochang2/development-diary/assets/63287638/897ae67d-e1c2-47e1-9cfa-dfc31724f097)

1. 바텀 시트는 open되면 아래에서 올라오고, close되면 아래로 내려간다.
2. wrapper(background) 영역과 content 영역이 나뉘어져 있다.
3. wrapper 영역을 누르면 바텀 시트가 close된다.
4. close 버튼 사용 여부를 바텀 시트를 사용하는 쪽에서 결정한다.
5. 스크롤 시 body의 스크롤 영역이 움직이면 안 된다.
6. 바텀 시트가 open된 상태에서 뒤로 가기 버튼 클릭 시 바텀 시트가 close된다.
7. 햄버거 메뉴가 존재하는데, "내가 받은 롤링페이퍼(바텀 시트에서 보여주는 내용)"와 같은 anchor를 클릭하면 페이지 이동 시 바텀 시트가 open된 상태여야 한다.
8. 사용자(페이지 방문자)가 바텀 시트 높이 조절하는 기능은 없다.

최대한 서드 파티 라이브러리를 사용하지 않고 웹 페이지에서 바텀 시트를 만들고 싶어서 이것저것 시도해봤는데, 5가지 정도 예상치 못한 버그가 생겼었고, 그 중 3가지는 이미 알고 있던 내용으로 간단히 해결했다.  
첫 번째는 content 영역을 클릭할 때도 바텀 시트가 close되는 것이었다.  
이 내용은 content 영역에 클릭 시 `event.stopPropagation()`으로 해결했다.  
두 번째는 close 버튼 사용 여부에 따른 content 영역 조절이었다.  
close 버튼을 wrapper로 감싼 후 `position: sticky;`와 `padding: calc(close 버튼 height + wrapper에 대한 적절한 padding)`을 줘서 해결했다.  
세 번째는 직접 url을 입력해서 바텀 시트가 존재하는 페이지(마이 페이지)로 이동할 때, 바텀 시트가 아래로 내려가는 애니메이션이 동작하는 것이었다(기대하는 방식대로라면 바텀 시트가 처음에는 전혀 안 보였어야 한다).  
심지어 간헐적으로 동작해서 원인을 찾기 까다로웠는데 해결 방법은 의외로 간단했다.  
기본적으로 바텀 시트에 `top: 100vh`를 줬으면 됐다.

```scss
#bottom-sheet.background {
  $TRANSLATE_TIME: 0.5s;
  $TRANSFORM: top $TRANSLATE_TIME ease-in-out;

  top: 100vh;

  &.open {
    background-color: rgba(#000, 0.3);

    top: 0;
    transition: $TRANSFORM, background-color 0s linear $TRANSLATE_TIME;
  }

  &.close {
    background-color: transparent;

    top: 100vh;
    transition: $TRANSFORM, background-color 0s linear 0s;
  }
}
```

나머지 해결하기 까다로웠던 두 가지가 **스크롤 시 body의 스크롤 영역이 움직이면 안 된다**는 것과 **바텀 시트가 open된 상태에서 뒤로 가기 버튼 클릭 시 바텀 시트가 close되고, 햄버거 메뉴에서 바텀 시트가 존재하는 페이지로 이동하는 버튼 클릭 시 바텀 시트가 open된 상태로 노출되어야 한다**는 것이었다.

## 2. 스크롤

기본적으로 바텀 시트가 open되어 있을 때, 일반적으로 PC에서 마우스 스크롤, 모바일에서 스와이프가 바텀 시트에서 잘 동작했다.  
다만 간헐적으로 포커스가 body에 되고, body의 스크롤이 움직일 때가 있었다.  
의도치 않았기 때문에 이는 명확히 버그였고, 이를 해결하고자 했다.

### `preventDefault`, `stopPropagation` (X)

React에서 `HTMLDivElement`에는 `onScroll`(`document.addEventListener("scroll")`), `onScrollCapture` 이 존재한다.  
이 이벤트를 감지해서 `preventDefault`나 `stopPropagation`을 해서 바텀 시트가 open되어 있을 때 body의 스크롤을 막을 수 있을 거라 생각했지만 문제가 해결되지 않았다.

### `overscroll-behavior` (X)

내부 element에 스크롤 이벤트가 발생할 때 어떻게 처리할지 결정할 수 있는 css 속성이다.  
중첩된 element에 **둘 다** 스크롤이 존재할 때 마우스가 어디에 포커싱되어 있냐에 따라 행동을 제어할 수 있다.  
이 css 속성을 사용하면 바텀 시트에도 스크롤이 있을 때 body에 스크롤을 제어할 수 있었지만, 바텀 시트에 스크롤이 존재하지 않으면 여전히 body의 스크롤이 움직였다.

### `overflow-y` (X)

바텀 시트가 open되었을 때 body의 scroll을 `hidden`으로 만들고, 바텀 시트가 close되었을 때 body의 scroll을 `auto`로 돌려놓는 방법이다.  
원하는 기능은 구현되었지만 바텀 시트가 open, close될 때마다 스크롤이 생겼다, 감춰졌다 하면서 layout shift가 발생했다.  
scroll에 disabled 같은 속성은 없어서 다른 방법은 없었고, 이는 최선의 방법이 아닌 거 같아서 다른 방법을 다시 찾았다.

### scrollbar-width, overflow, position, top, width (O)

우선 모든 요소의 `scrollbar-width`를 없도록 만들었다.  
이렇게 하면 스크롤은 정상적으로 동작하지만 화면에서는 보이지 않는다.  
애초에 진행하고 있는 프로젝트가 모바일 이용자를 대상으로 만들고 있기 때문에 UI 상 크게 차이가 없었다.

```scss
* {
  scrollbar-width: none; /* 파이어폭스 */

  &::-webkit-scrollbar {
    display: none; /* 크롬, 사파리, 오페라, 엣지 */
  }
}
```

이러면 `overflow` 속성에 따른 layout shift 발생을 방지할 수 있다.  
또한 인터넷 서칭 중 IOS Chorome/Safari에선 `overflow: hidden;`이 정상적으로 작동되지 않는 경우가 있다고 해서 현재 body의 스크롤 위치를 기억해놓을 수 있는 코드를 추가했다.  
아래는 해당 로직을 훅으로 별도 분리한 코드이다.

```tsx
// use-body-scroll-lock.ts

type TScrollLock = {
  lockScroll: () => void;
  unlockScroll: () => void;
};

export default function useBodyScrollLock(): TScrollLock {
  let scrollPosition = 0;

  const lockScroll = () => {
    scrollPosition = window.scrollY;
    document.body.style.overflow = 'hidden';
    document.body.style.position = 'fixed';
    document.body.style.top = `-${scrollPosition}px`;
    document.body.style.width = '100%';
  };

  const unlockScroll = () => {
    document.body.style.removeProperty('overflow');
    document.body.style.removeProperty('position');
    document.body.style.removeProperty('top');
    document.body.style.removeProperty('width');
    window.scrollTo(0, scrollPosition);
  };

  return { lockScroll, unlockScroll };
}
```

```tsx
// BottomSheet.tsx

'use client';

import {
  useEffect,
  type ReactNode,
  type HTMLAttributes,
  type MouseEvent,
} from 'react';
import { Close } from '@/assets/icons';
import useBodyScrollLock from '@/hooks/use-body-scroll-lock';

type TProps = {
  children: ReactNode;
  isOpen: boolean;
  onClose: () => void;
  isHandlingHistory?: boolean;
  isShowingClose?: boolean;
  isControllingScroll?: boolean;
} & HTMLAttributes<HTMLDivElement>;

const BottomSheet = ({
  children,
  isOpen,
  onClose,
  isHandlingHistory = true,
  isShowingClose = true,
  isControllingScroll = true,
  ...props
}: TProps) => {
  // 기타 다른 로직

  const { lockScroll, unlockScroll } = useBodyScrollLock();

  useEffect(() => {
    if (!isControllingScroll) {
      return;
    }

    if (isOpen) {
      lockScroll();
    } else {
      unlockScroll();
    }
  }, [isControllingScroll, isOpen, lockScroll, unlockScroll]);

  return (
    <article
      id="bottom-sheet"
      className={`${isOpen ? 'open' : 'close'} background`}
      onClick={onClose}
    >
      <div className="wrapper" onClick={handlePropagation} {...props}>
        {isShowingClose && (
          <div className="close">
            <button onClick={onClose} className="fl-r">
              <Close className="sm" />
            </button>
          </div>
        )}
        <div className="content">{children}</div>
      </div>
    </article>
  );
};

export default BottomSheet;
```

## 3. history API

브라우저 히스토리(스택)를 조작하는 API이다.  
참고로 [BrowserRouter vs HistoryRouter](https://github.com/mochang2/development-diary/blob/main/033-vanilla%20js.md#9-history)에서도 비교했듯이 history API는 SPA에서 라우팅을 위해 자주 사용되는 API이다(물론 화면을 바꿔야하기 때문에 라이브러리나 프레임워크에 따라 추가적인 조작은 가미된다).  
~(이건 정확하진 않은데... 그래서 내 기억에 React에서 단순히 `history.pushState` 사용하면 state는 변경되어도 원하는 페이지로 바뀌지 않는다)~

### location API

반면 이 API는 현재 페이지의 URL을 조작하기 위한 API이다.

React 개발 도중 `location.href` 이런 문구를 몇 번 써봤어서... 그러면 history API하고 다른 게 뭐지...? 싶어서 한 번 공부하는 김에 정리해봤다.  
가장 헷갈렸던 부분은 `history.pushState`와 `location.href`였다.  

이 두 부분만 먼저 간단히 정리하자면 `history.pushState`는 HTTP 요청이나 페이지 새로고침이 없지만 어플리케이션 상태를 유지할 수 있다.  
하지만 `location.href`는 HTTP 요청이나 페이지 새로고침이 있지만 어플리케이션 상태를 유지할 수 있다.  

**각각의 API 상세 설명**

- `location.href`: protocol, domain, path, query string, hash 등 포함한 전체적인 URL을 return한다. 직접적으로 '=' 연산자를 통해 할당하고 URL을 수정할 수 있다.
- `location.protocol`: 현재 페이지의 protocol을 반환한다. 직접적으로 '=' 연산자를 통해 할당하고 URL을 수정할 수 있다.
- `location.hostname`: 현재 페이지의 hostname을 반환한다. 직접적으로 '=' 연산자를 통해 할당하고 URL을 수정할 수 있다.
- `location.pathname`: 현재 페이지의 hostname을 반환한다. 직접적으로 '=' 연산자를 통해 할당하고 URL을 수정할 수 있다.
- `location.search`: 현재 페이지의 query string 반환한다. 직접적으로 '=' 연산자를 통해 할당하고 URL을 수정할 수 있다.
- `location.hash`: 현재 페이지의 hash을 반환한다. 직접적으로 '=' 연산자를 통해 할당하고 URL을 수정할 수 있다.
- `location.assign`: *페이지 URL을 설정하고 새로고침*한다. 히스토리에 새로운 스택을 쌓는다.
- `location.replace`: *페이지 URL을 설정하고 새로고침*한다. 히스토에 최상단 스택을 대체한다.

- `history.back`: 히스토리의 이전 페이지를 불러온다.
- `history.forward`: 히스토리의 다음 페이지를 불러온다.
- `history.go`: 히스토리의 몇 번째 페이지를 불러온다.
- `history.pushState`: *페이지 새로고침 없이 state를 변경*한다. 히스토리에 새로운 스택을 쌓는다.
- `history.replaceState`: *페이지 새로고침 없이 state를 변경*한다. 히스토리에 최상단 스택을 대체한다.

~(여담이지만... 바텀 시트 작성 때 필요한 지식과는 크게 상관이 없는 비교인 것 같다)~

### 

(작성중)

## 4. next link

## 5. 구현 코드
