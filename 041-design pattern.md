## 0. 공부하게 된 계기

[클린 코드](https://github.com/mochang2/development-diary/blob/main/040-clean%20code%20js.md)를 공부하며 디자인 패턴에 대해서도 익숙해질 필요가 있다고 생각했다.  
~필기 테스트에서도 나왔고 면접에서도 공격받았던 질문...~

모든 디자인 패턴을 다 작성한다기 보다는 JS에서 주로 사용되는 패턴을 먼저 정리하고자 한다(이후 방향성이 바뀔 수도 있다).

목차  
[Builter or Constructor](#builder-or-constructor)
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
https://joshua1988.github.io/web-development/javascript/javascript-pattern-design/#%ED%8C%A9%ED%86%A0%EB%A6%AC-%ED%8C%A8%ED%84%B4  
https://sangcho.tistory.com/entry/%EC%9B%B9%EA%B0%9C%EB%B0%9C%EC%9E%90%EA%B0%80%EC%95%8C%EC%95%84%EC%95%BC%ED%95%A07%EA%B0%80%EC%A7%80%EB%94%94%EC%9E%90%EC%9D%B8%ED%8C%A8%ED%84%B4  
https://im-developer.tistory.com/141  
https://www.devh.kr/2021/Design-Patterns-In-JavaScript/  
https://gyoogle.dev/blog/design-pattern/Singleton%20Pattern.html

## 1. 디자인 패턴이란

> 간단하게 말해서 디자인 패턴은 설계자들이 "올바른" 설계를 "빨리" 만들 수 있도록 도와줍니다.

디자인 패턴은 선배들의 경험이 담긴 문제 해결 방법이다.  
어떤 문제나 수정 사항이 발생했을 때, 하나하나 시행착오를 겪으면서 다시 짓기에는 시간과 비용이 많이 든다.  
시행착오를 바탕으로 특정 상황에서 발생하는 문제 패턴을 발견하고 적용한 것을 **디자인 패턴**이라고 부른다.

디자인 패턴은 설계자로 하여금 재사용이 가능한 설계는 선택하고, 재사용을 방해하는 설계는 배제하도록 도와준다.  
또한 패턴을 쓰면 이미 만든 시스템의 유지보수나 문서화도 개선할 수 있고, 클래스의 명세도 정확하게 할 수 있으며, 객체 간의 상호작용 또는 설계 의도까지 명확하게 정의할 수 있다.

## 2. 생성 패턴

### Builder or Constructor

### Factory

### Singleton

한 클래스 안에 하나의 객체만 유지하도록 하여 race condition 문제를 해결하기 위한 패턴이다.

생성자가 여러 번 호출되어도 실제로 생성되는 객체는 하나이며 최초로 생성된 이후에 호출된 생성자는 이미 생성한 객체를 반환하도록 만든다.  
다음과 같은 특징이 있다.

- 장점
  - 메모리 낭비를 방지할 수 있다.
  - 클래스 인스턴스 간 데이터 공유가 가능하다.
- 단점
  - 혼자 너무 많은 일을 하거나 많은 데이터를 공유하면 개방-폐쇄 원칙에 위배된다.
  - 결합도가 높아지면 유지 보수나 테스트가 힘들어진다.

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

console.log(userModule.create('Julia', 'hello123'))
console.log(userModule.create('Julia', 'hello123'))
console.log(userModule.create('Julia', 'hello123'))
console.log(userModule.create('Paul', 'hello456'))

console.log(userModule.get('Julia'))
console.log(userModule.get('Paul'))
console.log(userModule.get('Mike'))
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
