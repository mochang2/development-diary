## 0. 공부하게 된 계기

React? Vue? 아무리 개발을 해봐도 근본을 모르면 말짱 도루묵인 것 같다.  
~과제 테스트 통과 못하는 건 덤~  
우테캠으로 과제 전형을 보고, vanilla js로 [virtual DOM을 비슷하게 따라 만드는 강의](https://junilhwang.github.io/TIL/Javascript/Design/Vanilla-JS-Virtual-DOM/)가 있다는 것을 알고 공부해야겠다 마음을 먹었다.

## 1. babel

 js 컴파일러. 소스 코드를 웹 브라우저가 처리할 수 있는 JS 버전으로 변환. 새로운 ESNext 문법을 사용하여 개발할 때, 지원하지 않는 브라우저에서 오류가 나지 않도록 하기 위해.
.babelrc

+ polyfill
=> 개발자가 특정 기능이 지원되지 않는 브라우저를 위해 사용할 수 있는 코드 조각이나 플러그인.

## 2. webpack

모듈 번들러. 주요 목적은 브라우저에서 사용할 수 있도록 JavaScript 파일을 번들로 묶는 것이지만, 리소스나 애셋(Image, Font 등)을 변환하고 번들링 또는 패키징할 때도 사용됨.
여러 개의 모듈(js, css, html, image 등)을 하나의 js로 묶어주는 모듈 번들러.
jsx를 해석해주는 babel을 적용할 수 있고(jsx -> React.createElement), 코드 최적화 수행, console.log()와 같이 실제 서비스에서는 필요 없는 코드를 자동으로 제거하는 등의 기능.
https://joshua1988.github.io/webpack-guide/webpack/what-is-webpack.html#%EC%9B%B9%ED%8C%A9%EC%97%90%EC%84%9C%EC%9D%98-%EB%AA%A8%EB%93%88

react.createchild

## 3. Node.js

기존의 JS는 HTML에 독립적으로 실행할 수 없는 프로그래밍 언어였다(`<script>` 태그 사용했어야 함).  
하지만 Node.js가 등장하면서 JS를 HTML에 독립적으로 실행할 수 있게 됐다.  
물론 그 이전에도 JS를 HTML에 독립적으로 실행할 수 있도록 만드는 시도가 있었으나 엔진(Chrome V8 JavaScript Engine 이전) 속도 문제로 실패했었다.  
[Node.js 공식 사이트](https://nodejs.org/)에 따르면 Node.js는

> Node.js는 Chrome V8 JavaScript 엔진으로 빌드된 JavaScript 런타임입니다.\
> Node.js는 이벤트 기반, Non-blocking I/O 모델을 사용해 가볍고 효율적입니다.\
> 이는 (I/O를 직접 수행하지 않으므로) 사용자가 프로세스의 교착상태에 대해 걱정할 필요가 없다는 말이기도 합니다.\
> Node.js의 패키지 생태계인 npm은 세계에서 가장 큰 오픈 소스 라이브러리 생태계이기도 합니다.\
> Node.js는 이벤트 루프를 시작하는 호출이 없으며, 각 연결에서 콜백이 실행되는데 실행할 작업이 없다면 Node.js는 대기합니다.
> Node.js는 스레드를 사용하지 않도록 설계되었지만 멀티 코어 환경의 장점을 얻지 못하는 것은 아닙니다. 'child_process.fork()' API를 사용해 자식 프로세스를 생성할 수 있습니다.

_cf) 런타임: 특정 언어로 만든 프로그램들을 실행할 수 있게 해주는 가상 머신의 상태. 다른 런타임으로는 웹 브라우저(크롬, 사파리 등)이 있음._

공식 사이트의 설명대로 Node.js는 서버의 역할도 수행할 수 있는 JS 런타임이다.  
Node.js는 서버 실행을 위해 필요한 http/https/http2 모듈을 제공한다.

이러한 특성 때문에 Node.js는 다음과 같은 개발에 용이하다.

- 정적 파일 서버
- 웹 응용프로그래밍
- 메시지 미들웨어
- HTML5 멀티 플레이어 게임용 서버

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
> The CLI runs from a terminal, and is how most developers interact with npm.  
> The registry is a large public database of JavaScript software and the meta-information surrounding it.

### yarn

_참고: https://developer0809.tistory.com/128_

[위키피디아](<https://ko.wikipedia.org/wiki/Yarn_(%ED%8C%A8%ED%82%A4%EC%A7%80_%EA%B4%80%EB%A6%AC%EC%9E%90)>)에 따르면 yarn은 Node.js 런타임 환경을 위해 페이스북이 개발한 소프트웨어 패키지 시스템이다.

다음은 npm과 비교했을 때의 yarn의 장점이 있다.

- yarn은 로컬 캐시로부터 패키지를 설치할 수 있다.
- yarn은 데이터 무결성 보장을 위해 체크섬을 사용하는 반면 npm은 SHA-512를 사용하여 다운로드된 패키지의 데이터 무결성을 검사한다.
- yarn은 병렬로 패키지를 설치하는 반면, npm은 순차적으로 설치해서 일반적으로 yarn의 다운로드 속도가 빠르다.
- yarn은 yarn.lock 또는 package.json 파일에 있는 파일만 설치한다. 반면에 npm은 다른 패키지를 즉시 포함시킬 수 있는 코드를 자동으로 실행하므로, 보안 시스템에 여러 가지 취약성이 발생한다. 따라서 yarn이 npm보다 보안이 강화된 것으로 간주된다.

_cf) 명령어_
동일한 명령어: `npm(yarn) init`, `npm(yarn) run`, `npm(yarn) test`, `npm(yarn) login`, `npm(yarn) logout`, `npm(yarn) link`, `npm(yarn) publish`, `npm(yarn) cache clean`

다른 명령어:

| npm                                  | yarn                           |
| ------------------------------------ | ------------------------------ |
| npm install                          | yarn(yarn install)             |
| npm install \<package\>              | yarn add \<package\>           |
| npm install --save-dev \<package\>   | yarn add --dev \<package\>     |
| npm uninstall \<package\>            | yarn remove \<package\>        |
| npm uninstall --save-dev \<package\> | yarn remove \<package\>        |
| npm update                           | yarn upgrade                   |
| npm update \<package\>               | yarn upgrade \<package\>       |
| npm install --global \<package\>     | yarn global add \<package\>    |
| npm uninstall --global \<package\>   | yarn global remove \<package\> |

### package.json

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
    "react": "0.10.0"
  },
  "devDependencies": {
    "supertest": "^4.0.2"
  }
}
```

devDependencies는 개발할 때만 사용되고 배포할 때는 포함되지 않는 패키지들을 적는 곳으로 `-D` 옵션을 추가해서 설치한다.

참고로 모듈 버전은 일반적으로 "x.x.x"로 구성되는데, 첫 번째 자리 수는 대대적인 변화가 있을 경우, 두 번째 자리 수는 버그 픽스 등 적당한 변화가 있을 경우, 세 번째 자리 수는 아주 미묘한 변화가 있을 경우 1씩 올려서 사용한다.  
모듈 버전을 관리할 때 "x.x.x"는 세 자리 수 모두 똑같은 버전을 설치하라는 의미이다.  
`~`는 두 번째 자리 수까지 똑같은 버전을 설치하라는 의미이다. 예를 들어 `~0.0.1`은 `>=0.0.1 <0.1.0`을 의미한다.  
`^`은 첫 번째 자리 수까지 똑같은 버전을 설치하라는 의미이다. 예를 들어 `^1.0.2`은 `>=1.0.2 <2.0`을 의미한다.

### package.json vs package-lock.json(yarn.lock)

_참고: https://velog.io/@songyouhyun/Package.json%EA%B3%BC-Package-lock.json%EC%9D%98-%EC%B0%A8%EC%9D%B4 , https://hyunjun19.github.io/2018/03/23/package-lock-why-need/_

[geeks for geeks](https://www.geeksforgeeks.org/difference-between-package-json-and-package-lock-json-files/)에 나온 내용을 먼저 표로 정리해보면 다음과 같다.

| package.json                                                                         | pacakge-lock.json                                                                                                                             |
| ------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------- |
| It contains basic information about the project.                                     | It describes the exact tree that was generated to allow subsequent installs to have the identical tree.                                       |
| It is mandatory for every project.                                                   | It is automatically generated for those operations where npm modifies either node_modules tree or package.json(install or up 등이 발생할 때). |
| It records important metadata about the project.                                     | It allows future devs to install the same dependencies in the project.                                                                        |
| It contains information such as name, description, author, script, and dependencies. | It contains the name, dependencies, and locked version of the project.                                                                        |

위 표에서 나온 것처럼 package-lock.json은 모듈의 '정확한 버전 정보'를 가지고 있기 때문에 package-lock.json이 존재하면 `npm install(yarn)`은 package.json을 이용해서 node_modules를 생성하지 않고 package-lock.json을 이용해서 node_modules를 생성한다.

처음부터 정확한 정보를 package.json에 명시하면 되지 않겠냐고 생각하겠지만,  
package-lock.json을 별도로 사용하면 프로젝트 패키지의 중요한 버그 수정이 이루어질 때마다 프로젝트의 package.json에 적혀 있는 버전을 항상 수정하지 않고 version range(^ 또는 `)로 해결할 수 있다.
