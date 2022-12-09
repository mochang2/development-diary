## 0. 공부하게 된 계기

본 글은 https://github.com/ryanmcdermott/clean-code-javascript 를 번역한 글이다.  
그냥 복붙 없이 내 손으로 한글자 한글자 타이핑하며 익히려고 추가한 페이지이다.

1. [Variables](#variables)
2. [Functions](#functions)
3. [Objects and Data Structures](#objects-and-data-structures)
4. [Classes](#classes)
5. [SOLID](#solid)
6. [Testing](#testing)
7. [Concurrency](#concurrency)
8. [Error Handling](#error-handling)
9. [Formatting](#formatting)
10. [Comments](#comments)
11. [Translation](#translation)

참고  
https://738.github.io/clean-code-typescript/

## Variables

**의미 있고 발음 가능한 변수 이름을 사용하라.**

Bad:

```javascript
const yyyymmdstr = moment().format('YYYY/MM/DD')
```

Good:

```javascript
const currentDate = moment().format('YYYY/MM/DD')
```

**같은 유형의 변수는 같은 단어를 사용하라.**

Bad:

```javascript
getUserInfo()
getClientData()
getCustomerRecord()
```

Good:

```javascript
getUser()
```

**검색 가능한 이름을 사용하라.**

우리는 코드를 쓰기보다 더 많이 읽는다. 우리가 작성한 코드가 읽을 수 있고 검색이 가능해야 한다. 프로그램을 이해할 때 의미있는 변수 이름을 짓지 않으면 읽는 사람으로 하여금 어려움을 느끼게 할 수 있다. 검색 가능한 이름을 지어라. ESLint는 이름 없는 상수를 식별할 수 있도록 도와준다.

Bad:

```javascript
// what the heck is 86400000 for?
setTimeout(callback, 86400000)
```

Good:

```javascript
const MILLISECONDS_PER_DAY = 60 * 60 * 124 * 1000

setTimeout(callback, MILLISECONDS_PER_DAY)
```

**설명 가능한 이름을 사용하라.**

Bad:

```javascript
const address = 'One Infinite Loop, Cupertino 95014'
const cityZipCodeRegex = /^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?$/

saveCityZipCode(
  address.match(cityZipCodeRegex)[1],
  address.match(cityZipCodeRegex)[2]
)
```

Good:

```javascript
const address = 'One Infinite Loop, Cupertino 95014'
const cityZipCodeRegex = /^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?$/
const [_, city, zipCode] = address.match(cityZipCodeRegex) || []

saveCityZipCode(city, zipCode)
```

**암시적인 변수를 사용하지 마라.**

Bad:

```javascript
const locations = ['Austin', 'New York', 'San Francisco']
locations.forEach((l) => {
  doStuff()
  doSomeOtherStuff()
  // ...
  // ...
  // ...
  // Wait, what is `l` for again?
  dispatch(l)
})
```

Good:

```javascript
const locations = ['Austin', 'New York', 'San Francisco']
locations.forEach((location) => {
  doStuff()
  doSomeOtherStuff()
  // ...
  // ...
  // ...
  dispatch(location)
})
```

**불필요한 context를 추가하지 마라.**

class나 object의 이름이 말해주는 것을 property의 이름에서 반복하지 마라.

Bad:

```javascript
const Car = {
  carMake: 'Honda',
  carModel: 'Accord',
  carColor: 'Blue',
}
```

Good:

```javascript
const Car = {
  make: 'Honda',
  model: 'Accord',
  color: 'Blue',
}
```

**short circuiting이나 conditionals 보다는 default parameter를 사용하라**

default parameter는 오직 `undefined`에 대해서만 동작한다는 것을 기억하라. loosy compare시 `false`로 의미가 되는 `''`, `""`, `false`, `null`, `0`, `NaN`은 동작하지 않는다.

Bad:

```javascript
function createMicrobrewery(name) {
  const breweryName = name || 'Hipster Brew Co.'
  // ...
}
```

Good:

```javascript
function createMicrobrewery(name = 'Hipster Brew Co.') {
  // ...
}
```

## Functions

**함수 인자는 2개 이하가 이상적이다.**

함수 인자가 적을수록 테스팅하기 쉬워진다.

3개 이상의 인자를 받고 있다면 그 함수는 두 가지 이상의 동작을 하고 있는지 의심하라.

특히 JS는 간단히 `object`를 만들 수 있기 때문에 `class`를 위한 많은 보일러 플레이트가 필요하지 않다. 그리고 ES6에 구조 분해 할당 문법이 도입되며 더 의미 있는 변수로 인자를 입력받아 사용할 수 있으며 Linter는 사용하지 않는 변수를 파악하기 수월하게 만들어준다.

Bad:

```javascript
function createMenu(title, body, buttonText, cancellable) {
  // ...
}

createMenu('Foo', 'Bar', 'Baz', true)
```

Good:

```javascript
function createMenu({ title, body, buttonText, cancellable }) {
  // ...
}

createMenu({
  title: 'Foo',
  body: 'Bar',
  buttonText: 'Baz',
  cancellable: true,
})
```

**함수는 한 가지 일만 해야 한다.**

함수가 두 가지 이상의 일을 한다면 수정하거나 테스트하기 어렵다. 한 가지 일을 할 때 비로소 리팩토링이 쉽다.

Bad:

```javascript
function emailClients(clients) {
  clients.forEach((client) => {
    const clientRecord = database.lookup(client)
    if (clientRecord.isActive()) {
      email(client)
    }
  })
}
```

Good:

```javascript
function emailActiveClients(clients) {
  clients.filter(isActiveClient).forEach(email)
}

function isActiveClient(client) {
  const clientRecord = database.lookup(client)
  return clientRecord.isActive()
}
```

**함수 이름은 그 자체로 무엇을 하는지 나타내야 한다.**

Bad:

```javascript
function addToDate(date, month) {
  // ...
}

const date = new Date()

// It's hard to tell from the function name what is added
addToDate(date, 1)
```

Good:

```javascript
function addMonthToDate(month, date) {
  // ...
}

const date = new Date()
addMonthToDate(1, date)
```

**함수는 한 가지 추상화 레벨을 가져야 한다.**

하나의 함수 안에 전혀 추상화가 되지 않은 코드, 추상화가 된 코드 등이 섞이면 이해하기 어려워진다.

Bad:

```javascript
function parseBetterJSAlternative(code) {
  const REGEXES = [
    // ...
  ]

  const statements = code.split(' ')
  const tokens = []
  REGEXES.forEach((REGEX) => {
    statements.forEach((statement) => {
      // ...
    })
  })

  const ast = []
  tokens.forEach((token) => {
    // lex...
  })

  ast.forEach((node) => {
    // parse...
  })
}
```

Good:

```javascript
function parseBetterJSAlternative(code) {
  const tokens = tokenize(code)
  const syntaxTree = parse(tokens)
  syntaxTree.forEach((node) => {
    // parse...
  })
}

function tokenize(code) {
  const REGEXES = [
    // ...
  ]

  const statements = code.split(' ')
  const tokens = []
  REGEXES.forEach((REGEX) => {
    statements.forEach((statement) => {
      tokens.push(/* ... */)
    })
  })

  return tokens
}

function parse(tokens) {
  const syntaxTree = []
  tokens.forEach((token) => {
    syntaxTree.push(/* ... */)
  })

  return syntaxTree
}
```

**중복된 코드를 제거하라.**

중복된 코드가 존재한다면 리팩토링 시 최소 두 군데 이상 반복적으로 고쳐야 하는 불편함이 발생한다.

하지만 중복된 코드를 제거하기 위해 'bad' 추상화를 사용한다면 더 안 좋은 코드가 되니 SOLID 원칙을 따라야 한다.

Bad:

```javascript
function showDeveloperList(developers) {
  developers.forEach((developer) => {
    const expectedSalary = developer.calculateExpectedSalary()
    const experience = developer.getExperience()
    const githubLink = developer.getGithubLink()
    const data = {
      expectedSalary,
      experience,
      githubLink,
    }

    render(data)
  })
}

function showManagerList(managers) {
  managers.forEach((manager) => {
    const expectedSalary = manager.calculateExpectedSalary()
    const experience = manager.getExperience()
    const portfolio = manager.getMBAProjects()
    const data = {
      expectedSalary,
      experience,
      portfolio,
    }

    render(data)
  })
}
```

Good:

```javascript
function showEmployeeList(employees) {
  employees.forEach((employee) => {
    const expectedSalary = employee.calculateExpectedSalary()
    const experience = employee.getExperience()

    const data = {
      expectedSalary,
      experience,
    }

    switch (employee.type) {
      case 'manager':
        data.portfolio = employee.getMBAProjects()
        break
      case 'developer':
        data.githubLink = employee.getGithubLink()
        break
    }

    render(data)
  })
}
```

**default object는 Object.assign을 이용하라.**

Bad:

```javascript
const menuConfig = {
  title: null,
  body: 'Bar',
  buttonText: null,
  cancellable: true,
}

function createMenu(config) {
  config.title = config.title || 'Foo'
  config.body = config.body || 'Bar'
  config.buttonText = config.buttonText || 'Baz'
  config.cancellable =
    config.cancellable !== undefined ? config.cancellable : true
}

createMenu(menuConfig)
```

Good:

```javascript
const menuConfig = {
  title: 'Order',
  // User did not include 'body' key
  buttonText: 'Send',
  cancellable: true,
}

function createMenu(config) {
  let finalConfig = Object.assign(
    {
      title: 'Foo',
      body: 'Bar',
      buttonText: 'Baz',
      cancellable: true,
    },
    config
  )
  return finalConfig
  // config now equals: {title: "Order", body: "Bar", buttonText: "Send", cancellable: true}
  // ...
}

createMenu(menuConfig)
```

**flag를 함수 파라미터로 이용하지 마라.**

Bad:

```javascript
function createFile(name, temp) {
  if (temp) {
    fs.create(`./temp/${name}`)
  } else {
    fs.create(name)
  }
}
```

Good:

```javascript
function createFile(name) {
  fs.create(name)
}

function createTempFile(name) {
  createFile(`./temp/${name}`)
}

// 두 함수를 부르는 로직에서 어떤 것을 부를지 구분한다.
```

**side effect를 피하라(part1).**

input만을 이용해서 (공유되는 scope의 변수를 바꾸지 않고) output을 만들도록 함수를 구현하라.

Bad:

```javascript
// Global variable referenced by following function.
// If we had another function that used this name, now it'd be an array and it could break it.
let name = 'Ryan McDermott'

function splitIntoFirstAndLastName() {
  name = name.split(' ')
}

splitIntoFirstAndLastName()

console.log(name) // ['Ryan', 'McDermott'];
```

Good:

```javascript
function splitIntoFirstAndLastName(name) {
  return name.split(' ')
}

const name = 'Ryan McDermott'
const newName = splitIntoFirstAndLastName(name)

console.log(name) // 'Ryan McDermott';
console.log(newName) // ['Ryan', 'McDermott'];
```

**side effect를 피하라(part2).**

input만을 이용해서 (메모리 값을 바꾸지 않고) output을 만들도록 함수를 구현하라.

주의 사항:  
아주 가끔 input에 대한 변경이 필요할 경우가 있지만 정말 드물다. 대부분 side effect가 없도록 리팩토링 가능하다.  
크기가 큰 `object`를 복사하는 것은 expensive한 작업이다. 다행히 `immutable.js`와 같은 좋은 라이브러리가 있으니 빠르고 cheap하게 `object`를 복사할 수 있다.

Bad:

```javascript
const addItemToCart = (cart, item) => {
  cart.push({ item, date: Date.now() })
}
```

Good:

```javascript
const addItemToCart = (cart, item) => {
  return [...cart, { item, date: Date.now() }]
}
```

**전역 함수에 쓰지 마라.**

전역을 오염시키는 것은 좋은 습관이 아니다. 왜냐하면 다른 library와 충돌이 날 수 있기 때문이다.

`prototype`을 직접 건들기 보다는 ES6에 `class` 문법이 나왔으니 이를 활용하라.

Bad:

```javascript
Array.prototype.diff = function diff(comparisonArray) {
  const hash = new Set(comparisonArray)
  return this.filter((elem) => !hash.has(elem))
}
```

Good:

```javascript
class SuperArray extends Array {
  diff(comparisonArray) {
    const hash = new Set(comparisonArray)
    return this.filter((elem) => !hash.has(elem))
  }
}
```

**명령형 프로그래밍 보다 함수형 프로그래밍을 지향하라.**

JS는 Haskell과 같은 완전 함수형 언어는 아니지만 함수형에 가까운 언어이다. 함수형 프로그래밍은 더욱 깔끔하며 예측하기 쉽다.

Bad:

```javascript
const programmerOutput = [
  {
    name: 'Uncle Bobby',
    linesOfCode: 500,
  },
  {
    name: 'Suzie Q',
    linesOfCode: 1500,
  },
]

let totalOutput = 0

for (let i = 0; i < programmerOutput.length; i++) {
  totalOutput += programmerOutput[i].linesOfCode
}
```

Good:

```javascript
const programmerOutput = [
  {
    name: 'Uncle Bobby',
    linesOfCode: 500,
  },
  {
    name: 'Suzie Q',
    linesOfCode: 1500,
  },
]

const totalOutput = programmerOutput.reduce(
  (totalLines, output) => totalLines + output.linesOfCode,
  0
)
```

**조건문을 (의미를 파악할 수 있도록) 감싸라.**

Bad:

```javascript
if (fsm.state === 'fetching' && isEmpty(listNode)) {
  // ...
}
```

Good:

```javascript
function shouldShowSpinner(fsm, listNode) {
  return fsm.state === 'fetching' && isEmpty(listNode)
}

if (shouldShowSpinner(fsmInstance, listNodeInstance)) {
  // ...
}
```

**부정 조건문을 지양하라.**

Bad:

```javascript
function isDOMNodeNotPresent(node) {
  // ...
}

if (!isDOMNodeNotPresent(node)) {
  // ...
}
```

Good:

```javascript
function isDOMNodePresent(node) {
  // ...
}

if (isDOMNodePresent(node)) {
  // ...
}
```

**조건문 자체를 지양하는 방법으로 구조화하라.**

`if`, `switch`를 사용하는 것 자체로, 함수가 하나의 행위를 하는 것이 아닌지 의심할 여지가 있다.

Bad:

```javascript
class Airplane {
  getCruisingAltitude() {
    switch (this.type) {
      case '777':
        return this.getMaxAltitude() - this.getPassengerCount()
      case 'Air Force One':
        return this.getMaxAltitude()
      case 'Cessna':
        return this.getMaxAltitude() - this.getFuelExpenditure()
    }
  }
}
```

Good:

```javascript
class Airplane {
  getCruisingAltitude() {
    throw new Error('subclass는 반드시 구현해야 하는 메서드입니다.')
  }
}

class Boeing777 extends Airplane {
  getCruisingAltitude() {
    return this.getMaxAltitude() - this.getPassengerCount()
  }
}

class AirForceOne extends Airplane {
  getCruisingAltitude() {
    return this.getMaxAltitude()
  }
}

class Cessna extends Airplane {
  getCruisingAltitude() {
    return this.getMaxAltitude() - this.getFuelExpenditure()
  }
}
```

**타입 체킹을 지양하라(part1).**

JS는 인터프리터 언어로 run time에 타입을 확인한다. 일관된 인터페이스를 사용하여 타입 체킹이 필요하지 않게 하라.

Bad:

```javascript
function travelToTexas(vehicle) {
  if (vehicle instanceof Bicycle) {
    vehicle.pedal(this.currentLocation, new Location('texas'))
  } else if (vehicle instanceof Car) {
    vehicle.drive(this.currentLocation, new Location('texas'))
  }
}
```

Good:

```javascript
function travelToTexas(vehicle) {
  vehicle.move(this.currentLocation, new Location('texas'))
}
```

**타입 체킹을 지양하라(part2).**

primitive type을 사용해 part1과 같은 방식으로 되지 않는다면 TS 도입을 고려하라. JS에서 타입 체킹 코드를 사용하면 로직 함수보다 타입 체킹 함수가 더 큰 부분을 차지하기도 한다.

Bad:

```javascript
function combine(val1, val2) {
  if (
    (typeof val1 === 'number' && typeof val2 === 'number') ||
    (typeof val1 === 'string' && typeof val2 === 'string')
  ) {
    return val1 + val2
  }

  throw new Error('Must be of type String or Number')
}
```

Good:

```javascript
function combine(val1, val2) {
  return val1 + val2
}
```

**과도하게 최적화하지 마라.**

모던 브라우저에서 이미 자원 최적화가 내부적으로 지원된다.

Bad:

```javascript
// On old browsers, each iteration with uncached `list.length` would be costly
// because of `list.length` recomputation. In modern browsers, this is optimized.
for (let i = 0, len = list.length; i < len; i++) {
  // ...
}
```

Good:

```javascript
for (let i = 0; i < list.length; i++) {
  // ...
}
```

**사용하지 않는 과거 코드는 지워라.**

사용하지 않는 코드는 보관할 필요가 없다. history는 git과 같은 형상 관리 시스템으로 관리하라.

Bad:

```javascript
function oldRequestModule(url) {
  // ...
}

function newRequestModule(url) {
  // ...
}

const req = newRequestModule
inventoryTracker('apples', req, 'www.inventory-awesome.io')
```

Good:

```javascript
function newRequestModule(url) {
  // ...
}

const req = newRequestModule
inventoryTracker('apples', req, 'www.inventory-awesome.io')
```

## Objects and Data Structures

**getter setter 사용하라.**

- 객체 속성을 얻는 것 이상으로 무언가를 더 하고 싶을 때, 코드 안에서 관련된 모든 접근자를 찾고 변경하지 않아도 된다.
- set을 사용할 때 검증 로직을 추가하는 것이 간단하다.
- 내부의 API를 캡슐화할 수 있다.
- 값을 조회하고 설정할 때 로그를 기록하고 에러를 처리하는 것이 쉽다.
- 서버에서 객체 속성을 불러올 때 lazy loading 할 수 있다.

Bad:

```javascript
function makeBankAccount() {
  // ...

  return {
    balance: 0,
    // ...
  }
}

const account = makeBankAccount()
account.balance = 100
```

Good:

```javascript
function makeBankAccount() {
  // this one is private
  let balance = 0

  // a "getter", made public via the returned object below
  function getBalance() {
    return balance
  }

  // a "setter", made public via the returned object below
  function setBalance(amount) {
    // ... validate before updating the balance
    balance = amount
  }

  return {
    // ...
    getBalance,
    setBalance,
  }
}

const account = makeBankAccount()
account.setBalance(100)
```

**private member를 갖는 객체를 생성하라.**

ES5이전에는 클로저를 이용해서 만들 수 있다.

Bad:

```javascript
const Employee = function (name) {
  this.name = name
}

Employee.prototype.getName = function getName() {
  return this.name
}

const employee = new Employee('John Doe')
console.log(`Employee name: ${employee.getName()}`) // Employee name: John Doe
delete employee.name
console.log(`Employee name: ${employee.getName()}`) // Employee name: undefined
```

Good:

```javascript
function makeEmployee(name) {
  return {
    getName() {
      return name
    },
  }
}

const employee = makeEmployee('John Doe')
console.log(`Employee name: ${employee.getName()}`) // Employee name: John Doe
delete employee.name
console.log(`Employee name: ${employee.getName()}`) // Employee name: John Doe
```

## Classes

**ES5의 함수보다 ES6의 클래스를 지향하라.**

일반적으로 클래스가 더 가독성이 좋다. 다만, 아주 작은 함수라면 클래스 overkill일 수 있으니 잘 고려하라.

Bad:

```javascript
const Animal = function (age) {
  if (!(this instanceof Animal)) {
    throw new Error('Instantiate Animal with `new`')
  }

  this.age = age
}

Animal.prototype.move = function move() {}

const Mammal = function (age, furColor) {
  if (!(this instanceof Mammal)) {
    throw new Error('Instantiate Mammal with `new`')
  }

  Animal.call(this, age)
  this.furColor = furColor
}

Mammal.prototype = Object.create(Animal.prototype)
Mammal.prototype.constructor = Mammal
Mammal.prototype.liveBirth = function liveBirth() {}

const Human = function (age, furColor, languageSpoken) {
  if (!(this instanceof Human)) {
    throw new Error('Instantiate Human with `new`')
  }

  Mammal.call(this, age, furColor)
  this.languageSpoken = languageSpoken
}

Human.prototype = Object.create(Mammal.prototype)
Human.prototype.constructor = Human
Human.prototype.speak = function speak() {}
```

Good:

```javascript
class Animal {
  constructor(age) {
    this.age = age
  }

  move() {
    /* ... */
  }
}

class Mammal extends Animal {
  constructor(age, furColor) {
    super(age)
    this.furColor = furColor
  }

  liveBirth() {
    /* ... */
  }
}

class Human extends Mammal {
  constructor(age, furColor, languageSpoken) {
    super(age, furColor)
    this.languageSpoken = languageSpoken
  }

  speak() {
    /* ... */
  }
}
```

**메서드 체이닝을 사용하라.**

메서드에서 간단히 `this`를 return하기만 하면 된다.

Bad:

```javascript
class Car {
  constructor(make, model, color) {
    this.make = make
    this.model = model
    this.color = color
  }

  setMake(make) {
    this.make = make
  }

  setModel(model) {
    this.model = model
  }

  setColor(color) {
    this.color = color
  }

  save() {
    console.log(this.make, this.model, this.color)
  }
}

const car = new Car('Ford', 'F-150', 'red')
car.setColor('pink')
car.save()
```

Good:

```javascript
class Car {
  constructor(make, model, color) {
    this.make = make
    this.model = model
    this.color = color
  }

  setMake(make) {
    this.make = make

    return this
  }

  setModel(model) {
    this.model = model

    return this
  }

  setColor(color) {
    this.color = color

    return this
  }

  save() {
    console.log(this.make, this.model, this.color)

    return this
  }
}

const car = new Car('Ford', 'F-150', 'red').setColor('pink').save()
```

**inheritance보다 composition을 지향하라.**

inheritance가 생각이 난다면 composition으로 해결할 수 없는지 한 번 더 생각하라. GoF에서도 composition이 더 좋은 패턴이라고 말한다.

만약 inheritance가 더 좋은 경우가 있다면 아마 다음과 같은 경우일 것이다.

- 'has-a' 관계가 아니라 'is-a' 관계(`User` - `UserDetails` vs `Human` - `Animal`)일 때.
- base class를 재사용할 수 있는 경우.
- base class를 변경하여 derived class를 전체적으로 변경하려는 경우.

Bad:

```javascript
class Employee {
  constructor(name, email) {
    this.name = name
    this.email = email
  }

  // ...
}

// Bad because Employees "have" tax data. EmployeeTaxData is not a type of Employee
class EmployeeTaxData extends Employee {
  constructor(ssn, salary) {
    super()
    this.ssn = ssn
    this.salary = salary
  }

  // ...
}
```

Good:

```javascript
class EmployeeTaxData {
  constructor(ssn, salary) {
    this.ssn = ssn
    this.salary = salary
  }

  // ...
}

class Employee {
  constructor(name, email) {
    this.name = name
    this.email = email
  }

  setTaxData(ssn, salary) {
    this.taxData = new EmployeeTaxData(ssn, salary)
  }
  // ...
}
```

## SOLID

**단일 책임 원칙, SRP(Single Responsibility Priniciple)**

클린 코드에서 말하듯이, '클래스를 변경할 때는 단 한 가지 이유만 존재해야 한다'. 클래스를 변경하는 많은 시간을 줄이는 것은 중요하다. 왜냐하면 너무 많은 기능이 한 클래스에 있고 그 안에서 하나의 기능을 수정한다면, 다른 종속된 모듈에 어떻게 영향을 줄지 이해하는 것이 어렵기 때문이다.

Bad:

```javascript
class UserSettings {
  constructor(user) {
    this.user = user
  }

  changeSettings(settings) {
    if (this.verifyCredentials()) {
      // ...
    }
  }

  verifyCredentials() {
    // ...
  }
}
```

Good:

```javascript
class UserAuth {
  constructor(user) {
    this.user = user
  }

  verifyCredentials() {
    // ...
  }
}

class UserSettings {
  constructor(user) {
    this.user = user
    this.auth = new UserAuth(user)
  }

  changeSettings(settings) {
    if (this.auth.verifyCredentials()) {
      // ...
    }
  }
}
```

**개방 폐쇄 원칙, OCP(Open / Closed Principle)**

이 원칙은 기본적으로 새로운 기능이 추가될 때 기존 코드를 변경하지 않고 새 기능을 추가할 수 있도록 해야 한다는 원칙이다.

Bad:

```javascript
class AjaxAdapter extends Adapter {
  constructor() {
    super()
    this.name = 'ajaxAdapter'
  }
}

class NodeAdapter extends Adapter {
  constructor() {
    super()
    this.name = 'nodeAdapter'
  }
}

class HttpRequester {
  constructor(adapter) {
    this.adapter = adapter
  }

  fetch(url) {
    if (this.adapter.name === 'ajaxAdapter') {
      return makeAjaxCall(url).then((response) => {
        // transform response and return
      })
    } else if (this.adapter.name === 'nodeAdapter') {
      return makeHttpCall(url).then((response) => {
        // transform response and return
      })
    }
  }
}

function makeAjaxCall(url) {
  // request and return promise
}

function makeHttpCall(url) {
  // request and return promise
}
```

Good:

```javascript
class AjaxAdapter extends Adapter {
  constructor() {
    super()
    this.name = 'ajaxAdapter'
  }

  request(url) {
    // request and return promise
  }
}

class NodeAdapter extends Adapter {
  constructor() {
    super()
    this.name = 'nodeAdapter'
  }

  request(url) {
    // request and return promise
  }
}

class HttpRequester {
  constructor(adapter) {
    this.adapter = adapter
  }

  fetch(url) {
    return this.adapter.request(url).then((response) => {
      // transform response and return
    })
  }
}
```

**리스코프 치환 원칙, LSP(Liskov Substitution Principle)**

만약 부모 클래스와 자식 클래스가 있다면, 부모 클래스와 자식 클래스는 잘못된 결과 없이 서로 교환하여 사용될 수 있d어야 한다는 원칙이다.

Bad:

```javascript
class Rectangle {
  constructor() {
    this.width = 0
    this.height = 0
  }

  setColor(color) {
    // ...
  }

  render(area) {
    // ...
  }

  setWidth(width) {
    this.width = width
  }

  setHeight(height) {
    this.height = height
  }

  getArea() {
    return this.width * this.height
  }
}

class Square extends Rectangle {
  setWidth(width) {
    this.width = width
    this.height = width
  }

  setHeight(height) {
    this.width = height
    this.height = height
  }
}

function renderLargeRectangles(rectangles) {
  rectangles.forEach((rectangle) => {
    rectangle.setWidth(4)
    rectangle.setHeight(5)
    const area = rectangle.getArea() // BAD: Returns 25 for Square. Should be 20.
    rectangle.render(area)
  })
}

const rectangles = [new Rectangle(), new Rectangle(), new Square()]
renderLargeRectangles(rectangles)
```

Good:

```javascript
class Shape {
  setColor(color) {
    // ...
  }

  render(area) {
    // ...
  }
}

class Rectangle extends Shape {
  constructor(width, height) {
    super()
    this.width = width
    this.height = height
  }

  getArea() {
    return this.width * this.height
  }
}

class Square extends Shape {
  constructor(length) {
    super()
    this.length = length
  }

  getArea() {
    return this.length * this.length
  }
}

function renderLargeShapes(shapes) {
  shapes.forEach((shape) => {
    const area = shape.getArea()
    shape.render(area)
  })
}

const shapes = [new Rectangle(4, 5), new Rectangle(4, 5), new Square(5)]
renderLargeShapes(shapes)
```

**인터페이스 분리 원칙, ISP(Interface Segregation Principle)**

_cf)_  
TS로 보는 것이 더 좋은 설명 같다: https://github.com/mochang2/development-diary/blob/main/039-solid.md#5-i---%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4-%EB%B6%84%EB%A6%AC-%EC%9B%90%EC%B9%99interface-segregation-principle

JS는 인터페이스가 없기 때문에 이 원칙은 다른 것들처럼 엄격하게 적용되지 않는다. 그러나 JS의 부족한 타입 시스템과도 중요하고 관련이 있다.

ISP는 "클라이언트가 사용하지 않는 인터페이스에 의존하도록 강요해서는 안 된다."라고 말한다. 인터페이스는 duck typing 때문에 JS에서 암묵적인 계약이다.

JS에서 이 원리를 보여주는 좋은 예는 큰 설정 객체를 필요로 하는 클래스를 위한 것이다. 대부분의 경우 클라이언트가 모든 설정을 필요로 하지 않기 때문에 많은 양의 옵션을 설정하지 않아도 좋다. 옵션으로 설정하면 "뚱뚱한 인터페이스"를 사용하지 않도록 방지할 수 있습니다.

Bad:

```javascript
class DOMTraverser {
  constructor(settings) {
    this.settings = settings
    this.setup()
  }

  setup() {
    this.rootNode = this.settings.rootNode
    this.settings.animationModule.setup()
  }

  traverse() {
    // ...
  }
}

const $ = new DOMTraverser({
  rootNode: document.getElementsByTagName('body'),
  animationModule() {}, // Most of the time, we won't need to animate when traversing.
  // ...
})
```

Good:

```javascript
class DOMTraverser {
  constructor(settings) {
    this.settings = settings
    this.options = settings.options
    this.setup()
  }

  setup() {
    this.rootNode = this.settings.rootNode
    this.setupOptions()
  }

  setupOptions() {
    if (this.options.animationModule) {
      // ...
    }
  }

  traverse() {
    // ...
  }
}

const $ = new DOMTraverser({
  rootNode: document.getElementsByTagName('body'),
  options: {
    animationModule() {},
  },
})
```

**의존성 역전 원칙, DIP(Dependency Inversion Principle)**

_cf)_  
TS로 보는 것이 더 좋은 설명 같다: https://github.com/mochang2/development-diary/blob/main/039-solid.md#6-d---%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%97%AD%EC%A0%84-%EC%9B%90%EC%B9%99dependency-inversion-principle

이 원칙은 두 가지 필수적인 사항을 명시한다.

- 상위 레벨 모듈은 하위 레벨 모듈에 의존해서는 안 된다. 둘 다 추상화에 의존해야 한다.
- 추상화는 세부사항에 의존해서는 안 된다. 세부 사항은 추상화에 따라 달라져야 한다.

앞서 언급했듯이 자바스크립트에는 인터페이스가 없기 때문에 의존하는 추상화는 암묵적인 계약이다. 즉, 객체/클래스가 다른 객체/클래스에 노출되는 메서드 및 속성이다. 아래 예제에서 암묵적으로 `InventoryTracker`에 대한 모든 요청 모듈이 `requestItems` 메서드를 갖는다고 가정한다.

Bad:

```javascript
class InventoryRequesterV1 {
  constructor() {
    this.REQ_METHODS = ['HTTP']
  }

  requestItem(item) {
    // ...
  }
}

class InventoryRequesterV2 {
  constructor() {
    this.REQ_METHODS = ['WS']
  }

  requestItem(item) {
    // ...
  }
}

class InventoryTracker {
  constructor(items) {
    this.items = items

    // BAD: We have created a dependency on a specific request implementation.
    // We should just have requestItems depend on a request method: `request`
    this.requester = new InventoryRequesterV1()
  }

  requestItems() {
    this.items.forEach((item) => {
      this.requester.requestItem(item)
    })
  }
}

const inventoryTracker = new InventoryTracker(['apples', 'bananas'])
inventoryTracker.requestItems()
```

Good:

```javascript
class InventoryTracker {
  constructor(items, requester) {
    this.items = items
    this.requester = requester
  }

  requestItems() {
    this.items.forEach((item) => {
      this.requester.requestItem(item)
    })
  }
}

class InventoryRequesterV1 {
  constructor() {
    this.REQ_METHODS = ['HTTP']
  }

  requestItem(item) {
    // ...
  }
}

class InventoryRequesterV2 {
  constructor() {
    this.REQ_METHODS = ['WS']
  }

  requestItem(item) {
    // ...
  }
}

// By constructing our dependencies externally and injecting them, we can easily
// substitute our request module for a fancy new one that uses WebSockets.
const inventoryTracker = new InventoryTracker(
  ['apples', 'bananas'],
  new InventoryRequesterV2()
)
inventoryTracker.requestItems()
```

## Testing

테스트는 배포보다 중요하다. 테스트가 없거나 부족한 경우, 코드를 배포할 때마다 당신은 어떤 것이 작동하지 않을지 확실하지 않을 것이다. 적절한 양의 테스트를 구성하는 것은 당신의 팀에게 달려있지만, 100%의 커버리지를 가진다면 마음의 평화를 얻을 것이다.

**한번에 하나의 개념만 테스트하라.**

Bad:

```javascript
import assert from 'assert'

describe('MomentJS', () => {
  it('handles date boundaries', () => {
    let date

    date = new MomentJS('1/1/2015')
    date.addDays(30)
    assert.equal('1/31/2015', date)

    date = new MomentJS('2/1/2016')
    date.addDays(28)
    assert.equal('02/29/2016', date)

    date = new MomentJS('2/1/2015')
    date.addDays(28)
    assert.equal('03/01/2015', date)
  })
})
```

Good:

```javascript
import assert from 'assert'

describe('MomentJS', () => {
  it('handles 30-day months', () => {
    const date = new MomentJS('1/1/2015')
    date.addDays(30)
    assert.equal('1/31/2015', date)
  })

  it('handles leap year', () => {
    const date = new MomentJS('2/1/2016')
    date.addDays(28)
    assert.equal('02/29/2016', date)
  })

  it('handles non-leap year', () => {
    const date = new MomentJS('2/1/2015')
    date.addDays(28)
    assert.equal('03/01/2015', date)
  })
})
```

## Concurrency

**callback이 아니라 `Promise`를 지향하라.**

Bad:

```javascript
import { get } from 'request'
import { writeFile } from 'fs'

get(
  'https://en.wikipedia.org/wiki/Robert_Cecil_Martin',
  (requestErr, response, body) => {
    if (requestErr) {
      console.error(requestErr)
    } else {
      writeFile('article.html', body, (writeErr) => {
        if (writeErr) {
          console.error(writeErr)
        } else {
          console.log('File written')
        }
      })
    }
  }
)
```

Good:

```javascript
import { get } from 'request-promise'
import { writeFile } from 'fs-extra'

get('https://en.wikipedia.org/wiki/Robert_Cecil_Martin')
  .then((body) => {
    return writeFile('article.html', body)
  })
  .then(() => {
    console.log('File written')
  })
  .catch((err) => {
    console.error(err)
  })
```

**`Promise`보다 `async`/`await`를 지향하라.**

Bad:

```javascript
import { get } from 'request-promise'
import { writeFile } from 'fs-extra'

get('https://en.wikipedia.org/wiki/Robert_Cecil_Martin')
  .then((body) => {
    return writeFile('article.html', body)
  })
  .then(() => {
    console.log('File written')
  })
  .catch((err) => {
    console.error(err)
  })
```

Good:

```javascript
import { get } from 'request-promise'
import { writeFile } from 'fs-extra'

async function getCleanCodeArticle() {
  try {
    const body = await get('https://en.wikipedia.org/wiki/Robert_Cecil_Martin')
    await writeFile('article.html', body)
    console.log('File written')
  } catch (err) {
    console.error(err)
  }
}

getCleanCodeArticle()
```

## Error Handling

**catch한 에러를 무시하지 마라.**

콘솔(`console.log`)에 에러를 기록하는 것은 그다지 좋지 않다. `try`/`catch`에서 코드의 일부를 감싸는 것은 에러가 발생할 수 있다고 생각한다는 것을 의미하므로 에러가 발생할 때에 대한 계획을 세우거나 코드 경로를 만들어야 한다.

+) `Promises`에 대한 에러도 무시하지 마라.

Bad:

```javascript
try {
  functionThatMightThrow()
} catch (error) {
  console.log(error)
}
```

Good:

```javascript
try {
  functionThatMightThrow()
} catch (error) {
  // One option (more noisy than console.log):
  console.error(error)
  // Another option:
  notifyUserOfError(error)
  // Another option:
  reportErrorToService(error)
  // OR do all three!
}
```

## Formatting

**일관된 capitalization을 사용하라.**

Bad:

```javascript
const DAYS_IN_WEEK = 7
const daysInMonth = 30

const songs = ['Back In Black', 'Stairway to Heaven', 'Hey Jude']
const Artists = ['ACDC', 'Led Zeppelin', 'The Beatles']

function eraseDatabase() {}
function restore_database() {}

class animal {}
class Alpaca {}
```

Good:

```javascript
const DAYS_IN_WEEK = 7
const DAYS_IN_MONTH = 30

const SONGS = ['Back In Black', 'Stairway to Heaven', 'Hey Jude']
const ARTISTS = ['ACDC', 'Led Zeppelin', 'The Beatles']

function eraseDatabase() {}
function restoreDatabase() {}

class Animal {}
class Alpaca {}
```

**함수 caller와 callee는 가까이 둬라.**

위에서 아래로 순서대로 읽을 수 있도록 함수와 메서드를 배치하라. 반드시 순서대로 함수와 메서드를 배치할 수 없다면 최대한 가까이 둬라.

Bad:

```javascript
class PerformanceReview {
  constructor(employee) {
    this.employee = employee
  }

  lookupPeers() {
    return db.lookup(this.employee, 'peers')
  }

  lookupManager() {
    return db.lookup(this.employee, 'manager')
  }

  getPeerReviews() {
    const peers = this.lookupPeers()
    // ...
  }

  perfReview() {
    this.getPeerReviews()
    this.getManagerReview()
    this.getSelfReview()
  }

  getManagerReview() {
    const manager = this.lookupManager()
  }

  getSelfReview() {
    // ...
  }
}

const review = new PerformanceReview(employee)
review.perfReview()
```

Good:

```javascript
class PerformanceReview {
  constructor(employee) {
    this.employee = employee
  }

  perfReview() {
    this.getPeerReviews()
    this.getManagerReview()
    this.getSelfReview()
  }

  getPeerReviews() {
    const peers = this.lookupPeers()
    // ...
  }

  lookupPeers() {
    return db.lookup(this.employee, 'peers')
  }

  getManagerReview() {
    const manager = this.lookupManager()
  }

  lookupManager() {
    return db.lookup(this.employee, 'manager')
  }

  getSelfReview() {
    // ...
  }
}

const review = new PerformanceReview(employee)
review.perfReview()
```

## Comments

**비즈니스 로직이 복잡할 때만 주석을 달아라.**

좋은 코드는 주석이 아닌, 그 자체로 본인의 기능을 설명한다.

Bad:

```javascript
function hashIt(data) {
  // The hash
  let hash = 0

  // Length of string
  const length = data.length

  // Loop through every character in data
  for (let i = 0; i < length; i++) {
    // Get character code.
    const char = data.charCodeAt(i)
    // Make the hash
    hash = (hash << 5) - hash + char
    // Convert to 32-bit integer
    hash &= hash
  }
}
```

Good:

```javascript
function hashIt(data) {
  let hash = 0
  const length = data.length

  for (let i = 0; i < length; i++) {
    const char = data.charCodeAt(i)
    hash = (hash << 5) - hash + char

    // Convert to 32-bit integer
    hash &= hash
  }
}
```

**필요 없는 코드를 주석으로 남겨두지 마라.**

필요하다면 버전 관리를 이용하라.

Bad:

```javascript
doStuff()
// doOtherStuff();
// doSomeMoreStuff();
// doSoMuchStuff();

/**
 * 2016-12-20: Removed monads, didn't understand them (RM)
 * 2016-10-01: Improved using special monads (JP)
 * 2016-02-03: Removed type-checking (LI)
 * 2015-03-14: Added combine with type-checking (JR)
 */
function combine(a, b) {
  return a + b
}
```

Good:

```javascript
doStuff()

function combine(a, b) {
  return a + b
}
```

**포지션 표시를 주석으로 하지 마라.**

Bad:

```javascript
////////////////////////////////////////////////////////////////////////////////
// Scope Model Instantiation
////////////////////////////////////////////////////////////////////////////////
$scope.model = {
  menu: 'foo',
  nav: 'bar',
}

////////////////////////////////////////////////////////////////////////////////
// Action setup
////////////////////////////////////////////////////////////////////////////////
const actions = function () {
  // ...
}
```

Good:

```javascript
$scope.model = {
  menu: 'foo',
  nav: 'bar',
}

const actions = function () {
  // ...
}
```
