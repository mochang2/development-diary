## 0. 공부하게 된 계기
면접으로 자주 나온다는 js에 관한 질문들을 단순히 답만 외우는 게 아니라 자세히 정리하고 싶어졌다.

## 1. GC(garbage collection)
_출처: https://blog.sessionstack.com/how-javascript-works-memory-management-how-to-handle-4-common-memory-leaks-3f28b94cfbec , https://developer.mozilla.org/ko/docs/Web/JavaScript/Memory_Management _  


<img src="https://user-images.githubusercontent.com/63287638/178136632-c33e7ae6-c245-4cd9-a1c9-63a6c434d678.png" alt="memory lifecycle" width="50%" />

위 사진에서 볼 수 있듯이 메모리는 크게 3가지의 life cycle을 가지고 있다.  
할당한 뒤에 사용되고, 사용되지 않으면 해제된다.  
C와 같은 저레벨 언어에서는 allocate와 release를 malloc(calloc 등)과 free를 통해서 개발자가 직접 명령해야 하지만 JS, JAVA, Python 등은 별도의 Garbage Collector가 동작해서 메모리 관리를 해준다.  

```javascript
var n = 374 // allocates memory for a number

var s = 'sessionstack' // allocates memory for a string

var o = { // allocates memory for an object and its contained values
  a: 1,
  b: null
}

var a = [1, null, 'str']  // (like object) allocates memory for the array

function f(a) { // allocates memory for a function
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
  var o1 = {};
  var o2 = {};
  o1.p = o2; // o1 references o2
  o2.p = o1; // o2 references o1. This creates a cycle.
}
```

### GC 전략 2: mark and sweep algorithm
사용되는 메모리는 __mark__ 한 뒤에 mark되지 않은 메모리들을 __sweep__ 하는 알고리즘이다.  
GC의 root(웹은 window, node는 global)를 지정한 뒤 해당 루트를 타고, 타고 내려갔을 때 참조되지 않는 메모리를 release하는 방법이다.  
하지만 이 방법에도 문제점은 존재했다.  
'언제' Garbage Collector가 동작해야 하는지 모른다는 것이다.  
일반적으로 대부분의 Garbage Collector는 allocate한 뒤에 동작하는데, 만약 한번에 엄청 많은 allocation이 일어난 직후 더이상의 allocation이 일어나지 않는다면 메모리가 정리되지 않게 된다.  
하지만 이는 극단적인 경우이므로 일반적으로 자주 발생하지는 않는다.  

이를 해결하는 방법이 수동 메모리 release인데, 위에서 얘기했듯이 JS에서는 제공하지 않는다.  
Node에서는 디버깅을 위한 추가 옵션과 도구를 제공한다. [참고](https://developer.mozilla.org/ko/docs/Web/JavaScript/Memory_Management#node.js)  

##### GC 전략 3: mark and sweep algorithm 최적화 기법

* generational collection(세대별 수집) – 객체를 '새로운 객체’와 '오래된 객체’로 나눈다. 객체 상당수는 생성 이후 제 역할을 빠르게 수행해 금방 쓸모가 없어지는데, 이런 객체를 '새로운 객체’로 구분한다. 가비지 컬렉터는 새로운 객체를 엄격한 기준에 따라 메모리에서 제거한다. 일정 시간 이상 동안 살아남은 객체는 '오래된 객체’로 분류하고, 가비지 컬렉터가 덜 감시한다.
* incremental collection(점진적 수집) – 방문해야 할 객체가 많다면 모든 객체를 한 번에 방문하고 mark 하는데 상당한 시간이 소모된다. 자바스크립트 엔진은 이런 현상을 개선하기 위해 가비지 컬렉션을 여러 부분으로 분리한 다음, 각 부분을 별도로 수행한다. 작업을 분리하고, 변경 사항을 추적하는 데 추가 작업이 필요하긴 하지만, 긴 지연을 짧은 지연 여러 개로 분산시킬 수 있다는 장점이 있다.
* idle-time collection(유휴 시간 수집) – 가비지 컬렉터가 실행에 주는 영향을 최소화하기 위해 CPU가 유휴 상태일 때에만 가비지 컬렉션을 실행한다.

### memory leak 막는 방법

__global variable 사용 최소화__

```javascript
function f(a) {

}
```


## 2. prototype

implicit reference: prototype
explicit reference: a = { b: 1 }

## 3. closure


## 4. this
+ arrow function vs function

## 5. event bubbling

## 6. async vs defer
