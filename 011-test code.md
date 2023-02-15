## 0. 공부하게 된 계기

토이 프로젝트할 때와 달리 회사 프로젝트는 상시 켜두기 때문에, 그리고 누구일지 모르는 사람들이 다수가 사용하기 때문에  
어떤 일이 벌어질지 예측을 할 수가 없다.  
최대한 모든 상황을 담아서 테스트를 해야되고 프로젝트가 커질수록 미들웨어 간의 관계 등에 의해서 예상치 못한 부분에서 에러가 발생할 수 있다.  
따라서 테스트 코드를 작성해야 돼서 공부를 시작했다.  
~그런데... 프로젝트가 무기한 잠정중단이 됐다... 아마 부하테스트 정도만 하고 그 이상의 테스트 코드는 안 짤 듯 싶다~
https://ko.myservername.com/differences-between-unit-testing 와 https://tecoble.techcourse.co.kr/post/2021-05-25-unit-test-vs-integration-test-vs-acceptance-test/ 를 참고했다.

## 1. 필요성

### 실시간 코드 반영

테스트 코드는 방금 작성한 코드를 당장 테스트 하기 위해서만 작성되는 것이 아니다.  
이후 다른 변경 사항으로 인해 발생 가능한 결함을 찾아내는 역할을 한다.

### 자동 테스트 도구

직접 손으로 테스트하는 것보다 명령어 한 줄(jest test 등)으로 자동화시키는 것이 더 안정적이고 실수가 줄어든다.

### 테스트 작성 시간

물론 처음에는 오래 걸리겠지만, 프로젝트가 커질수록 해당 시간은 보잘 것 없게 된다.  
즉, 투자할 가치가 있는 시간인 것이다.

### QA 프로세스

QA만으로 모든 테스트를 커버할 수 없고, 개발자 입장에서 테스트를 하고 넘겨줘야 QA 프로세스가 감소할 수 있다.

### 클라이언트 테스트

UI 테스트가 쉽지는 않지만 불가능한 부분도 아니다.  
서버에 대한 테스트가 중요하겠지만 클라이언트 테스트도 무시할 부분이 아니다.

## 2. 테스트 종류

### 단위 테스트(Unit Test)

개발한 **모듈**(프로그램의 기본 단위)이 의도대로 동작하는가에 초점이 맞춰져 있다. 일반적으로 Class나 Method 범위로 테스트를 진행한다.  
의존성이 있는 코드와 함께 테스트하는 Sociable 테스트(자식 컴포넌트까지 포함해서 렌더링)와 모듈에 의해 실행되는 코드를 테스트 더블로 대체하는 Solitary 테스트(자식 컴포넌트를 mocking해서 렌더링)가 있다.  
단위 크기가 작을수록 복잡성이 낮아지므로 동작을 표현하기 더 쉬워진다.

+) 테스트 더블  
테스트를 진행하기 어려운 경우 이를 대신해 테스트를 진행할 수 있도록 만들어주는 객체.

### 통합 테스트(Integration Test)

**단위 테스트가 끝난 모듈과 외부 라이브러리 또는 DB와 같이 개발자가 변경할 수 없는 모듈까지 함께** 진행되는 테스트로 모듈 간 상호작용이 정상적으로 수행되는지에 초점이 맞춰져 있다.  
통합 테스트를 통해 단위 테스트에서 발견하기 어려운 버그를 찾을 수 있다.

### 인수 테스트(Acceptance Test)

사용자 스토리(시나리오)에 맞춰 수행하는 테스트이다.  
앞선 두 테스트들과 달리 비즈니스 쪽에 초점을 둔다.  
프로젝트에 참여하는 사람들(기획자, 클라이언트 대표, 개발자 등)이 토의해서 시나리오를 만들고 개발자는 이에 의거해서 코드를 작성한다.  
개발자 혼자서 직접 시나리오를 제작할 수 있지만 고객 관점 측면에서 놓치는 부분이 생길 수 있다.  
따라서 직접 고객과 대면하는 팀으로부터 시나리오와 피드백을 받아 개발할 수 있는 테스트이다.

핸드폰의 주요 부품인 배터리와 SIM 카드에 대해 테스트한다고 가정하면  
단위 테스트는 배터리의 수명, 용량 및 기타 매개 변수를 확인하며 SIM 카드가 활성화되었는지 확인하는 것이다.  
통합 테스트는 배터리와 SIM 카드가 일체화되어 휴대폰을 시작하기 위해 조립하는 것이다.  
기능 테스트는 휴대폰의 기능 및 배터리 사용량은 물론 SIM 카드 설비 등을 확인하는 것이다.

_참고)_  
유닛 -> E2E로 갈 수록 테스트의 단위 크기가 커지고 테스트를 위한 비용이 증가한다.  
Google Test Automation Conference에서는 테스트의 비율로서 E2E는 10%, 통합은 20%, 유닛은 70%로 테스트 비율을 제안했다.

~꼭 따라야 할 필요는 없을 것 같지만,~  
TDD는 애자일처럼 다음 순서를 무한 반복하는 개발 주기를 가진다.

1. Write a failing test: 실패하는 테스트를 만들어라. 통과해야만 하는 예외를 추가하는 작업이다.
2. Make the test pass: 실패하던 테스트를 성공하도록 만들어라. 포인트는 코드의 퀄리티는 신경쓰지 않고 빠르게 기능만 하도록 만드는 것이다.
3. Refactor: 개발된 코드들에 대해 모든 중복을 제거하고 추가적인 정리를 진행한다. 다시 1로 돌아간다.

## 3. FIRST 단위 테스트 원칙

단위 테스트는 가장 작은 단위의 테스트이며, 모든 테스트의 시작점이다.  
FIRST는 Fast, Isolated, Repeatable, Self-validating, Timely의 약자이다.

- Fast: 유닛 테스트는 빨라야 한다.
- Isolated: 다른 테스트에 종속적인 테스트는 절대로 작성하지 않는다.
- Repeatable: 테스트는 실행할 때마다 같은 결과를 만들어야 한다.
- Seft-validating: 테스트는 스스로 결과물이 옳은지 그른지 판단할 수 있어야 한다. 특정 상태를 수동으로 미리 만들어야 동작하는 테스트 등은 작성하지 않는다.
- Timely: 유닛 테스트는 프로덕션 코드가 테스트를 성공하기 직전에 구성되어야 한다. Test Driven Development(TDD) 방법론에 적합한 원칙이지만 실제로 적용되지 않는 경우도 있다.

## 4. jest

### node.js 테스트 툴 비교

<img src="https://user-images.githubusercontent.com/63287638/182154642-c447f412-c782-4027-9f91-0ee8b1ffa434.png" alt="node.js 테스트 툴 비교" width="75%" height="auto" />

### 설치

배포할 때는 필요없는 패키지이므로 개발용으로 설치한다.  
이에 예외는 없을 것 같다.

```sh
npm install -D jest
```

### 실행 명령어

package.json에 "test"를 "jest"로 수정한다.  
npm test로 jest를 실행할 수 있다.  
global 옵션을 줘서 npx로 실행할 수도 있지만 일반적으로는 이렇게 많이 실행한다.

```json
"scripts: {
	"test": "jest"
}
```

### test

```typescript
test('description), () => {
	expect(어떤 행위).toBe(기댓값)
})
```

위와 같은 방식이 기본 형식이다.

### describe

여러 테스트들을 하나의 그룹으로 묶는데 사용한다.  
아래는 그 예시다.

```typescript
describe('/', () => {
  test('/readiness :GET', async () => {
    const { status } = await request.get('/healthz/readiness')

    expect(status).toEqual(200)
  })

  test('/liveness :GET', async () => {
    const { status } = await request.get('/healthz/liveness')

    expect(status).toEqual(200)
  })
})
```

### beforeAll, afterAll, beforeEach, afterEach

각각은 모든 테스트 전,  
모든 테스트 후,  
describe로 묶인 각각의 테스트 전에,  
describe로 묶인 각각의 테스트 후에  
실행하고자 있는 것이 있을 때 사용한다.  
아래와 같이 사용할 수 있다.

```typescript
beforeEach(() => {
	console.log('123')
})

afterAll(() => {
	sequelize.sync(force: true)
}
```

### Matcher

1. toEqual() : \~\~와 같은 값을 기대한다는 뜻으로 toBe()로는 같지 않다고 뜨는데 toEqual()는 같다고 나오는 경우가 있다. 특히 객체와 같은 경우가 그렇다. 따라서 특별한 경우가 아니면 toEqual()를 사용하기를 권고한다.
2. toBeTruthy(), toBeFalsy(): 이름에서 알 수 있듯이 boolean 값을 return하기를 기대하고 쓰는 matcher이다. js와 같은 경우에는 1이 true, 0이 false 간주한다.
3. toHaveLength(n): 배열의 길이를 체크한다.
4. toContain('Example'): 특정 원소가 배열에 들어있는지를 테스트할 때 쓰인다.
5. toMatch('문자열 또는 정규표현식'): 문자열과 일치하는지 또는 정규표현식에 일치하는지 쓰인다.
6. toThrow(argument?): 함수는 인자도 받는데 문자열을 넘기면 예외 메세지를 비교하고 정규식을 넘기면 정규식 체크를 해준다.

### tset.each

테스트 코드도 유지보수의 대상이다.  
최대한 사람의 직접적인 수정이 덜 필요하게 만들어야 하기 때문에 다음과 같이 반복문적인 부분을 제공한다.

```javascript
test.each([[999], [0], [-123], [NaN], ['string'], [12.34]])(
  `옵션이 ~~이 아니라면 에러를 반환한다.`,
  (command) => {
    expect(() => {
      // 함수 실행
    }).toThrow()
  }
)
```

### Mocking

###### 우테코 프리코스의 코드를 이용한 예제. 참고로 `MissionUtils`는 우테코에서 제공하는 모듈이다.

#### jest.fn

사용자 입력이나 랜덤값 등 테스트를 진행하기 위해 필요한 동작을 하는 가짜(mock) 함수로 만들어준다.

```javascript
// 원래는 콘솔의 입력을 받는 함수
const mockQuestions = (answers) => {
  MissionUtils.Console.readLine = jest.fn()
  answers.reduce((acc, input) => {
    return acc.mockImplementationOnce((_, callback) => {
      callback(input)
    })
  }, MissionUtils.Console.readLine)
}

// 원래는 범위 안에 있는 숫자 중에서 랜덤하게 선택해서 반환하는 함수
const mockRandoms = (numbers) => {
  MissionUtils.Random.pickNumberInRange = jest.fn()
  numbers.reduce((acc, number) => {
    return acc.mockReturnValueOnce(number)
  }, MissionUtils.Random.pickNumberInRange)
}

test('횟수 1번만에 성공한다.', () => {
  mockRandoms([1, 0, 1])
  mockQuestions(['3', 'U', 'D', 'U'])

  const app = new App()
  app.play()

  // ...
})
```

#### jest.spyOn

어떤 객체에 속한 함수의 구현을 가짜로 대체하지 않고, 해당 함수의 호출 여부와 어떻게 호출되었는지만을 알아내야 할 때가 사용한다.  
아래와 같이 콘솔에 출력된 내용을 확인하는 용도로 사용할 수 있다.

```javascript
const getLogSpy = () => {
  const logSpy = jest.spyOn(MissionUtils.Console, 'print')
  logSpy.mockClear()
  return logSpy
}

const expectLogContains = (received, logs) => {
  logs.forEach((log) => {
    expect(received).toEqual(expect.stringContaining(log))
  })
}

test('횟수 1번만에 성공한다.', () => {
  const logSpy = getLogSpy()

  const app = new App()
  app.play()

  const log = getOutput(logSpy)
  expectLogContains(log, [
    '최종 게임 결과',
    '[ O |   | O ]',
    '[   | O |   ]',
    '게임 성공 여부: 성공',
    '총 시도한 횟수: 1',
  ])
})
```

## 5. 후기

### 오류 났던 부분들

0. 내가 진행했던 프로젝트는 express + typescript + yarn berry로 이루어져 있다.
1. winston 로그를 남기기 위해 app-root-path라는 모듈을 사용했었는데, jest랑 같이 사용하려다 보니 계속 오류가 났다. yarn berry로 모듈을 관리하면서 지정된 path를 제대로 읽어오지 못하는 것 같았다. 그래서 app-root-path라는 모듈을 지우고 rootDir(상대경로)를 하드 코딩하여 해결했다.
2. ts-jest가 필요없다. ES6 문법인 `import`를 인식하지 못해서 검색해보니 `require`을 사용하거나 babel 관련 모듈을 다운받으라고 했다. `require`로 바꾸면 프로젝트 전체 문법을 바꿔야 하니 포기했고 babel을 사용해도 해결이 되지 않았다. (지금 생각해보니 1번 문제였는데 babel 문제로 착각했던 것 같다).
3. jest, jest-extended 모듈을 받고 빌드 후 테스트를 진행했다. 빌드하는 시간까지 포함돼서 테스트 시간이 조금 더 걸렸지만 무사히 jest 모듈을 이용해 테스트를 진행할 수 있었다.

### 설정 파일

- tsconfig.json

설정에 대한 설명은 https://www.typescriptlang.org/ko/docs/handbook/tsconfig-json.html 에 자세히 나와 있다.

```json
{
  "compilerOptions": {
    "target": "es2021",
    "module": "commonjs",
    "moduleResolution": "node",
    "types": ["node", "jest"],
    "declaration": true,
    "sourceMap": true,
    "outDir": "./dist",
    "removeComments": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
    "resolveJsonModule": true
  },
  "exclude": ["node_modules", "dist"]
}
```

- jest.config.ts

```typescript
module.exports = {
  testEnvironment: 'node',
  testMatch: ['**/__tests__/*.test.js'],
  testTimeout: 120000,
}
```

- index.ts(app.ts에서 짠 기본 설명 미들웨어들을 거치고 난 후의 app을 실행하는 코드)

```typescript
const port = ~~
const app = initApp()

const serve = async () => {
	try {
		checkRequiredEnvs()

		await connectMongoDb()

		const server = http.createServer(app)

		server.listen(port, () => {
			console.log('서버가 실행되었습니다.')
		})
	} catch(err) {
		console.error(err)
	}
}

serve()

export default app // 테스트 코드에서 이 app 변수를 import 하기 위함.
// http.createServer가 진행된 app을 import해야 테스트 코드(supertest)가 제대로 동작한다고 함.
// app.address is not a function 에러가 발생할 시 아래 주소 참고.
// https://stackoverflow.com/questions/33986863/mocha-api-testing-getting-typeerror-app-address-is-not-a-function 참고
```

## 6. 테스트 방법론

#### AAA(Arrange, Act, Assert) 패턴

참고  
https://hijuworld.tistory.com/80?category=1097105  
https://codechacha.com/en/unittest-aaa-pattern/  
https://velog.io/@fkszm3/testing-unit-test%EA%B8%B0%EC%B4%88-AAA-Pattern

unit test 시 좋은 테스트 코드를 만들기 위한 패턴 중 하나이다.

-   Arrange: 테스트에 필요한 변수나 객체를 생성. 필요에 따라 mock 객체를 만듦. 길어질 수 있지만, 다른 테스트 코드와 유사성이 있다면 공통으로 분리하는 것도 좋음.
-   Act: 테스트할 코드를 실행. 보통 한두 줄로 표현 가능.
-   Assert: 실행한 코드가 설계한 대로 동작했는지 검증(`expect` 등).

```js
it(`버튼을 클릭하고 5초 뒤에는, p 태그 안에 "버튼이 눌리지 않았다." 라고 쓰여진다.`, () => {
    // Arrange
    jest.useFakeTimers();
    const { getByText } = render(<Button />);
    const buttonElement = getByText("button");

    // Act
    fireEvent.click(buttonElement);
    act(() => {
        // state 변경 시 감싸지 않으면 warning 발생
        jest.advanceTimersByTime(5000); // 5초가 흘렀다고 가정. 실제 테스트 시간이 5초가 걸리진 않음.
    });

    // Assert
    const p = getByText("버튼이 눌리지 않았다.");
    expect(p).not.toBeNull();
    expect(p).toBeInstanceOf(HTMLParagraphElement);
});
```

#### GWT(Given-When-Then) 패턴

참고: https://martinfowler.com/bliki/GivenWhenThen.html , https://softwareengineering.stackexchange.com/questions/308160/differences-between-given-when-then-gwt-and-arrange-act-assert-aaa

AAA의 또다른 이름이라고 생각해도 된다.  
AAA가 TDD의 용어이며, 개발자 지향적인 용어라면,  
GWT는 BDD의 용어이며, 비즈니스 지향적인 용어이다.

-   Arrange == Given
-   Act == When
-   Assert == Then

## 7. FE에서 테스트란

### 테스트 환경

**브라우저 환경** => headless 브라우저를 사용하여 개발이 완료된 후 배포할 때 CI와 연동해서 테스트하는 방식 권장됨.

-   장점
    -   모든 Web API에 접근할 수 있음.
    -   브라우저 호환성 및 기기 호환성 테스트를 진행할 수 있음.
-   단점
    -   느림.
    -   브라우저 런처 별도 설치 필요.

**Node.js 환경** => jest와 같은 테스트 도구에서 DOM을 가상으로 구현하는 라이브러리 활용

-   장점
    -   빠름
-   단점
    -   Web API 사용 못 함.
    -   페이지 내비게이션이나 로그아웃 같은 것들은 테스트할 수 없음.
    -   브라우저 호환성 및 기기 호환성 테스트를 못 함.

### E2E 테스트

참고  
https://fe-developers.kakaoent.com/2023/230209-e2e/  
https://brunch.co.kr/@jiwonleeqa/241  
https://medium.com/delivus/e2e-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EA%B5%AC%EC%B6%95%EA%B8%B0-used-aws-step-functions-2fccb930218c  
https://hyperconnect.github.io/2022/01/28/e2e-test-with-playwright.html  
https://ui.toast.com/posts/ko_20210818

~현재 프로젝트에서 당장 사용하기 보다는, 유닛 테스트와 통합 테스트가 자리를 잡고 유의미하다고 생각이 들면 도입을 제안해야겠다.~  
개인적으로 생각하는 E2E 테스트의 가장 큰 장점은 QA 과정에서도 발견하지 못할 버그를 찾을 수 있으며 QA 전인 개발 과정에서 버그를 찾는 것이 시간과 비용 측면에서 많은 세이브가 된다는 것이다.

많은 회고록들을 보면 E2E는 어렵다고 하며 심지어 도입했다가 걷어내는 팀도 있다고 한다.  
왜 그런 일들이 발생했을까 고민해보면 다음과 같은 이유가 있을 것이다.

-   QA 팀의 하위 호환. QA 팀과 협의가 없다면 의미가 없어짐.
-   테스트 코드 관리의 어려움.
-   오래 걸리는 테스트 속도.

카카오엔터테인먼트 팀에서는 이러한 문제에도 불구하고(다만 아직 QA팀과 협업은 진행하지 않았다고 함) E2E 테스트에서 시나리오를 작성함으로써 전체적인 프로젝트 흐름을 파악하고, 다른 사람의 코드를 수정할 때 부담이 되는 점을 보완하고자 도입을 시도했다.  
이 덕분에 프론트엔드에서 예상치 못한 버그를 찾아내거나 API에서 발생한 사이드 이펙트를 발견하기도 했다고 한다.  
그 팀이 지킨 원칙은 다음과 같다.

1. QA 팀과 같이 기획서 기반의 시나리오 작성(테스트 코드 자체가 하나의 문서가 됨).
2. 예상치 못한 에러를 찾기 위해 Mock 데이터 사용 지양.
3. husky 또는 github hook을 통해 pre push 단계에서 E2E 검증.
4. 병렬 테스트 툴(`Sorry-Cypress`) 사용.

+) 시간이 오래 거릴기 때문에 CI/CD 중에 테스트를 돌리는 것도 하나의 방법.

### 무엇을 테스트해야 하는가?

참고  
https://ui.toast.com/posts/ko_20210630  
https://www.freecodecamp.org/news/testing-react-hooks/  
https://team.modusign.co.kr/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C%EC%97%90%EC%84%9C-%EC%9D%98%EB%AF%B8%EC%9E%88%EB%8A%94-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%BD%94%EB%93%9C-%EC%9E%91%EC%84%B1%ED%95%98%EA%B8%B0-4992409c7f2d  
https://blog.mathpresso.com/%EB%AA%A8%EB%8D%98-%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%A0%84%EB%9E%B5-1%ED%8E%B8-841e87a613b2  
https://blog.mathpresso.com/%EB%AA%A8%EB%8D%98-%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%A0%84%EB%9E%B5-2%ED%8E%B8-de069e271b3d  
https://learn-react-test.vlpt.us/#/

Right-BICEP는 무엇을 테스트할지에 대한 가이드를 제공한다.  
**입력-실행-결과** 식으로 짤 수 있도록 집중해야 한다.

-   Right: 올바른 결과를 보여주는가. 일어날 만한 상황들에 대해서만 테스트를 수행한다.
-   Boundary-conditions: 경계 조건을 준수했는가. 경계 조건은 추가적인 내용이 있다.
    -   Conformance: 값이 예상하는 형식과 일치하는가.
    -   Ordering: 값의 집합이 적절하게 정렬되었거나 정렬되지 않았는가.
    -   Range: 값이 예상된 범위(최대/최소값)내 있는가.
    -   Reference: 직접 제어할 수 없는 외부 항목을 참조하고 있는가.
    -   Existance: 값이 존재하는가. (null check)
    -   Cardinality: 값이 없거나 하나이거나 엄청 많거나할 때의 경우에 대한 수행을 확인했는가.
    -   Time: 모든 작업이 시간순에 맞게 제대로 수행되는가(적시에).
-   Inverse-relationships: 역 관계를 검사할 수 있는가.
-   Cross checking using other means: 다른 수단(라이브러리나 툴)을 사용해 교차 검증이 가능한가.
-   Error conditions: 오류 조건을 강제로 일어나게 만들 수 있는가.
-   Performance characteristics: 성능 조건은 기준에 부합하는가.

다음은 프론트엔드에서 테스트할 수 있는 대상이다.  
크게 3가지로 구분하자면 *시각적 요소, 사용자 이벤트 처리, API 통신*이다.

-   **컴포넌트 및 기능 테스트**: 사용자가 이 컴포넌트와 상호작용하는 방식을 생각하며 코드를 짠다. (개인적인 생각) 이벤트가 존재하는 컴포넌트부터 즉, 다른 컴포넌트의 basic이 되는 컴포넌트는 제외하고 테스트 코드를 짜는 게 맞지 않을까 싶다.
    -   렌더링됐을 때 화면에 나와야 되는 element들이 잘 있는지
    -   user interaction이 발생하면 URL 이동을 포함하여 변경되야 하는 부분이 잘 되는지
    -   API 요청이 완료된 후에 데이터를 잘 렌더링 하는지
    -   무한 로딩 상황이 있는지(time-out 시 어떻게 처리되는지)
    -   사용자가 빠져나갈 수 없는 에러 상황이 있는지
    -   style이 올바른지. (다음부터는 개인적인 생각) 정말 스타일이 중요한 사항 또는 stacking context로 인해 user interaction이 방해되는 상황 정도만 테스트하는게 맞을 것 같음
    -   함수가 인자에 따라 (에러를 포함하여) 올바르게 처리하는지
    -   크로스 브라우징, 크로스 플랫폼이 잘 되는지
    -   시각, 청각적인 요소가 빠지진 않았는지
-   **성능 테스트**
    -   네트워크 등의 상황을 고려하여 bottleneck이 없는지
-   **접근성 테스트**
    -   a11y를 잘 지켰는지

+) 테스트 코드도 코드이다.  
즉, 선언적으로, 반복적으로 작성되는 부분은 함수로 구현하면 효율적이다.

```ts
it("뭔가 수행한다.", async () => {
    const onSubmit = jest.fn();
    const onCancel = jest.fn();
    const result = render(
        <ComplexForm onSubmit={onSubmit} onCancel={onCancel} />
    );

    expect(result.getByLabelText("First Name")).toBeInTheDocument();
    expect(result.getByLabelText("Last Name")).toBeInTheDocument();

    await act(async () => {
        userEvent.click(result.getByLabelText("Over 21?"));
    });

    expect(result.getByLabelText("Favorite Drink?")).toBeInTheDocument();
});
```

위 코드보다는 아래 코드가 시나리오를 모르는 누군가가 봐도 이해하기 쉬울 것이다.

```ts
it("뭔가 수행한다.", async () => {
    const { FirstNameInput, LastNameInput, clickIsOver21, FavoriteDrinkInput } =
        renderComplexForm();

    expect(FirstNameInput()).toBeInTheDocument();
    expect(LastNameInput()).toBeInTheDocument();

    await clickIsOver21();

    expect(FavoriteDrinkInput()).toBeInTheDocument();
});
```
