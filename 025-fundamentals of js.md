## 0. 공부하게 된 계기

면접으로 자주 나온다는 js에 관한 질문들을 단순히 답만 외우는 게 아니라 자세히 정리하고 싶어졌다.

### 목차

- [GC](https://github.com/mochang2/development-diary/blob/main/025-fundamentals%20of%20js.md#1-gcgarbage-collection)
- [prototype](https://github.com/mochang2/development-diary/blob/main/025-fundamentals%20of%20js.md#2-prototype)
- [closure](https://github.com/mochang2/development-diary/blob/main/025-fundamentals%20of%20js.md#3-closure)
- [this](https://github.com/mochang2/development-diary/blob/main/025-fundamentals%20of%20js.md#4-this)
- [event bubbling](https://github.com/mochang2/development-diary/blob/main/025-fundamentals%20of%20js.md#5-event-bubbling)
- [async vs defer](https://github.com/mochang2/development-diary/blob/main/025-fundamentals%20of%20js.md#6-async-vs-defer)
- [event loop](https://github.com/mochang2/development-diary/blob/main/025-fundamentals%20of%20js.md#7-event-loop)

## 1. GC(garbage collection)

_출처: https://blog.sessionstack.com/how-javascript-works-memory-management-how-to-handle-4-common-memory-leaks-3f28b94cfbec , https://developer.mozilla.org/ko/docs/Web/JavaScript/Memory_Management_

<img src="https://user-images.githubusercontent.com/63287638/178136632-c33e7ae6-c245-4cd9-a1c9-63a6c434d678.png" alt="memory lifecycle" width="50%" />

위 사진에서 볼 수 있듯이 메모리는 크게 3가지의 life cycle을 가지고 있다.  
할당한 뒤에 사용되고, 사용되지 않으면 해제된다.  
C와 같은 저레벨 언어에서는 allocate와 release를 malloc(calloc 등)과 free를 통해서 개발자가 직접 명령해야 하지만 JS, JAVA, Python 등은 별도의 Garbage Collector가 동작해서 메모리 관리를 해준다.

```javascript
let n = 374 // allocates memory for a number

let s = 'sessionstack' // allocates memory for a string

let o = {
  // allocates memory for an object and its contained values
  a: 1,
  b: null,
}

let a = [1, null, 'str'] // (like object) allocates memory for the array

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
  variable = 'text text'
  // == window.variable 또는 global.variable = 'text text'
  // == this.variable = 'text text'
  // 이러한 전역 오염을 막기 위해서는 var, let, const 반드시 사용
}
```

**필요 없는 Timer나 callback**

```javascript
let serverData = loadData()

setInterval(function () {
  let what = document.getElementById('what')
  if (what) {
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
      console.log('hi')
    }
  }

  theThing = {
    // replaceThing이 아래에서 호출되면서 더이상 null이 아님.
    longStr: new Array(1000000).join('*'),
    someMethod: function () {
      console.log('message')
    },
  }
}

setInterval(replaceThing, 1000)
```

**DOM 객체 참조**

```javascript
let elements = {
  button: document.getElementById('button'),
  image: document.getElementById('image'),
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

OOP에서 class처럼 동일한 속성을 지정해주기 위한 방법이다 정도로 이해할 수도 있다.  
ES6에 도입된 class가 있지만 (MDN에 따르면) class는 문법적인 양념일 뿐, JS는 여전히 프로토타입 기반의 언어이며 내부 동작 방식도 prototype을 이용한다.

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
  this.badRequest = 400
  this.unauthorized = 403
  this.notFound = 404
}

const imError = new error()
const urError = new error()

console.log(imError) // error { badRequest: 400, unauthorized: 403, notFound: 404 }
console.log(urError) // error { badRequest: 400, unauthorized: 403, notFound: 404 }
console.log(imError.info) // undefined

error.prototype.info = '에러에러에러'

console.log(imError) // error { badRequest: 400, unauthorized: 403, notFound: 404 }
console.log(urError) // error { badRequest: 400, unauthorized: 403, notFound: 404 }
console.log(imError.info) // 에러에러에러. prototype chaining. imError은 info가 없으나 error는 info가 있음.
console.log(urError.info) // 에러에러에러
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
const a = { a: 1 }
// a ---> Object.prototype ---> null

const b = Object.create(a)
// b ---> a ---> Object.prototype ---> null
console.log(b.a) // 1 (상속됨)

const c = Object.create(b)
// c ---> b ---> a ---> Object.prototype ---> null

const d = Object.create(null)
// d ---> null
console.log(d.hasOwnProperty) // undefined이다. 왜냐하면 d는 Object.prototype을 상속받지 않기 때문이다.
```

다만 https://velog.io/@thms200/Object.create- 를 참고하면 알 수 있듯이 `Object.create()`이나 `new`생성자를 통해 부모 객체의 기능을 물려받으면 생성자 함수를 사용할 수 없기 때문에 `Rectangle.prototype = Object.create(Shape.prototype); Rectangle.prototype.constructor = Rectangle;`과 같이 생성자 property를 별도로 명시해줘야 한다.

```javascript
function Animal() {
  this.type = 'Animal'
}

function Mammal(type) {
  this.subType = 'Mammal'
}

Mammal.prototype = new Animal()
// Mammal.prototype.constructor = Mammal

function Cat(name) {
  this.name = name
}

Cat.prototype = new Mammal('cat')
// Cat.prototype.constructor = Cat

const siamese = new Cat('siamese')

console.log(siamese)
console.log(Object.getPrototypeOf(siamese))
console.log(Object.getPrototypeOf(Object.getPrototypeOf(siamese)))
console.log(
  Object.getPrototypeOf(Object.getPrototypeOf(Object.getPrototypeOf(siamese)))
)

```

위에서 주석처리한 줄에 따라 아래와 같은 서로 다른 결과가 나온다.  
![wo constructor](https://user-images.githubusercontent.com/63287638/181062303-d70b6ad4-cc5e-415a-8bc9-3e5f38f58e25.png)  

![with constructor](https://user-images.githubusercontent.com/63287638/181062312-6d274056-1516-400b-bb1d-69a2aaa9e703.png)



## 3. closure

흔히 함수 내에서 함수를 정의하고 사용하면 클로저라고 말하지만, 정확히 말하자면 내부에 선언된 함수가 외부 함수의 지역변수를 사용했을 때만 클로저라고 선언된다.  
MDN에 따르면 `클로저는 주변의 상태의 참조와 함께 번들로 묶인 함수의 조합입니다. 즉, 클로져는 우리에게 inner함수에서 outer함수의 스코프에 접근을 가능하게 해줍니다. 자바스크립트에서 클로저는 함수가 생성될 때마다 생성됩니다.`  
일반적으로 정의한 함수를 리턴하고 사용은 바깥에서 함으로써 클로저를 사용한다.

아래 예제 코드들을 통해서 클로저란 무엇인가를 알 수 있을 것이다.

```javascript
function func() {
  const test = '123'
  console.log(test)
}

func() // 123
console.log(test) // error
```

위 예시처럼 함수 밖에서 name을 호출하는 것은 당연히 에러를 발생시킬 것이다.  
하지만 아래와 같이 사용하면 `outer` 내부에서 선언한 test에 접근할 수 있다.  
위에서 이야기한대로 `inner`는 해당 함수가 선언된 상태를 기억하고 있기 때문이다.

```javascript
function outer() {
  const test = '123'
  console.log(test)

  return function inner() {
    const variable = '456!'
    console.log(test + variable)
  }
}

const getTest = outer() // 123
getTest() // 123456!
```

클로저를 활용하는 유명한 예제가 있다.

```javascript
var i
for (i = 0; i < 10; i++) {
  setTimeout(function () {
    console.log(i) // 10 열 번 출력
  }, 100)
}

// IIFE 활용
var i
for (i = 0; i < 10; i++) {
  ;(function (j) {
    setTimeout(function () {
      console.log(j) // 1 ~ 10 출력
    }, 100)
  })(i)
}
```

위에서 '정확히 말하자면 내부에 선언된 함수가 외부 함수의 지역변수를 사용했을 때만 클로저라고 선언된다'고 말했다.  
아래 코드를 보면 해당 내용의 의미를 알 수 있다.

```javascript
function outer() {
  const test = '123'

  if (true) {
    const variable = '456!'

    return function inner() {
      console.log(variable)
    }
  }
}

const aa = outer()
console.dir(aa) // [[Scopes]] 내부에 closure가 없음
```

```javascript
function outer() {
  const test = '123'

  if (true) {
    const variable = '456!'
    return function inner() {
      console.log(test + variable)
    }
  }
}

const aa = outer()
console.dir(aa) // 아래와 같이 [[Scopes]] 내부에 closure 존재
```

<img alt="closure" src="https://user-images.githubusercontent.com/63287638/178147027-061f6477-6aa4-4d32-9284-e2346f909f20.png" width="350" />

### 클로저 장점 1. 은닉화

ES6에서 class 문법이 생기면서 private을 사용할 수 있지만, [prototype](https://github.com/mochang2/development-diary/blob/main/025-fundamentals%20of%20js.md#2-prototype)처럼 JS는 여전히 프로토타입 기반의 언어이며 내부 동작 방식이 같지 않다.

```javascript
function hello(name) {
  this._name = name
}

Hello.prototype.say = function () {
  console.log('Hello, ' + this._name)
}

const hello1 = new hello('123')
const hello2 = new hello('456')
const hello3 = new hello('789')

hello1.say() // 'Hello, 123'
hello2.say() // 'Hello, 456'
hello3.say() // 'Hello, 789'
hello1._name = 'sudo'
hello1.say() // 'Hello, sudo'
```

위 코드는 Hello의 private 변수인 \_name(JS 네이밍 컨벤션에 따르면 private variable)에 외부에서 마음껏 접근할 수 있다.  
하지만 아래와 같이 코딩하면 불가능해진다.

```javascript
function hello(name) {
  const _name = name

  return function () {
    console.log('Hello, ' + _name)
  }

  /* 또는
  return function() {
    console.log('Hello, ' + name)
  }
  */
}

const hello1 = new hello('123')
const hello2 = new hello('456')
const hello3 = new hello('789')
```

### 클로저 장점 2. 전역 변수의 사용을 줄임

count와 같이 많이 사용할 수 있는 변수의 이름을 사용해서 전역 오염시키지 않을 수 있다.

```javascript
let count = 0
function handleCilck() {
  count++
  return count
}

// 대신에 아래처럼

function handleCilck() {
  let count = 0
  return function () {
    count++
    return count
  }
}
```

### 클로저 장점 3. 재사용률을 높임

~모든 함수의 특징이기 때문에 따로 예제 코드가 필요 없을 것 같다~

### 주의할 점: 메모리 관리

클로저를 통해 내부 변수를 참조하는 동안에는 내부 변수가 차지하는 메모리를 GC가 회수하지 않는다.  
따라서 클로저 사용이 끝나면 참조를 제거하는 것이 좋다.  
c++에서 delete를 쓰던 것처럼 객체에 null을 넣어주면 된다.

```javascript
function hello(name) {
  const _name = name

  return function () {
    console.log('Hello, ' + _name)
  }
}

const hello1 = new hello('123')

hello1 = null // release
```

## 4. this

JS는 함수가 어떻게 불리느냐에 따라서 this에 대한 서로 다른 4가지 용법을 가지고 있다.

### 일반 함수

웹에서는 window, node에서는 global과 같다.

```javascript
function func() {
  return this === global
}

console.log(func()) // true
```

### 객체의 값으로서의 함수

객체 그 자체를 가리킨다.

```javascript
const obj = {
  value: 123,
  key: function func() {
    return this.value
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
  console.log(this)
}
new func() // func {}

function func2() {
  console.log(this.key)
  this.key = 'value'
  console.log(this.key)
}
new func2()
// undefined
// value

function func3() {
  this.age = 100
  return 3
}
const a = new func3()
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
  log: foo,
}

foo.call(ken, 1, 2, 3) // 35
```

### 참고 arrow function vs 일반 function

[MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/Arrow_functions)에 따르면 arrow function은 일반 function과 비교했을 때 다음과 같은 특성이 있다.

> - this나 super에 대한 바인딩이 없고, methods 로 사용될 수 없습니다.
> - new.target 키워드가 없습니다.
> - 일반적으로 스코프를 지정할 때 사용하는 call, apply, bind methods를 이용할 수 없습니다.
> - 생성자(Constructor)로 사용할 수 없습니다.
> - yield를 화살표 함수 내부에서 사용할 수 없습니다

여기서 주목할 것은 첫 번째 속성으로, arrow function은 this에 대한 binding이 위에 예시로 제시한 함수들처럼 동적으로 진행되지 않는다.  
즉, 해당 함수가 어떻게 선언되었냐에 따라 결정되지 않는다는 것이다.  
arrow function을 부른 위치가 어디냐에 따라서 해당 함수 내부에서의 this가 정적으로 결정되어 있다.

```javascript
function func() {
  console.log('Inside func:', this.aa // 13

  return {
    aa: 25,
    arrow: () => console.log('Inside aa:', this.aa), // 13
  }
}

func.call({ aa: 13 }).func()
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
const div3 = document.getElementById('div3')
div3.addEventListener('click', (e) => {
  alert('click div3')
})

const div2 = document.getElementById('div2')
div2.addEventListener('click', (e) => {
  alert('click div2')
})

const div1 = document.getElementById('div1')
div1.addEventListener('click', (e) => {
  alert('click div1')
})
```

### stopPropagation

이벤트가 전파되는 것을 막는 것이다.  
아래 [이벤트 캡처링](https://github.com/mochang2/development-diary/edit/main/025-fundamentals%20of%20js.md#event-capturing)에서도 동일한 원리로 동작한다.

```javascript
const div3 = document.getElementById('div3')
div3.addEventListener('click', (e) => {
  // e.stopPropagation(); // 여기에 쓰면 click div3까지만 실행된다
  alert('click div3')
})

const div2 = document.getElementById('div2')
div2.addEventListener('click', (e) => {
  // e.stopPropagation(); // 여기에 쓰면 click div2까지만 실행된다
  alert('click div2')
})

const div1 = document.getElementById('div1')
div1.addEventListener('click', (e) => {
  alert('click div1')
})
```

### event capturing

이벤트가 전파되는데, bubbling과는 반대 방향으로 전파된다.  
즉, 부모의 이벤트가 먼저 실행되는 것이다.  
아래와 같은 코드면 `click div1` -> `click div2` -> `click div3` 순으로 실행된다.

```javascript
const div3 = document.getElementById('div3')
div3.addEventListener(
  'click',
  (e) => {
    alert('click div3')
  },
  { capture: true }
)

const div2 = document.getElementById('div2')
div2.addEventListener(
  'click',
  (e) => {
    alert('click div2')
  },
  { capture: true } // 호기심에 이 부분을 빼보니 div2의 이벤트가 가장 늦게 실행됐다
)

const div1 = document.getElementById('div1')
div1.addEventListener(
  'click',
  (e) => {
    alert('click div1')
  },
  { capture: true }
)
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
    <input type="checkbox" id="item1" />
    <label for="item1">이벤트 버블링 학습</label>
  </li>
  <li>
    <input type="checkbox" id="item2" />
    <label for="item2">이벤트 캡쳐 학습</label>
  </li>
</ul>
```

```javascript
// input에 이벤트 추가
var inputs = document.querySelectorAll('input')
inputs.forEach(function (input) {
  input.addEventListener('click', function (event) {
    alert('clicked')
  })
})

// 새로운 element를 추가하는 코드
var itemList = document.querySelector('.itemList')

var li = document.createElement('li')
var input = document.createElement('input')
var label = document.createElement('label')
var labelText = document.createTextNode('이벤트 위임 학습')

input.setAttribute('type', 'checkbox')
input.setAttribute('id', 'item3')
label.setAttribute('for', 'item3')
label.appendChild(labelText)
li.appendChild(input)
li.appendChild(label)
itemList.appendChild(li)
```

위에 코드(`addEventListener` 코드)까지는 새로운 TODO가 생기지 않다면 잘 동작할 것이다.  
하지만 새로운 TODO가 추가될 경우(`addEventListener` 아래 코드) 원하는 click 이벤트가 잘 동작하지 않는다.  
이럴 때 event delegation을 이용할 수 있는데, `.itemList` 자체에 이벤트를 다는 것이다.

```javascript
// 위에 쓴 JS 코드 다 지우고
var itemList = document.querySelector('.itemList')
itemList.addEventListener('click', function (event) {
  alert('clicked')
})
```

## 6. async vs defer

_출처: https://jae04099.tistory.com/entry/HTML-script-%ED%83%9C%EA%B7%B8%EB%8A%94-%EC%96%B4%EB%94%94%EC%97%90-%EC%9C%84%EC%B9%98-%ED%95%B4%EC%95%BC-%ED%95%A0%EA%B9%8C_

[5](https://github.com/mochang2/development-diary/edit/main/025-fundamentals%20of%20js.md#5-event-bubbling)에서 예시로 쓴 코드에서처럼 HTML에서 script 모듈을 가져올 때 defer 옵션을 주지 않으면 JS에서 getElementById 등의 DOM과 관련된 상호작용하는 코드가 제대로 동작하지 않는다.  
div1, div2, div3가 undefined 객체라고 뜨는데 이는 HTML에서 JS를 실행시키는 시점과 관련이 있다.

웹에서 파싱이란 네트워크로 받은 HTML과 CSS 파일을 토큰화시키고 Parse Tree를 생성하는 것이다.  
이 Parse Tree를 DOM 트리로 만들어 렌더링하는 것이다.  
HTML을 화면에 보이기 위해 파싱하는 도중에 파싱을 멈추는 경우가 있는데, 바로 script 태그를 만날 때다.

script 태그는 실행 전에 다운로드 과정을 거친다.  
다음은 script 태그의 위치에 따른 구분이다.

### head 태그 내부

script 파일이 너무 크다면 HTML을 파싱하다가 사용자에게 빈 화면을 보여줄 수 있다.  
또한, script 내에서 DOM 요소를 조작한다면 존재하지 않는 DOM 요소에 접근하려는 시도이므로 제대로 된 접근이 되지 않을 수도 있다.

```html
<html>
  <head>
    <script src="~~"></script>
  </head>
  <body>
    <!-- ... -->
  </body>
</html>
```

![head 내부](https://user-images.githubusercontent.com/63287638/178145304-29b634ad-d2dd-4db1-9c52-df30b05b505a.png)

### body 태그 마지막

head 태그에 넣은 방법보다 HTML 콘텐츠를 빠르게 보여줄 수 있다.  
하지만 웹이 자바스크립트에 의존적이라면 사용자 경험을 안 좋게 만들 수 있다.

```html
<html>
  <head>
    <!-- ... -->
  </head>
  <body>
    <!-- ... -->
    <script src="~~"></script>
  </body>
</html>
```

![body의 마지막](https://user-images.githubusercontent.com/63287638/178145324-9301a9f4-8184-4ed7-8dae-5a5012de7b24.png)

### async

head 태그 내부에서만 사용 가능한 옵션이다.  
자바스크립트에 의존적인 웹을 좀 더 빨리 실행시킬 수 있다.  
하지만 HTML 파싱이 자바스크립트 파일을 실행시키는 동안 멈추게 돼 사용자 경험이 좋지 않을 수 있다.  
또한 script 태그가 여러 개일 경우, 순서에 상관 없이 먼저 다운로드 받아지는 script를 실행시키기 때문에 해당 프로젝트가 script 파일의 실행 순서에 영향을 받는다면 문제가 될 수 있다.

```html
<html>
  <head>
    <script async src="~~"></script>
  </head>
  <body>
    <!-- ... -->
  </body>
</html>
```

![async](https://user-images.githubusercontent.com/63287638/178145367-efe0f799-618f-402d-96cc-8c80b35f798d.png)

### defer

head 태그 내부에서만 사용 가능한 옵션이다.  
async처럼 script 다운을 HTML을 파싱할 때 진행하지만, 다른 점은 실행을 HTML 파싱이 전부 완료된 뒤에 한다는 것이다.  
여러 개의 script 태그를 이용해도 미리 다운을 다 받고 HTML도 끝까지 파싱시킨 후에 실행되기 때문에 async의 단점을 보완할 수 있다.

```html
<html>
  <head>
    <script defer src="~~"></script>
  </head>
  <body>
    <!-- ... -->
  </body>
</html>
```

![defer](https://user-images.githubusercontent.com/63287638/178145371-3b64a83a-f709-43b2-96b7-c6ee5a33798c.png)

## 7. Event Loop

_참고: https://medium.com/sjk5766/javascript-%EB%B9%84%EB%8F%99%EA%B8%B0-%ED%95%B5%EC%8B%AC-event-loop-%EC%A0%95%EB%A6%AC-422eb29231a8, https://velog.io/@thms200/Event-Loop-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EB%A3%A8%ED%94%84_

JS는 싱글 스레드 기반이며 비동기적으로 동작한다.  
싱글 스레드이기 때문에 한 순간에 하나의 작업만 가능하지만, 비동기로 동작하기 때문에 단일 스레드임에도 불구하고 짧은 시간 동안 많은 작업을 수행할 수 있다.

다만, JS 자체가 비동기 동작을 지원하는 것이 아닌, **브라우저** 가 가진 요소(node에서는 libuv 라이브러리)를 이용해서 비동기 동작을 처리한다.  
이 부분이 Event Loop이다.

### 사전 지식

JS의 내부 구조에 대한 설명을 하기에 앞서 이후에 계속 나올 Execution Context와 Call Stack을 먼저 정리하겠다.

##### Execution Context

Execution Context는 JS 코드가 실행되는 환경을 의미하며 두 가지 타입의 Execution Context가 있다.  
하나는 Global Execution Context로 JS 엔진에 의해 처음 코드가 실행될 때 생성된다.  
또다른 종류로는 Function Execution Context가 있는데, 이것은 함수가 호출될 때마다 생성된다.  
모든 함수는 호출되는 시점에 자신만의 Execution Context를 가진다.

Execution Context는 Creation, Execution 두 단계에 걸쳐서 동작한다.  
Creation 단계에는 다음 세 가지 일이 발생한다.

- Window 객체(Function Execution Context 같은 경우는 arguments 객체) 생성
- this 객체 생성
- 호이스팅이 일어나는 변수(var로 선언한 변수)는 undefined를 할당하고 함수 선언문은 실제로 메모리 할당됨

그 다음 단계인 Execution 단계에서는 코드를 한 줄씩 실행하고 변수에 실제 값을 할당한다.

##### Call Stack

Call Stack은 이러한 Execution Context를 저장하는 자료 구조이다.  
Stack이라는 이름에서 알 수 있듯이 LIFO 구조이며, c에서 함수가 호출돼서 스택에 저장되는 방식과 같이 동작한다.

```javascript
function first() {
  console.log('first')
  second()
}

function second() {
  console.log('second')
}

first()
```

위 함수는 아래와 같은 순서로 동작한다.

![call stack](https://user-images.githubusercontent.com/63287638/179402612-a655c611-dda0-4080-92cb-18985d356b63.png)  
_출처: https://medium.com/sjk5766/call-stack%EA%B3%BC-execution-context-%EB%A5%BC-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90-3c877072db79_

에러도 위와 같은 방식으로 동작하기 때문에 에러가 발생했을 때 처음 발생한 위치를 추적해갈 수 있는 것이다.

### Event Loop 동작 방식

<img src="https://user-images.githubusercontent.com/63287638/179403074-a3be0979-9d90-46c7-b07d-e0a38383e0d6.png" alt="event loop" width="80%" />

##### JS Engine

위 그림에서 보듯이 JS Engine은 Memory Heap과 Call Stack으로 이루어져 있다.  
위에서 이야기한 JS가 싱글 스레드라는 의미는 Call Stack이 하나라는 이야기이다.  
Memory Heap은 메모리 할당(변수, 함수 등이 담김)이 이루어지는 곳이며,  
Call Stack은 코드가 실행될 때 LIFO 형식으로 쌓이는 곳이다.

##### Web API

Web API는 브라우저(또는 libuv)에서 제공하는 API로 DOM, Ajax, Timer 등이 있다.  
Call Stack에서 실행된 비동기 함수는 Web API를 호출하고, Web API는 콜백함수를 Callback Queue에 넣는다.

##### Event Table

특정 event가 발생했을 때 어떤 callback 함수가 호출되어야 하는지를 알고 있는 자료구조이다.

##### Callback Queue

비동기적으로 실행되어야 할 콜백함수가 보관되는 공간이다.  
예를 들어 setTimeout 타이머 완료 후 실행되는 함수나 addEventListener에서 이벤트가 발생했을 때 실행되는 함수 등이 보관된다.

##### Event Loop

Event Loop가 하는 역할 자체는 간단하다.  
Call Stack을 지켜보고 있다가 Call Stack이 비었을 경우, Callback Queue에서 함수를 꺼내 Call Stack에 집어넣음으로써 실행되게 한다.  
이러한 반복적인 행동을 틱(tick)이라 한다.

```javascript
console.log('1')

setTimeout(() => {
  console.log('3')
}, 3000)

console.log('2')
```

위 코드에 따른 결과는 `1 2 3` 순이다.  
코드 동작 방식은 다음과 같다.

> 1. Global Execution Context가 생성되면서 window(또는 global) 객체가 생성되고 (변수는 없으므로 pass) 함수를 위한 메모리가 할당된다.
> 2. console.log('1')이 Call Stack에 들어갔다가 나오면서 콘솔에 '1'이 출력된다.
> 3. setTimeout 부분이 Call Stack에 들어갔다가 나오면서 Timer Web API를 호출하고, setTimeout의 콜백 부분은 3초 후에 Callback Queue에 들어간다.
> 4. console.log('2')이 Call Stack에 들어갔다가 나오면서 콘솔에 '2'가 출력된다.
> 5. Event Loop가 Call Stack이 비어있음을 확인한 뒤 Callback Queue에 들어갔던 타이머 콜백을 Call STack에 추가한다.
> 6. 콜백이 실행되면서 console.log('3')이 Call Stack에 쌓였다가 사라지면서 콘솔에 '3'이 출력된다.
> 7. 타이머 콜백이 Call Stack에서 사라진다.
> 8. 모든 Execution Context이 사라진다.

### Micro Task Queue

`Promise`의 thenable 메소드와 관련된 콜백이 들어가는 Queue를 말한다.  
위에서 이야기한 Callback Queue에 있는 콜백들보다 먼저 실행된다.

```javascript
const waitASecond = () => {
  let start = Date.now()
  let now = start

  while (now - start < 1000) {
    now = Date.now()
  }
}

console.log('1')

setTimeout(function () {
  console.log('2')
}, 0)

let promise = new Promise((resolve, reject) => resolve())

promise.then((resolve) => console.log('3')).then((resolve) => console.log('4'))

waitASecond()

console.log('5')
```

위의 내용들을 정리했을 때 코드는 `1 5 3 4 2` 순으로 출력된다.

### Animation Frames

requestAnimationFrame API가 실행되면 콜백이 Animation Frames으로 담긴다.  
크롬 기준으로 Microtask Queue > Animation Frames > Callback Queue(Task Queue) 순으로 실행된다.

~requestAnimationFrame API를 사용해본 것이 아니라 정확하지 않을 수 있다. 추후에 사용하는 일이 있다면 자세히 알아봐야겠다~
