## Event Loop

참고
https://medium.com/sjk5766/javascript-%EB%B9%84%EB%8F%99%EA%B8%B0-%ED%95%B5%EC%8B%AC-event-loop-%EC%A0%95%EB%A6%AC-422eb29231a8  
https://velog.io/@thms200/Event-Loop-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EB%A3%A8%ED%94%84

JS는 싱글 스레드 기반이며 비동기적으로 동작한다.  
싱글 스레드이기 때문에 한 순간에 하나의 작업만 가능하지만, 비동기로 동작하기 때문에 단일 스레드임에도 불구하고 짧은 시간 동안 많은 작업을 수행할 수 있다.

다만, JS 자체가 비동기 동작을 지원하는 것이 아닌, **브라우저** 가 가진 요소(node에서는 libuv 라이브러리)를 이용해서 비동기 동작을 처리한다.  
이 부분이 Event Loop이다.

### 사전 지식

JS의 내부 구조에 대한 설명을 하기에 앞서 이후에 계속 나올 Execution Context와 Call Stack을 먼저 정리하겠다.

##### Execution Context

**코드를 실행하는데 필요한 환경을 제공하는 객체. 변수 식별자(record)와 외부 scope와의 연결(outer) 등을 포함**

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
  console.log('first');
  second();
}

function second() {
  console.log('second');
}

first();
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
console.log('1');

setTimeout(() => {
  console.log('3');
}, 3000);

console.log('2');
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
  let start = Date.now();
  let now = start;

  while (now - start < 1000) {
    now = Date.now();
  }
};

console.log('1');

setTimeout(function () {
  console.log('2');
}, 0);

let promise = new Promise((resolve, reject) => resolve());

promise.then((resolve) => console.log('3')).then((resolve) => console.log('4'));

waitASecond();

console.log('5');
```

위의 내용들을 정리했을 때 코드는 `1 5 3 4 2` 순으로 출력된다.

### Animation Frames

requestAnimationFrame API가 실행되면 콜백이 Animation Frames으로 담긴다.  
크롬 기준으로 Microtask Queue > Animation Frames > Callback Queue(Task Queue) 순으로 실행된다.

~requestAnimationFrame API를 사용해본 것이 아니라 정확하지 않을 수 있다. 추후에 사용하는 일이 있다면 자세히 알아봐야겠다~
