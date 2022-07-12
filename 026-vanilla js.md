## 0. 공부하게 된 계기
React? Vue? 아무리 개발을 해봐도 근본을 모르면 말짱 도루묵인 것 같다.  
~코딩 테스트나 과제 테스트 통과 못하는 건 덤~  
우테캠으로 과제 전형을 보고, vanilla js로 virtual DOM을 비슷하게 따라 만드는 강의가 있다는 것을 알고 공부해야겠다 마음을 먹었다.

## 1. babel

비효율적인 것은 잘 몰랐으나 node modules를 사용하다 보면 패키지가 깨지는 경우가 많았다.  
비효율성과 패키지 관리를 위해 yarn v2에 출시되었다고 한다.

### 단점 1) 비효율적인 의존성 검색

## 2. webpack

위의 문제를 해결하기 위해 yarn berry는 pnp(Plug n Play) 전략을 사용한다. 아래 terminal 명령어를 통해 yarn berry를 사용할 수 있다.

```
npm install -g yarn
cd ../path/to/some-package
yarn set version berry
// yarn set version berry가 안되면 yarn policies set-version을 사용
```

yarn berry는 node_modules 디렉토리를 생성하지 않는다.  
대신 .yarn/cache 폴더에 의존성의 정보가 저장되고, .pnp.cjs 파일에 의존성을 찾을 수 있는 정보가 기록된다.  
.pnp.cjs를 이용하면 디스크 I/O 없이 어떤 패키지가 어떤 라이브러리에 의존하는지, 각 라이브러리는 어디에 위치하는지를 바로 알 수 있다.

### react.createchild


## 3. Node.js
기존의 JS는 HTML에 독립적으로 실행할 수 없는 프로그래밍 언어였다(`<script>` 태그).  
하지만 Node.js가 등장하면서 JS를 HTML에 독립적으로 실행할 수 있게 됐다.  
물론 그 이전에도 JS를 HTML에 독립적으로 실행할 수 있도록 만드는 시도가 있었으나 엔진(Chrome V8 JavaScript 엔진 이전) 속도 문제로 실패했었다.  
[Node.js 공식 사이트](https://nodejs.org/)에 따르면 Node.js는

> Node.js는 Chrome V8 JavaScript 엔진으로 빌드된 JavaScript 런타임입니다.\
Node.js는 이벤트 기반, Non-blocking I/O 모델을 사용해 가볍고 효율적입니다.\
이는 (I/O를 직접 수행하지 않으므로) 사용자가 프로세스의 교착상태에 대해 걱정할 필요가 없다는 말이기도 합니다.\
Node.js의 패키지 생태계인 npm은 세계에서 가장 큰 오픈 소스 라이브러리 생태계이기도 합니다.\
Node.js는 이벤트 루프를 시작하는 호출이 없으며, 각 연결에서 콜백이 실행되는데 실행할 작업이 없다면 Node.js는 대기합니다.
Node.js는 스레드를 사용하지 않도록 설계되었지만 멀티 코어 환경의 장점을 얻지 못하는 것은 아닙니다. 'child_process.fork()' API를 사용해 자식 프로세스를 생성할 수 있습니다.
  
_cf) 런타임: 특정 언어로 만든 프로그램들을 실행할 수 있게 해주는 가상 머신의 상태. 다른 런타임으로는 웹 브라우저(크롬, 사파리 등)이 있음._
  
공식 사이트의 설명대로 Node.js는 서버의 역할도 수행할 수 있는 JS 런타임이다.  
Node.js는 서버 실행을 위해 필요한 http/https/http2 모듈을 제공한다.  

이러한 특성 때문에 Node.js는 다음과 같은 개발에 용이하다.  

* 정적 파일 서버
* 웹 응용프로그래밍
* 메시지 미들웨어
* HTML5 멀티 플레이어 게임용 서버

### 1) NVM(Node Version Manager)
Node.js의 버전을 관리하는 도구이다.  
협업, 여러 가지 프로젝트를 동시에 진행해야 할 때와 라이브러리 버전 호환 문제에 유용하게 사용할 수 있다.  
파이썬으로 개발할 때 virtualenv를 사용하는 이유와 같다.  

Node.js를 PC에 설치하면 `node -v`로 버전을 확인할 수 있다.  
지금 사용중인 Node.js 버전이 필요한 버전과 맞지 않으면 [NVM Github](https://github.com/coreybutler/nvm-windows/)에 가서 설치할 수 있다.  

이후 `nvm install <버전>`으로 원하는 버전을 설치할 수 있다.  
`nvm list`를 입력하면 설치되어 사용할 수 있는 Node.js 버전이 조회되며  
`nvm use <버전>`으로 사용할 버전을 선택할 수 있다.  
마지막으로 다시 `node -v`를 입력하면 버전이 바뀐 것을 확인할 수 있다.  


### 2) NPM(Node Pacakge Module)
세상에서 가장 큰 소프트웨어 레포지토리이다.  
jQuery, react, vue 등의 다양한 오픈 소스들도 올라가 있고, 사설 모듈을 올려서 회사 내에서만 사용할 수도 있다.  

[NPM 공식문서](https://docs.npmjs.com/about-npm)에 따르면 다음과 같은 특징이 있다.  

> Use the website to discover packages, set up profiles, and manage other aspects of your npm experience. For example, you can set up organizations to manage access to public or private packages.  
The CLI runs from a terminal, and is how most developers interact with npm.  
The registry is a large public database of JavaScript software and the meta-information surrounding it.

##### package.json
npm을 통해 설치된 패키지 목록을 관리하고 프로젝트의 정보 및 기타 실행 스크립트를 작성하는 파일이다.  
프로젝트를 처음 시작할 때, `npm init` 명령어를 통해 설치할 수도 있고, 직접 editor에서 생성해서 작성할 수도 있다.  
이후 `npm install` 또는 `npm install package.json` 또는 `yarn` 또는 `yarn install` 등으로 관련 모듈들을 설치할 수 있다.  
package.json은 다음과 같이 구성되어 있다.  

```json
{
  "name": "project",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "axios": "^0.27.2",
    "cookie-parser": "~1.4.4",
    "react": "0.10.0",
  },
  "devDependencies": {
    "supertest": "^4.0.2"
  }
}
```

devDependencies는 개발할 때만 사용되고 배포할 때는 포함되지 않는 패키지들을 적는 곳으로 `-D` 옵션을 추가해서 설치한다.  

참고로 모듈 버전은 일반적으로 "x.x.x"로 구성되는데, 첫 번째 자리 수는 대대적인 변화가 있을 경우, 두 번째 자리 수는 버그 픽스 등 적당한 변화가 있을 경우, 세 번째 자리 수는 아주 미묘한 변화가 있을 경우 1씩 올려서 사용한다.  
모듈 버전을 관리할 때 "x.x.x"는 세 자리 수 모두 똑같은 버전을 설치하라는 의미이다.  
`~`는 두 자리 자리 수까지 똑같은 버전을 설치하라는 의미이다. 예를 들어 `~0.0.1`은 `>=0.0.1 <0.1.0`을 의미한다.  
`^`은 첫 번재 자리 수까지 똑같은 버전을 설치하라는 의미이다. 예를 들어 `^1.0.2`은 `>=1.0.2 <2.0`을 의미한다.

##### package.json vs package-lock.json(yarn.lock)
_참고: https://velog.io/@songyouhyun/Package.json%EA%B3%BC-Package-lock.json%EC%9D%98-%EC%B0%A8%EC%9D%B4 , https://hyunjun19.github.io/2018/03/23/package-lock-why-need/_  

[geeks for geeks](https://www.geeksforgeeks.org/difference-between-package-json-and-package-lock-json-files/)에 나온 내용을 먼저 표로 정리해보면 다음과 같다.  

| package.json                                                 | pacakge-lock.json                                                 |
| ---------------------------------------------------------- | ---------------------------------------------------------- |
| It contains basic information about the project.       | It describes the exact tree that was generated to allow subsequent installs to have the identical tree.                                   |
| It is mandatory for every project.                    | It is automatically generated for those operations where npm modifies either node_modules tree or package.json(install or up 등이 발생할 때).                                     |
| It records important metadata about the project.                 | It allows future devs to install the same dependencies in the project.                              |
| It contains information such as name, description, author, script, and dependencies.                          | It contains the name, dependencies, and locked version of the project.               |

위 표에서 나온 것처럼 package-lock.json은 모듈의 '정확한 버전 정보'를 가지고 있기 때문에 package-lock.json이 존재하면 `npm install(yarn)`은 package.json을 이용해서 node_modules를 생성하지 않고 package-lock.json을 이용해서 node_modules를 생성한다.  

처음부터 정확한 정보를 package.json에 명시하면 되지 않겠냐고 생각하겠지만,  
package-lock.json을 별도로 사용하면 프로젝트의 패키지의 중요한 버그 수정이 이루어질 때마다 프로젝트의 package.json에 적혀 있는 버전을 항상 수정하지 않고 version range(^나 `)로 해결할 수 있다.  

### 3) Chrome v8 engine
이벤트 루프

Node.js는 싱글 스레드?


