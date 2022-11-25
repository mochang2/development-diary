## 0. 공부하게 된 계기

structural typing을 하는 TS에 필수적인 개념이라고 생각한다.  
개념은 쉬워서 금방 아~ 하고 넘어갔지만 다시 설명하려고 하니까 못 하겠어서 글로 정리하기로 했다.

참고  
https://soopdop.github.io/2020/12/09/duck-typing/  
https://ko.wikipedia.org/wiki/%EB%8D%95_%ED%83%80%EC%9D%B4%ED%95%91  
https://basarat.gitbook.io/typescript/main-1/nominaltyping

## 1. Duck Typing

duck typing은 duck test에서 유래했다.

> 만약 어떤 새가 오리처럼 걷고, 헤엄치고, 꽥꽥거리는 소리를 낸다면 나는 그 새를 오리라고 부를 것이다. 오리인지 100% 확실하지는 않지만 이 정도의 추론 단서라면 오리라고 판단하기에 무리가 없다.

duck typing은 dynamic typing의 한 종류로, 객체의 변수 및 메소드의 집합이 객체의 타입을 결정하는 것을 말한다.  
클래스 상속이나 인터페이스 구현으로 타입을 구분하는 대신, duck typing은 객체가 어떤 타입에 걸맞은 변수와 메소드를 지니면 객체를 해당 타입에 속하는 것으로 간주한다.

### 수도 코드 예시

아래는 duck typing을 JS로 최대한 나타낸 본 것이다.

```text
function calculate(a, b, c) => return (a+b)*c
```

```text
a = calculate (1, 2, 3)
b = calculate ([1, 2, 3], [4, 5, 6], 2)
c = calculate ('apples ', 'and oranges, ', 3)
```

```text
print to_string a // 9
print to_string b // [1, 2, 3, 4, 5, 6, 1, 2, 3, 4, 5, 6]
print to_string c // apples and oranges, apples and oranges, apples and oranges,
```

### 프로그래밍 패턴으로서 Duck Typing

JS에서 `apperance`와 `quack`을 체크하여 duck인지 판단을 아래와 같이 할 수 있다.

```javascript
const duck = {
  appearance: 'feathers', // 깃털을 가졌다.
  quack: function duck_quack(what) {
    console.log(what + ' quack-quack!')
  }, // 꽥꽥거리는 기능을 가졌다.
  color: 'black', // 검은색이다.
}

const someAnimal = {
  appearance: 'feathers', // 깃털을 가졌다
  quack: function animal_quack(what) {
    console.log(what + ' whoof-whoof!')
  }, // 꽥꽥거리는 기능을 가졌다. 소리가 좀 다를 뿐.
  eyes: 'yellow', // 눈이 노랗다.
}

// 오리인지 판단하는 함수. 깃털이 있고 꽥꽥거리는 기능이 있으면 오리이다.
function check(who) {
  if (who.appearance == 'feathers' && typeof who.quack == 'function') {
    who.quack('I look like a duck!\n')
    return true
  }
  return false
}

console.log(check(duck)) // true
console.log(check(someAnimal)) // true
```

`key in object`이나 `typeof`을 사용하면 JS에서 타입에 대한 방어적인 코드를 작성할 수 있었다.  
하지만 type이 런타임에 결정된다는 점 때문에 방어 코드가 너무 방대해지는 단점이 생길 수 있다.

이러한 단점을 해결하고자 나온 것이 JS의 슈퍼셋 TS이다.  
TS 컴파일러는 컴파일 시점에 duck typing과 같은 방식으로 타입을 검사하여 컴파일 에러를 발생시킨다.  
이것을 structural typing이라 한다.

## 2. Structural Typing vs Nominal Typing

TS(JS)는 JAVA나 C\#과 달리 아래와 같이 사용할 수도 있다.

```typescript
class Duck {
  constructor() {}

  quack() {
    console.log('꽥꽥')
  }

  describeFeather() {
    console.log('회색 깃털을 가지고 있습니다.')
  }
}

class SomeAnimal {
  constructor() {}

  quack() {
    console.log('나는 꽥꽥 소리칠 수 있으니 duck이려나')
  }

  describeFeather() {
    console.log('나는 회색 깃털을 가지고 있으니 duck이려나')
  }
}

let a = new Duck()
let b = new SomeAnimal()

a = b
a.describeFeather() // '나는 꽥꽥 소리칠 수 있으니 duck이려나'
a.quack() // '나는 회색 깃털을 가지고 있으니 duck이려나'
```

이처럼 **구조**가 같으면 같은 타입을 간주하는 방식을 structural typing이라 한다.  
타입의 이름, 패키지나 모듈의 위치와 상관 없이 내부적으로 같은 구조를 띄고 있으면 같은 타입으로 간주하는 것이다.  
반면, **이름**을 기준으로 타입을 간주하는 방식을 nominal typing이라 한다.

### Nominal Typing in TS

TS는 nominal typing을 지원하지 않는다.  
structural typing은 전통적인 nominal typing에 비해 덜 엄격하고 의도치 않은 동작을 할 수 있지만 JS의 슈퍼셋이라 어쩔 수 없는 부분인 것 같다.  
하지만 다음과 같이 nominal typing을 지원하는 것처럼 코드를 구현할 수 있다.  
(아직은 이런 것을 쓸 일이 있나 싶긴 하다)

1. literal types

```typescript
/** Generic Id type */
type Id<T extends string> = {
  type: T
  value: string
}

/** Specific Id types */
type FooId = Id<'foo'>
type BarId = Id<'bar'>

/** Optional: constructors functions */
const createFoo = (value: string): FooId => ({ type: 'foo', value })
const createBar = (value: string): BarId => ({ type: 'bar', value })

let foo = createFoo('sample')
let bar = createBar('sample')

foo = bar // Error
foo = foo // Okay
```

2. `enum`

```typescript
// FOO
enum FooIdBrand {
  _ = '',
}
type FooId = FooIdBrand & string

// BAR
enum BarIdBrand {
  _ = '',
}
type BarId = BarIdBrand & string

// Usage Demo
let fooId: FooId
let barId: BarId

// Safety!
fooId = barId // error
barId = fooId // error

// Newing up
fooId = 'foo' as FooId
barId = 'bar' as BarId

// Both types are compatible with the base
let str: string
str = fooId
str = barId
```

3. `inteface`

```typescript
// FOO
interface FooId extends String {
  _fooIdBrand: string // To prevent type errors
}

// BAR
interface BarId extends String {
  _barIdBrand: string // To prevent type errors
}

// Usage Demo
var fooId: FooId
var barId: BarId

// Safety!
fooId = barId // error
barId = fooId // error
fooId = <FooId>barId // error
barId = <BarId>fooId // error

// Newing up
fooId = 'foo' as any
barId = 'bar' as any

// If you need the base string
var str: string
str = fooId as any
str = barId as any
```
