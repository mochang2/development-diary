## 0. 공부하게 된 계기

[클린 코드](https://github.com/mochang2/development-diary/blob/main/040-clean%20code%20js.md)를 공부하며 디자인 패턴에 대해서도 익숙해질 필요가 있다고 생각했다.  
~필기 테스트에서도 나왔고 면접에서도 공격받았던 질문...~

모든 디자인 패턴을 다 작성한다기 보다는 JS에서 주로 사용되는 패턴을 먼저 정리하고자 한다(이후 방향성이 바뀔 수도 있다).

목차  
[Builder](#builder)  
[Factory](#factory)  
[Singleton](#singleton)  
[Prototype](#prototype)  
[Composite](#composite)  
[Decorator](#decorator)  
[Proxy](#proxy)  
[Facade](#facade)  
[Command](#command)  
[Iterator](#iterator)  
[Mediator](#mediator)  
[Observer](#observer)

참고  
https://joshua1988.github.io/web-development/javascript/javascript-pattern-design  
https://sangcho.tistory.com/entry/%EC%9B%B9%EA%B0%9C%EB%B0%9C%EC%9E%90%EA%B0%80%EC%95%8C%EC%95%84%EC%95%BC%ED%95%A07%EA%B0%80%EC%A7%80%EB%94%94%EC%9E%90%EC%9D%B8%ED%8C%A8%ED%84%B4  
https://im-developer.tistory.com/141  
https://www.devh.kr/2021/Design-Patterns-In-JavaScript/

## 1. 디자인 패턴이란

> 간단하게 말해서 디자인 패턴은 설계자들이 "올바른" 설계를 "빨리" 만들 수 있도록 도와줍니다.

디자인 패턴은 사람들의 경험이 담긴 문제 해결 방법이다.  
어떤 문제나 수정 사항이 발생했을 때, 하나하나 시행착오를 겪으면서 다시 짓기에는 시간과 비용이 많이 든다.  
기존에 누군가 겪은 시행착오를 바탕으로 특정 상황에서 발생하는 문제 패턴을 발견하고 적용한 것을 **디자인 패턴**이라고 부른다.

디자인 패턴은 설계자로 하여금 재사용이 가능한 설계는 선택하고, 재사용을 방해하는 설계는 배제하도록 도와준다.  
또한 패턴을 쓰면 이미 만든 시스템의 유지보수나 문서화도 개선할 수 있고, 클래스의 명세도 정확하게 할 수 있으며, 객체 간의 상호작용 또는 설계 의도까지 명확하게 정의할 수 있다.

### 객체

참고  
https://ko.wikipedia.org/wiki/%EA%B0%9D%EC%B2%B4_(%EC%BB%B4%ED%93%A8%ED%84%B0_%EA%B3%BC%ED%95%99)  
http://klausbreaktime.blogspot.com/2017/07/blog-post.html  
https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=tipsware&logNo=221029211791

공통적으로 사용되는 '객체'의 개념을 먼저 정리하고자 한다.  
가장 중요한 오해인 `객체 지향 == 클래스`를 해소하기 위한 내용이다.  
클래스는 객체 타입을 정의하기 위한 한 가지 문법일 뿐이다.

관점에 따라 '객체'에 대한 정의가 달라진다.

- 모델링 관점에서는 명확한 의미를 담고 있는 대상 또는 개념
- 프로그래머 관점에서는 클래스에서 생성된 변수
- 소프트웨어 개발 관점에서는 객체의 상태를 나타내는 데이터(property)와 그 데이터를 처리하거나 참조하는 함수(method)
- 객체지향 프로그래밍 관점에서는 소프트웨어 관점에서 본 데이터를 속성으로 변경하여 속성과 메서드 형태로 구현

쉽지는 않지만 한 마디로 객체를 정의하자면 **행위나 작업을 표준화시켜서 메모리에 표현된 것** 정도로 이야기할 수 있다.  
시스템은 상호작용하는 자율적인 객체들의 공동체로 구성된다.  
여기서 자율적인 객체란 상태와 행위를 함께 지니며 부여받은 *역할*에 대한 *책임*을 다하며 다른 객체와 *협력*하는 존재를 말한다.

객체 지향의 핵심은 책임을 적절히 분배 받은 객체 간의 협력이다.

객체의 책임은 크게 '하는 것'과 '아는 것'의 두 가지 범주로 분류된다.

- 하는 것
  - 객체를 생성하거나 계산을 하는 등의 스스로 하는 것
  - 다른 객체의 행동을 시작시키는 것
  - 다른 객체의 활동을 제어하고 조절하는 것
- 아는 것
  - 개인적인 정보에 관해 아는 것
  - 관련된 객체에 관해 아는 것
  - 자신이 유도하거나 계산할 수 있는 것에 관해 아는 것

객체의 특징은 다음과 같다.

1. 객체는 동작해야 한다.
2. 객체는 자율적이어야 한다. 자신이 어떤 책임을 수행하는지만 공개하고 어떻게 수행하는지는 자신이 스스로 결정해서 감춰야 한다(은닉). 상태는 그저 객체가 적절히 행동할 수 있게 도와주는 용도일 뿐이다. 객체지향의 세계에서 객체는 다른 객체의 상태에 직접적으로 접근할 수도, 상태를 변경할 수도 없다. 자율적인 객체는 스스로 자신의 상태를 책임져야 한다.

## 2. 생성 패턴

생성 패턴은 **인스턴스 만드는 절차를 추상화**하는 패턴이다.  
생성 패턴에 속하는 패턴들은 객체를 생성, 합성하는 방법이나 객체의 표현 방법을 시스템과 분리한다.  
생성 패턴을 이용하면 무엇이 생성되고, 누가 이것을 생성하며, 이것이 어떻게 생성되는지, 언제 생성할 것인지 결정하는 데 유연성을 확보할 수 있게 된다.

생성 패턴에는 중요한 두 가지 이슈가 있다.

- 시스템이 어떤 Concrete Class를 사용하는지에 대한 정보를 캡슐화한다.
- 이들 클래스의 인스턴스들이 어떻게 만들고 어떻게 결합하는지에 대한 부분을 완전히 가린다.

### Builder

참고  
https://ttum.tistory.com/35
https://pplenty.tistory.com/15  
https://dev-youngjun.tistory.com/197  
https://johngrib.github.io/wiki/pattern/builder/

**객체를 생성하는 클래스와 표현하는 클래스를 분리하여 동일한 절차에서도 서로 다른 표현을 생성하는 방법을 제공하기 위한 패턴이다.**

optional한 property가 많을 때 순서에 대한 관리가 어려워지는 문제를 해결할 수 있다(다만 JS와 같은 경우는 객체 리터럴을 손쉽게 만들 수 있으므로 단순히 이러한 이유라면 사용하지 않는 것이 더 좋을 것 같다).

다음과 같은 특징이 있다.

- 장점
  - 생성 과정이 복잡한 인스턴스를 일관된 프로세스로 생성할 수 있다.
  - 멤버 변수의 초기화와 검증을 각각의 멤버 변수별로 분리해서 작성할 수 있다.
  - 동일한 프로세스에서 서로 다른 객체를 생성할 수 있다.
- 단점
  - 빌더 클래스를 선언함으로써 코드가 더 길어져 관리해야 할 클래스가 많아지고 구조가 복잡해질 수 있다.

#### 코드 예시

여행 계획을 다루는 class가 있다고 하자.

- 여행 날짜
- 몇박 며칠
- n일차에 할 일
- 숙소
- 참여자 정보
- 경비

등을 관리하는 class이다.  
해당 클래스가 만약 당일치기 정보도 다룬다면 '숙소'와 같은 내용은 선택적인 사항이 될 것이다.  
이럴 때 Builder 패턴이 유용하다.

다음과 같은 방법으로 구현한다.

1. `Builder` 클래스를 static하게 내부에서 생성한다. 이때 관례적으로 Builder라는 이름을 뒤에 붙인다.
2. `Builder` 클래스의 생성자는 public으로 두고 필수 값들에 대해 생성자의 parameter로 받는다.
3. optional property는 method chaining을 이용하여 입력 받는다.

```javascript
class TourPlan {
  #date;
  #accomodations;
  // ...

  constructor(builder) {
    this.#date = builder.getDate();
    this.#accomodations = builder.getAccomodations();
    // ...
  }

  static TourPlanBuilder = class {
    #date;
    #accomodations;
    // ...

    constructor({ date }) {
      this.#date = date;
      this.#accomodations = null;
      // ...
    }

    getDate() {
      return this.#date;
    }

    getAccomodations() {
      return this.#accomodations;
    }

    setAccomodations(accomodations) {
      // 검증 코드, 초기화 코드 추가할 수 있음
      this.#accomodations = accomodations;

      return this;
    }

    // ...

    build() {
      return new TourPlan(this);
    }
  };
}
```

```javascript
// 당일치기라면
const tourplan1 = new TourPlan.TourPlanBuilder({ date: '2022-12-31' }).build();

// 당일치기가 아니라면
const tourplan2 = new TourPlan.TourPlanBuilder({ date: '2022-12-31' })
  .setAccomodations('두바이 완전 비싼 7성급 호텔')
  .build();
```

### Factory

참고  
https://readystory.tistory.com/117  
https://readystory.tistory.com/119  
https://biggwang.github.io/2019/06/28/Design%20Patterns/%5BDesign%20Patterns%5D%20%ED%8C%A9%ED%86%A0%EB%A6%AC%20%ED%8C%A8%ED%84%B4,%20%EB%8F%84%EB%8C%80%EC%B2%B4%20%EC%99%9C%20%EC%93%B0%EB%8A%94%EA%B1%B0%EC%95%BC-%EA%B8%B0%EB%B3%B8%20%EC%9D%B4%EB%A1%A0%ED%8E%B8/  
https://velog.io/@ellyheetov/Factory-Pattern  
https://dev-youngjun.tistory.com/195

**객체 생성 하는 코드를 분리하여 클라이언트 코드와의 결합도(의존성)를 낮추기 위한 패턴이다.**  
여기서 클라이언트 코드란 객체 생성과 관련된 코드를 호출시키는 부분을 이야기한다.  
Factory Method 패턴이라고도 한다.

팩토리 패턴에서는 객체를 생성하기 위한 인터페이스를 정의하는데, 어떤 클래스의 인스턴스를 만들지는 서브 클래스에서 결정하게 만든다.  
어떤 클래스가 자신이 생성해야 하는 객체의 클래스를 예측할 수 없거나 생성할 객체를 기술하는 책임을 서브 클래스에 전가할 때 사용된다.

- 장점
  - 종속성을 낮추고 결합을 느슨하게 하여 확장을 용이하게 한다.
  - 클라이언트 구현 객체들 사이에 추상화를 제공한다.
- 단점
  - 클래스가 많아진다.
  - 클라이언트가 creator 클래스를 반드시 상속해 product 클래스를 생성해야 한다(아래 코드 예시 확인).

#### 코드 예시

```javascript
class Product {
  constructor() {}
}

class User {
  constructor() {
    this.product = new Product();
  }
}
```

위 `Product` 클래스와 `User` 클래스는 의존 관계에 있어, 결합도가 높다.  
특히 아래와 같은 코드에선 확장할 때 큰 문제가 있다.

```javascript
class Product {
  constructor({ type }) {}
}

class NormalUser extends User {
  // 일반 사용자, 관리자 등
  constructor() {
    this.product = new Product({});
  }
}

class AbnormalUser extends User {
  // 밴 당한 사용자, 휴면 계정의 사용자 등
  constructor() {
    this.product = new Product({});
  }
}
```

만약 `Product` 클래스 생성자를 변경할 일이 생긴다면, 모든 `User` 관련 클래스들에 변경 사항이 생긴다.  
아래와 같이 `Factory`라는 하나의 클래스에서 인스턴스를 생성해줌으로써(예시를 `class` 문법을 썼을 뿐이지 그냥 함수를 써도 무방) `User` 관련 클래스들은 `Product`의 생성자가 변경되어도 추가적으로 수정할 필요가 없다.

```javascript
class Product {
  // ...
}

class Factory {
  static getInstance() {
    return new Product();
  }
}

class NormalUser {
  constructor() {
    this.product = Factory.getInstance();
  }
}
```

팩토리 패턴이라고 할 때 많이 나오는 예시로, 아래와 같은 경우도 있다.  
팩토리 클래스에서 어떤 타입의 인스턴스가 반환될지 결정(은닉)해 해당 인스턴스를 반환하는 것이다.

```javascript
class UserFactory {
  static getInstance({ type }) {
    switch (type) {
      case 'normal':
        return new NormalUser();
      case 'abnormal':
        return new AbnormalUser();

        throw new Error('invalid user type');
    }
  }
}

class NormalUser extends User {
  // ...
}

class AbnormalUser extends User {
  // ...
}

// app.js
const user1 = UserFactory.getInstance({ type: 'normal' });
```

다만 위 `UserFactory` 클래스와 같은 경우는 개방 폐쇄 원칙에 위반된다.  
`ConsiderableUser` 등 다양한 `User`를 확장한 클래스가 생기면 `UserFactory` 클래스도 변경해야 되기 때문이다.  
`UserFactory`에서 `if-else`나 `switch-case`를 걷어내는 방법이 아래의 추상 팩토리 패턴이다.

#### Abstract Factory

추상 팩토리 패턴은 인풋으로 서브 클래스에 대한 식별 데이터를 받은 것이 아니라 또 하나의 팩토리 클래스를 받는다.

바로 코드 예시로 보겠다.  
객체 지향 문법이 필요해서 이번에는 TS를 사용했다.

```typescript
abstract class User {
  constructor() {}

  login() {}

  logout() {}
}

class NormalUser extends User {
  // ...
}

class AbnormalUser extends User {
  // ...
}

interface userFactory {
  createUser: () => void;
}

class NormalUserFactory implements userFactory {
  constructor() {}

  createUser() {
    return new NormalUser();
  }
}

class AbnormalUserFactory implements userFactory {
  constructor() {}

  createUser() {
    return new AbnormalUser();
  }
}

// consumer class
class UserFactory {
  static getUser(factory: userFactory) {
    return new factory.createUser();
  }
}

// app.js
const adminUser = UserFactory.getUser(new NormalUserFactory());
const bannedUser = UserFactory.getUser(new AbormalUserFactory());
```

### Singleton

참고  
https://gyoogle.dev/blog/design-pattern/Singleton%20Pattern.html

**한 클래스 안에 하나의 객체만 유지하도록 하여 race condition 문제를 해결하기 위한 패턴이다.**

생성자가 여러 번 호출되어도 실제로 생성되는 객체는 하나이며 최초로 생성된 이후에 호출된 생성자는 이미 생성한 객체를 반환하도록 만든다.  
다음과 같은 특징이 있다.

- 장점
  - 메모리 낭비를 방지할 수 있다.
  - 클래스 인스턴스 간 데이터 공유가 가능하다.
- 단점
  - 혼자 너무 많은 일을 하거나 많은 데이터를 공유하면 개방-폐쇄 원칙에 위배된다.
  - 결합도가 높아지면 유지 보수나 테스트가 힘들어진다.

#### 코드 예시

`class` 문법이 나오기 전까지는 다음과 같이 사용했다.

```javascript
var userModule = (function () {
  var userAddress = {};
  var users = [];
  var userId = 0;

  return {
    create: (username, password) => {
      if (!userAddress.hasOwnProperty(username)) {
        var user = { id: userId, username, password };
        userAddress[username] = users.length + '';
        users.push(user);
        userId++;

        return user;
      } else {
        return users[userAddress[username]];
      }
    },
    get: (username) => {
      return userAddress[username] ? users[userAddress[username]] : null;
    },
  };
})();

console.log(userModule.create('Julia', 'hello123')); // { id: 0, username: 'Julia', password: 'hello123' }
console.log(userModule.create('Julia', 'hello123')); // { id: 0, username: 'Julia', password: 'hello123' }
console.log(userModule.create('Julia', 'hello123')); // { id: 0, username: 'Julia', password: 'hello123' }
console.log(userModule.create('Paul', 'hello456')); // { id: 1, username: 'Paul', password: 'hello456' }

console.log(userModule.get('Julia')); // { id: 0, username: 'Julia', password: 'hello123' }
console.log(userModule.get('Paul')); // { id: 1, username: 'Paul', password: 'hello456' }
console.log(userModule.get('Mike')); // null
```

ES7 이후에는 `static`이라는 너무 간단한 명령어가 생겼다.

```javascript
class UserModule {
  static userModule;

  constructor() {
    if (userModule) {
      return userModule;
    }

    // 없으면 다른 동작
  }
}
```

### Prototype

참고  
https://www.devkuma.com/docs/design-pattern/prototype/  
https://lee1535.tistory.com/76  
https://keencho.github.io/posts/prototype-pattern/  
https://readystory.tistory.com/122

**original 객체를 새로운 객체에 복사해 사용자의 필요에 따라 이를 수정하는 메커니즘을 제공하는 패턴이다.**

![프로토타입 패턴 구조](https://user-images.githubusercontent.com/63287638/221845994-3993921b-3d13-4943-8441-c9e95fd6e877.png)

`Prototype` 인터페이스는 임의의 인터페이스를 복제하는 메서드를 가진다.  
대부분의 경우 `clone()` 메서드 하나만 선언되어 있다.  
`ConcretePrototype`과 `SubClassPrototype`은 `Prototype` 인터페이스를 구현하는 클래스를 생성한다.  
이 클래스들은 원본 객체의 데이터를 복사하는 것 말고도 연결된 객체의 복사와 관련된 작업이나 전의 의존성에서 벗어나게 하는 작업 등을 수행할 수 있다.

코드가 구현 클래스에 의존하지 않아야 하는 경우나 객체를 초기화하는 방법만 다를 뿐 서브 클래스의 수를 줄이려는 경우 프로토타입 패턴을 사용할 수 있다.  
다음과 같은 특징이 있다.

- 장점
  - 구현 클래스에 직접 연결하지 않고 객체를 복사할 수 있다.
  - 복잡한(리소스가 많이 소요되는) 오브젝트를 보다 편리하게(더 적은 리소스로) 만들 수 있다.
  - 중복되는 초기화 코드를 제거할 수 있다.
- 단점
  - 순환 참조가 존재하는 경우 복제가 까다로울 수 있다.
  - 깊은 복사를 할지, 얕은 복사를 할지 선택해야 한다.

#### 코드 예시

첫 번째 예시는, 초기화 시 API를 통해 데이터를 가져오는 객체가 존재한다고 하자.  
일반적으로 네트워크 통신 및 DB 접근이 객체를 복사하는 것보다 비용이 비싸므로 비슷한 객체를 여럿 만들어야 할 때 매번 `new` 키워드로 생성한다면 많은 비용이 발생할 것이다.  
이러한 문제를 해결하기 위해 다음과 같이 코드를 작성할 수 있다.

```js
class DataFetchComponent {
  constructor(data = []) {
    this.data = data;
  }

  async fetch() {
    // 비동기 처리가 되었다고 가정하자.
    const { data } = await request('api-url');

    this.data = data;
  }

  clone() {
    return new DataFetchComponent(this.data.map((datum) => datum));
  }
}

async function useData() {
  const component = new DataFetchComponent();
  await component.fetch();

  return {
    forSomeReasons: component.clone(),
    forOtherReasons: component.clone(),
  };
}
```

두 번째 예시로, 하나의 객체를 통해 다양한 객체를 생성할 수 있다.

```js
const harryPorterBookPrototype = {
  title: 'Harry Porter',
  price: '$15',
  author: 'J.K.Rolling',
};

const harryPorterEnglishBook = Object.assign({}, harryPorterBookPrototype);
const harryPorterKoreanBook = Object.assign({}, harryPorterBookPrototype, {
  title: '해리 포터',
});
const harryPorterJapaneseBook = Object.assign({}, harryPorterBookPrototype, {
  title: 'ハリーポッター',
});

console.log(harryPorterEnglishBook); // { title: 'Harry Porter', price: '$15', author: 'J.K.Rolling' }
console.log(harryPorterKoreanBook); // { title: '해리 포터', price: '$15', author: 'J.K.Rolling' }
console.log(harryPorterJapaneseBook); // { title: 'ハリーポッター', price: '$15', author: 'J.K.Rolling' }
```

## 3. 구조 패턴

https://refactoring.guru/ko/design-patterns/structural-patterns  
https://www.devkuma.com/docs/design-pattern/structural/

구조 패턴은 **구조를 유연하고 효율적으로 유지하면서 객체들과 클래스들을 더 큰 구조로 조립하는 방법을 설명**하는 패턴이다.  
예를 들어 서로 다른 인터페이스를 지닌 2개의 객체를 묶어 단일 인터페이스를 제공하거나 객체들을 서로 묶어 새로운 기능을 제공한다.  
여기서 중요한 것은 인터페이스나 구현을 복합하는 것이 아니라 객체를 합성하는 방법을 제공한다는 것이다.

이 패턴을 이용하면 서로 독립적으로 개발한 클래스 라이브러리를 애초에 하나였던 것처럼 사용할 수 있다.  
또한 여러 인터페이스를 합성하여 서로 다른 인터페이스들의 통일된 추상을 제공한다.

### Composite

참고  
https://mygumi.tistory.com/343  
https://readystory.tistory.com/131

**복합 객체(group of object)나 단일 객체(an object)를 동일하게 취급하는 것을 목적으로 하는 패턴이다.**

OOP에서 Composite는 하나 이상의 유사한 객체를 구성으로 설계된 객체로, 모두 유사한 기능을 나타낸다.  
일반적으로 트리 구조에서 리프와 브랜치는 구별해서 사용되어 복잡성이 늘어나지만, 이 패턴을 사용하면 그러한 가능성이 줄어든다.  
또한 Directory-File 관계처럼 전체-부분 관계를 나타낼 때 유용하다.

아래는 Composite 패턴 UML이다.

![Compositie UML](https://user-images.githubusercontent.com/63287638/222100256-a6285673-dc87-4de7-9624-206bf9d95047.png)

`Client`는 컴포넌트를 사용하는 곳이다.  
`Component`는 모든 컴포넌트들을 추상화한 개념으로, `Leaf`와 `Composite`에 대한 inteface나 abstract class이다.  
`Leaf`는 `Composite`의 구성 요소이며 `Component`를 구현한 구현체로, 다른 컴포넌트에 대한 참조를 가지면 안 된다.  
`Composite`는 `Leaf` 객체들로 구성되어 있으며 `Component`를 구현한 구현체다.  
일반적으로 `Composite`는 `Leaf`를 관리(add, remove 등)하기 위한 추가적인 메서드가 필요하다.

다음과 같은 특징을 가지고 있다.

- 장점
  - 객체들이 모두 같은 타입으로 취급되기 때문에 새로운 클래스 추가가 용이하다.
  - 단일 객체 및 집합 객체를 구분하지 않고 코드 작성이 가능하여 사용자 코드가 단순해진다.
  - 런타임 단일 객체와 집합 객체를 구분하지 않고 일관된 프로그래밍이 가능하다.
- 단점
  - 설계가 지나치게 범용성을 많이 가져 Composite의 구성 요소에 제약을 가하기 힘들다.

#### 코드 예시

~최대한 파일과 디렉터리를 활용하려는 예제를 쓰려다 보니 정말 쓸모 없고 억지스러울 수 있다.~

모든 파일을 read / write 기능이 있다(그 외 기능도 있겠지만 여기서는 간단히 하기 위해 생략).  
일반 파일은 read를 하면 파일 내용을 읽어오고, write를 하면 파일에 내용을 쓴다(여기서는 overwrite만 한다고 가정한다).  
디렉터리는 read를 하면 내부에 저장된 파일들의 내용을 전부 읽어오고, write를 하면 모든 파일에 동일한 내용을 쓴다.  
Composite 패턴을 사용하면 (실제로는 구분해야겠지만) 사용자는 일반 파일인지 디렉터리인지 확인할 필요 없이 interface에 정의된 메서드만으로 파일 읽기, 쓰기를 다룰 수 있게 된다.

```ts
abstract class FFile {
  // 전역 객체 File이 존재해서 FFile이라고 명명
  read() {
    console.error('read를 반드시 구현해야 함');
  }

  write(text: string) {
    console.error('write를 반드시 구현해야 함');
  }
}

class NormalFile extends FFile {
  constructor(private text: string = '') {
    super();
  }

  read() {
    console.log(this.text);
  }

  write(text: string) {
    this.text = text;
  }
}

class DirectoryFile extends FFile {
  constructor(private files: NormalFile[] = []) {
    super();
  }

  read() {
    this.files.forEach((file) => file.read());
  }

  write(text: string) {
    this.files.forEach((file) => file.write(text));
  }

  append(newFile: NormalFile) {
    this.files.push(newFile);
  }
}

class Client {
  constructor() {
    const normalFile1 = new NormalFile();
    normalFile1.write('일반 파일1입니다.');
    normalFile1.read(); // 일반 파일1입니다.

    const normalFile2 = new NormalFile();
    normalFile2.write('일반 파일2입니다.');
    normalFile2.read(); // 일반 파일2입니다.

    const directoryFile = new DirectoryFile();
    directoryFile.append(normalFile1);
    directoryFile.append(normalFile2);
    directoryFile.read(); // 일반 파일1입니다. -> 일반 파일2입니다.

    directoryFile.write('디렉터리입니다.');
    directoryFile.read(); // 디렉터리입니다. * 2
  }
}

new Client();
```

위 방법은 `NormalFile`와 `DirectoryFile`를 다르게 취급하고 있다.  
하지만 둘은 공통의 조상 클래스 `FFile`을 가지고 있으므로 리스코프 치환 원칙을 이용해 `Composite` 즉, `DirectoryFile`을 수정할 수 있다.

```ts
class DirectoryFile extends FFile {
  constructor(private files: FFile[] = []) {
    super();
  }

  // ...

  append(newFile: FFile) {
    this.files.push(newFile);
  }
}
```

### Decorator

참고  
https://readystory.tistory.com/195  
https://gmlwjd9405.github.io/2018/07/09/decorator-pattern.html  
https://coding-factory.tistory.com/713  
https://refactoring.guru/ko/design-patterns/decorator

**런타임에 객체의 기능(책임)을 수정할 수 있게 하는 패턴이다.**

Decorator는 말 그대로 꾸며주는 놈이다.  
기본 기능을 가지고 있는 클래스를 하나 만들고 이외에 부가적인 기능들을 추가하기 편하도록 설계하는 방법을 제공한다.

Decorator 패턴은 Composite 패턴과 비슷하지만 다른 점이 두 가지 있다.  
첫 번째는 하나의 자식 컴포넌트만 존재하는 Composite 패턴과 달리 Decorator 패턴은 여러 자식 컴포넌트가 존재할 수 있다.  
두 번째는 Decorator 패턴은 래핑된 객체에 추가 책임이 존재하는 반면, Composite 패턴은 자식들의 결과를 요악하기만 한다.

아래는 Decorator 패턴 UML이다.

![Decorator UML](https://user-images.githubusercontent.com/63287638/224022707-2dab5b7f-21b9-4371-a68c-fbdd431541d9.png)

`Component`는 기본 기능을 뜻하는 `ConcreteComponent`와 추가 기능을 뜻하는 `Decorator`의 공통 기능을 정의하고 일반적으로 인터페이스, 추상 클래스를 사용한다.  
`ConcreteComponent`는 기본 기능을 구현한 클래스이다.  
`Decorator` 많은 수가 존재할 수 있는, 구체적인 `Decorator`의 공통 기능을 제공한다.  
`ConcreteDecoratorA`, `ConcreteDecoratorB`는 `Decorator`의 하위 클래스로 기본 기능에 추가되는 개별적인 기능을 구현한 클래스이다.

다음과 같은 특징을 가지고 있다.

- 장점
  - 새 자식 클래스를 만들지 않고도 객체의 행동을 확장할 수 있다(아래 코드 예시에서 자세히 다룸).
  - `Component`나 `ConcreteComponent`, `Decorator` 등을 변경할 필요가 없기 때문에 _런타임에_ 객체들에서부터 책임들을 추가하거나 제거할 수 있다.
  - 객체를 여러 데코레이터로 래핑하여 여러 행동들을 합성할 수 있다.
- 단점
  - 래퍼들의 스택에서 특정 래퍼를 제거하기가 어렵다.
  - 데코레이터의 행동이 순서에 의존하지 않는 방식으로 데코레이터를 구현하기 어렵다.
  - 계층들의 초기 설정 코드가 복잡할 수 있다.

데코레이터 패턴은 다음과 같은 상황에서 사용하기 좋다.

1. 클래스의 요소들을 계속해서 수정을 하면서 사용하는 구조가 필요한 경우.
2. 여러 요소들을 조합해서 사용하는 클래스 구조인 경우.
3. 컴파일 단계에서가 아닌 런타임 단계에서 복합 방법이나 대상을 변경이 필요한 경우.

#### 코드 예시

다양한 기능이 추가될 수 있는 클래스에서 Decorator 패턴은 서브 클래스를 생성하는 것보다 유연한 방법을 제공한다.  
내비게이션을 구현한다고 하자.  
기본적으로 아래처럼 화면에 도로를 표시하는 클래스가 있다.

```ts
class RoadDisplay {
  draw() {
    console.log('기본 도로 표시 ');
  }
}
```

만약 화면에 각각 차선 표시와 교통량 표시, 교차로 표시를 할 수 있는 기능이 추가되어야 한다고 하자.  
다음과 같은 세 개의 클래스가 추가되어 사용될 수 있을 것이다.

```ts
class RoadDisplayWithLane extends RoadDisplay {
  constructor() {
    super();
  }

  draw() {
    super.draw();
    console.log('차선 표시 ');
  }
}

class RoadDisplayWithTraffic extends RoadDisplay {
  constructor() {
    super();
  }

  draw() {
    super.draw();
    console.log('교통량 표시 ');
  }
}

class RoadDisplayWithIntersection extends RoadDisplay {
  constructor() {
    super();
  }

  draw() {
    super.draw();
    console.log('교차로 표시 ');
  }
}

let roadDisplay: RoadDisplay = new RoadDisplay();
roadDisplay.draw(); // 기본 도로 표시

roadDisplay = new RoadDisplayWithLane();
roadDisplay.draw(); // 기본 도로 표시 + 차선 표시
```

하지만 만약 여러 가지 기능의 조합이 필요하다면?  
예를 들어 기본 정보 + 교차로 + 교통량 표시의 기능이 필요하다면, 또는 기본 정보 + 교통량 + 교차로 표시의 기능이 필요하다면 또다른 서브 클래스를 만들어야 할 것이다.  
심하면 아래와 같은 다이어그램이 나올 수 있다.

![dirty subclasses](https://user-images.githubusercontent.com/63287638/224026948-320d7adf-aca9-4bce-8522-658af7bcdb96.png)

이를 Decorator 패턴으로 해결하면 다음과 같이 작성할 수 있다.

```ts
abstract class Display {
  draw() {
    console.error('이거 구현해야 합니다.');
  }
}

class RoadDisplay extends Display {
  draw() {
    console.log('기본 도로 표시 ');
  }
}

abstract class DisplayDecorator extends Display {
  // 데코레이터 패턴의 공통 클래스
  private display!: Display;

  constructor(display: Display) {
    super();
    this.display = display;
  }

  draw() {
    this.display.draw();
  }
}

class LaneDecorator extends DisplayDecorator {
  constructor(display: Display) {
    super(display);
  }

  draw() {
    super.draw();
    this.drawLane();
  }

  private drawLane() {
    console.log('차선 표시 ');
  }
}

class TrafficDecorator extends DisplayDecorator {
  constructor(display: Display) {
    super(display);
  }

  draw() {
    super.draw();
    this.drawTraffic();
  }

  private drawTraffic() {
    console.log('교통량 표시 ');
  }
}

class IntersectionDecorator extends DisplayDecorator {
  constructor(display: Display) {
    super(display);
  }

  draw() {
    super.draw();
    this.drawIntersection();
  }

  private drawIntersection() {
    console.log('교차로 표시 ');
  }
}

const road = new RoadDisplay();
road.draw(); // 기본 도로 표시

const roadWithLane = new LaneDecorator(new RoadDisplay());
roadWithLane.draw(); // 기본 도로 표시 + 차선 표시

const roadWithTraffic = new TrafficDecorator(new RoadDisplay());
roadWithTraffic.draw(); // 기본 도로 표시 + 교통량 표시

const roadWithAll = new TrafficDecorator(
  new LaneDecorator(new IntersectionDecorator(new RoadDisplay()))
);
roadWithAll.draw(); // 기본 도로 표시 + 교차로 표시 + 차선 표시 + 교통량 표시

const roadWithAll2 = new LaneDecorator(
  new TrafficDecorator(new IntersectionDecorator(new RoadDisplay()))
);
roadWithAll2.draw(); // 기본 도로 표시 + 교차로 표시 + 교통량 표시 + 차선 표시
```

### Proxy

참고  
https://readystory.tistory.com/132  
https://coding-factory.tistory.com/711  
https://refactoring.guru/ko/design-patterns/proxy

**다른 객체로 접근하는 것을 통제하기 위해서 그 객체의 surrogate나 placeholder의 역할을 하는 객체를 제공하는 패턴이다.**

Proxy는 Reverse, Forward Proxy Server에서 사용하는 단어와 의미가 같다.  
surrogate나 placeholder가 전처리를 한 뒤 실제 서비스를 호출한다.  
약간 React에서 HOC하고 비슷한 용도로 사용할 수 있다고도 생각이 들었다.

Decorator와 비슷한 구조를 가지고 있으나 사용하는 의도는 다르다.  
두 패턴 모두 한 객체가 일부 작업을 다른 객체에 위임해야 하는 합성 원칙을 기반으로 한다.  
하지만 Proxy는 자체적으로 자신의 서비스 객체의 수명 주기를 관리하는 반면 Decorator의 합성은 클라이언트에 의해 제어된다.

아래는 Proxy 패턴 UML이다.

![Proxy UML](https://user-images.githubusercontent.com/63287638/224519577-c00acd1e-99eb-49d4-b17b-172ecd32ada7.png)

`Client`는 컴포넌트를 사용하는 곳이다.  
`Subject`는 `Proxy`와 `RealSubject`가 공통적으로 가지고 있는 속성을 명세화한 객체로, 인터페이스 또는 가상 클래스를 통해 구현한다.  
`Proxy`는 `Client`에서 사용하는 객체로, surrogate, placeholder 역할을 한다. 내부적으로 전처리 후 `RealSubject`를 호출한다.  
`RealSubject`는 `Client`가 원하는 내용(`DoAction`)을 실제로 행하는 객체이다.

다음과 같은 특징을 가지고 있다.

- 장점
  - 클라이언트들이 알지 못하는 상태에서 서비스 객체를 제어할 수 있다.
  - 클라이언트들이 신경 쓰지 않을 때 서비스 객체의 수명 주기를 관리할 수 있다.
  - 프록시는 서비스 객체가 준비되지 않았거나 사용할 수 없는 경우에도 작동한다(일종의 lazy loading처럼).
- 단점
  - 새로운 클래스들을 많이 도입해야 하므로 코드가 복잡해질 수 있다.
  - 객체를 한 단계 더 거쳐야 하므로 성능이 낮아질 수 있다.

Proxy 패턴은 사용하기 쉬운 만큼 다양한 패턴이 존재한다.  
그 중 자주 사용되는 3가지 정도 사용 예시가 있다.

1\) _가상 프록시_  
필요로 하는 시점까지 객체 생성을 연기하고, 해당 객체가 생성된 것처럼 동작하도록 만들고 싶을 때 사용한다.  
리소스가 많이 요구되는 작업은 해당 작업이 진짜 필요로 할 때까지 뒤로 미룬다.  
예를 들어 많은 이미지들을 처리해야 하는 경우, 이미지가 정말 필요하다는 요청이 올 때까지 미뤄 다른 작업의 우선 순위와 속도를 높일 수 있다.

2\) _원격 프록시_  
원격 객체에 대한 접근을 surroage 역할을 하는 객체가 대신해, 서로 다른 주소 공간에 있는 객체를 마치 같은 주소 공간에 있는 것처럼 동작하게 만드는 패턴이다.  
Google Docs가 그 예시이다.  
(내 생각) 그 외에도 서버와 통신할 때 Proxy 서버를 둬, redirect / LB / 접근 제어 처리 / 프로토콜에 따른 요청 분해 등을 한 다음 서버 자원을 다루는 API를 호출하는 것도 원격 프록시의 한 종류라고 생각한다.

3\) _보호 프록시_  
객체에 대한 접근 권한을 제어하거나 객체마다 접근 권한을 달리하고 싶을 때 사용하는 패턴이다.

이외에도 로깅을 남기는 처리 등을 하도록 Proxy를 만들 수 있다.

#### 코드 예시

다른 코드들을 둘러봤을 때 `RealSubject`를 `Proxy` 내부에 선언한 뒤 `Proxy`에서만 사용해도 될 것 같은데, 그런 코드 예시는 딱히 없었다.  
그래서 아래 코드 예시도 `RealSubject`를 `Proxy` 내부에 굳이 구현은 안 했지만, Builder 패턴처럼 `RealSubject`를 `Proxy` 내부에 선언하고 사용하는 것도 일종의 방법일 것 같다.

아래에서 `System`은 OS의 기능을 수행하게끔 하는 클래스로, 임의로 만든 가상의 클래스이다.

첫 번째 예시로 지연된 생성 처리를 위한 Proxy이다.  
쪼매 억지일 수 있지만 그러려니 하자.

```ts
interface ImageSubject {
  display: () => Promise<void>;
}

class Image implements ImageSubject {
  constructor(private fileName: string) {}

  async display() {
    console.log('Displaying', this.fileName);

    const image = await this.loadFromDisk();
    api.response({
      image,
    });
  }

  async loadFromDisk() {
    console.log('Loading', this.fileName);

    return await System.load(this.fileName);
  }
}

class ProxyImage implements ImageSubject {
  private image: ImageSubject;

  constructor(private fileName: string) {}

  async resizeImage(size) {
    await System.changeImageFile(this.fileName, {
      resize: true,
      size,
    });
  }

  async display() {
    if (!image) {
      this.image = new Image(this.fileName);
    }
    await this.image.display();
  }
}

(async () => {
  const image = ProxyImage('puppy.png');

  await image.resizeImage(640);
  // 이외에도 Image 객체가 직접 필요없지만 display 하기 전까지 처리해야 할 작업들 요청

  await image.display();
})();
```

두 번째 예시로 접근 권한 제어이다.  
시스템 명령어를 사용할 때 일부 명령어, 예를 들어 파일 삭제 명령어 등은 되돌릴 수 없이 위험한 명령어일 수 있다.  
그러므로 root가 아니면 명령어를 실행하지 못하게 제한할 필요가 있다.

```ts
interface SystemCommandSubject {
  runCommand: (command: string) => void;
}

interface UserInformation {
  username: string;
  password: string;
}

class SystemCommandExecutor implements SystemCommandSubject {
  runCommand(command: string) {
    System.run(command);
  }
}

class SystemCommandProxy implements SystemCommandSubject {
  static privilegedCommands = ['rm', 'chown'];

  private isRoot!: boolean;
  private executor!: SystemCommandSubject;

  constructor(userInformation: UserInformation) {
    this.isRoot = this.isRightRootInformation(userInformation);
    this.executor = new SystemCommandExecutor();
  }

  isRightRootInformation(userInformation: UserInformation): boolean {
    return System.checkUser(userInformation);
  }

  runCommand(command: string) {
    const canRun =
      this.isRoot ||
      !SystemCommandProxy.privilegedCommands.some((privilegedCommand) =>
        command.startsWith(privilegedCommand)
      );
    if (canRun) {
      this.executor.runCommand(command);

      return;
    }

    console.error('권한이 없습니다');
  }
}

const privilegedCommander = new SystemCommandProxy({
  username: 'root',
  password: '0000',
}); // correct root
const unprivilegedCommander = new SystemCommandProxy({
  usename: 'not-root',
  passowrd: '0000',
}); // incorrect root

privilegedCommander('rm'); // 정상 작동
unprivilegedCommander('rm'); // 권한이 없습니다
```

세 번째 예시로 캐싱이다.  
외부 라이브러리를 입맛에 맞게 변경하거나 JAVA와 같은 언어에서 `final`로 선언된 클래스에 대한 조작이 필요할 경우 사용할 수 있다.  
예를 들어 유튜브 API를 제공하는 클래스를 이용해서 본인의 서비스에서 이용해야 하지만 유튜브 API 자체에는 캐싱이 지원되지 않는다고 하자.

```ts
interface YouTubeAPI {
  listVideos: () => void;
  getVideoInformation: (id: string) => void;
}

// 유튜브에서 제공하는 클래스
class ThirdPartyYouTube implements YouTubeAPI {
  listVideos() {
    // ...
  }

  getVideoInformation() {
    // ...
  }
}

class CachedYouTube implements YouTubeAPI {
  constructor(
    private service: YouTubeAPI,
    private cachedList = null,
    private cachedVideoInformation = null
  ) {}

  listVideos() {
    if (!this.cachedList) {
      this.cachedList = this.service.listVideos();
    }
    return this.cachedList;
  }

  getVidoeInformation() {
    if (!this.cachedVideoInformation) {
      this.cachedVideoInformation = this.service.getVidoeInformation();
    }
    return this.cachedVideoInformation;
  }
}
```

### Facade

참고  
https://refactoring.guru/ko/design-patterns/facade  
https://lktprogrammer.tistory.com/42  
https://readystory.tistory.com/m/193

**서브시스템을 더 쉽게 사용할 수 있도록 higher-level 인터페이스를 정의하고, 제공하는 패턴이다.**

Facade는 건물의 정면이라는 뜻으로, Facade 객체는 어떤 소프트웨어의 다른 커다란 코드 부분에 대해 간략화된 인터페이스를 제공한다.  
라이브러리나 프레임워크(하위 시스템)에 있는 다양한 객체들의 집합이 클라이언트 코드에서 동작시키려면 이러한 모든 객체들을 초기화하고, 종속성 관계를 추적하고, 올바른 순서로 메서드를 실행해야 한다.  
이 때문에 클라이언트 코드의 비즈니스 로직이 하위 시스템에 종속적이게 될 가능성이 크다.  
또한 하위 시스템의 모든 기능이 필요하지 않을 때, 클라이언트에서 필요한 기능만 제공하게 만들 수 있다.

아래는 Facade 패턴 UML이다.

![Facade UML](https://user-images.githubusercontent.com/63287638/230818099-c69a55db-6070-4e5b-ae0f-8b61838c1507.PNG)

`Client`는 컴포넌트를 사용하는 곳이다.  
`Facade`는 하위 시스템 기능들의 특정 부분에 편리하게 접근할 수 있는 기능을 제공한다. `Facade`는 클라이언트의 요청을 어디로 보내야 하는지와 움직이는 모든 부품을 어떻게 작동해야 하는지 알고 있다.  
`Addtional Facade`는 `Facade`의 책임을 분산한다.  
`Subsystem`는 실제 `Client`가 원하는 요청을 동작하는 객체이다. 하지만 `Facade`를 이용하기 때문에 `Client`와 `Subsystem`은 서로를 모르는 상태이다.

다음과 같은 특징을 가지고 있다.

- 장점
  - 복잡한 하위 시스템에서 코드를 별도로 분리할 수 있다.
  - 하위 시스템을 업그레이드하거나 교체할 때 비용이 감소한다. 사용자는 Facade 객체의 구현만 바꾸면 되기 때문이다.
- 단점
  - Facade 객체의 책임이 과중해질 수 있다.

#### 코드 예시

동영상을 올리는 기능을 만들어야 한다고 하자.  
직접 인코딩 기능을 구현하기에는 복잡한데, 인코딩 기능을 가진 비디오 변환 라이브러리가 너무 거대한 기능을 가지고 있다고 하자.  
이때 인코딩 기능만을 제공하는 Facade 객체(`VideoConverter`)를 만들 수 있다.

```ts
// convert.ts

declare module Converter {
  type Codec = ''; // ...
  class VideoFile {
    /* ... */
  }
  class OggCompressionCodec {
    /* ... */
  }
  class MPEG4CompressionCodec {
    /* ... */
  }
  class BitrateReader {
    /* ... */
  }
  // class AudioMixer, class Something ...
}
```

```ts
// VideoConverter
type VideoExtension = 'mp4' | 'ogg';

class VideoConvertor {
  convert(filePath: string, format: VideoExtension = 'mp4'): File {
    const video = new VideoFile(filepath);
    const codec = getCodec(format);

    return new File(BitrateReader.convert(video, codec));
  }

  getCodec(format: VideoExtension): Codec {
    switch (format) {
      case 'ogg':
        return new OggCompressionCodec();
      case 'mp4':
      default:
        return new MPEG4CompressionCodec();
    }
  }
}
```

```ts
// Client
const converter = new VideoConverter();
const convertedVideo = converter.convert('test.ogg');
convertedVideo.save();
```

만약 위와 같이 `VideoConverter` 객체가 존재하지 않는다면, 모든 클라이언트는 `convert` 모듈을 import한 뒤 일부 기능만 사용하고, 그 구현을 반복해야 한다.

## 4. 행위 패턴

https://coding-factory.tistory.com/708  
https://www.devkuma.com/docs/design-pattern/structural/

> 메서드 호출을 실체화하는 것이다. 이 말은 메서드를 객체로 바꿀 수 있다는 의미이다.

행위 패턴은 객체 사이의 알고리즘이나 책임 분배에 관련된 패턴이다.  
한 객체가 혼자 수행할 수 없는 작업을 여러 개의 객체로 어떻게 분배하는지, 또 그렇게 하면서도 객체 사이의 결합도를 최소화하는 것에 중점을 둔다.

공통적으로 사용하는 '행위'를 추상화할 수도 있다.  
게임을 구현한다고 하자.  
게임 캐릭터는 장애물을 피하기 위해 jump를 할 수 있다.
하지만 재미 요소를 위해 특정 맵에서나 특정 적 캐릭터도 jump를 해야할 수 있다.  
이때 jump 명령어를 추상화한다면 jump에 대한 높이 등이 바뀔 때 모든 객체를 수정할 필요가 없이 jump 명령어만 바꾸면 된다.

### Command

참고  
https://www.youtube.com/watch?v=Q0Vfr6Ajk9U  
https://luv-n-interest.tistory.com/1089  
https://gmlwjd9405.github.io/2018/07/07/command-pattern.html  
https://velog.io/@newtownboy/%EB%94%94%EC%9E%90%EC%9D%B8%ED%8C%A8%ED%84%B4-%EC%BB%A4%EB%A7%A8%EB%93%9C%ED%8C%A8%ED%84%B4Command-Pattern

Command 패턴은 **요청을 객체로 캡슐화하여 서로 다른 사용자의 매개변수화, 요청 저장 또는 로깅, 연산의 취소를 지원하게 만드는 패턴**이다.

Command 패턴은 행위 패턴의 한 종류로서, 요구 사항을 객체로 캡슐화한다.  
실행될 기능을 캡슐화함으로써 재사용성이 높은 클래스를 설계하는 패턴으로 하나의 추상 클래스에 메서드를 만들어 각 명령이 들어오면 그에 맞는 서브 클래스가 선택되어 실행되는 패턴이다.  
(`Invoker` 내부적으로 모든 명령을 관리한 뒤 switch case 등으로 어떤 명령어를 실행해야 옳을지 판단하는 로직이 있을 수 있다. 다만 아래 예시 코드에서는 명령을 Client에서 직접 제어하도록 만들었다)
하나의 객체가 '행위'할 수 있는 내용이 많다면 사용해볼 수 있다.

![Command UML](https://user-images.githubusercontent.com/63287638/226109068-07165f8a-c018-4d11-b467-9b006fc0d267.png)

위 사진은 일반적인 Command 패턴의 UML이다.  
`Command` 는 실행될 기능을 `execute` 메서드로 선언한 인터페이스 또는 추상 클래스이다.  
`ConcreteCommand`는 `Command`를 구현한, 실제로 실행되는 기능을 나타내는 객체이다.  
`Invoker`는 `ConcreteCommand` 객체들의 `execute`를 요청하는 호출자이다.  
`Receiver`는 `ConcreteCommand`의 기능을 실행하기 위해 사용하는 수신자 클래스이다. `ConcreteCommand`의 내부 변수로서 저장되어 사용되기도 한다.

다음과 같은 특징이 있다.

- 장점
  - 객체 간 의존성(어떤 객체(A)에서 다른 객체(B)의 메서드를 실행하려면 객체(B)를 참조하고 있어야 하는)을 제거할 수 있다.
  - 기능이 수정되거나 변경이 일어날 때 A클래스의 코드를 수정없이 기능에 대한 클래스를 정의하면 되므로 시스템이 확장성이 있으면서 유연성을 가질 수 있다.
  - 커맨드 단위의 별도의 액션 등이 가능하고 커맨드 상속 및 조합을 통해 더 정교한 커맨드를 구현할 수 있다.
- 단점
  - 설계가 복잡하다.

#### 코드 예시

요새 스마트 TV는 되게 많은 일을 할 수 있다.  
기본적인 TV 기능, 인터넷 검색, 유튜브 시청, TV 연결 등등.  
이렇게 다양한 기능을 할 수 있는 객체에 Command 패턴을 적용시켜보자.

참고로 아래에 `Network`, `Connection` 각각 인터넷 연결, smartView 연결 등을 구현한 가상의 클래스이다.

처음에는 아무 패턴 없이 스마트 TV를 구현해보겠다.

```ts
class TV {
  display() {
    console.log('화면에 출력');
  }
}

class SmartTV extends TV {
  async searchThroughInternet(keyword: string) {
    const internet = await Network.connect().openBrowser();
    const searchResult = internet.search(keyword);

    this.display(searchResult);
  }

  async playYouTube() {
    const youtube = await Network.connect().openBrowser('https://youtube.com');

    this.display(youtube);
  }
}
```

위 `SmartTv` 객체를 보면 인터넷 연결 후 검색 결과 display, 유튜브 연결 후 검색 결과 display 등 하나의 객체가 하는 일이 너무 많다.  
smartView 연결을 구현하면 해당 객체가 책임지는 영역이 더 많아질 것이다.  
단일 책임 원칙에 위배되기 때문에 인터넷 연결, 유튜브 연결 등은 다른 객체에게 위임하도록 해보자.

```ts
// TV 클래스는 동일

class BrowserGUI {
  // ...
}

class SearchEngine {
  async searchKeyword(keyword: string): BrowserGUI {
    const internet = await Network.connect().openBrowser();
    const searchResult = internet.search(keyword);

    return searchResult;
  }
}

class YoutubePlayer {
  async turnOn(): BrowserGUI {
    const youtube = await Network.connect().openBrowser('https://youtube.com');

    return youtube;
  }
}

class SmartTV extends TV {
  constructor(private searchEngine, private youtubePlayer) {}

  async searchThroughInternet(keyword: string) {
    const searchResult = await this.searchEngine.searchKeyword(keyword);

    this.display(searchResult);
  }

  async playYouTube() {
    const youtube = await this.yotubePlayer.turnOn();

    this.display(youtube);
  }
}
```

만약 여기서 스마트폰 연결까지 진행한다면?  
별도의 클래스를 선언할 뿐만 아니라 `SmartTV`의 생성자, 내부 변수 수정 그리고 추가적인 메서드 구현까지 해야 한다.  
변경될 사항이 너무 많다는 뜻이다.  
다음 코드는 이러한 문제를 Command 패턴을 이용해 해결한 코드이다.

```ts
// Command
// GUI는 Command 패턴과 직접적인 관련이 없음

abstract class GUI {
  // ...
}

class BrowserGUI extends GUI {
  // ...
}

class SmartPhoneGUI extends GUI {
  // ...
}

interface Command {
  execute: (args: any) => Promise<GUI>;
}

class SearchEngine implements Command {
  async execute(keyword: string): BrowserGUI {
    const internet = await Network.connect().openBrowser();
    const searchResult = internet.search(keyword);

    return searchResult.view();
  }
}

class YoutubePlayer implements Command {
  async execute(): BrowserGUI {
    const youtube = await Network.connect().openBrowser('https://youtube.com');

    return youtube.view();
  }
}

class SmartPhoneConnector implements Command {
  async execute(smartViewId: string): SmartPhoneGUI {
    const smartphone = await Connection.connectSmartPhone(smartViewId);

    return smartphone.view();
  }
}
```

```ts
// Invoker

class TV {
  display() {
    // 화면에 출력하는 로직
  }
}

class SmartTV extends TV {
  private command!: Command;

  setCommand(command: Command) {
    this.command = command;
  }

  async display(args: any) {
    const view = await this.command.execute(args);

    super.display(view);
  }
}
```

```ts
// Client

(async function () {
  const smartTv = new SmartTv();

  // 무언가 검색하고 싶다면
  smartTv.setCommand(new SearchEngine());
  await smartTv.display('github.com/mochang2');

  // 유튜브를 보고 싶다면
  smartTv.setCommand(new YoutubePlayer());
  await smartTv.display();

  // 스마트폰을 연결하고 싶다면
  smartTv.setCommand(new SmartPhoneConnector());
  await smartTv.display();
})();
```

### Iterator

참고  
https://velog.io/@cham/Design-Pattern-%EC%9D%B4%ED%84%B0%EB%A0%88%EC%9D%B4%ED%84%B0-%ED%8C%A8%ED%84%B4-iterator-pattern  
https://flower0.tistory.com/446  
https://refactoring.guru/ko/design-patterns/iterator

**객체를 저장하는 방식(컬렉션)은 보여주지 않으면서도 클라이언트가 객체들에게 일일이 접근할 수 있게 해주는 방법이다.**  
이 패턴의 구현 방법은 반복 작업을 `Iterator`를 이용해 캡슐화하는 것이다.

컬렉션은 데이터를 효율적으로 저장하기만 하면 되는데, 순회 알고리즘을 추가할수록 책임이 비대해지는 문제가 발생한다.  
또는 여러 컬렉션을 동시에 다루는 작업이 필요할 수 있다.  
예를 들어 내부 객체를 `Tree`로 저장한 A 객체와 `Stack`으로 저장한 B 객체가 있다고 하자.  
그리고 `Client`가 해당 A, B의 내부 객체를 동시에 순회해야 할 일이 있다고 하면, `Client`는 `Tree`, `Stack`을 각각 순회하는 코드와 A, B가 내부적으로 어떻게 구현되어 있는지 알고 있어야 한다.  
하지만 `Tree`를 순회하는 그리고, `Stack`을 순회하는 `Iterator`를 공통적으로 선언해서 사용한다면 `Client`는 `Iterator`만 알고 있으면 된다.

![Iterator UML](https://user-images.githubusercontent.com/63287638/227749570-e2dc8cff-31c2-4e0d-bfb8-2e9a338b5a36.png)

위 사진은 일반적인 Iterator 패턴의 UML이다.  
`Iterator`는 컬렉셔의 순회에 필요한 작업들을 선언하는 인터페이스 혹은 가상 클래스이다.  
`ConcreteIterator`는 컬렉션 순회를 위한 특정 알고리즘들을 구현한다. 내부적으로 순회의 진행 상황을 추적함(몇 번째까지 돌았는지 등)으로써 여러 `Iterator`들이 같은 컬렉션을 독립적으로 순회할 수 있어야 한다.  
`IterableCollection`은 컬렉션과 호환되는 `Iterator`들을 가져오기 위한 하나 이상의 메서드들을 선언한다.  
`ConcreteCollection`은 내부 객체를 다루는 역할을 할 뿐만 아니라 `Iterator`를 반환하는 메서드도 존재한다.

Iterator 패턴은 다음과 같은 특징이 있다.

- 장점
  - 컬렉션을 서로 다른 방식으로 다루는 객체들을 동시에 다룰 수 있다.
  - `Iterator` 객체는 알고리즘 자체를 구현하는 것 외에도 모든 순회 세부 정보들(예: 현재 위치 및 남은 요소들의 수)을 캡슐화하며, 이 때문에 여러 `Iterator`들이 서로 독립적으로 동시에 같은 컬렉션을 통과할 수 있다.
  - 순회를 지연하고 필요할 때 계속할 수 있다.
  - Factory 패턴을 함께 사용하여 컬렉션 자식 클래스들이 해당 컬렉션들과 호환되는 다양한 유형의 반복자들을 반환하도록 할 수 있다.
- 단점
  - 내장 컬렉션(JS의 `Array`, `Set` 등)과 같이 단순한 컬렉션들만 작동하는 경우 과도하게 코드가 늘어날 수 있다.
  - 일부 특수 컬렉션들은 `Iterator`로 순회하면 덜 효율적일 수도 있다.

#### 코드 예시

한식집 알촌과 일식집 기소야가 있다고 하자.  
알촌은 중복된 메뉴가 있어도 상관없다고 생각해 `Array`로 다루고 있고, 기소야는 메뉴가 중복되면 안 된다고 생각해 `Set`으로 다루고 있다.

```ts
// ConcreteCollection

interface Food {
  name: string;
  price: number;
  type: string;
}

type menuType = Food[] | Set<Food>; // 더 상위 collection으로 묶으면 좋겠지만 간단한 코드라 임시 방편으로 작성

interface Menu {
  addFood: (food: Food) => void;
  getMenu: () => menuType;
}

class KoreanMenu implements Menu {
  constructor(private menu: Food[] = []) {}

  addFood(food: Food) {
    this.menu.push(food);
  }

  getMenu() {
    return this.menu;
  }
}

class JapaneseMenu implements Menu {
  constructor(private menu: Set<Food> = new Set()) {}

  addFood(food: Food) {
    this.menu.add(food);
  }

  getMenu() {
    return this.menu;
  }
}
```

```ts
// Client

class AlchonKiosk {
  static foods: Food[] = [
    { name: '돼지국밥', price: 8000, type: 'korean' },
    { name: '해물파전', price: 18000, type: 'korean' },
    { name: '육회비빔밥', price: 9000, type: 'korean' },
  ];

  private menu!: Food[];

  constructor() {
    this.initializeMenu();
  }

  initializeMenu() {
    this.menu = [...AlchonKiosk.foods];
  }
}

class KisoyaKiosk {
  static foods: Food[] = [
    { name: '돼지국밥', price: 8000, type: 'korean' },
    { name: '해물파전', price: 18000, type: 'korean' },
    { name: '육회비빔밥', price: 9000, type: 'korean' },
  ];

  private menu!: Food[];

  constructor() {
    this.initializeMenu();
  }

  initializeMenu() {
    this.menu = new Set(KisoyaKiosk.foods);
  }
}
```

어느 날 갑자기 알촌이 너무 잘 나가서 기소야를 합병했다고 하자.  
기존까지 각자 개발해 판매했기 때문에 큰 문제는 없었지만 이제는 메뉴를 각각 다른 컬렉션에 담는 것이 문제가 된다.  
키오스크(손님들은 메뉴 주문을 위해 키오스크를 이용한다)는 갑자기 `Array`로 된 메뉴들도 나열해야 하고, `Set`으로 된 메뉴들도 나열해야 손님들이 올바르게 주문을 할 수 있게 된 상황이다.  
Iterator 패턴으로 상황을 타개해볼 수 있다.

```ts
// Iterator

// Iterator 인터페이스가 존재하므로 _ 추가
interface Iterator_<T> {
  getNext: () => T;
  hasMore: () => boolean;
}
```

```ts
// ConcreteIterator

class ArrayIterator<T> implements Iterator_<T> {
  constructor(private array: T[], private index = 0) {}

  getNext() {
    return this.array[this.index++];
  }

  hasMore() {
    return this.index >= this.array.length || !this.array[this.index]
      ? false
      : true;
  }
}

class SetIterator<T> implements Iterator_<T> {
  // graceful하지 못하지만 그냥 대충 그러려니 하자 ㅎㅎ...
  private array!: T[];

  constructor(set: Set<T>, private index = 0) {
    this.array = [...set];
  }

  getNext() {
    return this.array[this.index++];
  }

  hasMore() {
    return this.index >= this.array.length || !this.array[this.index]
      ? false
      : true;
  }
}
```

```ts
// IterableCollection, ConcreteCollection

interface Food {
  name: string;
  price: number;
  type: string;
}

type menuType = Food[] | Set<Food>; // 더 상위 collection으로 추상화하면 좋겠지만 간단한 코드라 임시 방편으로 작성

interface Menu {
  addFood: (food: Food) => void;
  getMenu: () => menuType;
  createIterator: () => Iterator_<Food>;
}

class KoreanMenu implements Menu {
  constructor(private menu: Food[] = []) {}

  addFood(food: Food) {
    this.menu.push(food);
  }

  getMenu() {
    return this.menu;
  }

  createIterator() {
    return new ArrayIterator<Food>(this.getMenu());
  }
}

class JapaneseMenu implements Menu {
  constructor(private menu: Set<Food> = new Set()) {}

  addFood(food: Food) {
    this.menu.add(food);
  }

  getMenu() {
    return this.menu;
  }

  createIterator() {
    return new SetIterator<Food>(this.getMenu());
  }
}
```

```ts
// Client

class Kiosk {
  static foods: Food[] = [
    { name: '돼지국밥', price: 8000, type: '한식' },
    { name: '해물파전', price: 18000, type: '한식' },
    { name: '육회비빔밥', price: 9000, type: '한식' },
    { name: '카레', price: 8000, type: '일식' },
    { name: '초밥', price: 18000, type: '일식' },
    { name: '돈까스', price: 10000, type: '일식' },
  ];

  private menus!: Menu[];

  constructor() {
    this.initializeMenu();
  }

  private initializeMenu() {
    const alchonMenu = new KoreanMenu();
    const koreanFoods = Kiosk.foods.filter((food) => food.type === '한식');
    koreanFoods.forEach((food) => alchonMenu.addFood(food));

    const kisoyaMenu = new JapaneseMenu();
    const japaneseFoods = Kiosk.foods.filter((food) => food.type === '일식');
    japaneseFoods.forEach((food) => kisoyaMenu.addFood(food));

    this.menus = [alchonMenu, kisoyaMenu];
  }

  displayMenu() {
    this.menus.forEach((menu) => {
      const iterator = menu.createIterator();

      this.iterate(iterator);
    });
  }

  private iterate(iterator: Iterator_<Food>) {
    while (iterator.hasMore()) {
      const food: Food = iterator.getNext();

      console.log(`음식: ${food.name} - ${food.price}원, ${food.type}`);
    }
  }
}

const kiosk = new Kiosk();
kiosk.displayMenu();
// [LOG]: "음식: 돼지국밥 - 8000원, 한식"
// [LOG]: "음식: 해물파전 - 18000원, 한식"
// [LOG]: "음식: 육회비빔밥 - 9000원, 한식"
// [LOG]: "음식: 카레 - 8000원, 일식"
// [LOG]: "음식: 초밥 - 18000원, 일식"
// [LOG]: "음식: 돈까스 - 10000원, 일식"
```

### Mediator

참고  
https://refactoring.guru/ko/design-patterns/mediator  
https://velog.io/@cham/Design-Pattern-%EC%A4%91%EC%9E%AC%EC%9E%90-%ED%8C%A8%ED%84%B4Mediator-Pattern  
https://brownbears.tistory.com/568

**클래스 간의 복잡한 관계들을 캡슐화하여 하나의 클래스에서 관리하도록 처리하는 패턴이다.**  
Mediator 패턴을 이용하면 객체 간의 혼란스러운 의존 관계를 줄일 수 있다.  
M개의 객체 사이의 서로 데이터를 주고받을 일이 있을 경우, 최대 M(M + 1) / 2개의 관계가 생긴다.  
이때 `Mediator` 객체만 하나 추가한다면 이를 다시 M개로 줄일 수 있다.  
Mediator 패턴은 M:N 관계를 M:1 관계로 만듦으로써 복잡도를 내리므로 유지 보수 및 확장성에 유리하다.

기존 패턴이 아래와 같은 관계를 가지고 있다고 하자.

![Mediator 패턴 적용 이전](https://user-images.githubusercontent.com/63287638/229271366-ac2c3f82-a5ce-494e-915f-5144f5cb368f.jpg)

이에 Mediator 패턴을 적용하면 아래와 같은 관계로 변경될 수 있다.  
만약 `Button`이나 `TextField` 등의 공통 요소도 추상화가 가능하다면 오른쪽 사진과 같이 더 단순화하여 표현할 수 있다.

![Mediator 패턴 적용 이후](https://user-images.githubusercontent.com/63287638/229271369-07ef9a07-3839-4161-a853-3e44ab2df366.jpg)

아래는 Mediator 패턴 UML이다.  
다른 패턴들에 비해 꽤나 심플하다.

![Mediator UML](https://user-images.githubusercontent.com/63287638/229271206-a9c48633-4c42-43df-9bf5-1c0667792c40.jpg)

`Mediator`는 `Colleague` 객체 간의 상호 참조를 위한 인터페이스이다. 클라이언트 등록, 실행 등의 메소드를 정의한다.  
`ConcreteMediator`는 `Mediator` 구현 클래스이다. `Colleague` 간의 상호 참조, 데이터 전달 등의 역할을 한다.  
`Colleague`는 다른 `Colleague`와의 상호 참조를 위한 인터페이스이다.  
`ConcreteColleage`는 `Colleague` 구현 클래스이다. `Mediator`를 통해 다른 `Colleague`와의 상호 참조가 가능하다.

간단하게만 다른 패턴과 비교해보겠다.  
Facade 패턴: 통신을 위해 인터페이스를 설계하고 제공한다는 점에서 두 패턴은 동일하지만 Facade는 단방향 통신만, Mediator는 단방향 통신뿐만 아니라 양방향 통신 또한 가능하다.  
Observer 패턴: 대부분의 경우 두 패턴 중 하나를 구현할 수 있으나, 때로는 두 패턴을 동시에 적용할 수 있다. 예를 들어 Mediator 객체는 publisher의 역할을 맡고, 컴포넌트들은 Mediator의 이벤트들을 구독 및 구독 취소하는 구독자들의 역할을 맡으면 Observer 패턴과 매우 유사해질 수 있다.

- 장점
  - 효율적인 자원 관리(리소스 풀 등)를 가능하게 한다.
  - 객체간의 통신을 위해 서로간에 직접 참조할 필요가 없다(느슨한 결합).
  - 객체들 간 수정을 하지 않고 관계를 수정할 수 있다.
- 단점
  - 특정 application 로직에 맞춰져있기 때문에 다른 application에 재사용하기 힘들다.
  - Mediaotr 객체에 권한이 집중화되어 굉장히 복잡질 수 있다.

#### 코드 예시

채팅 프로그램은 일종의 Mediator 패턴의 구현체로 볼 수 있다.  
M명의 사람들이 1:1 채팅 프로그램에 참여하고 있을 때, Mediator가 없다면 M명의 사람들은 각각 M-1개의 연결을 관리해야 하며, 만약 각자의 권한 등이 다르다면 이에 대한 처리도 각자 해야 한다.  
그리고 일반 유저뿐만 아니라 모든 유저들에게 메시지를 보낼 수 있지만 본인은 메시지를 받지 않고, 특정 유저만 집어서도 메시지를 보낼 수 있는 관리자도 존재한다고 해보자.  
출발지, 도착지, 권한 확인, 송신, 수신, 연결 관리 등 각각의 객체들이 하는 책임이 많아진다.  
하지만 여기에 `Mediator`가 존재하면 권한 확인, 연결 관리 등의 책임을 분산할 수 있다.

```ts
// ConcreteColleage

interface ChatUser {
  name?: string;
  send(message: string, to: ChatUser): void;
  receive(message: string, from: ChatUser): void;
}

class User implements ChatUser {
  private mediator: ChatMediator;
  public name: string;

  constructor(name: string, mediator: ChatMediator) {
    this.name = name;
    this.mediator = mediator;
  }

  send(message: string, to: ChatUser) {
    console.log(`${this.name} sends "${message}" to ${to.name}`);
    this.mediator.send(message, this, to);
  }

  receive(message: string, from: ChatUser) {
    console.log(`${this.name} received "${message}" from ${from.name}`);
  }
}

class Admin implements ChatUser {
  private mediator: ChatMediator;

  constructor(mediator: ChatMediator) {
    this.mediator = mediator;
  }

  send(message: string, to: ChatUser) {
    console.log(`Admin sends "${message}" to ${to.name}`);
    this.mediator.send(message, this, to);
  }

  receive(message: string, from: ChatUser) {
    // Admin does not receive messages
  }

  sendToAll(message: string) {
    console.log(`Admin sends "${message}" to all users`);
    this.mediator.sendToAll(message, this);
  }

  sendToSome(message: string, users: ChatUser[]) {
    console.log(
      `Admin sends "${message}" to ${users.map((u) => u.name).join(', ')}`
    );
    this.mediator.sendToSome(message, this, users);
  }
}
```

```ts
// Mediator

class ChatMediator {
  private users: ChatUser[];

  constructor() {
    this.users = [];
  }

  addUser(user: ChatUser) {
    this.users.push(user);
  }

  send(message: string, from: ChatUser, to: ChatUser) {
    if (to instanceof Admin) {
      console.log(`Cannot send message to admin`);
      return;
    }
    to.receive(message, from);
  }

  sendToAll(message: string, from: ChatUser) {
    this.users
      .filter((u) => u !== from)
      .forEach((u) => u.receive(message, from));
  }

  sendToSome(message: string, from: ChatUser, users: ChatUser[]) {
    users
      .filter((u) => u !== from && !(u instanceof Admin))
      .forEach((u) => u.receive(message, from));
  }
}
```

```ts
// Client

const mediator = new ChatMediator();
const user1 = new User('Alice', mediator);
const user2 = new User('Bob', mediator);
const user3 = new User('Charlie', mediator);
const admin = new Admin(mediator);

// Add the objects to the mediator
mediator.addUser(user1);
mediator.addUser(user2);
mediator.addUser(user3);
mediator.addUser(admin);

// User 1 sends messages to user 2
user1.send('Hi Bob. How are you? Do you want to grab lunch?', user2);

// User 2 sends messages to user 3
user2.send('Hey Charlie. Want to go to the movies tonight?', user3);

// User 3 sends messages to user 1
user3.send("Hi Alice, long time no see. Sure, let's do lunch tomorrow", user1);

// Admin sends messages to all users
admin.sendToAll(
  'Important announcement: the server will be down for maintenance tonight'
);
```

### Observer

참고  
https://velog.io/@haero_kim/%EC%98%B5%EC%A0%80%EB%B2%84-%ED%8C%A8%ED%84%B4-%EA%B0%9C%EB%85%90-%EB%96%A0%EB%A8%B9%EC%97%AC%EB%93%9C%EB%A6%BD%EB%8B%88%EB%8B%A4  
https://velog.io/@hanna2100/%EB%94%94%EC%9E%90%EC%9D%B8%ED%8C%A8%ED%84%B4-2.-%EC%98%B5%EC%A0%80%EB%B2%84-%ED%8C%A8%ED%84%B4-%EA%B0%9C%EB%85%90%EA%B3%BC-%EC%98%88%EC%A0%9C-observer-pattern  
https://gyoogle.dev/blog/design-pattern/Observer%20Pattern.html

**객체 간의 결합도를 낮추기 위한 패턴이다.**
publisher / subscriber 패턴이라고도 불린다.

어떠한 객체의 상태가 변경될 때마다 다른 객체들도 변하거나, 특정 동작을 유도하고 싶을 때 보통 사용된다.  
observer는 subject의 변화 상태를 구독하다가 변화가 발생되었다는 이벤트를 받으면 등록된 동작을 수행한다.

다음과 같은 특징이 있다.

- 장점
  - 실시간으로 한 객체의 변경 사항을 다른 객체에 전파할 수 있다.
  - 느슨한 결합으로 subject, observer를 독립적으로 사용할 수 있다. 이로 인해 언제든지 observer를 새로 추가, 삭제가 가능하며 변경되어도 서로에게 영향을 미치지 않을 수 있다.
- 단점
  - 상태 관리가 힘들 수도 있다.
  - 데이터 배분을 잘 해야 한다.

JS의 `addEventListener`가 이러한 패턴을 응용했다고 볼 수 있다.  
콜백을 전달함으로써 observer 패턴을 구현할 수도 있지만 다음 예제에서는 subject가 observer의 주소값을 내부적으로 가지고 있는 형태로 구현했다.

#### 코드 예시

날씨 정보를 다루는 클래스가 있다고 가정하자.

```javascript
class WeatherController {
  measureChanges() {
    const temperature = getTemperature();
    const humidity = getHumidity();
    const pressure = getPressure();

    currentWeatherView.update(temperature, humidity, pressure);
    forecastWeatherView.update(temperature, humidity, pressure);
    staticsticView.update(temperature, humidity, pressure);
  }
}
```

만약 위의 코드에서 풍향과 같은 새로운 정보를 추가하고자 하면 `measureChanges` 메서드 또한 변경해야 하는 불편함이 존재한다.

observer 패턴은 주로 interface를 많이 사용하지만 JS는 없는 문법이니 base 클래스로 강제화했다.

```javascript
const ERROR = '상속받은 클래스는 해당 메서드를 구현해야 합니다.';

class ViewController {
  registerView(view) {
    throw new Error(ERROR);
  }

  removeView(view) {
    throw new Error(ERROR);
  }

  updateViews() {
    throw new Error(ERROR);
  }
}

class View {
  update({ temperature, humidity, pressure }) {
    throw new Error(ERROR);
  }
}
```

```javascript
class WeatherController extends ViewController {
  constructor() {
    super();

    this.views = [];
  }

  registerView(view) {
    // view 추가
  }

  removeView(view) {
    // view 삭제
  }

  updateViews() {
    // this.views 순회하며 update 호출
  }
}

class CurrentWeatherView extends View {
  constructor(controller) {
    super();

    controller.registerView(this);
  }

  update({ temperature, humidity, pressure }) {
    // 화면 업데이트
  }
}
```
