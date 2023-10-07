## Closure

### Lexical Scope

클로저를 알기 전에 lexical scope가 무엇인지 먼저 알아야 한다.  
lexical scope란 함수와 변수의 scope를 선언된 위치를 기준으로 정한다는 의미이다.  
이와 반대로 dynamic scope는 함수나 변수의 scope를 호출된 시점을 기준으로 정한다는 의미이다.
아래 함수를 실행하면 `const num = 10`에서 선언한 `num`이 쓰이지 않는 것을 알 수 있다.

```javascript
const num = 1;

function a() {
  const num = 10;
  b();
}

function b() {
  console.log(num);
}

a(); // 1
b(); // 1
```

이제 클로저를 알아보자.
흔히 함수 내에서 함수를 정의하고 사용하면 클로저라고 말하지만, 정확히 말하자면 내부에 선언된 함수가 외부 함수의 지역변수를 사용했을 때만 클로저라고 선언된다.  
MDN에 따르면 `클로저는 주변의 상태의 참조와 함께 번들로 묶인 함수의 조합입니다. 즉, 클로져는 우리에게 inner함수에서 outer함수의 스코프에 접근을 가능하게 해줍니다. 자바스크립트에서 클로저는 함수가 생성될 때마다 생성됩니다.`  
일반적으로 정의한 함수를 리턴하고 사용은 바깥에서 함으로써 클로저를 사용한다.

아래 예제 코드들을 통해서 클로저란 무엇인가를 알 수 있을 것이다.

```javascript
function func() {
  const test = '123';
  console.log(test);
}

func(); // 123
console.log(test); // error
```

위 예시처럼 함수 밖에서 name을 호출하는 것은 당연히 에러를 발생시킬 것이다.  
하지만 아래와 같이 사용하면 `outer` 내부에서 선언한 test에 접근할 수 있다.  
위에서 이야기한대로 `inner`는 해당 함수가 선언된 상태를 기억하고 있기 때문이다.

```javascript
function outer() {
  const test = '123';
  console.log(test);

  return function inner() {
    const variable = '456!';
    console.log(test + variable);
  };
}

const getTest = outer(); // 123
getTest(); // 123456!
```

클로저를 활용하는 유명한 예제가 있다.

```javascript
var i;
for (i = 0; i < 10; i++) {
  setTimeout(function () {
    console.log(i); // 10 열 번 출력
  }, 100);
}

// IIFE 활용
var i;
for (i = 0; i < 10; i++) {
  (function (j) {
    setTimeout(function () {
      console.log(j); // 1 ~ 10 출력
    }, 100);
  })(i);
}
```

위에서 '정확히 말하자면 내부에 선언된 함수가 외부 함수의 지역변수를 사용했을 때만 클로저라고 선언된다'고 말했다.  
아래 코드를 보면 해당 내용의 의미를 알 수 있다.

```javascript
function outer() {
  const test = '123';

  if (true) {
    const variable = '456!';

    return function inner() {
      console.log(variable);
    };
  }
}

const aa = outer();
console.dir(aa); // [[Scopes]] 내부에 closure가 없음
```

```javascript
function outer() {
  const test = '123';

  if (true) {
    const variable = '456!';
    return function inner() {
      console.log(test + variable);
    };
  }
}

const aa = outer();
console.dir(aa); // 아래와 같이 [[Scopes]] 내부에 closure 존재
```

<img alt="closure" src="https://user-images.githubusercontent.com/63287638/178147027-061f6477-6aa4-4d32-9284-e2346f909f20.png" width="350" />

### 클로저 장점 1. 은닉화

ES6에서 class 문법이 생기면서 private을 사용할 수 있지만, 그 전에는 변수 앞에 `_`를 붙여서 private property라는 것을 표현했다.

```javascript
function hello(name) {
  this._name = name;
}

Hello.prototype.say = function () {
  console.log('Hello, ' + this._name);
};

const hello1 = new hello('123');
const hello2 = new hello('456');
const hello3 = new hello('789');

hello1.say(); // 'Hello, 123'
hello2.say(); // 'Hello, 456'
hello3.say(); // 'Hello, 789'
hello1._name = 'sudo';
hello1.say(); // 'Hello, sudo'
```

위 코드는 Hello의 private 변수인 \_name(JS 네이밍 컨벤션에 따르면 private variable)에 외부에서 마음껏 접근할 수 있다.  
하지만 아래와 같이 코딩하면 불가능해진다.

```javascript
function hello(name) {
  const _name = name;

  return function () {
    console.log('Hello, ' + _name);
  };

  /* 또는
  return function() {
    console.log('Hello, ' + name)
  }
  */
}

const hello1 = new hello('123');
const hello2 = new hello('456');
const hello3 = new hello('789');
```

### 클로저 장점 2. 전역 변수의 사용을 줄임

count와 같이 많이 사용할 수 있는 변수의 이름을 사용해서 전역 오염시키지 않을 수 있다.

```javascript
let count = 0;
function handleCilck() {
  count++;
  return count;
}

// 대신에 아래처럼

function handleCilck() {
  let count = 0;
  return function () {
    count++;
    return count;
  };
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
  const _name = name;

  return function () {
    console.log('Hello, ' + _name);
  };
}

const hello1 = new hello('123');

hello1 = null; // release
```
