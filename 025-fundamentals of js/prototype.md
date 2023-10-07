## Prototype

참고  
https://www.youtube.com/watch?v=wUgmzvExL_E

OOP에서 class처럼 동일한 속성을 지정해주기 위한 방법이다 정도로 이해할 수도 있다.  
ES6에 도입된 class가 있지만 (MDN에 따르면) class는 문법적인 양념일 뿐, JS는 여전히 프로토타입 기반의 언어이며 내부 동작 방식도 prototype을 이용한다.

상속 관점에서 자바스크립트의 유일한 생성자는 객체뿐이다.  
각각의 객체는 \[Prototype\]이라는 은닉(private) 속성을 가지는데 자신의 프로토타입이 되는 다른 객체를 가리킨다.  
그 객체의 프로토타입 또한 프로토타입을 가지고 있고 이것이 반복되다, 결국 null을 프로토타입으로 가지는 오브젝트에서 끝난다.  
null은 더 이상의 프로토타입이 없다고 정의되며, 프로토타입 체인의 종점 역할을 한다.

cf) prototype.\_\_proto\_\_ 이라는 속성이 존재하는데 이는 객체가 만들어지기 위해 사용된 원형인 프로토타입 객체를 숨은 링크로 참조하는 역할을 한다. [MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/proto)에 따르면 deprecated됐으며 `Object.getPrototypeOf(변수)`를 사용하기를 권장하고 있다.

### 함수와 객체의 내부 구조

```javascript
function Func() {}

let f1 = new Func();
let f2 = new Func();
```

위와 같은 코드는 아래와 같은 내부 구조를 가진다.

<img src="https://user-images.githubusercontent.com/63287638/178142162-7a6174d6-7f9f-4d33-8caa-b92d2a9e1190.png" alt="함수와 객체의 내부 구조" width="75%" />

### 예제 코드

```javascript
function error() {
  this.badRequest = 400;
  this.unauthorized = 403;
  this.notFound = 404;
}

const imError = new error();
const urError = new error();

console.log(imError); // error { badRequest: 400, unauthorized: 403, notFound: 404 }
console.log(urError); // error { badRequest: 400, unauthorized: 403, notFound: 404 }
console.log(imError.info); // undefined

error.prototype.info = '에러에러에러';

console.log(imError); // error { badRequest: 400, unauthorized: 403, notFound: 404 }
console.log(urError); // error { badRequest: 400, unauthorized: 403, notFound: 404 }
console.log(imError.info); // 에러에러에러. prototype chaining. imError은 info가 없으나 error는 info가 있음.
console.log(urError.info); // 에러에러에러
```

### Array의 비밀

구글에 'js array map'이라고 검색하면 MDN 사이트의 제목으로 `Array.prototype.map()`이 나온다.  
내가 `const whatArray = [1,2,3]` 이라고 선언하면 `whatArray.map(~)`이나 `whatArray.sort()` 등의 내장 함수를 사용할 수 있는 이유이다.  
실제로 `console.log(Array.prototype)` 이라고 하면 아래와 같은 결과물이 출력된다.

<img src="https://user-images.githubusercontent.com/63287638/178142653-4234dc4a-9b3c-483c-b877-62b3b2c39876.png" alt="array prototype" width="400" />

### Object.create

ES5에 객체를 생성하는 새로운 방법이 도입됐었다.  
Object.create를 이용해서 새로운 객체를 만들면, 생성된 객체의 프로토타입은 이 메소드의 첫 번째 인수로 지정된다.

```javascript
const a = { a: 1 };
// a ---> Object.prototype ---> null

const b = Object.create(a);
// b ---> a ---> Object.prototype ---> null
console.log(b.a); // 1 (상속됨)

const c = Object.create(b);
// c ---> b ---> a ---> Object.prototype ---> null

const d = Object.create(null);
// d ---> null
console.log(d.hasOwnProperty); // undefined이다. 왜냐하면 d는 Object.prototype을 상속받지 않기 때문이다.
```

다만 https://velog.io/@thms200/Object.create- 를 참고하면 알 수 있듯이 `Object.create()`이나 `new`생성자를 통해 부모 객체의 기능을 물려받으면 생성자 함수를 사용할 수 없기 때문에 `Rectangle.prototype = Object.create(Shape.prototype); Rectangle.prototype.constructor = Rectangle;`과 같이 생성자 property를 별도로 명시해줘야 한다.

```javascript
function Animal() {
  this.type = 'Animal';
}

function Mammal(type) {
  this.subType = 'Mammal';
}

Mammal.prototype = new Animal();
// Mammal.prototype.constructor = Mammal // Mammal class의 생성자는 Mammal이다

function Cat(name) {
  this.name = name;
}

Cat.prototype = new Mammal('cat');
// Cat.prototype.constructor = Cat // Cat class의 생성자는 Cat이다

const siamese = new Cat('siamese');

console.log(siamese);
console.log(Object.getPrototypeOf(siamese));
console.log(Object.getPrototypeOf(Object.getPrototypeOf(siamese)));
console.log(
  Object.getPrototypeOf(Object.getPrototypeOf(Object.getPrototypeOf(siamese)))
);
```

위에서 주석처리한 줄에 따라 아래와 같은 서로 다른 결과가 나온다.  
![wo constructor](https://user-images.githubusercontent.com/63287638/181062303-d70b6ad4-cc5e-415a-8bc9-3e5f38f58e25.png)

![with constructor](https://user-images.githubusercontent.com/63287638/181062312-6d274056-1516-400b-bb1d-69a2aaa9e703.png)

### Class에서 method로서 arrow function을 사용하게 되면 생기는 문제들

참고  
https://simsimjae.tistory.com/452  
https://www.charpeni.com/blog/arrow-functions-in-class-properties-might-not-be-as-great-as-we-think

위 블로그들을 읽고, React class component에서 arrow function method를 사용하면 안 되나? 라는 의문이 들었다.  
먼저 나의 결론을 말하자면 _arrow function method 써도 상관없다_ 이다.

다음에 나열할 모든 문제의 원인은 class가 transpile되면서 생기는 문제이다.

아래와 같이 class를 선언했다고 하자.

```js
class A {
  static color = 'red';
  counter = 0;

  handleClick = () => {
    this.counter++;
  };

  handleLongClick() {
    this.counter++;
  }
}
```

이를 es7으로 transpiling하면 다음과 같은 결과가 나온다.

```js
'use strict';

function _defineProperty(obj, key, value) {
  // ...
}
function _toPropertyKey(arg) {
  // ...
}
function _toPrimitive(input, hint) {
  // ...
}
class A {
  constructor() {
    _defineProperty(this, 'counter', 0);
    _defineProperty(this, 'handleClick', () => {
      this.counter++;
    });
  }
  handleLongClick() {
    this.counter++;
  }
}
_defineProperty(A, 'color', 'red');
```

`_defineProperty`, `_toPropertyKey`, `_toPrimitive`은 선언이므로 무시한다고 하고, 중요한 것은 `handleClick`이라고 선언했던 arrow function method가 `counter`와 같이 instance 변수처럼 `constructor` 내부로 들어갔다는 사실이다.

#### constructor 내부로 들어가면 무슨 문제가 생기는데?

1. overriding이 제대로 동작하지 않을 수 있다.

위와 같이 선언해도 `A` 클래스에서는 아무 문제가 없다.

```js
class A {
  handleClick = () => {
    console.log('A.handleClick');
  };

  handleLongClick() {
    console.log('A.handleLongClick');
  }
}

console.log(A.prototype); // {constructor: ƒ, handleLongClick: ƒ}
new A().handleClick(); // A.handleClick
new A().handleLongClick(); // A.handleLongClick
```

하지만 `B`, `C` 클래스가 `A`를 상속한다면 문제가 발생한다.

```js
class B extends A {
  handleClick = () => {
    super.handleClick();

    console.log('B.handleClick');
  };

  handleLongClick() {
    super.handleLongClick();

    console.log('B.handleLongClick');
  }
}

console.log(B.prototype); // A {constructor: ƒ, handleLongClick: ƒ}
console.log(B.prototype.__proto__); // {constructor: ƒ, handleLongClick: ƒ}
new B().handleClick(); // Uncaught TypeError: (intermediate value).handleClick is not a function
new B().handleLongClick(); // A.handleLongClick + B.handleLongClick
```

```js
class C extends A {
  handleClick() {
    super.handleClick();

    console.log('C.handleClick');
  }
}

console.log(C.prototype); // A {constructor: ƒ, handleClick: ƒ}
console.log(C.prototype.__proto__); // {constructor: ƒ, handleLongClick: ƒ}
new C().handleClick(); // A.handleClick
```

이는 상속받은 `B`, `C`가 인스턴스화될 때 내부적으로 `A`의 생성자를 호출하는데, 이때 `A`의 생성자 내부의 `this`는 상속받은 클래스를 가리키기 때문이다.

2. 모킹이 힘들다.

```js
class A {
  handleClick = () => {
    console.log('A.handleClick');
  };

  handleLongClick() {
    console.log('A.handleLongClick');
  }
}

console.log(A.prototype.handleClick); // undefined
console.log(A.prototype.handleLongClick); // [Function: handleLongClick]
```

`prototype`으로 직접 접근하면 `handleClick` 메서드는 정의되지 않음을 알 수 있다.  
`handleClick` 메서드는 인스턴스화할 때, 즉 `new` 키워드로 클래스가 생성됐을 경우에만 **인스턴스 내부에서** 초기화되기 때문에 다른 객체들은 prototype chaining에 의한 영향을 받지 않는다.  
이로 인해 모킹이 까다로워진다.

3. 성능이 느리다?!

`prototype` 내의 함수를 호출하면 JS 엔진은 이를 최적화할 수 있다.  
하지만 여러 인스턴스 내에서 각각 함수를 호출하면 JS 엔진은 이를 서로 다른 함수라고 생각해 최적화할 수 없다.  
이에 대해 [블로그](https://www.charpeni.com/blog/arrow-functions-in-class-properties-might-not-be-as-great-as-we-think)에서 실험한 결과가 있는데, 사진을 첨부해보겠다.

![class method performance1](https://user-images.githubusercontent.com/63287638/229267609-888b25ad-e495-4599-a4e8-b6a9a6e667ef.jpg)

![class method performance2](https://user-images.githubusercontent.com/63287638/229267610-106d970d-4a63-4d68-a6d4-8f40ecbded0f.jpg)

arrow function으로 method를 선언한 클래스들이 훨씬 느린 것을 알 수 있다.

##### React class component에서 arrow function 성능은 어떠한가

내가 직접 benchmark로 돌려봤을 때는 오히려 arrow function으로 method를 선언한 클래스가 훨씬 빨랐다. ~띠용?!~  
그래서 여러 가지 검색을 해봤지만 React는 0.x.x 버전 이후로 function method를 auto binding(normal function으로 선언해도 arrow function처럼 동작하도록 `constructor`에서 자동 binding해주는 기능) 하는 기능을 지원하지 않으며, arrow function을 선언함으로써 성능에 문제가 생겼다는 글은 존재하지 않았다.  
React가 최적화에 신경을 쓴 것도 있겠지만 다음 내용도 어느 정도 신빙성이 있었다.

> Arrow functions can sometimes be faster than normal functions when used in callback functions, since they are more concise and do not create a new `this` context. It's worth noting that these performance issues(arrow function method vs normal function method) are specific to this particular use case(위 블로그 실험 결과) of arrow functions and do not necessarily apply to arrow functions used in other contexts. In general, the performance differences between arrow functions and normal functions are usually negligible, as I mentioned in my previous answer.

위 블로그에서 실행한 내용는 컴포넌트를 과도하게 많이 선언하고 실행한 결과이다.  
즉, 실제 프로젝트에서 arrow function 선언으로 인해 렌더링이 느려질 걱정은 안 해도 될 것 같고, 만약 렌더링이 느려진다면 다른 원인을 분석하는 게 맞을 것 같다.

**결론**
React로 웹사이트를 만들 때 굳이 컴포넌트 간 상속을 만들 구조가 많지 않다.  
또한 메서드 하나하나에 대해 모킹할 일도 적다.  
테스트를 한다면 차라리 렌더링이 잘 되는지, API call이 되는지, 버튼이 disabled 됐는지 등을 테스트하기 때문이다.  
마지막으로 성능에 대해서 걱정할 정도는 아니다.  
따라서 나의 결론은 _arrow function이든 normal function이든 편한 거 쓰자!_ 이다.
