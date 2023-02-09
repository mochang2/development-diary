## 0. 공부하게 된 계기

TS를 어떻게 하면 TS 용도에 맞게 쓸 수 있을까 하여 공부한 내용을 정리하고자 한다.

[TS 시작하기](https://github.com/mochang2/development-diary/new/main#1-ts-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0)  
[Type Casing](https://github.com/mochang2/development-diary/new/main#2-type-casing)  
[Symbol](https://github.com/mochang2/development-diary/new/main#3-symbol)  
[Tuple](https://github.com/mochang2/development-diary/new/main#4-tuple)  
[TS만의 타입](https://github.com/mochang2/development-diary/new/main#5-ts%EB%A7%8C%EC%9D%98-%ED%83%80%EC%9E%85)  
[declare](https://github.com/mochang2/development-diary/new/main#6-declare)  
[tsconfig.json 내 compileOptions 정리](https://github.com/mochang2/development-diary/new/main#7-tsconfigjson-%EB%82%B4-compileoptions-%EC%A0%95%EB%A6%AC)  
[interface 사용법](https://github.com/mochang2/development-diary/new/main#8-interface-%EC%82%AC%EC%9A%A9%EB%B2%95)  
[class](https://github.com/mochang2/development-diary/new/main#9-class)  
[generic 사용법](https://github.com/mochang2/development-diary/new/main#10-generic-%EC%82%AC%EC%9A%A9%EB%B2%95)

## 1. TS 시작하기

-   `npm i typescript`: tsc 명령어 사용 가능
-   `(npx) tsc --init`: 해당 디렉터리 내부에 기본적인 `tsconfig.json` 파일 생성

or

-   VS Code 내장 컴파일러 사용

## 2. Type Casing

TS의 primitive types는 모두 소문자(ex. string, number 등).  
따라서 아래와 같은 방법으로 사용하면 안 됨.

```ts
function reverse(s: String): String {
    return s.split("");
}

// String은 Object를 만드는 생성자
```

## 3. Symbol

고유하고 수정 불가능한 값을 만들어주므로 주로 object의 key값으로 사용해 접근을 제어하는데 씀.  
`new Symbol('some_primitive_types')`와 같은 방식이 아닌 `Symbol('some_primitive_types')`로 사용.

## 4. Tuple

```ts
const array: (string | number)[] = ["string", 0];

// 0번째 인자는 반드시 string, 두 번째 인자는 number인 형태가 존재한다고 가정
// TS는 아래와 같은 타입이 가능

const tuple: [string, number] = ["string", 0];
```

## 5. TS만의 타입

-   `any`: 최상위 타입. 어떠한 타입도 해당 변수에 할당 가능. 사용 지양.
-   `unknown`: any보다 type-safe. 어떠한 타입도 해당 변수에 할당 가능. 다만 `unknown` 타입의 값이 `any`나 `unknown` 타입을 제외한 타입의 변수에는 할당 불가능.

```ts
let notSure: unknown;
notSure = 1;
notSure = "maybe a string instead";

// any type에는 unknown 타입 할당 가능
let anyType: any;
anyType = notSure;

// any, unknown 이외의 type에는 unknown 타입 할당 불가능
let numberType: number;
numberType = notSure; // compile error!
```

-   `never`: 모든 타입의 subtype이며 모든 타입에 할당 가능. 하지만 `never`에는 어떠한 것(`any` 포함)도 할당할 수 없음.
-   `void`: 함수의 리턴 형식으로만 사용. 오직 `undefined`만이 `void`에 할당될 수 있음.

```ts
let a: string = "hello";

if (typeof a !== "string") {
    let b: never = a;
}
```

## 6. declare

[개념 정리](https://it-eldorado.tistory.com/127)  
[모듈 type 없을 때 사용법](https://kong-dev.tistory.com/164)

## 7. `tsconfig.json` 내 compileOptions 정리

-   strict: true일 경우 noImplicitAny, strictNullChecks, strictFunctionTypes 등 모든 타입 체킹을 킴.
-   noImplicitAny: TS가 타입 추론 중 `any` 타입이라고 판단하는데, 이를 명시하지 않은 경우 에러 발생시키는 옵션.
-   strictNullChecks: 모든 타입에 자동으로 포함되어 있는 `null`과 `undefined`를 제거하지 않으면 에러를 발생시키는 옵션. 해당 옵션이 없다면 모든 타입은 `null`과 `undefined` 값을 가질 수 있음.
-   noImplicitReturns: 모든 함수가 명시적으로 return되지 않으면 에러를 발생시키는 옵션.
-   strictFunctionTypes: 함수를 할당할 시에 함수의 매개변수 타입이 같거나 슈퍼 타입인 경우가 아니면 에러를 발생시키는 옵션.
-   noImplicitThis: (클래스에서 제외하고)첫 번째 매개변수 자리에 this를 넣고, this에 대한 타입을 표현하지 않으면 에러를 발생시키는 옵션.
-   typeRoots: 설정하지 않으면, (TS 2.0부터 사용 가능해진) `node_modules/@types` 의 모든 경로를 찾아서 사용. 사용하면 배열 안에 있는 경로들에서만 가져옴.
-   types: 배열 안의 모듈 또는 `node_modules/@types`에서 이름을 가져오며 typeRoots와 같이 사용하지 않음.
-   target: 빌드의 결과물 버전 지정. 보통 브라우저라면 ES5, node.js라면 그 이상의 버전 설정.
-   lib: 기본 type definition 라이브러리로 어떤 것을 사용할 것인지 지정. 브라우저 종류나 노드 버전에 따라 적절한 선택 필요.

## 8. interface 사용법

#### interface implements in class

```ts
interface IPerson1 {
    name: string;
    age?: number;
    hello(): void;
}

class Person implements IPerson1 {
    name!: string;
    age?: number | undefined;

    constructor(name: string) {
        this.name = name;
    }

    hello(): void {
        console.log(`ㅎㅇ ${this.name} 이다.`);
    }
}

const p1: IPerson1 = new Person("Mark"); // 타입 명시 안 해도 Person이라고 알아서 추론함
```

#### vs type

> 그래서 TypeScript 팀은 가능한 Type Alias보단 Interface를 사용하고, 합 타입 혹은 튜플 타입을 반드시 써야 되는 상황이면 Type Alias를 사용하도록 권장하고 있습니다.

출처: https://medium.com/humanscape-tech/type-vs-interface-%EC%96%B8%EC%A0%9C-%EC%96%B4%EB%96%BB%EA%B2%8C-f36499b0de50

> interface는 합성될 때 캐시가 되지만 type은 아니다.

출처: https://github.com/microsoft/TypeScript/wiki/Performance#preferring-interfaces-over-intersections

기본적으로 type은 해당 type을 지칭하는 *이름*을 새로 만드는 것이라면, interface는 새로운 타입을 만들어내는 것이라고 생각하면 됨.

```ts
type EatType = (food: string) => void;

interface IEat {
    (food: string) => void;
}
```

```ts
type PersonList = string[];

interface IPersonList {
    [index: number]: string;
}
```

```ts
// 상속
interface ErrorHandling {
    success: boolean;
    error?: { message: string };
}

interface ArtistsData {
    artists: { name: string }[];
}

type ArtistsResponseType = ArtistsData & ErrorHandling;

interface IArtistsResponse extends ArtistsData, ErrorHandling {}
```

```ts
// union
interface Bird {
    fly(): void;
    layEggs(): void;
}

interface Fish {
    swim(): void;
    layEggs(): void;
}

type PerType = Bird | Fish;

// interface 는 표현 불가능
```

```ts
// Declaration Merging
// type은 같은 이름을 사용하면 에러 발생
interface MergingInterface {
    a: string;
}

interface MergingInterface {
    b: string;
}

let mi: MergingInterface;
console.log(mi.a, mi.b); // 에러 나지 않음. 선언이 합쳐지기 때문
```

## 9. class

기본 접근 제어자는 `public`.  
strict 옵션이 켜져 있다고 가정.

```ts
class Person {
    name: string; // 초기화 안 됐다는 에러
    age: number; // 초기화 안 됐다는 에러

    public constructor(name: string, age: number) {}
}
```

아래 방식으로 에러 해결 가능.

```ts
class Person {
    name: string;
    age: number;

    public constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
    }
}

class Person {
    public constructor(public name: string, private age: number) {}
}
```

#### abstract class vs interface

_cf) 참고_  
`class`도 `implements`가 가능하지만 굳이...? 강제화 할 거면 `interface`로 지정하는 게 맞을 것 같음.

`abstract class`

-   그 자체로서 property를 가질 수 있음.
-   여러 `class`를 동시에 `extends` 받을 수 없음.
-   (자바와 다르게) `implements` 될 수 있지만 property나 method를 구현해야 됨.
-   클래스 멤버에 대한 **구현 세부 정보를 포함**할 수 있음.

`interface`

-   그 자체로서 property를 가질 수 없음. 단순히 선언만 가능.
-   여러 `interface`를 동시에 `implements` 할 수 있음.
-   서로 다른 `interface`가 `extends`할 수 있으며 `class`가 `implements`할 수 있음.
-   어떤 **행동에 대한 명세일 뿐**, 그 자체가 상속 관계를 나타내진 않음.

```ts
abstract class Job {
    readonly nickname: string;
    constructor(nickname: string) {
        this.nickname = nickname;
    }
    greet(): void {
        console.log(`My nickname is ${this.nickname}`);
    }
    abstract attack(): void;
}
class Warrior extends Job {
    attack() {
        console.log("검을 사용해 공격!");
    }
}
class Magician extends Job {
    attack() {
        console.log("마법을 사용해 공격!");
    }
}
// const job: Job = new Job('what'); Error!
// TS2511: Cannot create an instance of an abstract class.
const warrior: Job = new Warrior("warrior1");
warrior.greet(); // My nickname is warrior1
warrior.attack(); // 검을 사용해 공격!

const magician: Job = new Magician("magician1");
magician.greet(); // My nickname is magician1
magician.attack(); // 마법을 사용해 공격!
```

## 10. generic 사용법

```ts
// function
interface HelloFunctionG {
    <T>(message: T): T;
}

const f: HelloFunctionG = <T>(message: T): T => {
    return message;
};
```

```ts
// class extends
class Person<T extends string | number> {
    private _name: T;

    constructor(name: T) {
        this._name = name;
    }
}
```

```ts
interface IPerson {
    name: string;
    age: number;
}

const person: IPerson = {
    name: "lee",
    age: 123,
};

function getProp<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
}

function setProp<T, K extends keyof T>(obj: T, key: K, value: T[K]): void {
    obj[key] = value;
}
```
