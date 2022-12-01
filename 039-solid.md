## 0. 공부하게 된 계기

[클린 코드](https://github.com/mochang2/development-diary/blob/main/040-clean%20code%20js.md)에 대해 정리하기 전에 SOLID를 정리하고자 한다.  
JS는 객체 지향스럽게 코딩하기 위한 기능이 다소 부족해서 TS로 최대한 예제 코드를 구성했다.

참고 및 사진 출처  
https://ko.wikipedia.org/wiki/SOLID_(%EA%B0%9D%EC%B2%B4_%EC%A7%80%ED%96%A5_%EC%84%A4%EA%B3%84)  
https://www.nextree.co.kr/p6960/  
https://blog.itcode.dev/posts/2021/08/13/single-responsibility-principle  
https://www.digitalocean.com/community/conceptual-articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design  
https://738.github.io/clean-code-typescript/#solid

## 1. 개념

SOLID란 객체 지향 프로그래밍 및 설계의 다섯 가지 기본 원칙이다.  
프로그래머사 시간이 지나도 유지 보수와 확장이 쉬운 시스템을 만들고자 할 때 이 원칙들을 함께 적용할 수 있다.  
SOLID 원칙들은 소프트웨어 작업에서 프로그래머사 소스 코드가 읽기 쉽게 될 때까지 소프트웨어 소스 코드를 리팩토링할 수 있도록 적용할 수 있는 지침이다.

## 2. S - 단일 책임 원칙(Single Responsibility Priniciple)

~SOLID 원칙 중 가장 직관적인 이름 같다.~

단일 책임 원칙이란 **하나의 객체는 반드시 하나의 동작에 대한 책임을 갖는다**는 원칙이다.  
객체의 책임이 많아질수록 해당 객체의 변경에 따른 영향도의 양과 범위가 매우 커진다.  
단일 책임 원칙은 특정 객체의 책임 의존성 과중을 지양하기 위한 원칙이다.

모듈화가 강해질수록 다른 객체와의 의존성, 연관성은 떨어진다.  
반대로 이야기하면 모듈화가 약해질수록 다른 객체와의 의존성, 연관성은 크게 늘어나며 다른 클래스의 프로퍼티나 메서드에 접근이 가능해져 객체 지향 원칙에 따라 클래스를 나누는 이유가 사라질 수 있다.

### 적용 방법

혼재된 각 책임을 각각의 개별 클래스로 분할하여 클래스 당 하나의 책임만을 맡도록 하는 것이다.  
여기서 관건은 **책임만 분리하는 것이 아니라 분리된 두 클래스 간의 관계의 복잡성을 줄이는 것**이다.  
만약 각각의 클래스들이 유사하고 비슷한 책임을 중복해서 갖고 있다면 상위 클래스를 선언하는 것도 하나의 방법이다.

책임을 기존의 클래스로 모으는 것 외에도 새로운 클래스를 만드는 것이 하나의 방법일 수 있다.  
즉, **산발적으로 여러 곳에 분포된 책임들을 한 곳으로 모으는 것**이 중점이다.

### 코드로 보는 단일 책임 원칙

#### 단일 책임 원칙이 지켜지지 않는다면

```typescript
class App {
  // 게임 진행

  constructor() {
    this.init() // 데이터 초기화
    this.play()
  }

  play() {
    // 어떤 동작
  }

  processGame() {
    // 게임 진행
  }

  controlState() {
    // 게임 진행에 필요한 state 변경
  }

  getInput() {
    // 콘솔로부터 입력을 받음
  }

  printOutput() {
    // 콘솔에 출력함
  }
}
```

`App` 클래스가 하길 원하는 행동은 게임에 대한 전체적인 control이다.

하지만 위 코드에서 `App` 클래스는 다음과 같은 동작들을 한다.

- 입력을 받음
- 입력에 따라 state를 변경
- state 변경에 따른 다음 process 진행
- 출력

이렇게 하나의 객체에 너무 많은 책임이 몰릴 경우 프로젝트에서 해당 객체의 의존성이 높아진다.  
각자의 코드가 서로 의존될 경우, 코드 수정에 따른 영향도 역시 높아지고 범위 또한 넓어진다.  
이는 리팩토링 시 새로운 곳에서 에러가 발생시켜 또 다른 리팩토링을 야기할 수 있기 때문에 지양해야 한다.

#### 단일 책임 원칙을 지키려면

~(best practice로서 동작시키기에는 best가 아닐 수 있겠지만)~

각각의 역할을 나눌 필요가 있다.  
`App`은 여전히 state 변경에 따른 전체 게임 진행을 담당한다.  
하지만 입력을 받고, 출력하는 부분은 다른 객체에게 책임을 분산하고 해당 객체를 호출하게 만들 수 있다(`InputController`).  
게임에 대한 진행도 다른 객체에게 위임할 수 있다(`GameController`).  
또한 입력 시 state를 변경하길 원한다면 callback을 넘겨줌으로써 해결할 수 있다(`OutputController`).

```typescript
class App {
  // 게임 진행

  #inputController
  #outputController
  #state

  constructor() {
    this.init() // 데이터 초기화
    this.play()
  }

  init() {
    this.#inputController = new InputController()
    this.#outputController = new OutputController()
    this.#gameController = new GameController()
    this.#state = state.START
  }

  play() {
    // 비동기 처리 되어 있다고 가정
    this.#inputController.getInput(this.controlState)

    const result = this.#gameController.processGame(this.#state)
    this.#outputController(result)
  }

  controlState = () => {
    // 게임 진행에 필요한 state 변경
  }
}

class InputController {
  constructor() {}

  getInput(setGameState: (state: string) => {}) {
    try {
      readline('입력해주세요', (state: string) => {
        this.validateState(state)

        setGameState(state)
      })
    } catch (error) {
      this.handleError(error)
    }
  }

  validateState(state: string) {
    if (isInvalidState(state)) {
      throw new Error(error.INVALID_STATE)
    }
  }
}

class GameController {
  constructor() {}

  processGame(state: number): string {
    // state에 따른 게임 진행
  }
}

class OutputController {
  constructor() {}

  printResult(result: string) {
    // 출력 형식에 따라 출력
  }
}
```

추가적으로, 단일 책임 원칙이 잘 지켜지지 않는다고 생각되면 각각의 메서드, 함수는 하나의 동작만 하는지 생각해 보는 것도 좋은 방법이다.

## 3. O - 개방 폐쇄 원칙(Open Close Principle)

개방 폐쇄 원칙이란 **객체의 확장은 개방적으로, 객체의 수정은 폐쇄적으로 하라**는 원칙이다.  
기능이 변하거나 확장 가능하지만, 해당 기능의 _기존_ 코드는 수정하면 안 된다는 뜻이다.

~엥? 뭔 소리?~  
만약 객체를 수정할 때 해당 객체만 수정할 뿐만 아니라 해당 객체에 의존하는 다른 코드까지 고쳐야 한다면 좋은 설계가 아니라는 것이다.  
예를 들어 `debounce`를 사용하는 코드에서 callback이 변경한다고 `debounce` 자체를 고쳐야 한다면 안된다는 뜻이다.

### 적용 방법

1. 변경(확장)될 것과 변하지 않을 것을 구분한다.
2. 구현에 의존하기 보다는 인터페이스에 의존하도록 한다.

### 코드로 보는 개방 폐쇄 원칙

#### 개방 폐쇄 원칙이 지켜지지 않는다면

Pos기에 어떤 카드가 들어오느냐에 따라 구매 동작이 달라지길 원한다고 하자.

```typescript
class Card {
  constructor(type) {
    this.type = type
  }

  ApayOff(price: number) {
    // ACard에 따라 동작
  }

  BpayOff(price: number) {
    // ACard에 따라 동작
  }

  CpayOff(price: number) {
    // ACard에 따라 동작
  }
}

class Pos {
  purchase(card: Card, price: number) {
    switch (card.type) {
      case 'A':
        return card.ApayOff(price)
      case 'B':
        return card.BpayOff(price)
      case 'C':
        return card.CpayOff(price)
      default:
        return
    }
  }
}
```

만약 `Pos`가 다룰 수 있는 카드의 종류가 더 생긴다고 하자.  
`Card` 클래스 내부에 `DpayOff`, `EpayOff`... 등의 메서드가 더 생길 뿐만 아니라 `Pos` 클래스 자체에도 `case`문을 지속적으로 더 추가해야 한다.  
다루는 카드의 종류가 다양해졌기 때문에 `Card`와 관련된 부분은 고쳐야겠지만 `Pos`까지 다루는 부분은 비효율적이다.

#### 개방 폐쇄 원칙을 지키려면

TS에서 `interface`를 정의함으로써 각 카드마다 `payOff` 구현을 강제화하고, `Pos` class에서는 `Card` 타입 대신 구현한 `interface` 타입을 인자로 받으면 된다.

```typescript
interface Purchaseable {
  payOff(price: number): purchaseState // 결제 결과
}

class ACard implements Purchaseable {
  payOff(price: number) {
    // ACard에 따라 동작
  }
}

class BCard implements Purchaseable {
  payOff(price: number) {
    // BCard에 따라 동작
  }
}

class CCard implements Purchaseable {
  payOff(price: number) {
    // CCard에 따라 동작
  }
}

class Pos {
  purchase(card: Purchaseable, price: number) {
    return card.payOff(price)
  }
}
```

만약 JS라면 `class extends`를 쓴 뒤 프로토타입 체이닝을 통해 최상위 객체의 `payOff` 프로토타입에 접근한다면 에러를 발생시킴으로써 해결할 수 있다.

```javascript
class Card {
  payOff(price) {
    throw new Error('하위 클래스에서 구현해야 하는 메서드입니다.')
  }
}

class ACard extends payOff {
  payOff(price) {
    // ACard에 따라 동작
  }
}
```

## 4. L - 리스코프 치환 원칙(The Liskov Substitution Principle)

리스코프 치환 원칙이란 **부모 객체를 호출하는 동작에서 자식 객체로 부모 객체를 완전히 대체할 수 있어야 한다**는 원칙이다.  
즉, (`extends` 또는 `implements` 한)하위 타입은 상위 타입이 약속한 규약(public interface 등)을 지켜야 한다는 뜻이다.

### 적용 방법

1. 만약 두 개체가 똑같은 일을 한다면 둘을 하나의 클래스로 표현하고 이들을 구분할 수 있는 필드를 둔다.
2. 똑같은 연산을 제공하지만 동작 방식이 다르다면 인터페이스 상속을 이용한다.
3. 공통된 연산이 없다면 완전 별개인 2개의 클래스를 만든다.
4. 만약 두 개체가 하는 일에 추가적으로 무언가를 더 한다면 상속을 한다.
5. 상속이 어렵다면 합성을 사용하는 것이 좋다. 만약 반드시 상속 구조가 필요하다면 extract subclass, push down field, push down method 등의 리팩토링 기법을 사용한다.

### 코드로 보는 리스코프 치환 원칙

#### 리스코프 치환 원칙이 지켜지지 않는다면

많이 사용하는 예제로 직사각형, 정사각형 예제가 있다.  
일반적인 수학에서 정사각형은 직사각형이다.  
그렇다면 정사각형 클래스가 직사각형 클래스를 상속할 수 있는 것 아닌가? 라는 물음이 생길 것이다.

```typescript
class Rectangle {
  constructor() {
    this.width = 0
    this.height = 0
  }

  setWidth(newWidth: number) {
    this.width = newWidth
  }

  setHeight(newHeight: number) {
    this.height = newHeight
  }

  getArea(): number {
    return this.width * this.height
  }
}

class Square extends Rectangle {
  constructor() {
    super()

    this.width = 0
    this.height = 0
  }

  setWidth(newWidth: number) {
    this.width = newWidth
    this.height = newWidth
  }

  setHeight(newHeight: number) {
    this.height = newHeight
    this.width = newHeight
  }
}
```

위의 코드 선언은 별 문제가 없어보이지만 사용시 리스코프 치환 원칙을 위배한다.

```typescript
const rectangle1 = new Rectangle()
rectangle1.setWidth(5)
rectangle1.setHeight(10)

console.log(rectangle1.getArea()) // 50

const rectangle2 = new Square()
rectangle2.setWidth(5)
rectangle2.setHeight(10)

console.log(rectangle2.getArea()) // 50이 아닌 100
```

`rectangle1`을 `Rectangle`의 인스턴스로 만들 때는 예상했던 대로 동작을 한다.  
하지만 `rectangle2`를 `Rectangle`의 하위 객체인 `Square`의 인스턴스로 만들면 동작이 달라진다.

#### 리스코프 치환 원칙을 지키려면

위 적용 방법 중

> 2. 똑같은 연산을 제공하지만 동작 방식이 다르다면 인터페이스 상속을 이용한다.

를 적용한다.

```typescript
interface FourAngledShape {
  setWidth(newWidth: number): void
  setHeight(newHeight: number): void
  getArea(): number
}

class Rectangle implements FourAngledShape {
  constructor() {
    this.width = 0
    this.height = 0
  }

  setWidth(newWidth: number) {
    this.width = newWdith
  }

  setHeight(newHeight: number) {
    this.height = newHeight
  }

  getArea() {
    return this.width * this.height
  }
}

class Square implements FourAngledShape {
  constructor() {
    this.width = 0
    this.height = 0
  }

  setWidth(newWidth) {
    this.width = newWidth
    this.height = newWidth
  }

  setHeight(newHeight) {
    this.width = newHeight
    this.height = newHeight
  }

  getArea() {
    return this.width * this.height
  }
}
```

이제 더이상 `Rectangle`과 `Square`가 상속 관계가 아니므로 리스코프 치환 원칙을 만족하는 코드가 되었다.

위와 같은 방법 이외에도 TS는 `abstract`라는 키워드를 지원하므로 `Shape`라는 상위 추상 클래스를 만드는 것도 방법이다.

## 5. I - 인터페이스 분리 원칙(Interface Segregation Principle)

인터페이스 분리 원칙이란 **객체는 자신이 사용하지 않는 인터페이스에 의존하지 않아야 한다**는 원칙이다.  
즉, 어떤 클래스가 다른 클래스에 종속될 때에는 가능한 최소한의 인터페이스만을 사용해야 한다.  
다른 말로 '하나의 범용적인 인터페이스보다는 여러 개의 구체적인 인터페이스에 의존해야 한다.'로 표현할 수 있다.

### 적용 방법

1. 클래스 상속을 이용해 인터페이스를 나눈다.
2. 위임(특정 일의 책임을 다른 클래스나 메서드에 맡김)을 이용해 인터페이스를 나눈다.

### 코드로 보는 인터페이스 분리 원칙

#### 인터페이스 분리 원칙이 지켜지지 않는다면

```typescript
interface SmartPrinter {
  print()
  fax()
  scan()
}

class AllInOnePrinter implements SmartPrinter {
  print() {
    // ...
  }

  fax() {
    // ...
  }

  scan() {
    // ...
  }
}

class EconomicPrinter implements SmartPrinter {
  print() {
    // ...
  }

  fax() {
    throw new Error('Fax not supported.')
  }

  scan() {
    throw new Error('Scan not supported.')
  }
}
```

`EconomicPrinter` 클래스는 사용하지 않는 `fax`, `scan` 메서드를 구현하게끔 강제되어 어쩔 수 없이 에러를 날리는 불필요한 메서드가 되었다.

#### 인터페이스 분리 원칙을 지키려면

```typescript
interface Printer {
  print()
}

interface Fax {
  fax()
}

interface Scanner {
  scan()
}

class AllInOnePrinter implements Printer, Fax, Scanner {
  print() {
    // ...
  }

  fax() {
    // ...
  }

  scan() {
    // ...
  }
}

class EconomicPrinter implements Printer {
  print() {
    // ...
  }
}
```

`Printer`, `Fax`, `Scanner` 인터페이스를 각각 선언하고 이를 `implements`하면 불필요한 인터페이스 구현을 방지할 수 있다.

## 6. D - 의존성 역전 원칙(Dependency Inversion Principle)

의존성 역전 원칙이란 **객체는 저수준 모듈보다 고수준 모듈에 의미한다**는 원칙이다.

~엥 이것도 뭔 소리~  
다른 말로 표현하자면

- 상위 레벨의 모듈은 하위 레벨의 모듈에 의존하지 않아야 한다. 두 모듈 모두 추상화에 의존해야 한다.
- 추상 객체는 세부 객체에 의존하지 않아야 한다. 반대로 세부 객체가 추상 객체에 의존해야 한다.

경직된 객체보다 구현되지 않아 유연한 인터페이스가 더욱 확장 가능성이 높기 때문이다.

+) 다른 원칙의 하위호환 격이기 때문에 다른 원칙들을 준수한다면 자연스레 준수될 원칙이다.

### 적용 방법

_Layering_  
잘 구조화된 객체지향 아키텍처들은 각 레이어마다 잘 정의되고 통제되는 인터페이스를 통한 긴밀한 서비스들의 집합을 제공하는 레이어들로 구성되어 있다.  
이를 통해서 상위레벨의 모듈은 하위레벨의 모듈로의 의존성에서 벗어나 그 자체로 재사용되고 확장성도 보장받을 수 있다.

![DIP](https://user-images.githubusercontent.com/63287638/205083856-8be4ff42-d257-4e96-9ce7-ded3c4c24f69.png)

### 코드로 보는 의존성 역전 원칙

#### 의존성 역전 원칙이 지켜지지 않는다면

한손 무기와 캐릭터 클래스가 있고 다음과 같이 코드를 짠다면 어떤 문제가 발생할까?

```typescript
class OneHandSword {
  constructor(name: string, damage: number) {
    this.name = name
    this.damage = damage
  }

  attack(): number {
    return this.damage + this.calcuateBonusDamage()
  }
}

class Character {
  constructor(name: string, weapon: OneHandSword) {
    this.name = name
    this.weapon = weapon
  }

  attack(): number {
    return this.weapon.attack()
  }

  changeWeapon(weapon: OneHandSword) {
    this.weapon = weapon
  }
}
```

만약 무기가 양손 무기, 지팡이, 도끼, 창 등 다양해지면 `Character`는 다양해지는 무기에 따라 같이 변경되어야 한다.  
즉, 이는 개방 폐쇄 원칙도 위반하는 사례이다.

#### 의존성 역전 원칙을 지키려면

무기를 포괄할 수 있는 추상 인터페이스(클래스)를 선언한다면 해결할 수 있다.

```typescript
interface Attackable {
  attack(): number
}

class OneHandSword implements Attackable {
  constructor(name: string, damage: number) {
    this.name = name
    this.damage = damage
  }

  attack(): number {
    return this.damage + this.calcuateBonusDamage()
  }
}

class TwoHandSword implements Attackable {
  constructor(name: string, damage: number) {
    this.name = name
    this.damage = damage
  }

  attack(): number {
    return this.damage + this.calcuateBonusDamage()
  }
}

class Character {
  constructor(name: string, weapon: Attackable) {
    this.name = name
    this.weapon = weapon
  }

  attack(): number {
    return this.weapon.attack()
  }

  changeWeapon(weapon: Attackable) {
    this.weapon = weapon
  }
}
```

`Character`의 `weapon`이 범용적으로 사용할 수 있는 `Attackable interface`에 의존함으로써 무기가 지속적으로 추가되어도 `Character`는 변경되지 않아도 된다.
