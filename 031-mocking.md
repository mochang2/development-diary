## 0. 공부하게 된 계기

~면접에서 집요하게 공격받던 질문이다...~  
예전에 진행했던 프로젝트 중에 FE 개발만 많이 앞서 나가고 BE의 API가 개발될 때까지 기다렸던 적이 있다.  
그러한 일이 현직에서도 많이 발생하고 BE/FE 개발을 비동기(?)로 진행하기 위한 방법으로 mocking을 이용한다고 한다.

참고 및 사진 출처

- https://tech.kakao.com/2021/09/29/mocking-fe/
- https://www.soapui.org/learn/mocking/what-is-api-mocking/
- https://yujo11.github.io/web/json-server%EB%A1%9C%20Mock%20Server%EB%A7%8C%EB%93%A4%EA%B8%B0/
- https://gaemi606.tistory.com/entry/React-API-%EC%9A%94%EC%B2%AD-%ED%85%8C%EC%8A%A4%ED%8A%B8
- https://han-py.tistory.com/400

## 1. Mocking

'mock'의 사전적으로 모조품, 흉내라는 뜻이다.  
mocking이란 'real' object를 흉내낸 'fake' object를 만드는 행위이다.

API를 흉내내는 것이 모두 mocking은 아니며 흉내 정도에 따라 4가지로 구분한다.

- Stubbing: mostly a placeholder without real functionality
- Mocking: basic functionality required for a specific testing or development purpose
- Simulation: complete functionality for testing or development purposes
- Virtualization: imulation that is deployed into an operational, manageable and controllable environment

mocking은 보통 테스트 환경에서 많이 사용되며 FE에서 BE와의 통신을 흉내내는 것만을 말하는 것이 아니다.  
아래 사진과 같이 Legacy, Mobile Apps, Infrastructure, Databases, Peer Services, 3rd Party Components 간의 '흉내낸' 통신을 모두 mocking이라고 한다.

![mocking](https://user-images.githubusercontent.com/63287638/189320648-ec48b91f-5ca7-4ec1-bb72-f085b53c8469.png)

**제대로 mocking하는 방법**

- 기술적(스키마, 프로토콜 등)으로 100% 동일하게 만들기
- 행위를 기록할 수 있는 툴이나 로깅 사용하기
- negative test를 만들어서 일부러 예기치 않은 오류, 긴 응답 시간 등을 만듦으로써 클라이언트가 올바른 처리를 하도록 만들기
- 종속성 내에서 예기치 않은 변경이나 불규칙성에 영향을 받지 않고 원하는 만큼 테스트를 실행하기

### FE에서 Mocking이 필요한 이유

웹 개발의 가장 이상적인 상황은 다음과 같다.

> 요구 사항 분석 및 기획 -> BE 개발 -> FE 개발

하지만 모든 개발이 위와 같은 순서와 방식으로 진행되지 않는다.  
FE 개발을 BE 개발과 같이 시작할 수도, 요구 사항 분석 및 기획이 끝나기 전에 시작해야될 수도 있다.

이때 FE에서 BE의 API를 활용하는 것처럼 BE 개발에 종속적인 부분이 있다면, 해당 부분이 완성되기 전까지는 FE에서 개발을 진행할 수 없고, 그 부분이 진행된 후에나 개발이 가능하다.  
심지어 추가적인 수정사항이 발생했는데 BE 개발에 의존성이 있는 부분에 수정이 필요하다면, 이러한 비효율적인 과정을 반복 진행할 수밖에 없다.

그래서 카카오 엔터프라이즈는 다음과 같은 방식으로 개발을 진행한다고 한다.  
FE에서 BE와 종속적인 개발 영역이 있는 경우, FE 개발자는 필요한 API의 스펙을 사전 협의하여 BE 개발자에게 받고 다음과 같이 개발을 진행한다.

![mocking1](https://user-images.githubusercontent.com/63287638/189332537-1ff15954-2543-41cf-a883-a0062c68e393.png)

때로는 아래 방식처럼 API 자체에 대한 의존성을 일단 가져가지 않고, mocking을 통해 개발을 진행한 다음, 실제 사용해야 하는 API가 나온 시점에서 API의 실제 인터페이스에 맞게 컨버팅하는 과정을 따로 가지는 개발 방법으로 진행하기도 한다.

![mocking2](https://user-images.githubusercontent.com/63287638/189332541-3f9c2aac-64a3-4d13-ae6d-89bdf7449106.png)

## 2. 기존 Mocking 방식

### 라이브러리 없이

화면에 필요한 데이터의 상태를 애플리케이션의 내부 로직에 직접 필요한 화면에 붙이는 방식이 있다.  
이렇게 작업하는 방식은 구현이 쉬워 빠르게 적용할 수 있다는 장점이 있다.  
하지만 애플리케이션의 서비스 로직에 직접 mocking을 해야 해 애플리케이션 서비스 로직을 수정해야 하고, HTTP 메소드와 네트워크의 응답 상태에 따라 각각 대응하기가 쉽지 않다.

아래는 `useEffect` 훅을 사용하여 내가 사용했던 방식이다.  
~사실 이 방식이 mocking이라고 부를 수 있는 방식인지 모르겠다~

```tsx
interface PostType {}

const data = [
  // ...
]

function Component() {
  const [posts, setPosts] = useState<PostType[]>([])

  useEffect(() => {
    setPosts(data)
  }, [])

  return (
    {
      posts.length === 0
      ? <Loading />
      : <Wrapper>
          {posts.map((post) => {
            return (
              <Post key={post.id}>
                {post.name
              }</Post>
            )
          })}
        </Wrapper>
    }
  )
}
```

또는 과제 테스트에서 자주 사용되는 방식으로, json 파일을 만들어 읽는 방식도 가능하다.  
앞에 나온 방식과는 다르게 비동기 처리를 추가할 수 있다.

아래에서 `data` 디렉터리는 `public` 디렉터리 내부에 존재해야 한다. [참고](https://itprogramming119.tistory.com/entry/React-axios%EC%9D%98-url%EC%9D%84-%EB%A1%9C%EC%BB%AC%EC%97%90-%EC%9E%88%EB%8A%94-json%EC%9C%BC%EB%A1%9C-%EC%84%A4%EC%A0%95%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95)

```tsx
interface PostType {}

const api: AxiosInstance = axios.create({
  baseURL: '/data',
  headers: {}
})

function Component() {
  const [posts, setPosts] = useState<PostType[]>([])

  useEffect(() => {
    async function getFaqs() {
      const { data } = await api.get('/faq.json')

      setPosts(data)
    }

    getFaqs()
  }, [])

  return (...)
}
```

위와 같은 방식을 사용하면서 느꼈던 불편함은 다음과 같았다.

1. 실제로 API를 연동할 때 uri 입력부터 에러 처리 등 수정해야 될 부분이 많다.
2. 일부러 예기치 않은 오류를 만들거나, 긴 응답 시간을 만들어 로딩처리나 timeout을 테스트하려면 손봐야 할 부분이 많으며 실제 배포 시에는 해당 부분을 다시 전부 지워야 한다.

반대로, Mock 서버를 별도로 만드는 방법도 있다.  
이 방법은 웹 애플리케이션의 서비스 로직을 수정하지 않아도 된다는 장점이 있다.  
하지만, 데이터 상태 구현에는 직접 Mocking 대비 비용 및 공수가 상당히 들어간다(그래서 직접 해본 적은 없다).  
추가적으로 이를 로컬이 아닌 누군가에게 공유해야 한다면, Mock 서버를 원격으로 제공하기 위한 추가 환경 구성 작업 등에도 적지 않은 공수가 들어 효율적이지 않다.

### 라이브러리 사용하여

[json-server](https://yujo11.github.io/web/json-server%EB%A1%9C%20Mock%20Server%EB%A7%8C%EB%93%A4%EA%B8%B0/) 모듈 예시 - Mock Server 만들기

[axios-mock-adapter](https://gaemi606.tistory.com/entry/React-API-%EC%9A%94%EC%B2%AD-%ED%85%8C%EC%8A%A4%ED%8A%B8) 모듈 예시

## 3. MSW

실제 API를 사용하는 것처럼 네트워크 수준에서 mocking 하기 위한 방법을 제공해주는 모듈이다.  
~`axios-mock-adaptor`도 네트워크 단에서 mocking이 가능한 것 같지만 npm에 해당 내용은 없으며 npm trend에서 MSW가 더 많고 자료가 많으므로 MSW를 선택했다~

MSW(Mock Service Worker) 네트워크 요청을 가로채서 mocked response을 보내주는 역할을 한다.  
따라서, MSW 라이브러리를 통하면, 서버를 구축하지 않아도 API를 네트워크 수준에서 mocking 할 수 있다.

MSW가 이러한 역할을 할 수 있는 이유는 Service Worker를 통해 HTTP 요청을 가로채기 때문이다.

### Service Worker

Service Worker는 웹 애플리케이션의 메인 스레드와 분리된 별도의 백그라운드 스레드에서 실행시킬 수 있는 기술 중 하나이다.  
Service Worker 덕분에 애플리케이션의 UI(메인 스레드가 처리하는 부분) Block 없이 연산을 처리할 수 있다.

Service Worker는 다음과 같은 기능에 주로 사용된다.

1. 백그라운드 동기화할 때.
2. 높은 비용의 계산을 처리할 때.
3. 푸시 이벤트를 생성할 때.
4. 네트워크 요청을 가로챌 때. 애플리케이션과 서버 사이에서 Request를 가로채서 직접 Fetch에 대한 컨트롤도 할 수 있음. HTTP Request와 Response를 보고 캐싱 처리를 한다든지, 필요하다면 로깅을 하는 등 새로운 동작을 만들어 낼 수 있음.

다음은 msw 동작 원리를 그림으로 표현한 것이다.

![msw 동작 원리](https://user-images.githubusercontent.com/63287638/189345075-483be3fb-5cdf-488d-b6a2-cf564261856c.png)

### 코드 예시

1. `npm install msw` 또는 `yarn add msw` 실행
2. `npx msw init <PUBLIC DIR>` 실행. 여기서 `<PUBLIC DIR>`은 CRA로 react를 시작했다면 `public`.

2번까지 실행했다면 `<PUBLIC DIR>` 위치에 `mockserviceWorker.js` 파일이 생성된다.  
여러 `self.addEventListener` 코드가 동작하여 사용자의 request를 중간에서 가로챌 수 있게 된다.

3. `mocks/handlers.ts`를 작성

```typescript
import { rest, setupWorker } from 'msw'

// 인증이 추가되면 req.cookies 이용해서 토큰 확인 가능

const handlers = [
  rest.get('/api/faq/categories', (req, res, ctx) => {
    return res(
      ctx.status(200),
      ctx.delay(100),
      ctx.json({
        categories: [
          // ...
        ],
      })
    )
  }),
  rest.get('/api/faq', (req, res, ctx) => {
    return res(
      ctx.status(200),
      ctx.delay(100),
      ctx.json({
        faqs: [
          // ...
        ],
      })
    )
  }),
]

export default setupWorker(...handlers)
```

4. `index.tsx`를 수정

```tsx
// 추가
if (process.env.NODE_ENV === 'development') {
  // 개발 모드일 때만
  // require(동기적으로 모듈을 가져옴)를 사용할 수도 있음
  // eslint 설정에서 require를 사용하지 못하게 해서 import 비동기 처리를 함
  import('apis/handlers').then((worker) => worker.default.start())
}
```

5. api 코드 작성

```tsx
function Faq() {
  const [faqs, setFaqs] = useState<FaqType[]>([])

  useEffect(() => {
    async function getFaqs() {
      const {
        data: { faqs },
      } = await api.get('/faq')

      setFaqs(faqs)
    }

    getFaqs()
  }, [])

  return (
    <React.Fragment>
      {faqs.map((faq) => {
        // ...
      })}
    </React.Fragment>
  )
}
```

`yarn react-scripts start` 또는 `npm run react-scripts start`를 하여 올바르게 요청이 이루어지면 다음과 같은 내용이 console에 출력된다.

![result](https://user-images.githubusercontent.com/63287638/189372067-290f5db5-0c71-4f6d-b4c6-d8fbd74483e5.jpg)

다만 HMR될 때 가끔 맛탱이가 갈 때가 있다.  
내부 동작 방식까지는 잘 모르겠어서 이유는 모르겠다.  
그럴 때는 refresh하거나 수동으로 해당 페이지를 다시 들어가야겠다.
