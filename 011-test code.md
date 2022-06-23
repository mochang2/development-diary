## 0. 공부하게 된 계기
토이 프로젝트할 때와 달리 회사 프로젝트는 상시 켜두기 때문에, 그리고 누구일지 모르는 사람들이 다수가 사용하기 때문에  
어떤 일이 벌어질지 예측을 할 수가 없다.  
최대한 모든 상황을 담아서 테스트를 해야되고 프로젝트가 커질수록 미들웨어 간의 관계 등에 의해서 예상치 못한 부분에서 에러가 발생할 수 있다.  
따라서 테스트 코드를 작성해야 돼서 공부를 시작했다.  
~그런데... 프로젝트가 무기한 잠정중단이 됐다... 아마 부하테스트 정도만 하고 그 이상의 테스트 코드는 안 짤 듯 싶다~
https://ko.myservername.com/differences-between-unit-testing 와 https://tecoble.techcourse.co.kr/post/2021-05-25-unit-test-vs-integration-test-vs-acceptance-test/ 를 참고했다.
  
## 1. 필요성
##### 실시간 코드 반영
테스트 코드는 방금 작성한 코드를 당장 테스트 하기 위해서만 작성되는 것이 아니다.  
이후 다른 변경 사항으로 인해 발생 가능한 결함을 찾아내는 역할을 한다.  


##### 자동 테스트 도구
직접 손으로 테스트하는 것보다 명령어 한 줄(jest test 등)으로 자동화시키는 것이 더 안정적이고 실수가 줄어든다.

##### 테스트 작성 시간
물론 처음에는 오래 걸리겠지만, 프로젝트가 커질수록 해당 시간은 보잘 것 없게 된다.  
즉, 투자할 가치가 있는 시간인 것이다.

##### QA 프로세스
QA만으로 모든 테스트를 커버할 수 없고, 개발자 입장에서 테스트를 하고 넘겨줘야 QA 프로세스가 감소할 수 있다.

##### 클라이언트 테스트
UI 테스트가 쉽지는 않지만 불가능한 부분도 아니다.  
서버에 대한 테스트가 중요하겠지만 클라이언트 테스트도 무시할 부분이 아니다.

## 2. 테스트 종류
##### 단위 테스트(Unit Test)
응용 프로그램에서 테스트 가능한 가장 작은 소프트웨어를 실행하여 예상대로 동작하는지 확인하는 테스트이다.  
개별 모듈을 독립적으로 테스트하는 것으로, 일반적으로 클래스 또는 메소드 수준으로 정해진다.  
단위 크기가 작을수록 복잡성이 낮아지므로 동작을 표현하기 더 쉬워진다.  

##### 통합 테스트(Integration Test)
단위 테스트보다 더 큰 동작을 달성하기 위해 여러 모듈들을 모아 이들이 의도대로 협력하는지 확인하는 테스트이다.  
통합 테스트는 외부 라이브러리 등 개발자가 변경할 수 없는 부분까지 묶어서 검증할 수 있다.  
이는 DB에 접근하거나 전체 코드와 다양한 환경이 제대로 작동하는지 확인하는데 필요한 모든 작업을 수행할 수 있다.  
통합 테스트를 통해 단위 테스트에서 발견하기 어려운 버그를 찾을 수 있다.  

##### 인수 테스트(Acceptance Test)
사용자 스토리(시나리오)에 맞춰 수행하는 테스트이다.  
앞선 두 테스트들과 달리 비즈니스 쪽에 초점을 둔다.  
프로젝트에 참여하는 사람들(기획자, 클라이언트 대표, 개발자 등)이 토의해서 시나리오를 만들고 개발자는 이에 의거해서 코드를 작성한다.  
개발자 혼자서 직접 시나리오를 제작할 수 있지만 고객 관점 측면에서 놓치는 부분이 생길 수 있다.  
따라서 직접 고객과 대면하는 팀으로부터 시나리오와 피드백을 받아 개발할 수 있는 테스트이다.  

핸드폰의 주요 부품인 배터리와 SIM 카드에 대해 테스트한다고 가정하면  
단위 테스트는 배터리의 수명, 용량 및 기타 매개 변수를 확인하며 SIM 카드가 활성화되었는지 확인하는 것이다.  
통합 테스트는 배터리와 SIM 카드가 일체화되어 휴대폰을 시작하기 위해 조립하는 것이다.  
기능 테스트는 휴대폰의 기능 및 배터리 사용량은 물론 SIM 카드 설비 등을 확인하는 것이다.  

## 3. FIRST 단위 테스트 원칙
단위 테스트는 가장 작은 단위의 테스트이며, 모든 테스트의 시작점이다.  
FIRST는 Fast, Isolated, Repeatable, Self-validating, Timely의 약자이다.  

* Fast: 유닛 테스트는 빨라야 한다.
* Isolated: 다른 테스트에 종속적인 테스트는 절대로 작성하지 않는다.
* Repeatable: 테스트는 실행할 때마다 같은 결과를 만들어야 한다.
* Seft-validating: 테스트는 스스로 결과물이 옳은지 그른지 판단할 수 있어야 한다. 특정 상태를 수동으로 미리 만들어야 동작하는 테스트 등은 작성하지 않는다.
* Timely: 유닛 테스트는 프로덕션 코드가 테스트를 성공하기 직전에 구성되어야 한다. Test Driven Development(TDD) 방법론에 적합한 원칙이지만 실제로 적용되지 않는 경우도 있다.


## 4. jest
##### 설치
배포할 때는 필요없는 패키지이므로 개발용으로 설치한다.  
이에 예외는 없을 것 같다.  

```
npm install -D jest
```

##### 실행 명령어
package.json에 "test"를 "jest"로 수정한다.  
npm test로 jest를 실행할 수 있다.  
global 옵션을 줘서 npx로 실행할 수도 있지만 일반적으로는 이렇게 많이 실행한다.  

```
"scripts: {
	"test": "jest"
}
```

##### test

```typescript
test('description), () => {
	expect(어떤 행위).toBe(기댓값)
})
```

위와 같은 방식이 기본 형식이다.

##### describe
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

##### beforeAll, afterAll, beforeEach, afterEach
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

##### Matcher
1. toEqual() : \~\~와 같은 값을 기대한다는 뜻으로 toBe()로는 같지 않다고 뜨는데 toEqual()는 같다고 나오는 경우가 있다. 특히 객체와 같은 경우가 그렇다. 따라서 특별한 경우가 아니면 toEqual()를 사용하기를 권고한다.  
2. toBeTruthy(), toBeFalsy(): 이름에서 알 수 있듯이 boolean 값을 return하기를 기대하고 쓰는 matcher이다. js와 같은 경우에는 1이 true, 0이 false 간주한다.  
3. toHaveLength(n): 배열의 길이를 체크한다.  
4. toContain('Example'): 특정 원소가 배열에 들어있는지를 테스트할 때 쓰인다.  
5. toMatch('문자열 또는 정규표현식'): 문자열과 일치하는지 또는 정규표현식에 일치하는지 쓰인다.  
6. toThrow(argument?): 함수는 인자도 받는데 문자열을 넘기면 예외 메세지를 비교하고 정규식을 넘기면 정규식 체크를 해준다.

## 5. 후기
##### 오류 났던 부분들
0. 내가 진행했던 프로젝트는 express + typescript + yarn berry로 이루어져 있다.
1. winston 로그를 남기기 위해 app-root-path라는 모듈을 사용했었는데, jest랑 같이 사용하려다 보니 계속 오류가 났다. yarn berry로 모듈을 관리하면서 지정된 path를 제대로 읽어오지 못하는 것 같았다. 그래서 app-root-path라는 모듈을 지우고 rootDir(상대경로)를 하드 코딩하여 해결했다.
2. ts-jest가 필요없다. ES6 문법인 import를 인식하지 못해서 검색해보니 require을 사용하거나 babel 관련 모듈을 다운받으라고 했다. require로 바꾸면 프로젝트 전체 문법을 바꿔야 하니 포기했고 babel을 사용해도 해결이 되지 않았다. (지금 생각해보니 1번 문제였는데 babel 문제로 착각했던 것 같다).
3. jest, jest-extended 모듈을 받고 빌드 후 테스트를 진행했다. 빌드하는 시간까지 포함돼서 테스트 시간이 조금 더 걸렸지만 무사히 jest 모듈을 이용해 테스트를 진행할 수 있었다.

##### 설정 파일
* tsconfig.json

설정에 대한 설명은 https://www.typescriptlang.org/ko/docs/handbook/tsconfig-json.html 에 자세히 나와 있다.

```
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

* jest.config.ts

```
module.exports = {
  testEnvironment: 'node',
  testMatch: ['**/__tests__/*.test.js'],
  testTimeout: 120000,
}
```

* index.ts(app.ts에서 짠 기본 설명 미들웨어들을 거치고 난 후의 app을 실행하는 코드)

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

