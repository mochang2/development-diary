## 0. 공부하게 된 계기

면접으로 자주 나온다는 js에 관한 질문들을 단순히 답만 외우는 게 아니라 자세히 정리하고 싶어졌다.

## 1. GC(garbage collection)

*출처: https://blog.sessionstack.com/how-javascript-works-memory-management-how-to-handle-4-common-memory-leaks-3f28b94cfbec , https://developer.mozilla.org/ko/docs/Web/JavaScript/Memory_Management*

<img src="https://user-images.githubusercontent.com/63287638/178136632-c33e7ae6-c245-4cd9-a1c9-63a6c434d678.png" alt="memory lifecycle" width="50%" />

위 사진에서 볼 수 있듯이 메모리는 크게 3가지의 life cycle을 가지고 있다.  
할당한 뒤에 사용되고, 사용되지 않으면 해제된다.  
C와 같은 저레벨 언어에서는 allocate와 release를 malloc(calloc 등)과 free를 통해서 개발자가 직접 명령해야 하지만 JS, JAVA, Python 등은 별도의 Garbage Collector가 동작해서 메모리 관리를 해준다.

```javascript
let n = 374 // allocates memory for a number

let s = "sessionstack" // allocates memory for a string

let o = {
  // allocates memory for an object and its contained values
  a: 1,
  b: null,
}

let a = [1, null, "str"] // (like object) allocates memory for the array

function f(a) {
  // allocates memory for a function
  return a + 3
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
  let o1 = {}
  let o2 = {}
  o1.p = o2 // o1 references o2
  o2.p = o1 // o2 references o1. This creates a cycle.
}
```

### GC 전략 2: mark and sweep algorithm

사용되는 메모리는 **mark** 한 뒤에 mark되지 않은 메모리들을 **sweep** 하는 알고리즘이다.  
GC의 root(웹은 window, node는 global)를 지정한 뒤 해당 루트를 타고, 타고 내려갔을 때 참조되지 않는 메모리를 release하는 방법이다.  
하지만 이 방법에도 문제점은 존재했다.  
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
  variable = 'text text'
  // == window.variable 또는 global.variable = 'text text'
  // == this.variable = 'text text'
  // 이러한 전역 오염을 막기 위해서는 var, let, const 반드시 사용
}
```

**필요 없는 Timer나 callback**

```javascript
let serverData = loadData()

setInterval(function() {
    let renderer = document.getElementById('what')
    if(what) {
        what.innerHTML = serverData
    }
}, 5000)
```

위 코드는 'what'이라는 id를 가진 element가 삭제될 때 더이상 필요하지 않는 타이머인데, 이를 삭제해주는 코드가 존재하지 않는다.  
아래 코드는 이러한 처리가 잘 되어있는 예시이다.  

```javascript
let element = document.getElementById('launch-button')
let counter = 0

function onClick(event) {
   counter++
   element.innerHtml = 'text ' + counter
}
element.addEventListener('click', onClick)

// Do stuff

element.removeEventListener('click', onClick) // onClick 함수에 대한 참조를 제거
element.parentNode.removeChild(element)

// element가 유효한 범위를 벗어나게 되면,
// element와 onClick은 GC에 의해 release됨
```

**closure**

```javascript
let theThing = null

let replaceThing = function () {
  let originalThing = theThing
  
  let unused = function () {
    // theThing을 참조하는 originalThing이 아래에서 할당되므로 더이상 null이 아님
    // 사용되지 않는 함수지만 memory release되지 않음
    if (originalThing) {
      console.log("hi")
    }
  };
  
  theThing = { // replaceThing이 아래에서 호출되면서 더이상 null이 아님.
    longStr: new Array(1000000).join('*'),
    someMethod: function () {
      console.log("message")
    }
  };
};

setInterval(replaceThing, 1000)
```

**DOM 객체 참조**

```javascript
let elements = {
  button: document.getElementById('button'),
  image: document.getElementById('image')
}

function doStuff() {
  elements.image.src = 'http://example.com/image_name.png'
}

function removeImage() {
  // image를 지워도 button이 남아있기 때문에 elements는 메모리에 남아 있음
  document.body.removeChild(document.getElementById('image'))
}
```

위 예시는 극단적인 예시같지만, 만약 어떠한 변수가 html 테이블에서 td를 참조하고 있다고 가정할 때는 문제가 심각해진다.  
만약 테이블 자체를 삭제해도 td를 참조하고 있는 변수가 GC에 의해 release되지 않으면 테이블 전체가 메모리에 남아있게 된다.  


## 2. prototype
_참고: https://www.youtube.com/watch?v=wUgmzvExL_E_  

OOP에서 class에게 동일한 속성을 지정해주기 위한 방법이다 정도로 이해할 수도 있다.  
다만 내부 동작 방식이 조금 다르며, ES6에 도입된 class가 있지만 (MDN에 따르면) class는 문법적인 양념일 뿐, JS는 여전히 프로토타입 기반의 언어이며 내부 동작 방식이 같지 않다.  

상속 관점에서 자바스크립트의 유일한 생성자는 객체뿐이다.  
각각의 객체는 \[Prototype\]이라는 은닉(private) 속성을 가지는데 자신의 프로토타입이 되는 다른 객체를 가리킨다.  
그 객체의 프로토타입 또한 프로토타입을 가지고 있고 이것이 반복되다, 결국 null을 프로토타입으로 가지는 오브젝트에서 끝난다.  
null은 더 이상의 프로토타입이 없다고 정의되며, 프로토타입 체인의 종점 역할을 한다.  

cf) prototype.\_\_proto\_\_ 이라는 속성이 존재하는데 이는 객체가 만들어지기 위해 사용된 원형인 프로토타입 객체를 숨은 링크로 참조하는 역할을 한다. [MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/proto)에 따르면 deprecated됐으며 getPrototypeOf()를 사용하기를 권장하고 있다. 

### 함수와 객체의 내부 구조

```javascript
function Func() {}

let f1 = new Func()
let f2 = new Func()
```

위와 같은 코드는 아래와 같은 내부 구조를 가진다.  

<img src="https://user-images.githubusercontent.com/63287638/178142162-7a6174d6-7f9f-4d33-8caa-b92d2a9e1190.png" alt="함수와 객체의 내부 구조" width="75%" />

### 예제 코드

```javascript
function error() {
  this.badRequest = 400,
  this.unauthorized = 403,
  this.notFound = 404
}

const imError = new error()
const yourError = new error()

console.log(imError) // error { badRequest: 400, unauthorized: 403, notFound: 404 }
console.log(yourError) // error { badRequest: 400, unauthorized: 403, notFound: 404 }
console.log(imError.prototype) // undefined

error.prototype.info = "에러에러에러"

console.log(imError) // error { badRequest: 400, unauthorized: 403, notFound: 404 }
console.log(yourError) // error { badRequest: 400, unauthorized: 403, notFound: 404 }
console.log(imError.info) // 에러에러에러. prototype chaining. imError2은 info가 없으나 error는 info가 있음.
console.log(yourError.info) // 에러에러에러
```

### Array의 비밀
구글에 `js array map`이라고 검색하면 MDN 사이트의 제목으로 `Array.prototype.map()`이 나온다.  
내가 `const whatArray = [1,2,3]` 이라고 선언하면 `whatArray.map(~)`이나 `whatArray.sort()` 등의 내장 함수를 사용할 수 있는 이유이다.  
실제로 `console.log(Array.prototype)` 이라고 하면 아래와 같은 결과물이 출력된다.

<img src="https://user-images.githubusercontent.com/63287638/178142653-4234dc4a-9b3c-483c-b877-62b3b2c39876.png" alt="array prototype" width="400" />

### Object.create
ES5에 객체를 생성하는 새로운 방법이 도입됐었다.  
Object.create를 이용해서 새로운 객체를 만들면, 생성된 객체의 프로토타입은 이 메소드의 첫 번째 인수로 지정된다.

```javascript
const a = {a: 1}
// a ---> Object.prototype ---> null

const b = Object.create(a)
// b ---> a ---> Object.prototype ---> null
console.log(b.a); // 1 (상속됨)

const c = Object.create(b)
// c ---> b ---> a ---> Object.prototype ---> null

const d = Object.create(null)
// d ---> null
console.log(d.hasOwnProperty); // undefined이다. 왜냐하면 d는 Object.prototype을 상속받지 않기 때문이다.
```


## 3. closure



## 4. this
JS는 함수가 어떻게 불리느냐에 따라서 this에 대한 서로 다른 4가지 용법을 가지고 있다.

### 일반 함수
웹에서는 window, node에서는 global과 같다.

```javascript
function func() {
  return this === global;
}

console.log(func()) // true
```

### 객체의 값으로서의 함수
객체 그 자체를 가리킨다.

```javascript
const obj = {
  value: 123,
  key: function func() {
    return this.value;
  },
}

console.log(obj.key) // [Function: func]
console.log(obj.key()) // 123
```

### new로 생성된 함수
빈 객체를 반환하다.  
생성자 함수는 특이하게도 return문이 있음에도 불구하고 그 return문을 무시하고 this 객체를 return하는 특징이 있다.  

```javascript
function func() {
  console.log(this);
}
new func() // func {}

function func2() {
  console.log(this.key)
  this.key = "value"
  console.log(this.key)
}
new func2() // undefined

function func3() {
  this.age = 100
  return 3
}
const a = new func3() // value
console.log(a) // func3 { age: 100 }
```

### call, bind, apply
해당 함수의 첫 번째 인자로 전달되는 객체를 가리킨다.

```javascript
const age = 100

function foo() {
  console.log(this.age)
}

var ken = {
  age: 35,
  log: foo
}

foo.call(ken, 1, 2, 3); // 35
```

### 참고 arrow function vs 일반  function
[MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/Arrow_functions)에 따르면 arrow function은 일반 function과 비교했을 때 다음과 같은 특성이 있다. 

* this나 super에 대한 바인딩이 없고, methods 로 사용될 수 없습니다.
* new.target키워드가 없습니다.
* 일반적으로 스코프를 지정할 때 사용하는 call, apply, bind methods를 이용할 수 없습니다.
* 생성자(Constructor)로 사용할 수 없습니다.
* yield를 화살표 함수 내부에서 사용할 수 없습니다

여기서 주목할 것은 첫 번째 속성으로, arrow function은 this에 대한 binding이 위에 예시로 제시한 함수들처럼 동적으로 진행되지 않는다.  
즉, 어떤 환경에서 해당 함수가 불리느냐에 따라 결정되지 않는다는 것이다.  
arrow function을 부른 위치가 어디냐에 따라서 해당 함수 내부에서의 this가 정적으로 결정되어 있다.  

```javascript
const wrapper = {
  key: () => {
    return this;
  },
}

console.log(wrapper.key()) // {}
```

cf) 개인적인 생각이지만, React에서 컴포넌트를 선언할 때 일반 function보다는 arrow function을 사용하라고 한다. 아마 함수형 컴포넌트 이전에 클래스형 컴포넌트를 사용할 때 this를 많이 사용했는데, 이 this가 global 또는 window를 가리킬 수도 있어서 의도치 않게 동작할 수 있기 때문인 것 같다.


## 5. event bubbling
이벤트 버블링은 특정 화면 요소에서 이벤트가 발생했을 때 해당 이벤트가 더 상위의 화면 요소들로 전달되어 가는 특성을 의미한다.  
아래와 같이 코드를 짰을 경우, div3를 눌렀을 때, `click div3` -> `click div2` -> `click div1` 순으로 alert가 실행된다.  

```html
<!DOCTYPE html>

<html>
  <head>
    <title>test</title>
    <meta charset="utf-8" />
    <link rel="stylesheet" type="text/css" href="./test.css" />
    <script defer src="./test.js" type="text/javascript"></script>
    <!-- defer 옵션이 붙은 이유는 아래 6번째 소제목에서 설명할 예정 -->
  </head>
  <body>
    <div id="div1">
      <div id="div2">
        <div id="div3"></div>
      </div>
    </div>
  </body>
</html>
```

```css
body {
  width: 100%;
  height: 500px;
}

div {
  display: flex;
  justify-content: center;
  align-items: center;
}

#div1 {
  width: 100%;
  height: 100%;
  background-color: aquamarine;
}

#div2 {
  width: 50%;
  height: 50%;
  background-color: beige;
}

#div3 {
  width: 25%;
  height: 25%;
  background-color: goldenrod;
}
```

![event test code](https://user-images.githubusercontent.com/63287638/178144554-11968076-1eaa-47db-a43b-518bcbc48bd5.png)  

```javascript
// querySelector로 가져온 뒤 forEach를 돌리면 되는데 귀찮아서 일일이 다 씀...
const div3 = document.getElementById("div3");
div3.addEventListener("click", (e) => {
  alert("click div3");
});

const div2 = document.getElementById("div2");
div2.addEventListener("click", (e) => {
  alert("click div2");
});

const div1 = document.getElementById("div1");
div1.addEventListener("click", (e) => {
  alert("click div1");
});
```

### stopPropagation
이벤트가 전파되는 것을 막는 것이다.  
아래 [이벤트 캡처링](https://github.com/mochang2/development-diary/edit/main/025-fundamentals%20of%20js.md#event-capturing)에서도 동일한 원리로 동작한다.  

```javascript
const div3 = document.getElementById("div3");
div3.addEventListener("click", (e) => {
  // e.stopPropagation(); // 여기에 쓰면 click div3까지만 실행된다
  alert("click div3");
});

const div2 = document.getElementById("div2");
div2.addEventListener("click", (e) => {
  // e.stopPropagation(); // 여기에 쓰면 click div2까지만 실행된다
  alert("click div2");
});

const div1 = document.getElementById("div1");
div1.addEventListener("click", (e) => {
  alert("click div1");
});
```

### event capturing
이벤트가 전파되는데, bubbling과는 반대 방향으로 전파된다.  
즉, 부모의 이벤트가 먼저 실행되는 것이다.  
아래와 같은 코드면 `click div1` -> `click div2` -> `click div3` 순으로 실행된다.  

```javascript
const div3 = document.getElementById("div3");
div3.addEventListener(
  "click",
  (e) => {
    alert("click div3");
  },
  { capture: true }
);

const div2 = document.getElementById("div2");
div2.addEventListener(
  "click",
  (e) => {
    alert("click div2");
  },
  { capture: true } // 호기심에 이 부분을 빼보니 div2의 이벤트가 가장 늦게 실행됐다
);

const div1 = document.getElementById("div1");
div1.addEventListener(
  "click",
  (e) => {
    alert("click div1");
  },
  { capture: true }
);
```

### event delegation
하위 요소에 각각 이벤트를 붙이지 않고 상위 요소에서 하위 요소의 이벤트들을 제어하는 방식이다.  
focus와 같이 이벤트 전파가 되지 않거나 stopPropagation을 사용하지 않았을 때 사용 가능하다.  
또한 동적으로 element가 새로 생성될 때, 해당 객체에 대한 이벤트를 자동으로 할당할 수 있어서 유용하게 사용할 수 있는 기술이다.  

_출처: https://joshua1988.github.io/web-development/javascript/event-propagation-delegation/_

```html
<h1>오늘의 할 일</h1>
<ul class="itemList">
	<li>
		<input type="checkbox" id="item1">
		<label for="item1">이벤트 버블링 학습</label>
	</li>
	<li>
		<input type="checkbox" id="item2">
		<label for="item2">이벤트 캡쳐 학습</label>
	</li>
</ul>
```

```javascript
// input에 이벤트 추가
var inputs = document.querySelectorAll('input');
inputs.forEach(function(input) {
	input.addEventListener('click', function(event) {
		alert('clicked');
	});
});

// 새로운 element를 추가하는 코드
var itemList = document.querySelector('.itemList');

var li = document.createElement('li');
var input = document.createElement('input');
var label = document.createElement('label');
var labelText = document.createTextNode('이벤트 위임 학습');

input.setAttribute('type', 'checkbox');
input.setAttribute('id', 'item3');
label.setAttribute('for', 'item3');
label.appendChild(labelText);
li.appendChild(input);
li.appendChild(label);
itemList.appendChild(li);
```

위에 코드까지는 새로운 TODO가 생기지 않다면 잘 동작할 것이다.  
하지만 새로운 TODO가 추가될 경우 원하는 click 이벤트가 잘 동작하지 않는다.  
이럴 때 event delegation을 이용할 수 있는데, `.itemList` 자체에 이벤트를 다는 것이다.  

```javascript
// 위에 쓴 JS 코드 다 지우고
var itemList = document.querySelector('.itemList');
itemList.addEventListener('click', function(event) {
	alert('clicked');
});
```


## 6. async vs defer
