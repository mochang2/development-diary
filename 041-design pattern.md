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

## 2. 생성 패턴

생성 패턴은 **인스턴스 만드는 절차를 추상화**하는 패턴이다.  
생성 패턴에 속하는 패턴들은 객체를 생성, 합성하는 방법이나 객체의 표현 방법을 시스템과 분리한다.  
생성 패턴을 이용하면 무엇이 생성되고, 누가 이것을 생성하며, 이것이 어떻게 생성되는지, 언제 생성할 것인지 결정하는 데 유연성을 확보할 수 있게 된다.

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
  #date
  #accomodations
  // ...

  constructor(builder) {
    this.#date = builder.getDate()
    this.#accomodations = builder.getAccomodations()
    // ...
  }

  static TourPlanBuilder = class {
    #date
    #accomodations
    // ...

    constructor({ date }) {
      this.#date = date
      this.#accomodations = null
      // ...
    }

    getDate() {
      return this.#date
    }

    getAccomodations() {
      return this.#accomodations
    }

    setAccomodations(accomodations) {
      // 검증 코드, 초기화 코드 추가할 수 있음
      this.#accomodations = accomodations

      return this
    }

    // ...

    build() {
      return new TourPlan(this)
    }
  }
}
```

```javascript
// 당일치기라면
const tourplan1 = new TourPlan.TourPlanBuilder({ date: '2022-12-31' }).build()

// 당일치기가 아니라면
const tourplan2 = new TourPlan.TourPlanBuilder({ date: '2022-12-31' })
  .setAccomodations('두바이 완전 비싼 7성급 호텔')
  .build()
```

### Factory

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
  var userAddress = {}
  var users = []
  var userId = 0

  return {
    create: (username, password) => {
      if (!userAddress.hasOwnProperty(username)) {
        var user = { id: userId, username, password }
        userAddress[username] = users.length + ''
        users.push(user)
        userId++

        return user
      } else {
        return users[userAddress[username]]
      }
    },
    get: (username) => {
      return userAddress[username] ? users[userAddress[username]] : null
    },
  }
})()

console.log(userModule.create('Julia', 'hello123')) // { id: 0, username: 'Julia', password: 'hello123' }
console.log(userModule.create('Julia', 'hello123')) // { id: 0, username: 'Julia', password: 'hello123' }
console.log(userModule.create('Julia', 'hello123')) // { id: 0, username: 'Julia', password: 'hello123' }
console.log(userModule.create('Paul', 'hello456')) // { id: 1, username: 'Paul', password: 'hello456' }

console.log(userModule.get('Julia')) // { id: 0, username: 'Julia', password: 'hello123' }
console.log(userModule.get('Paul')) // { id: 1, username: 'Paul', password: 'hello456' }
console.log(userModule.get('Mike')) // null
```

ES7 이후에는 `static`이라는 너무 간단한 명령어가 생겼다.

```javascript
class UserModule {
  static userModule

  constructor() {
    if (userModule) {
      return userModule
    }

    // 없으면 다른 동작
  }
}
```

### Prototype

## 3. 구조 패턴

### Composite

### Decorator

### Proxy

## 4. 행위 패턴

### Command

### Iterator

### Mediator

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

JS의 `addEventlistener`가 이러한 패턴을 응용했다고 볼 수 있다.  
콜백을 전달함으로써 observer 패턴을 구현할 수도 있지만 다음 예제에서는 subject가 observer의 주소값을 내부적으로 가지고 있는 형태로 구현했다.\

#### 코드 예시

날씨 정보를 다루는 클래스가 있다고 가정하자.

```javascript
class WeatherController {
  measureChanges() {
    const temperature = getTemperature()
    const humidity = getHumidity()
    const pressure = getPressure()

    currentWeatherView.update(temp, humidity, pressure)
    forecastWeatherView.update(temp, humidity, pressure)
    staticsticView.update(temp, humidity, pressure)
  }
}
```

만약 위의 코드에서 풍향과 같은 새로운 정보를 추가하고자 하면 `measureChanges` 메서드 또한 변경해야 하는 불편함이 존재한다.

observer 패턴은 주로 interface를 많이 사용하지만 JS는 없는 문법이니 base 클래스로 강제화했다.

```javascript
const ERROR = '상속받은 클래스는 해당 메서드를 구현해야 합니다.'

class ViewController {
  registerView(view) {
    throw new Error(ERROR)
  }

  removeView(view) {
    throw new Error(ERROR)
  }

  updateViews() {
    throw new Error(ERROR)
  }
}

class View {
  update({ temperature, humidity, pressure }) {
    throw new Error(ERROR)
  }
}
```

```javascript
class WeatherController extends ViewController {
  constructor() {
    super()

    this.views = []
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
    super()

    controller.registerView(this)
  }

  update({ temperature, humidity, pressure }) {
    // 화면 업데이트
  }
}
```