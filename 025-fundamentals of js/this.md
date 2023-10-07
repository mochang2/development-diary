## this

JS는 함수가 어떻게 불리느냐에 따라서 this에 대한 서로 다른 4가지 용법을 가지고 있다.

### 일반 함수

웹에서는 window, node에서는 global과 같다.

```javascript
function func() {
  return this === global;
}

console.log(func()); // true
```

### 객체의 값으로서의 함수

객체 그 자체를 가리킨다.

```javascript
const obj = {
  value: 123,
  key: function func() {
    return this.value;
  },
};

console.log(obj.key); // [Function: func]
console.log(obj.key()); // 123
```

### new로 생성된 함수

빈 객체를 반환하다.  
생성자 함수는 특이하게도 return문이 있음에도 불구하고 그 return문을 무시하고 this 객체를 return하는 특징이 있다.

```javascript
function func() {
  console.log(this);
}
new func(); // func {}

function func2() {
  console.log(this.key);
  this.key = 'value';
  console.log(this.key);
}
new func2();
// undefined
// value

function func3() {
  this.age = 100;
  return 3;
}
const a = new func3();
console.log(a); // func3 { age: 100 }
```

### call, bind, apply

해당 함수의 첫 번째 인자로 전달되는 객체를 가리킨다.

```javascript
const age = 100;

function foo() {
  console.log(this.age);
}

var ken = {
  age: 35,
  log: foo,
};

foo.call(ken, 1, 2, 3); // 35
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
  console.log('Inside func:', this.aa); // 13

  return {
    aa: 25,
    arrow: () => console.log('Inside aa:', this.aa), // 13
  };
}

func.call({ aa: 13 }).arrow();
```

```javascript
// function을 이용해서 prototype을 사용할 때
// this, arrow function 주의
function Parent() {
  this.name = 'parent';
}

Parent.prototype.getName = () => {
  return this.name; // uses 'this' that is available at the time the function is evaluated. return 'aa' 와 같은 this를 사용하지 않는 것이면 잘 작동.
};

function Child() {
  this.name = 'child';
}

Child.prototype = new Parent();

Child.prototype.getName = function () {
  return this.name;
};

const p = new Parent();
const c = new Child();

console.log(p.getName()); // undefined. this는 빈 객체
console.log(c.getName()); // 'child'
```

cf) 개인적인 생각이지만, React에서 컴포넌트를 선언할 때 일반 function보다는 arrow function을 사용하라고 한다. 아마 함수형 컴포넌트 이전에 클래스형 컴포넌트를 사용할 때 this를 많이 사용했는데, 이 this가 global 또는 window를 가리킬 수도 있어서 의도치 않게 동작할 수 있기 때문인 것 같다. 다만 클래스간 상속할 때는 [arrow function이 위험한 이유](https://simsimjae.tistory.com/452)를 참고해서 arrow function이 꼭 필요한지 한 번더 생각해보자.

```javascript
class Parent {
  getName = () => {
    console.log('parent');
  };
}

// 위와 같이 선언하면 아래 같이 바뀜
/*
class Parent {
  constructor() {
    this.getName = () => {
      console.log('parent')
    }
  }
}
*/

class Child extends Parent {
  getName() {
    console.log('child');
  }
}

new Child().getName(); // parent
```
