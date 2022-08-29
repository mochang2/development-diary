## 0. 공부하게 된 계기

React를 개발하다 보면 매번 좋은 구조가 무엇인지 헷갈린다.  
좋은 구조에는 정답이 없고 프로젝트의 성격마다 좋은 구조가 다르다는 것은 알지만 그럼에도 어떤 구조가 좋은지 끊임없이 고민하게 되고 시간을 소비하게 된다.  
심지어 vanilla js로 개발할 때도 비슷한 고충을 겪는다.  
프론트엔드 아키텍처의 전반적인 흐름을 공부하면 그러한 시간을 줄까 싶어 일단 공부를 시작했다.

아래 코드들은 대부분 예전에 쓴 코드를 고쳐서 간단하게만 썼고, 자세한 문법은 공식 문서나 각종 커뮤니티를 찾아보는 것이 더 좋다.

참고 및 사진 출처

- https://developer.mozilla.org/ko/docs/Web/Guide/AJAX/Getting_Started
- https://velog.io/@teo/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C%EC%97%90%EC%84%9C-MV-%EC%95%84%ED%82%A4%ED%85%8D%EC%B3%90%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80%EC%9A%94
- https://velog.io/@teo/MVI-Architecture
- https://yozm.wishket.com/magazine/detail/1193/
- https://velog.io/@kykim_dev/%EA%B4%80%EC%8B%AC%EC%82%AC%EC%9D%98-%EB%B6%84%EB%A6%ACSeparation-of-Concerns-SoC%EC%99%80-Custom-Hook
- https://hwan1001.tistory.com/38
- https://github.com/mochang2/GGABI-Front
- https://velog.io/@kykim_dev/%EA%B4%80%EC%8B%AC%EC%82%AC%EC%9D%98-%EB%B6%84%EB%A6%ACSeparation-of-Concerns-SoC%EC%99%80-Custom-Hook
- https://snupi.tistory.com/185
- https://ko.reactjs.org/docs/context.html
- https://kyounghwan01.github.io/blog/React/react-context-api/#%E1%84%8B%E1%85%A8%E1%84%89%E1%85%B5
- https://velog.io/@yrnana/Context-API%EA%B0%80-%EC%A1%B4%EC%9E%AC%ED%95%98%EC%A7%80%EB%A7%8C-%EC%97%AC%EC%A0%84%ED%9E%88-%EC%82%AC%EB%9E%8C%EB%93%A4%EC%9D%B4-redux%EC%99%80-%EC%A0%84%EC%97%AD-%EC%83%81%ED%83%9C%EA%B4%80%EB%A6%AC-%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC%EB%A5%BC-%EC%93%B0%EB%8A%94-%EC%9D%B4%EC%9C%A0
- https://ui.toast.com/weekly-pick/ko_20200213

## 1. 사전지식

### AJAX

~깊게 들어가자면 밑도 끝도 없기 때문에 간단히 정리한다.~  
MDN 정의에 의하면 AJAX는 Asynchronous Javascript And XML의 약어이다.  
즉, 서버와 통신하기 위해 XMLHttpRequest 객체를 사용하는 것을 말한다.  
서버와 통신할 때 다양한 포맷으로 주고받을 수 있는데 그 중 하나가 XML인 것이고 JSON, HTML, 심지어 일반 텍스트도 가능하다.

AJAX가 가장 중요한 특징은 **비동기**이다.  
이 비동기 덕분에 SPA와 CSR 기술 구현이 가능해지고, UX가 비약적으로 증가했다.  
페이지 **새로고침 없이 서버에 요청이 가능**해졌기 때문이다.  
기존에 MPA(Multi Page Application)에서는 클라이언트는 서버로부터 받은 HTML을 브라우저 창에서 렌더링 하는 역할을 했다.  
만약 form 데이터를 보내서 새로운 페이지를 렌더링 해야 되면 새로운 페이지가 reload 되어야 했다.  
하지만 AJAX가 나오면서 비동기적으로 요청을 보내고 응답을 받아 웹페이지의 일부분만 갱신할 수 있게 되었다.

### 컴포넌트

W3C 정의에 따르면 웹 컴포넌트는 독립적인 뷰를 생성하기 위해서 HTML, CSS, JS를 한 곳에 묶어놓은 것이다.  
웹 컴포넌트는 모든 주요 브라우저에서 지원하는 웹 표준 기반의 재사용 가능한 클라이언트 사이드 컴포넌트이다.  
코드에서 원하는 부분을 캡슐화하는 훌륭한 방법을 제공하며 모든 웹 어플리케이션과 웹 페이지에서 재사용할 수 있다.  
어떤 프레임워크를 사용하는지 중요하지 않고 언제 어디서나 재사용 가능하게끔 만들어야 좋은 컴포넌트이다.

_참고) react 등의 라이브러리/프레임워크를 사용하지 않고 기존에 컴포넌트를 만들던 방식_

```javascript
// customElements.define() API를 이용하면 새로운 태그 이름을 만들 수 있다.
class CustomDiv extends HTMLElement {}
window.customElements.define('custom-div', CustomDiv)

const newDiv = document.createElement('custom-div')
document.body.appendChild(newDiv)
```

html 코드에서는 아래와 같이 사용하면 된다.

`<custom-div></custom-div>`

#### React에서의 컴포넌트

컴포넌트는 데이터(props)를 입력받아 View(state) 상태에 따라 DOM Node를 출력하는 함수이자 react로 만든 어플리케이션을 구성하는 최소 단위이다.

기존의 웹 프레임워크는 MVC방식으로 분리하여 관리하여 각 요소의 의존성이 높아 재활용이 어렵다는 단점이 있었다.  
반면 컴포넌트는 MVC의 뷰를 독립적으로 구성하여 재사용을 할 수 있고 이를 통해 새로운 컴포넌트를 쉽게 만들 수 있다.  
[아래](https://github.com/mochang2/development-diary/blob/main/028-architecture.md#flux)에도 나올 이야기이지만 좋은 구조를 위해서는 비즈니스 로직이 들어가면 컴포넌트의 재사용성이 상당히 떨어지기 때문에 가능하면 컴포넌트에 비즈니스 로직을 포함시키지 않아야 한다.  
(참고로 컴포넌트 이름은 항상 대문자로 시작하는데, react는 소문자로 시작하는 컴포넌트를 DOM 태그로 취급하기 때문이다)

#### 컴포넌트 관심사 분리

하나의 컴포넌트에게 한번에 너무 많은 일(concerns)을 부여하게 되면 그 코드를 읽는 사람은 혼란스러울 수 있다.  
가장 간단한 해결책은 한 번에 한 가지 걱정만 하도록 단위를 잘게 나누는 것이다.  
즉, 코드는 단위 별로 하나의 관심사만 갖도록 하고 그 관심사에 대해서만 충실히 동작하도록 만들어야 한다.  
(OOP의 SOLID 원칙에도서도 같은 개념을 사용한다)
관심사의 분리가 적절히 구현된 코드에서는 Loose Coupling (각각의 코드가 서로 얽혀있지 않고 독립적으로 잘 분리되어 있음)과 High Cohesive (유사한 내용끼리 비슷한 위치에 잘 모여 있음)와 같은 특성을 발견할 수 있다.

관심사 분리를 성공적으로 하면 다음과 같은 장점이 있다.

1. 코드가 더욱 명료. 자신이 어떤 일을 하고, 어떤 목적을 가지고 설계된 코드인지 보다 잘 드러나게 됨.
2. 코드 재사용성이 올라감. 여러 역할이 엉켜있는 코드보다, 역할 별로 잘 분리되어 있는 코드를 재사용하기가 쉬움.
3. 유지 보수가 용이. 변경 사항이 발생했을 때 해당 관심사에 연관된 코드만 수정하면 됨.
4. 테스트 코드를 작성하기 쉬움. 얽혀있는 로직보다 분리되어 있는 로직에 대한 테스트가 보다 더 간단해짐.

### FLUX와 REDUX

![flux 패턴](https://user-images.githubusercontent.com/63287638/187248174-056a59ff-638c-42ce-99ce-07e1589042ae.png)

flux는 facebook에서 (특히 알림 기능을 만들 때) MVC 패턴의 한계를 느끼고 만든 패턴인 반면, redux는 이러한 디자인 패턴을 비슷하게 적용한 '상태 관리' 라이브러리이다.  
reducer라는 개념이 등장하면서 flux가 deprecated 되었다고 한다.  
flux는 store가 여러 개인 반면, redux는 sotre가 하나라는 점,  
redux에선 sotre를 변경하는 여러 개의 reducer가 존재한다는 점,  
redux는 flux에 비해 thunk, saga, logging 등 미들웨어 생태계가 구축되었다는 점 등
둘 사이에는 사소한 차이가 존재하는데 근본적인 개념은 비슷하다.

![redux 구조](https://user-images.githubusercontent.com/63287638/187248427-19626449-2fa7-4b54-9606-8318ca617058.png)

위 사진은 redux를 적용한 구조이다.  
다음과 같은 순서로 상태 관리가 일어난다.

0. 모든 상태 관리는 store에서 발생한다. reducer를 제외하고 누구도 store가 관리하는 상태를 직접적으로 수정하면 안 된다.
1. 상태에 어떤 변화가 발생할 때 action을 store에 전달한다.
2. action은 객체 형태로 되어 있으며, 상태를 변화시킬 때 이 객체를 참조하여 변화를 일으킨다. 이를 dispatch라고 한다.
3. store가 action을 받으면 reducer가 상태를 변화시킨다.
4. 변한 상태는 다시 store에 저장한다.
5. store 안에 있는 상태가 바뀌면 store를 구독하는 컴포넌트들에 전달된다.

- action: 상태가 변화할 때 참조하는 객체.
- store: 상태를 저장하는 곳.
- reducer: 상태를 변화시키는 로직이 있는 함수. 반드시 순수 함수(같은 input -> 같은 output)여야 하고 `new Data()`나 `Math.random()` 등의 함수도 사용하면 안 됨.
- state: 컴포넌트에 최종 출력하기 전 거치는 중간 과정.
- dispatcher: 액션을 스토어에 전달. 전달은 보통 콜백 함수로 이루어짐.

#### 코드 예시

```javascript
// // 1. action 정의
// actionTypes.js

export const CHANGE_LAYOUT = "CHANGE_LAYOUT"
export const CHANGE_LAYOUT_WIDTH = "CHANGE_LAYOUT_WIDTH"
export const CHANGE_SIDEBAR_THEME = "CHANGE_SIDEBAR_THEME"
export const CHANGE_SIDEBAR_TYPE = "CHANGE_SIDEBAR_TYPE"

// actions.js
import {
  CHANGE_LAYOUT,
  CHANGE_LAYOUT_WIDTH,
  CHANGE_SIDEBAR_THEME,
  CHANGE_SIDEBAR_TYPE,
} from "./actionTypes.js"

export const changeLayout = layout => ({
  type: CHANGE_LAYOUT,
  payload: layout,
})

export const changePreloader = layout => ({
  type: CHANGE_PRELOADER,
  payload: layout,
})

export const changeLayoutWidth = width => ({
  type: CHANGE_LAYOUT_WIDTH,
  payload: width,
})

export const changeSidebarTheme = theme => ({
  type: CHANGE_SIDEBAR_THEME,
  payload: theme,
})

// // 2. reducer 함수 작성
// reducer.js
// @flow
import {
  CHANGE_LAYOUT,
  CHANGE_LAYOUT_WIDTH,
  CHANGE_SIDEBAR_THEME,
  CHANGE_SIDEBAR_TYPE,
} from "./actionTypes";

const INIT_STATE = {
  layoutType: "vertical",
  layoutWidth: "fluid",
  leftSideBarTheme: "light",
  leftSideBarType: "icon",
};

const Layout = (state = INIT_STATE, action) => {
  switch (action.type) {
    case CHANGE_LAYOUT:
      return {
        ...state,
        layoutType: action.payload,
      };
    case CHANGE_PRELOADER:
      return {
        ...state,
        isPreloader: action.payload,
      };
    case CHANGE_LAYOUT_WIDTH:
      return {
        ...state,
        layoutWidth: action.payload,
      };
    case CHANGE_SIDEBAR_THEME:
      return {
        ...state,
        leftSideBarTheme: action.payload,
      };
    default:
      return state;
  }
};

export default Layout;


// // 3. map~ToProps 작성
// component.js
import {
  CHANGE_LAYOUT,
  CHANGE_LAYOUT_WIDTH,
  CHANGE_SIDEBAR_THEME,
  CHANGE_SIDEBAR_TYPE,
} from "../store/actionTypes.js"

const mapActionToProps = (dispatch) => {
  return {
    onChangeLayout: () => {
      dispatch({ type: CHANGE_LAYOUT })
    }
    // ...
  }
}

const mapStateToProps = (state) => {
  return {
    layout: state.layoutType,
    // ...
  }
}

export default connect(mapActionToProps, mapStateToProps)(Component)


// // 4. 이후 props의 property를 참조하여 사용
```

## 2. 웹 프론트엔드 아키텍처

### 아키텍처

> IEEE에 따르면 컴포넌트란 구성 요소들 간의 관계, 환경, 설계와 발전을 관리하는 원칙으로 이루어진 시스템의 근본적인 구조이다.

초반에 규칙 없이 그냥 코드를 만들다 보면 덩치가 커지고 불편함이 생기는 순간이 온다.  
이러한 행위가 반복되어 하나의 특정 패턴이 만들어지고, 이러한 패턴들을 모두가 이해하고 따를 수 있도록 하는 구조를 아키텍처라고 부른다.

좋은 아키텍처의 가장 중요한 조건은 기능에 따라서든, 목적에 따라서든 분리한 **분류**이다.

### MVC

초창기 웹 서비스의 MVC 아키텍처에서  
*Model*은 데이터베이스를,  
*View*는 HTML, CSS, JS를 포함한 클라이언트 영역을,  
*Controller*는 이 둘 사이에서 라우터를 통해 데이터를 처리하고 새로운 HTML을 만들어서 보여주는 서버 영역을 이야기한다.

MVC로 나눠진 이유는 크게 두 가지이다.

1. 화면을 다루는 문제와 데이터를 다루는 문제의 성격이 달라서 분리하기 위함이었다.
2. Model과 View간의 의존관계를 최소화 해서 화면의 수정이 데이터 수정에 영향을 미치지 않고 반대로 데이터 수정이 화면의 수정에 영향을 미치지 않게 하기 위함이었다.

AJAX라는 기술이 추가된 후 react 전에 대세를 이루던 jQuery 시절에는 그 개념이 약간 바뀌었다.  
*Model*은 AJAX로부터 받은 데이터를,  
*View*는 HTML, CSS로 만들어지는 화면을,  
*Controller*는 중간에서 서버의 데이터를 받아서 화면을 바꾸고 이벤트를 처리해서 서버에 데이터를 전달하는 것을 이야기한다.  
(이때 Controller 역할을 했던 것이 jQuery이다)

> 가장 중요한 패러다임은 Model과 View의 종속성을 최대한 분리하는 것이다. HTML과 jQuery를 따로 관리하는 것이 주효했다.

### MVVM(angular, react, vue)

jQuery에서 데이터를 찾아서 데이터를 바꾸고, 이벤트를 연결하고, 이벤트를 수정하는 부분에서 반복적인 작업이 나타난다는 것을 발견했다.  
(Django, php 등을 상기시켜보면) 서버에서 개발할 때 HTML 코드에서 `{{ }}`나 `<%= %>`와 같은 치환자로 '선언적으로' 편하게 개발했던 부분을 상기시켜 템플릿 기반의 바인딩을 생각했다.

이 아키텍처에서 Model이 변하면 View를 수정하고, View에서 이벤트를 받아서 Model을 변경하는 Controller의 역할은 바뀌지 않는다.  
다만, 이를 구현하는 방식이 jQuery와 같은 DOM 조작에서 템플릿과 바인딩을 통한 '선언적인' 방법으로 변한 것이다.  
react 코드에서 `document.getElementById`와 같은 DOM API를 사용하지 않듯이, 코드에서 DOM을 조작하는 코드가 사라지고 이 기능들을 라이브러리/프레임워크가 담당하게 되었다.  
개발자는 화면에 그려져야 할 데이터만 react 등에 전달하면 되므로 **View를 그리는 Model만 다루게 되었다는 의미로 ViewModel이라고 부르며 이를 MVVM**이라고 한다.

> 이때의 패러다임은 Model과 View의 관점을 분리하려 하지 않고 하나의 템플릿으로 관리하려는 방식으로 발전했다. 기존에는 class나 id등으로 간접적으로 HTML에 접근하려고 했다면 직접적으로 HTML에 접근하는 방법으로 확장이 되었다.

### Container-Presenter

웹 서비스가 발전하면서 Page 단위가 아닌, 조금 더 작게 재사용할 수 있는 단위의 패턴으로 개발 방향이 바뀌었는데, 이것이 Component이다.  
컴포넌트 재사용을 위해 가급적 비즈니스 로직을 포함시키지 않아야 됐다.  
그래서 비즈니스 로직을 관장하고 있는 컴포넌트를 `Container` 컴포넌트라 하고, 비즈니스 로직은 가지고 있지 않고 데이터만 뿌려주는 형태의 컴포넌트를 `Presenter` 컴포넌트라 한다.

![container presenter](https://user-images.githubusercontent.com/63287638/187262255-2102109e-e9e2-4222-a7b4-713e9269055f.png)

이러한 형태로 개발하면 View에 집중하는 Presenter 컴포넌트는 단순히 주입 받은 정보를 렌더링할 뿐(ex. (state, props) ⇒ UI)이다.  
주입 받은 정보만 올바르다면 Presenter 컴포넌트는 항상 올바른 UI를 리턴하게 된다.  
로직과 분리되었기 때문에 여러 곳에서 재활용하기도 쉽고, Container 컴포넌트와 조합하기도 쉽다.

Container-Presenter 구조에 문제가 있었는데 props drilling이 필연적으로 발생한다는 것이었다.  
중간에 있는 컴포넌트들은 해당 `props`를 이용하지 않는데 하위 컴포넌트에게 `props`를 넘겨주기 위해 `props`를 받아야 했다.  
props drilling이 발생하면 다음과 같은 경우에 props에 대한 추적이 특히 어려워진다.

- 일부 데이터의 자료형을 바꾸게 되는 경우
- 프로퍼티의 이름이 중간에 변경되는 경우
- 필요보다 많은 프로퍼티를 전달하다가 컴포넌트를 분리하는 과정에서 필요 없는 프로퍼티가 계속 남는 경우
- 필요보다 적은 프로퍼티를 전달하면서 동시에 defaultProps를 과용한 결과로 필요한 프로퍼티가 전달되지 않는 경우

특히 hook의 등장으로 이 패턴(계층적인 패턴, Model과 View의 분리를 포함하는 것이 아님)을 권장되지 않는다고 하며 context api, redux, recoil 등 이를 해결할 다양한 상태 관리 라이브러리들이 등장했다.

#### 코드 예시

hook이 없었으므로 클래스 컴포넌트 형식으로 예시를 들었다.

```javascript
// // 기존
// UserList.js

import React from 'react'

class UserList extends React.Component {
  constructor() {
    this.state = {
      users: [],
    }
  }

  componentDidMount() {
    fetchUsers('/users')
      .then((res) => res.json())
      .then((res) => this.setState({ users: res.users }))
  }

  render() {
    return
    {
      this.state.users.length !== 0 ? (
        <ul>
          {this.state.users.map((user) => {
            return <li key={user.id}>{user.name}</li>
          })}
        </ul>
      ) : (
        <></>
      )
    }
  }
}
```

```javascript
// 분리한 뒤
// UserList.js
import React from 'react'

class UserList extends React.Component {
  constructor({ users }) {
    this.state.users = users
  }

  render() {
    return (
      <ul>
        {this.state.users.map(user => {
          return <li key={user.id}>{user.name}</li>
        })}
      </ul>
    )
  }
}

// UserListContainer.js
import React from 'react'

class UserListContainer extends React.Component {
  constructor() {
    this.state = {
      users: []
    }
  }

	componentDidMount() {
		fetchUsers("/users")
			.then(res => res.json())
			.then(res => this.setState({ users: res.users }))
	}

	render() {
		return <UserList users={this.state.users}>
	}
}
```

### FLUX

flux 패턴은 View를 각각의 MVC 컴포넌트 관점으로 보는 것이 아니라 하나의 큰 View로 이해한다.  
View에서는 Dispatch를 통해 Action을 전달하면 Action은 Reducer를 통해서 Store에 데이터를 보관하고 Store에 들어 있는 데이터는 다시 View로 연결되는 방식이다.  
기존 컴포넌트 단위의 MVC 개념에서 완전히 비즈니스 로직과 View를 분리했고 이를 '상태 관리(State Management)'라고 불렀다.

다만 flux 패턴의 가장 큰 문제는 높은 학습 곡선과 많은 양의 보일러 플레이트였다.  
이를 해결하기 위한 것이 Mobx나 Angular의 Rxjs 등이 등장했다 ~얘네는 잘 모르니 그러려니 하고 패스~.

> 해당 패러다임은 공통적으로 사용되는 비지니스 로직의 Layer와 View의 Layer를 완전히 분리되어 상태관리라는 방식으로 관리한다. 각각의 독립된 컴포넌트가 아니라 하나의 거대한 View 영역으로 간주한다. 둘 사이의 관계는 Action과 Reduce라는 인터페이스로 분리한며 Controller는 이제 양방향이 아니라 단반향으로 Cycle을 이루도록 설계한다.

### MVI(상태 관리 아키텍처)

사용자가 한 행동을 서로 다르지만 같은 결과를 만들어내기도 한다.
예를 들어, 사용자가 마우스 휠을 돌리는 행위와 (foucs된 창에서) 키보드 방향키를 누르는 행위는 결과적으로 같은 결과를 야기한다.  
이를 토대로 비즈니스 로직을

1. 사용자가 View를 통해 전달한 UI event를 어떠한 데이터가 변하게 할지 전달하는 역할과 - **Intent**
2. 전달 받은 요청에 따라서 적절히 데이터를 변화시키는 역할 - **Model**(변화를 감지하고 변경사항을 전파하는 영역 + 데이터를 변화하는 로직)

로 나눌 수 있다.

![mvi](https://user-images.githubusercontent.com/63287638/187268918-7be13bf1-1995-4a6c-a4c8-694653e585b1.png)

MVI 아키텍쳐가 기존의 MVC나 MVVM과 다르게 하나의 컴포넌트가 아니라 앱 전체에 적용이 된다.

> 데이터가 단방향으로 순환하며 전역적으로 구성된다. → 데이터의 흐름을 이해하고 디버깅을 하기 쉽다.
> View는 Model에 의존적이지만 비지니스 로직은 View와의 의존성이 없다. → UI변화 요구사항에 유연하게 대응할 수 있다.
> View의 생명주기와 무관하게 일관성 있는 상태를 갖는다. → 컴포넌트 생명주기에 따른 상태 동기화 문제를 해결한다.

### Context

props drilling의 또다른 해결책으로, 컴포넌트 트리에서 context라는 거대한 공통 조상을 만들고, 그 context로부터 데이터를 제공받는 방식이다.  
context는 react 컴포넌트 트리 안에서 전역적(global)이라고 볼 수 있는 데이터를 공유할 수 있도록 고안된 방법이다.  
이 방식 또한 View와 Model을 분리한다.

![context](https://user-images.githubusercontent.com/63287638/187270076-1ce2dba0-ef78-4351-af66-507fe5b88d72.png)

#### 코드 예시

react 16.3부터는 기본적으로 Context API란 것을 제공한다.  
또한 거의 모든 상태 관리 라이브러리들이 이 API를 이용해 개발되었다고 한다.

```javascript
// src/store.js
import React from 'react'

const Store = React.createContext(null) // context 객체를 만든다.
export default Store // 이 객체의 Provider와 Consumer를 이용한다
```

```javascript
// src/App.jsx
import React from 'react'
import Test from 'components/Test'
import Store from 'store'

class App extends React.Component {
  constructor(props) {
    super(props)
    this.changeMessage = () => {
      const { message } = this.state
      if (message === 'hello') {
        this.setState({
          message: 'by',
        })
      } else {
        this.setState({
          message: 'hello',
        })
      }
    }
    this.state = {
      test: 'testContext',
      message: 'hello',
      changeContext: this.changeMessage,
    }
  }

  render() {
    return (
      <Store.Provider value={this.state}>
        <Test test={this.state.test} />
      </Store.Provider>
    )
  }
}
```

```javascript
// src/components/Test.jsx

import React from 'react'
import Store from 'store'

class Test extends React.Component {
  render() {
    const { test } = this.props

    return (
      <div>
        <Store.Consumer>{(store) => store.message}</Store.Consumer>
        <Store.Consumer>
          {(store) => (
            <button type="button" onClick={store.changeContext}>
              btn
            </button>
          )}
        </Store.Consumer>
      </div>
    )
  }
}

export default Test
```

_참고) 위 예시는 단순히 context API를 사용하기 위한 방법이고 잘 사용하기 위한 방법과 provider hell을 알고 싶다면 [여기](https://velog.io/@yrnana/Context-API%EA%B0%80-%EC%A1%B4%EC%9E%AC%ED%95%98%EC%A7%80%EB%A7%8C-%EC%97%AC%EC%A0%84%ED%9E%88-%EC%82%AC%EB%9E%8C%EB%93%A4%EC%9D%B4-redux%EC%99%80-%EC%A0%84%EC%97%AD-%EC%83%81%ED%83%9C%EA%B4%80%EB%A6%AC-%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC%EB%A5%BC-%EC%93%B0%EB%8A%94-%EC%9D%B4%EC%9C%A0)_

### Atomic(Recoil, Svelte Store, Vue Composition, Jotai)

~아래 두 atomic이 섞어서 설명할 내용은 아닌 것 같아서 일단 나눴다~

#### 상태 관리로서의 atomic

거대한 View와 Store를 나누어 이분법으로 생각하자는 의견에는 동의하나 Action ~ Dispatch ~ Reducer와 같은 복잡한 구조를 가져야 하는가에 대한 방법에 대해서는 회의적인 시각으로 만들어진 방법이다.
간단한 문법으로 컴포넌트 외부에서 공통의 데이터를 set, get을 할 수 있게 하면서 동시에 동기화를 하고자 했다.  
이와 더불어 computed, derived, select 와 같은 반응형 기능을 제공하여 관련된 데이터의 동시 업데이트를 제공하고 있다.

#### 코드 예시

recoil의 예시이다

```javascript
// store.js
import { atom } from 'recoil'

export const ReactionsState = atom({
  key: 'review-reactions-state',
  default: [], // undefined 등도 가능
})
```

```javascript
// Reaction.js
import { useRecoilState } from 'recoil'
import { ReactionsState } from './store.js'

const Reaction = ({ reviewReaction }) => {
  const [reactions, setReactions] = useRecoilState(ReactionsState)
  // ...
}
```

#### 디자인 패턴으로서의 atomic

![atomic design](https://user-images.githubusercontent.com/63287638/187276003-b0a85ba3-13e7-4e11-8a67-c519a84fb15b.png)

버튼, 제목, 텍스트 입력 필드와 같은 가장 작은 구성 컴포넌트를 atom이라고 말한다.  
atom은 모든 컴포넌트들의 기초가 되는 블록이며, 더이상 분해할 수 없는 필수 요소이다.

atom을 합친 molecule은 2개 이상의 atom으로 구성되어 있으며 하나의 단위로 함께 동작하는 UI 컴포넌트들의 단순한 그룹이다.  
예를 들어 텍스트 입력 필드, 레이블, 오류 메시지 등을 말한다.

molecule을 합친 organism은 인터페이스의 개별적인 영역을 형성한다.

template은 컴포넌트들을 배치하고 설계의 구조를 보여준다.  
page의 실제 컴포넌트가 없을 경우 페이지가 어떻게 보일지에 대한 골격 구조이다.

page는 실제 컨텐츠들을 배치한 UI를 보여주며, 템플릿의 구체화된 인스턴스(UI)이다.

![atomic 폴더 구조](https://user-images.githubusercontent.com/63287638/187276324-b2ba6440-d4a0-4556-b797-f18b34ecf895.png)

- 장점
  - 어플리케이션과 분리하여 컴포넌트를 개발하고 테스트할 수 있다.
  - 통합 개발 시 백엔드 어플리케이션의 로직에 의존하지 않는다.
  - 특정 컴포넌트에 CSS가 강하게 결합되어 있어서 CSS를 잘 관리할 수 있다.
- 단점
  - 컴포넌트가 분리되어 있고 상위 컨테이너 컴포넌트의 사이즈를 결정할 수 없을 경우 media query를 사용하기 어렵다.

### React Query, SWR, Redux Query

보통 브라우저의 경우 데이터를 로컬에 보관하지 않기에 프론트엔드 대부분의 전역적인 상태관리가 필요한 이유는 서버와의 API 때문이다.  
왜냐하면 웹의 특성상 데이터의 보관과 조회, 수정이 서버에서 이루어져야 한다.  
따라서 비즈니스 로직이 대부분 백엔드(DB 등)에 보관된다.  
View는 서버의 데이터를 보여주고 서버에 Action을 전달만 하는 경우가 일반적이라 전역 상태관리를 통해 비즈니스 로직을 관리하기보다 다이렉트로 백엔드와 직접 연동을 하면서 필요한 로딩, 캐싱, 무효화, 업데이트 등 기존 상태관리에서 복잡하게 진행해야했던 로직들을 단순하게 만들어주는 방식도 생겨났다.  
이러한 방식을 통해서 API를 통한 전역 상태관리가 단순하게 되는 결과를 가져왔다.

특히 react query는

> 서버와의 fetch 영역을 Model로 보고, View는 react, Controller는 query와 mutation이라는 2가지 인터페이스를 통해서 캐싱, 동기화, refetch 등을 수행한다.

#### 코드 예시

react query를 예시로 들었다.

```javascript
// queries.js
import request from './api.js'
import { useQuery } from 'react-query'

const getData = async (url: string) => {
  try {
    const { data } = await request(url)
    return data
  } catch (err: any) {
    // handle error
  }
}

export const useFetchPushTime = () => {
  return useQuery(
    keys.all, // react query를 위한 키 설정 필요
    () => getData('some url'),
    {
      // 각종 옵션
    },
  )
}
```

```javascript
// component.js
import { useQuery } from 'react-query'

const Component = () => {
  const { data, status } = useQuery(
    [key.keyword],
    () => getStoreInfo(keyword),
  )

  return (
    // ...
  )
}
```
