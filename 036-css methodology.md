## 0. 공부하게 된 계기

[motion](https://github.com/mochang2/motion) (노션 클린 코딩 프로젝트)를 진행하기 전 컨벤션을 정하다가 CSS에도 컨벤션이 존재한다는 것을 알았다.  
가장 많이 사용되는 OOCSS, BEM, SMACSS를 정리한 뒤 프로젝트에 사용할 컨벤션을 정하고자 공부를 시작했다.

참고  
https://github.com/stubbornella/oocss/wiki  
https://www.smashingmagazine.com/2011/12/an-introduction-to-object-oriented-css-oocss/  
https://jmjmjm.tistory.com/44  
https://dev.to/nouran96/oocss-methodology-92d  
https://wit.nts-corp.com/2015/04/16/3538
https://blog.illunex.com/css-%EB%B0%A9%EB%B2%95%EB%A1%A0-oocss/

## 1. 공통점

같은 지향점을 가지고 있다.

- 코드의 재사용성을 높이기.
- 쉬운 유지보수.
- 확장 가능성.
- 클래스명 만으로도 무슨 의미인지 예측 가능하도록 하기.

## 2. OOCSS(Object Oriented CSS)

약어에서 알 수 있듯이 CSS를 모듈 방식으로 작성하여 중복을 줄이는 방법론이다.  
CSS object는 추상화될 수 있는 반복되는 패턴을 의미한다.

OOCSS의 주요 개념은 다음 두 가지이다.

- separate structure and skin
- separate container and content

### Separate Structure and Skin

structure은 `width`, `height`, `margin`, `padding` 등 invisible style을 의미한다.  
skin은 `color`, `font` 등 visible style을 의미한다.

한 웹 사이트에서는 대체적으로 한, 두 가지의 색으로 통일하다 보면 다음과 같이 똑같은 skin 스타일 코드가 반복적으로 사용되는 상황이 생기곤 한다.

```css
#button {
  width: 200px;
  height: 50px;
  padding: 10px;
  border: solid 1px #ccc;
  background: linear-gradient(#ccc, #222);
  box-shadow: rgba(0, 0, 0, 0.5) 2px 2px 5px;
}

#box {
  width: 400px;
  overflow: hidden;
  border: solid 1px #ccc;
  background: linear-gradient(#ccc, #222);
  box-shadow: rgba(0, 0, 0, 0.5) 2px 2px 5px;
}

#widget {
  width: 500px;
  min-height: 200px;
  overflow: auto;
  border: solid 1px #ccc;
  background: linear-gradient(#ccc, #222);
  box-shadow: rgba(0, 0, 0, 0.5) 2px 2px 5px;
}
```

이러한 코드가 존재할 때 다음과 같이 바꿔준다.

```css
.button {
  width: 200px;
  height: 50px;
}

.box {
  width: 400px;
  overflow: hidden;
}

.widget {
  width: 500px;
  min-height: 200px;
  overflow: auto;
}

.skin {
  border: solid 1px #ccc;
  background: linear-gradient(#ccc, #222);
  box-shadow: rgba(0, 0, 0, 0.5) 2px 2px 5px;
}
```

좀 더 적은 코드로 이루어지고 재사용성이 늘어난다.

좀 다른 예시로 HTML에서는 다음과 같이 사용할 수도 있다.

```html
<a href="#" class="btn-base cart">장바구니</a>
<a href="#" class="btn-base buy">구매</a>
```

### Separate Container and Content

container는 무엇인가를 포함한 것, 그리고 content는 이미지, 문단, 박스 등 container 안에 포함되어 있는 요소들을 이야기한다.  
CSS의 space selector나 '>' selector 보다는 content 스타일을 container로부터 분리하는 것을 추구한다.

```css
#sidebar {
  /*...*/
}

#sidebar .list {
  /*...*/
}

#sidebar .list .list-header {
  /*...*/
}

#sidebar .list .list-body {
  /*...*/
}
```

위 코드를 아래와 같이 바꿀 수 있다.

```css
.sidebar {
  /*...*/
}

.list {
  /*...*/
}

.list-header {
  /*...*/
}

.list-body {
  /*...*/
}
```

### 장단점

- 장점
  - CSS 코드 양을 줄일 수 있음. 따라서 브라우저 렌더링 시 스타일 시트 다운로드 속도를 증가시킴으로써 성능 향상을 기대할 수 있음.
  - 렌더링 엔진이 이미 계산된 값을 사용하려기 때문에(캐싱 등) CSS rule이 재사용될수록 성능 향상을 기대할 수 있음.
  - 재사용이 가능하므로 유지 보수성이 높음.
- 단점
  - HTML 코드가 복잡하여 가독성이 떨어질 수 있음.
  - OOP가 그러하듯, 크기가 작은 프로젝트에서 사용한다면 overkill일 수 있음.
  - 공통된 클래스가 많기 때문에 멀티 클래스(`<div class="aa bb cc">`와 같은 것)를 자주 사용하는데, 멀티 클래스가 많아질수록 유지 보수가 오히려 힘들 수 있음.

### 가이드라인

- space나 '>' 등과 같은 descendent selector 사용을 피하라.
- id 속성을 사용하지 말라는 의미는 아니다. 다만 id는 너무 강력하기 때문에 스타일을 위해서 사용한다기 보다, linking이나 js hook을 위해서 사용하라.
- `div.header` 등 HTML tag를 selector 앞에 추가하는 것을 자제하라.
- `!important` 사용을 자제하라.
- css grid를 사용하라.
- content는 '%' 등이 아닌 'px'로 크기를 지정하라.

### base classes

[wiki](https://github.com/stubbornella/oocss/wiki)에서 사용하는 base classess 들의 용도를 정리하려고 추가했다.

| property                                       | description                                                                                          |
| ---------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| `media`                                        | wrapper for the media object.                                                                        |
| `img`, `imgExt`                                | media object의 child node. 보통 link, image, flash의 wrapper로 사용됨.                               |
| `bd`                                           | media object에서 가장 중요. 반드시 필요하고 안에 무엇이든 들어갈 수 있음.                            |
| `data`                                         | wrapper for the data table                                                                           |
| `txtL`, `txtR`, `txtC`, `txtT`, `txtM`, `txtB` | `<table>`, `<tr>`, `<td>`, `<th>` 등에 적용할 수 있는 속성. 각각 좌, 우, 중간, 위, 중간, 아래를 의미 |
| `line`                                         | 가로 줄, 행                                                                                          |
| `unit`                                         | `line`은 한 section(column)으로 나눈 단위(base class).                                               |
| `sizeXofY`                                     | `unit`의 width를 분수로 표현한 것. extends `unit`.                                                   |
| `lastUnit`                                     | 각 line의 마지막 child에게 적용되는 속성. extends `unit`.                                            |
| `mod`                                          | 모든 container 모듈의 base class. 안에 무엇이든 들어갈 수 있음. 내부가 transparent함.                |
| `complex`                                      | 내부가 transparent함. `mod`나 `inner`만으로 구분짓기 어려운 내부를 구현. extends `mod`.              |
| `pop`                                          | backdrop같은 것처럼 외부가 transparent함. extends `mod`.                                             |
| `tl`, `tr`, `bl`, `br`                         | 블록의 왼쪽 상단 등 모서리를 복합적으로 표시하는 데 사용되는 표현 요소.                              |

~막상 정리하고 나니까 그닥 뭐 별거 없는거 같기도...~

## 3. BEM

## 4. SMACSS

## 5. 결론

motion 프로젝트에 어울리는, 나에게 어울리는 컨벤션은 무엇일까?
