## 0. 공부하게 된 계기

React에 대해 면접 전에 부랴부랴 외우기 보다 미리 '이해'하고 싶어서 정리하려고 한다.

## 1. 명령형 프로그래밍 vs 선언형 프로그래밍

[React 공식 페이지](https://ko.reactjs.org/)에서는 react를 선언형이라고 설명한다.  
UI 라이브러리/프레임워크에서 '선언적'이라는 특징이 react만의 고유한 특징이 아니라, svelte, vue 등도 갖는 특징이다.

### 개념

명령형 프로그래밍(imperative programming)은 코드로 원하는 결과를 달성해 나가는 **과정**에만 관심을 두는 프로그래밍 스타일이다.  
흔히들 'HOW'로 표현한다.

반면 선언형 프로그래밍(declarative programming)은 필요한 것을 어떤 것인지 **기술**하는데 관심을 두는 프로그래밍 스타일이다.
흔히들 'WHAT'으로 표현한다.  
함수형 프로그래밍(`map`, `filter` 등의 추상화를 이용하는 고차함수)이 선언형 프로그래밍의 한 종류이다.

### 예시

아래 예시는 단순히 명령형 프로그래밍과 선언형 프로그래밍의 차이를 알기 위한 예시이다.

```javascript
// // 명령형 프로그래밍
// 원하는 것을 달성하는 '방법'에 중점
let string = 'THis is the midday show with Cheryl Waters'
let urlFriendly = ''

for (var i = 0; i < string.length; i++) {
  if (string[i] === ' ') {
    urlFriendly += '-'
  } else {
    urlFriendly += string[i]
  }
}

console.log(urlFriendly)

// // 선언형 프로그래밍
// 추상화를 이용
const string = 'This is the midday show with Cheryl Waters'
const urlFriendly = string.replace(/ /g, '-')

console.log(urlFriendly)
```

아래는 'hello world'를 쓰기 위한 실제 DOM 조작과 관련된 예시이다.

```javascript
// Vanilla.js의 절차적 프로그래밍
const app = document.querySelector('#app')
const header = document.createElement('h1')
header.innerText = 'hello world'

app.appendChild(header)
```

```jsx
// react.js의 선언형 프로그래밍
function App() {
  return (
    <div id="app">
      <h1>hello world</h1>
    </div>
  )
}

// 바벨을 이용하지 않는다면
ReactDOM.render(<h1>hello world</h1>, document.querySelector('#app'))
```

(이상한 예시지만) 만약 위 코드에서 'hello'와 'world'를 각각의 `<h1>`으로 분리하고 싶다고 하자.  
vanilla.js는 아마 `const header2 = document.createElement('h1')`를 선언한 뒤 `appendChild`를 한 번 더 써야될 것이다.  
반면 react는 기존 `<h1>`태그를 수정한 뒤 아래에 `<h1>`태그를 추가해주면 된다.

### 선언형 프로그래밍의 장점

1. 가독성이 좋다(이거는 내가 단순히 jsx에 익숙해져서 그럴 수도 있는 것 같다).
2. 추상화를 이용하기 때문에 재사용하기 쉽다.
3. 결과 예측이 쉽다. 위 예시에서 vanilla.js는 최소 둘 이상의 변수가 필요하지만, react는 변수가 전혀 필요가 없다. 변수에 어떤 값들이 저장되었는지 트래킹을 해야되는데, 변수가 많이 사용되면 디버깅이 어려워진다.
4. 유지보수가 좋다(위 예시에서는 와닿지 않을 수도 있지만 많은 데이터를 갖는 object 형식을 사용한다면 와닿을 것이다)

+) 추가사항  
만약 react에서 DOM을 변경할 때 기존 element를 완전히 제거하고 새로운 element르르 생성한다면 다음과 같은 문제가 발생한다.

1. performance 저하한다.
2. 화면 깜빡임 발생하여 UX가 좋지 않다.
3. state가 사라진다.

이를 해결하기 위해 react에서는 [**virtual DOM**](https://github.com/mochang2/development-diary/blob/main/029-virtual%20DOM.md)과 **reconciliation**이라는 개념을 도입했다.

## 2. reconciliation
