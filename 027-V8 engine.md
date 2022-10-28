## 0. 공부하게 된 계기

Node.js가 널리 사용될 수 있는 계기가 V8 Engine가 나오면서라고 하니 공부하지 않을 수 없는 부분인 것 같다.  
참고한 글들 [1](https://evan-moon.github.io/2019/06/28/v8-analysis), [2](https://segmentfault.com/a/1190000040331440/en)처럼 내부 구조를 `git clone`해서 파본 것은 아니어도 정리할 필요는 있다는 생각이 들었다.

## 1. 개념

~자동차 v8 엔진의 이름을 따온 것 같다. 자동차 v6 실린더 엔진 어쩌구하고 비교했을 때 v8 엔진이 좋다\~ 식의 글들을 접하지만 자동차를 잘 모르니 패스~

V8은 웹 브라우저를 만드는 데 기반을 제공하는 오픈 소스 자바스크립트 엔진이다. 구글 크롬 브라우저와 안드로이드 브라우저에 탑재되어 있다. 내부적으로는 ECMAScript(ECMA - 262) 3rd Edition 규격의 c++로 작성되어 있다.

V8은 자바스크립트를 기계어가 아닌, 바이트코드(bytecode)로 컴파일하고 실행하는 방식을 사용한다.

## 2. 동작 방식

![v8compiler-pipeline](https://user-images.githubusercontent.com/63287638/179386645-888b9b74-9e50-451d-ae7a-2929f9d25365.png)  
_출처: JSConf EU 2017에서 발표한 Franziska Hinkelmann님의 자료_

위 사진에서 보듯이 V8은 크게 Parser, Ignition, TurboFan과 위 사진에는 없지만 Orinoco라는 GC로 구성된다.

참고로 Ignition은 엔진에 시동걸 때 사용하는 점화기이다. Ignition을 통해 내 소스 코드가 실행된다. 이후 너무 많이 호출되면(내 코드가 뜨거워지면) TurboFan으로 최적화해서 너무 과열되지 않게 식혀주는 역할을 하는 것으로 자동차 엔진의 네이밍을 따온 것이라고 한다.

### Parser

Parser는 소스 코드를 추상 트리 구문(AST)으로 변환시키는 역할을 한다.  
이 과정에서는 크게 두 가지, Lexical Analysis와 Syntax Analysis가 이루어진다.  
이 중 Lexical Analysis는 `Also called word segmentation, it is the process of converting a code in the form of a string into a sequence of tokens` 라는 과정이다.

JS에는 다음과 같은 토큰들이 존재한다.

> Keywords: var, let, const, etc.
> Identifier: consecutive characters not enclosed in quotation marks, which may be a variable, keywords such as if and else, or built-in constants such as true and false
> Operators: +, -, \*, / etc.
> Numbers: like hexadecimal, decimal, octal and scientific expressions, etc.
> String: the value of a variable, etc.
> Spaces: consecutive spaces, \n, \t, etc.
> Comment: Line comment or block comment
> Punctuation: braces {}, parentheses (), semicolons ;, colons :, etc.

`const a = 'hello word'`라는 코드는 `['const', 'a', '=', 'hello world']` 와 같은 토큰으로 구분되며 다음과 같은 Lexical Analaysis 결과를 낸다.

```json
[
  {
    "type": "Keyword",
    "value": "const"
  },
  {
    "type": "Identifier",
    "value": "a"
  },
  {
    "type": "Punctuator",
    "value": "="
  },
  {
    "type": "String",
    "value": "'hello world'"
  }
]
```

AST를 만드는 내부 함수

```cpp
// v8/src/ast/ast.cc
Literal* AstNodeFactory::NewNumberLiteral(double number, int pos) {
  int int_value;
  if (DoubleToSmiInteger(number, &int_value)) {
    return NewSmiLiteral(int_value, pos);
  }
  return new (zone_) Literal(number, pos);
}
```

를 거치면 다음과 같은 결과가 나온다고 한다.

```json
{
  "type": "Program",
  "body": [
    {
      "type": "VariableDeclaration",
      "declarations": [
        {
          "type": "VariableDeclarator",
          "id": {
            "type": "Identifier",
            "name": "a"
          },
          "init": {
            "type": "Literal",
            "value": "hello world",
            "raw": "'hello world'"
          }
        }
      ],
      "kind": "const"
    }
  ],
  "sourceType": "script"
}
```

### Ignition

Ignition은 인터프리터로 AST를 **바이트 코드** 로 변환시키고 실행시키는 역할을 한다.  
일반적으로 훨씬 빠르다고 생각되는 '기계어'가 아닌 '바이트 코드'이다.

~이 부분은 컴파일러를 만들어 보지 않는 이상 이해하기가 힘드니 그런갑다 하고 넘어가도 될 것 같다~  
JS는 동적 타이핑 언어이기 때문에 소스 코드를 실행하기 전에 알 수 없는 값들이 너무 많다.  
그래서 Ignition이 모든 소스 코드를 한 번에 해석하는 컴파일 방식이 아닌 한 줄씩 실행하는 인터프리터 방식을 사용함으로써 다음 세 가지 이점을 얻었다고 한다.

> 1. 메모리 사용량 감소. 자바스크립트 코드에서 기계어로 컴파일하는 것보다 바이트 코드로 컴파일하는 것이 더 편하다.
> 2. 파싱 시 오버헤드 감소. 바이트 코드는 간결하기 때문에 다시 파싱하기도 편하다.
> 3. 컴파일 파이프 라인의 복잡성 감소. Optimizing이든 Deoptimizing이든 바이트 코드 하나만 생각하면 되기 때문에 편하다.

Ignition은 다음과 같은 최적화 기법을 사용한다고 한다.

- Register Optimizer : Mainly to avoid unnecessary loading and storage of registers
- Peephole Optimizer : Find the reusable part of the bytecode and merge it
- Dead-code Elimination : Delete useless code and reduce the size of bytecode

V8 engine은 이렇게 바이트 코드로 전부 변환해놓기 때문에 처음 실행될 때는 시간이 걸리겠지만 그 이후부터는 거의 컴파일 언어에 가까운 성능을 보일 수 있다고 한다.

+) Ignition은 한 가지 업무를 더 한다. Ignition은 코드 실행을 지켜보고 실행 중에 함수가 얼마나 자주 불리는지, parameter들이 각 실행마다 얼마나 전달되는지 등을 기록하는데 이와 같은 업무는 다음에 나올 [TurboFan](https://github.com/mochang2/development-diary/blob/main/027-V8%20engine.md#TurboFan)과 관련이 있다.

### TurboFan

Ignition이 먼저 최적화한 뒤, TurboFan은 최적화된 바이트 코드를 보다 효율적인 기계 코드로 컴파일하여 저장한다.  
다음에 동일한 코드가 다시 실행될 때 해당 기계 코드가 직접 실행되어 코드의 실행 효율성이 크게 향상된다.  
특정 코드가 더 이상 핫 코드(자주 사용되는 코드)가 아닌 경우 TurboFan은 최적화 해제 프로세스를 수행하여 컴파일된 기계 코드를 바이트 코드로 복원하고 코드의 실행 권한을 Ignition으로 반환한다.

최적화하는 조건의 함수는 다음과 같이 생겼다고 한다.

```cpp
// v8/src/execution/rumtime-profiler.cc

// kHotAndStable: 자주 사용된다고 판단
// kSmallFunction: 코드 길이가 짧은 함수로 판단
// kDoNotOptimize: 최적화하지 않음

OptimizationReason RuntimeProfiler::ShouldOptimize(JSFunction function, BytecodeArray bytecode) {
  int ticks = function.feedback_vector().profiler_ticks(); // 함수 호출 횟수
  int ticks_for_optimization =
      kProfilerTicksBeforeOptimization +
      (bytecode.length() / kBytecodeSizeAllowancePerTick);
  if (ticks >= ticks_for_optimization) { // 호출 횟수가 특정 임계치를 넘으면
    return OptimizationReason::kHotAndStable;
  } else if (!any_ic_changed_ && bytecode.length() < kMaxBytecodeSizeForEarlyOpt) {
    return OptimizationReason::kSmallFunction;
  }
  // 해당 사항 없다면 최적화 x
  return OptimizationReason::kDoNotOptimize;
}
```

## 3. Node.js의 이벤트 루프

참고 및 사진 출처: https://www.korecmblog.com/node-js-event-loop/#nodejs%EC%9D%98-%EA%B5%AC%EC%A1%B0 , https://evan-moon.github.io/2019/08/01/nodejs-event-loop-workflow/

Node.js는 JS와 마찬가지로 싱글 스레드 논 블로킹이다.  
Node.js는 하나의 스레드로 동작하지만 I/O 작업이 발생하는 경우 이를 비동기적으로 처리하는 로직을 가지고 있어 블로킹 없이 수행할 수 있으며 이를 도와주는 것이 이벤트 루프이다.  
다만 좀 헷갈릴 수 있는 점은 Node.js는 싱글 스레드이지만 작업을 처리하는 별도의 스레드풀을 가지고 있다(무슨 말인지는 아래를 보면 이해할 수 있다).

### Node.js 이벤트 루프에 관한 궁금한 점 요약

**1. 이벤트 루프는 JS 엔진 내부에 있지 않다.** 이벤트 루프는 단지 JS 코드를 실행하기 위해 JS 엔진을 이용하는 것이다. 실제로 V8 엔진에는 이벤트 루프를 관리하는 코드가 없다.  
**2. 이벤트 루프에는 스택은 존재하지 않고 여러 개의 큐만 존재한다.**  
**3. 이벤트 루프는 단 하나의 스레드로 실행된다.**  
**4. setTimeout을 0으로 지정해도 최소 1ms 이상은 sleep하게 된다.**
**5. Node.js는 I/O 작업을 자신의 메인 스레드가 아닌 다른 스레드에게 위임함으로써 논 블로킹 I/O를 지원한다.**

### Node.js의 내부 구조

![노드의 내부 구조](https://user-images.githubusercontent.com/63287638/179388041-6c07a1a1-1249-4d60-a199-f2bc59a77bf1.png)

위 구조에서 핵심은 libuv와 비동기 I/O이다.  
libuv는 c++로 작성된 비동기 I/O 라이브러리로, 운영체제의 커널을 추상화한 라이브러리이다.  
만약 소스 코드에서 즉, Node.js에서 비동기 작업을 요청하면 libuv는 이 작업을 커널이 지원하는지 확인한다.  
만약 지원한다면 Node.js 대신 커널에게 비동기적으로 요청했다가 응답이 오면 그 응답을 Node.js에게 전달한다.  
만약 요청한 작업이 커널에서 지원하지 않는다면 워커 스레드가 담긴 풀에 이 요청을 처리할 것을 명령한 뒤, 응답을 전달받는다.  
실제로 node를 실행한 뒤 `ps` 등의 명령어로 확인해보면 여러 개의 node 스레드가 존재하는 것을 확인할 수 있다.  
스레드 풀에는 기본적으로 4개의 스레드가 있는데 최대 128개까지 늘릴 수 있다.  
아래 사진들은 위 동작 과정을 표현한 것이다.

![libuv-커널](https://user-images.githubusercontent.com/63287638/179858361-0d712180-c063-41f4-8b27-636017c861db.png)  
![libuv-워커-스레드-1](https://user-images.githubusercontent.com/63287638/179858479-ef6b75b2-91d9-4ba4-90eb-8f5ec7d8686f.png)

### 이벤트 루프 내부 구조

Node.js로 어떠한 JS 파일을 실행할 때의 과정은 다음과 같다.

1. Node.js는 우선 이벤트 루프를 만든다.
2. 이벤트 루프 바깥에서 해당 JS 파일을 처음부터 끝까지 실행한다.
3. 실행이 끝난 뒤에 (I/O 처리에 대한 콜백이 있는)이벤트 루프를 확인하여 이벤트 루프의 작업을 처리한다.
4. 처리할 작업이 없거나 처리가 전부 끝나면 `process.on('exit', callback)`을 실행하고 이벤트 루프를 종료한다.

이벤트 루프는 한 마디로 Node.js가 여러 비동기 작업을 관리하기 위한 구현체이다.  
그리고 내가 이해한 바로는 이 이벤트 루프는 Node.js에 단 하나 존재한다(스레드 풀에 각각 존재하는 게 아니다).  
구성은 아래 사진과 같다.

![event-loop](https://user-images.githubusercontent.com/63287638/179859058-5ba3af53-d8ff-4067-aac3-ae64b24ef154.png)

위 사진에서 네모박스는 특정 작업을 수행하기 위한 Phase를 의미한다.  
각각의 Phase는 해당 절차에서 처리할 작업들을 큐에 넣어서 보관한다.  
이 큐에는 이벤트 루프가 실행해야 하는 작업들이 순서대로 담겨있으며 만약 큐에 있는 작업들을 다 실행하거나 '시스템의 실행 한도에 다다르면' 다음 Phase로 넘어간다.  
`Timer Phase` -> `Pending Callbacks Phase` -> `Idle, Prepare Phase` -> `Poll Phase` -> `Check Phase` -> `Close Callbacks Phase` 이러한 순서를 거치며 한 Phase를 넘어가는 것을 Tick이라고 표현한다.  
점선으로 된 `nextTickQueue`나 `microTaskQueue`는 정확히 말하자면 이벤트 루프에 속한 큐는 아니며 [아래](https://github.com/mochang2/development-diary/edit/main/027-V8%20engine.md#nexttickqueue-micortaskqueue)에서 더 설명하겠다.

참고로 '시스템의 실행 한도에 다다르면' 이라는 표현을 쓴 이유는 스레드가 가리키고 있는 어떤 Phase에서 작업을 하고 있을 때 해당 작업을 완료하기 전에 다른 작업이 계속 들어올 경우 다른 Phase로 넘어가지 못하는 상황을 막기 위해 존재하는 limit 기능이다.

#### Timer Phase

타이머 콜백들과 관련이 있는 Phase이다.  
`Timer Phase`는 `setTimeout`이 호출되었을 때 타이머의 콜백을 큐에 바로 저장하는 것이 아니라 콜백을 언제 실행할 지에 관한 정보(타이머)를 min heap에 넣는다.  
(`Poll Phase` 단계는 아직 이야기하지 않았지만) 만약 `Poll Phase`에서 `setTimeout`을 3번 호출하면 `Timer Phase`에는 3개의 타이머가 저장된다.  
min heap 자료구조로 이루어져 있어서, 타이머들을 하나씩 `now - registeredTime >= delta`와 같은 방식으로 가장 빨리 실행해야 되는 타이머가 지금 실행되어야 하는 것이 맞는지를 확인한 뒤 실행한다(참고 자료는 `===`로 표현했었는데 실제로는 지정한 시간보다 더 늦게 실행될 수 있으며로 `>=`가 맞는 것 같다. 그리고 여기서 delta는 `setTimeout`에서 두 번째 인자로 주는 sleep하는 ms 단위를 이야기한다).

#### Pending Callbacks Phase

pending queue에 담기는 콜백들을 관리하는 Phase이다.  
이 콜백들은 이전 이벤트 루프에서 실행되지 않은 I/O 콜백들이다.  
위에서 얘기한 것처럼 '시스템의 실행 한도'에 다다른 경우 처리하지 못하고 넘어간 작업들을 쌓아놓은 페이즈이다.  
참고로 에러 핸들러 콜백 또한 pending queue에 들어온다.

#### Idle, Prepare Phase

이 Phase은 Node.js의 내부적인 관리를 위한 페이즈로 자바스크립트를 실행하지 않는다.  
공식 문서에서도 별다른 설명이 없고 코드의 직접적인 실행에 영향을 미치지 않는다.

#### Poll Phase

watcher_queue 내부에 파일 읽기의 응답 콜백, HTTP 응답 콜백 등과 같이 수행해야 할 작업들이 있다면 이 작업들을 실행한다.  
다만 여기서는 `Timer Phase`와 달리 큐에 담긴 순서대로 I/O 작업이 완료되어 콜백 또한 차례대로 실행된다는 보장이 없기 때문에 운영 체제가 FD(File Descriptor, 네트워크 소켓 등을 말함)가 준비되었다고 알리면 이벤트 루프는 이에 해당하는 watcher를 찾아 watcher가 맡고 있는 콜백을 실행한다.

만약 더 실행할 콜백들이 없다면 check_queue, pending_queue, closing_callbacks_queue를 검사한다.  
진행해야 할 작업이 있다면 `Poll Phase`는 종료되고 다음 Phase로 넘어가지만 진행해야 할 작업이 없다면 `Check Phase`와 `Timer Phase`를 검사한 뒤 실행할 수 있는 타이밍이 되면(`Timer Phase`와 같은 경우 간다고 바로 실행할 수 없을 수도 있으므로) 다음 Phase로 넘어간다.

#### Check Phase

오직 `setImmediate`의 콜백만을 위한 Phase이다.  
`setImmediate`가 호출되면 가지고 있는 콜백이 `Check Phase`의 큐에 담긴다.

`process.nextTick`과 비교할 때 이름에서 헷갈리는 사항이 생길 수 있다.  
[공식 문서](https://nodejs.org/ko/docs/guides/event-loop-timers-and-nexttick/#process-nexttick-setimmediate)에서도 인정한 내용인데 `process.nextTick`은 _즉시_ 실행되고 `setImmediate`는 _다음 tick_ 에 실행된다.

#### Close Callbacks Phase

이벤트 루프가 Close callback들과 함께 종료되면 이벤트 루프는 다음에 돌아야할 루프가 있는지 다시 체크 하게 된다.  
만약 아니라면 그대로 이벤트 루프는 종료된다.  
하지만 만약 더 수행해야할 작업들이 남아 있다면 이벤트 루프는 다음 순회를 돌기 시작하고 다시 `Timer Phase`부터 시작한다.

#### NextTickQueue, microTaskQueue

아래 설명하는 방식은 Node.js v11 이후의 동작 방식이다.

이 두 개의 큐는 이벤트 루프의 일부가 아니라 libuv에 구현되어 있지 않고 Node.js에 구현되어 있다.  
또한 이 큐들은 '시스템의 실행 한도'에 영향받지 않고 큐가 비워질 때까지 콜백들을 실행한다(따라서 이를 이용한 재귀 함수는 무한 반복되지 않도록 특히 조심해야 한다).  
`nextTickQueue`는 `process.nextTick()`의 콜백을 관리하며 `microTaskQueue`는 `Promise.resolve`된 콜백을 관리한다.  
JS처럼 micro라는 이름이 붙은 `microTaskQueue`가 우선순위를 가질 것 같지만, `nextTickQueue`가 `microTaskQueue`보다 높은 우선순위를 가지고 있다.

`nextTickQueue`와 `microTaskQueue`는 현재 실행하고 있는 작업이 끝나면 즉시 실행된다.  
아래 예제 코드를 보면 와닿을 것이다.

```javascript
setTimeout(() => {
  console.log(1)
  process.nextTick(() => {
    console.log(2)
  })
  Promise.resolve().then(() => {
    console.log(3)
  })
}, 0)

setTimeout(() => {
  console.log(4)
}, 0)

// 실행순서 1, 2, 3, 4
```

1. Node.js가 Timer Phase에 진입
2. 우선 Timer Phase에 있는 큐를 확인하고 console.log(1) 실행
3. process.nextTick과 Promise.resolve를 호출해 nextTickQueue와 microTaskQueue에 콜백을 등록
4. 현재 실행하고 있는 작업이 끝났으므로 Node.js는 nextTickQueue와 microTaskQueue에 작업이 있음을 확인
5. Timer Phase의 큐를 확인하지 않고 우선순위가 높은 nextTickQueue 부터 확인
6. console.log(2) 출력
7. Node.js는 nextTickQueue가 비었음을 확인하고 우선순위가 낮은 microTaskQueue 확인
8. console.log(3) 출력
9. microTaskQueue가 비었음을 확인하고 다시 Node.js는 Timer Phase에 있는 큐를 확인하고 console.log(4) 실핼
10. 현재 실행하고 있는 작업이 끝났으므로 Node.js는 nextTickQueue와 microTaskQueue에 작업이 있음을 확인
11. Timer Phase의 큐가 비었음을 확인하고 Pending Callbacks Phase로 이동

### 코드와 함께 정리

> 1. 실행 결과를 예측해보자

```javascript
setTimeout(() => {
  console.log('timeout')
}, 0)

setImmediate(() => {
  console.log('immediate')
})
```

위 결과는 예측할 수 없다.  
왜냐면 이벤트 루프가 `Timer Phase`에 진입할 때 타이머를 찾을 수도 있고, 못 찾을 수도 있기 때문이다.  
이는 컴퓨터의 성능이나 외부 작업에 의한 딜레이에 영향을 받을 수 있다.  
반면 아래와 같이 쓴다면 어떻게 될까?

```javascript
fs.readFile('test.txt', () => {
  setTimeout(() => {
    console.log('timeout')
  }, 0)

  setImmediate(() => {
    console.log('immediate')
  })
})
```

위 결과는 반드시 `immediate`가 먼저 출력된다.  
파일 읽기는 OS 커널에서 비동기 API를 제공하지 않으므로 스레드 풀에 작업을 이양한다.  
작업이 완료되면 이벤트 루프는 `Pending Callbacks Phase`의 큐에 작업의 콜백을 등록한다(내가 볼 때는 `Polling Phase`가 맞는 것 같지만 참고 자료에 따르면...).  
이벤트 루프가 `Pending Callbacks Phase`를 지날 때 해당 콜백을 실행하는데, `setTimeout`은 `Timer Phase`의 큐에 등록되며, `setImmediate`는 `Check Phase`의 큐에 등록된다.  
따라서 이벤트 루프의 순서에 따라 `Check Phase`에 있는 `setImmediate`가 먼저 실행된다.

> 2. 실행 시간을 비교해보자

```javascript
// 1)
var i = 0
var start = new Date()
function foo() {
  i++
  if (i < 1000) {
    setImmediate(foo)
  } else {
    var end = new Date()
    console.log('Execution time: ', end - start)
  }
}

foo()

// 2)
var i = 0
var start = new Date()
function foo() {
  i++
  if (i < 1000) {
    setTimeout(foo, 0)
  } else {
    var end = new Date()
    console.log('Execution time: ', end - start)
  }
}

foo()
```

`setImmediate`로 재귀 함수를 실행했느냐, `setTimeout`으로 재귀 함수를 실행했느냐의 차이이다.  
실행 결과에 따르면 후자의 시간이 압도적으로 느리다.  
`setTimeout`에 0ms를 주더라도 이는 반드시 0ms 후에 작동한다는 것이 아닌 폴링에 걸리는 시간이 0이라는 것이다.  
또한 시간을 비교하고 편차를 알아내는 작업이 CPU를 더 소모하기 때문에 더 느릴 수밖에 없다.

> 3. 실행 결과를 예측해보자

```javascript
var i = 0
function foo() {
  i++
  if (i > 20) {
    return
  }
  console.log('foo')
  setTimeout(() => {
    console.log('setTimeout')
  }, 0)
  process.nextTick(foo)
}

setTimeout(foo, 2)
```

`nextTickQueue`는 매 Tick마다 실행되는 것이 아닌! 실행할 작업이 없다면 바로 실행된다고 앞서 이야기했다.  
따라서 재귀 호출로 `nextTickQueue`에 들어간 모든 콜백들을 실행하고 나서야 `Timer Phase`의 콜백을 처리할 수 있기 때문에 결과는 `foo` \* 20번 뒤에 `setTimeout` \* 20번이 출력된다.

## 4. 메모리 구조

V8 Engine의 메모리 구조는 아래 사진과 같다.

![v8 memory structure](https://user-images.githubusercontent.com/63287638/180597792-1d9feead-6d75-4b9a-bd9b-3dff72336404.png)
_출처: https://deepu.tech/memory-management-in-v8/_

### 힙

다음과 같은 구성 요소를 가지고 있다.

- **New Space**: 대부분의 새 object들이 존재. Scavenger라는 minor GC가 관리하는 두 개의 작은 semi space(from-space와 to-space)가 존재. 새 object에 대한 메모리를 할당하고자 할 때 메모리가 부족하면, fragmentation을 없앰으로써 새 object를 compact하고 clean하게 관리하기 위해 사용됨.
- **Old Space**: new Space에서 오래 남아있는 object들이 옮겨짐. Mark and Sweep 알고리즘을 사용하는 major GC에 의해 관리됨. 다른 object에 대한 포인터를 가지고 있는 Old pointer space와 데이터만 가지고 있는 Old data space로 이루어짐.
- **Large object space**: 크기가 큰 object가 저장됨.
- **Code space**: JIT(Just In Time) compiler가 compile된 code block을 저장. V8 Engine 중에서 유일하게 실행 가능한 메모리.

### 스택

두 가지 종류의 데이터가 저장된다.

- **primitive type의 value**
  - number, string, null, undefined, symbol, boolean, bigint 7가지 종류가 있음.
  - primitive type에 대해서 재할당(`let`, `var`으로 선언된 변수)되면 주소가 가리키고 있는 값이 바뀌는 것이 아니라 가리키고 있는 주소 자체가 바뀌고, 이전에 주소는 참조되지 않으면 GC에 의해 메모리에서 사라짐.
  - 변수에는 값이 저장된 콜 스택 메모리의 주소값이 저장됨.
  - 변수 식별자 자체는 콜스택 상의 '실행 컨텍스트(Execution Context)의 렉시컬 환경(Lexical Environment)'이라는 곳에 저장됨.
- **객체 주소**
  - 힙에 저장되는 object(함수 포함)들에 대한 참조 값.

아래 사진처럼 표현할 수 있다.

![js stack heap](https://user-images.githubusercontent.com/63287638/180597950-4aba0533-c211-4de8-8c25-8cd9df04ff38.png)
_출처: https://charming-kyu.tistory.com/19_
