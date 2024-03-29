## 0. 공부하게 된 계기

double nc에서 다양한 프로젝트가 있었는데 그 중 일부는 node modules로 패키지를 관리하지 않고 yarn berry로 패키지를 관리하고 있었다.  
node modules로 관리할 때의 단점을 많이 커버할 수 있다고 한다.

참고
https://toss.tech/article/node-modules-and-yarn-berry  
https://blog.dramancompany.com/2023/02/%EB%A6%AC%EB%A9%A4%EB%B2%84-%EC%9B%B9-%EC%84%9C%EB%B9%84%EC%8A%A4-%EC%A2%8C%EC%B6%A9%EC%9A%B0%EB%8F%8C-yarn-berry-%EB%8F%84%EC%9E%85%EA%B8%B0/  
https://kasterra.github.io/setting-yarn-berry/

## 1. 사전 지식 - Node.js

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

#### 프론트엔드 개발에서도 Node.js가 필요한 이유

1. **최신 스펙으로 개발 가능.** JS의 스펙 발전 속도에 비해 브라우저의 발전 속도는 느림. 바벨과 같은 도구를 사용하며 웹팩, NPM 같은 노드 기술로 만들어진 환경을 사용해야 프론트엔드 개발환경을 갖출 수 있음.
2. **빌드 자동화.** 코딩 결과물을 바로 서버에 올리지 않음. 파일을 압축하고, 코드를 난독화하고, 폴리필을 추가하는 등 개발 이외의 작업이 필요. 추가적으로 라이브러리 의존성이나 테스트를 자동화할 수도 있음.
3. **개발 환경 커스터마이징.** CRA로 제공되는 것 이외에 커스터마이징이 필요할 수도 있음.

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

### `yarn`

_참고: https://developer0809.tistory.com/128_

[위키피디아](<https://ko.wikipedia.org/wiki/Yarn_(%ED%8C%A8%ED%82%A4%EC%A7%80_%EA%B4%80%EB%A6%AC%EC%9E%90)>)에 따르면 yarn은 Node.js 런타임 환경을 위해 페이스북이 개발한 소프트웨어 패키지 시스템이다.

다음은 npm과 비교했을 때의 yarn의 장점이다.

- yarn은 로컬 캐시로부터 패키지를 설치할 수 있다.
- yarn은 데이터 무결성 보장을 위해 체크섬을 사용하는 반면 npm은 SHA-512를 사용하여 다운로드된 패키지의 데이터 무결성을 검사한다.
- yarn은 병렬로 패키지를 설치하는 반면, npm은 순차적으로 설치해서 일반적으로 yarn의 다운로드 속도가 빠르다.
- yarn은 yarn.lock 또는 package.json 파일에 있는 파일만 설치한다. 반면에 npm은 다른 패키지를 즉시 포함시킬 수 있는 코드를 자동으로 실행하므로, 보안 시스템에 여러 가지 취약성이 발생한다. 따라서 yarn이 npm보다 보안이 강화된 것으로 간주된다.

+) 2023년 10월 보강된 내용  
[yarn vs npm 2023](https://www.copycat.dev/blog/yarn-vs-npm/)에 여전히 yarn이 더 빠르다고 나오긴 하지만 이는 npm이 로컬 캐시가 없어서 그런 것은 아니다.  
~파이프라인 개선하다가 npm 캐시가 제대로 동작하는 건가 확인하고 싶어서 공부한 결과~ 윈도우 기준 `${Drive}:\Users\${User}\AppData\local\npm-cach\_cacache` 안에 tarball과 메타 데이터 파일들이 해시된 이름으로 존재한다.  
기본 캐시 기간은 `npm config set` 명령어나 `.npmrc` 파일을 통해 수정하지 않는 한 영원하다.  

`npm install`은 로컬 캐시를 참조하는 과정을 포함하여 다음과 같은 순서로 동작한다.  

1. 캐시 확인: npm 캐시에 존재하는지 확인한다.
2. 캐시 hit: 캐시를 바라보는 Symbolic link 또는 Hard link를 `${project_home}/node_modules`에 생성한다(복사랑은 약간 다름). 이러한 방법을 통해 디스크 공간을 효율적으로 활용하고, 네트워크를 통한 설치를 최소화함으로써 설치 속도를 향상시킬 수 있다.
3. 캐시 miss: 캐시에 존재하지 않는다면 npm에서 존재하지 않는 모듈들을 설치하며 npm 캐시를 생성한다.

_cf) 캐시 관련되어 참고할 명령어_  

- `npm install --verbose`: npm install의 자세한 과정(캐시 validation 포함)을 알 수 있음.
- `npm install`은 기본적으로 npm 캐시를 생성.
- `npm cache add <pacakge-name>@<version>`: npm 캐시 생성.

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

### `package.json`

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
다만 [github issue](https://github.com/facebook/create-react-app/issues/10219)를 참고하면 알 수 있듯이 CRA로 react를 시작하면 모든 패키지들이 dependencies에 설치된다.  
이는 빌드 중에 모든 것이 번들로 제공되고 웹팩에 의해 사용하지 않는 모듈들은 제거되기 때문에 실제로 런타임 의존성이 없다.  
즉, react에서는(~front에서는 이라고 표현해도 틀리지 않을 듯~) 둘을 구분하는 것이 의미가 없다.

참고로 모듈 버전은 일반적으로 "x.x.x"로 구성되는데, 첫 번째 자리 수는 대대적인 변화가 있을 경우, 두 번째 자리 수는 버그 픽스 등 적당한 변화가 있을 경우, 세 번째 자리 수는 아주 미묘한 변화가 있을 경우 1씩 올려서 사용한다.  
모듈 버전을 관리할 때 "x.x.x"는 세 자리 수 모두 똑같은 버전을 설치하라는 의미이다.  
`~`는 두 번째 자리 수까지 똑같은 버전을 설치하라는 의미이다. 예를 들어 `~0.0.1`은 `>=0.0.1 <0.1.0`을 의미한다.  
`^`은 첫 번째 자리 수까지 똑같은 버전을 설치하라는 의미이다. 예를 들어 `^1.0.2`은 `>=1.0.2 <2.0`을 의미한다.

### `package.json` vs `package-lock.json`(`yarn.lock`)

_참고: https://velog.io/@songyouhyun/Package.json%EA%B3%BC-Package-lock.json%EC%9D%98-%EC%B0%A8%EC%9D%B4 , https://hyunjun19.github.io/2018/03/23/package-lock-why-need/_

[geeks for geeks](https://www.geeksforgeeks.org/difference-between-package-json-and-package-lock-json-files/)에 나온 내용을 먼저 표로 정리해보면 다음과 같다.

| package.json                                                                         | pacakge-lock.json                                                                                                                             |
| ------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------- |
| It contains basic information about the project.                                     | It describes the exact tree that was generated to allow subsequent installs to have the identical tree.                                       |
| It is mandatory for every project.                                                   | It is automatically generated for those operations where npm modifies either node_modules tree or package.json(install or up 등이 발생할 때). |
| It records important metadata about the project.                                     | It allows future devs to install the same dependencies in the project.                                                                        |
| It contains information such as name, description, author, script, and dependencies. | It contains the name, dependencies, and locked version of the project.                                                                        |

위 표에서 나온 것처럼 package-lock.json은 모듈의 '정확한 버전 정보'를 가지고 있기 때문에 package-lock.json이 존재하면 `npm install(yarn add)`은 package.json을 이용해서 node_modules를 생성하지 않고 package-lock.json을 이용해서 node_modules를 생성한다.

같은 `package.json`을 사용해도 서로 다른 `node_modules`를 생성하는 경우가 발생한다.

1. npm 버전이 다를 때
2. 버전을 명시하지 않고 version range를 사용할 때
3. 내가 사용하는 패키지를 의존하고 있는 패키지가 새로운 버전으로 배포되었을 때

와 같은 경우이다.

2번과 같은 문제를 해결하기 위해 처음부터 정확한 정보를 package.json에 명시하면 되지 않겠냐고 생각하겠지만,  
`package-lock.json`을 별도로 사용하면 프로젝트 패키지의 중요한 버그 수정이 이루어질 때마다 프로젝트의 `package.json`에 적혀 있는 버전을 항상 수정하지 않고 version range(^ 또는 `)로 해결할 수 있다.

### `node_modules/.package-lock.json`

`npm install`로 dependencies에 있는 패키지들을 설치하니 `.package-lock.json`이라는 파일이 있었다.  
[공식문서](https://docs.npmjs.com/cli/v8/configuring-npm/package-lock-json)를 확인해보니 `node_modules` 폴더를 반복적으로 처리하지 않기 위해 v7부터 추가된 '숨겨진' lock 파일이라고 한다.

이 파일은 트리에 대한 정보를 포함하며, 다음 조건이 충족될 경우 `node_modules` 계층을 읽는 대신에 사용된다.

- 참조하는 모든 패키지 폴더가 `node_modules` 계층에 있고,
- `node_modules` 계층 구조에 lock 파일이 추적하지 않는 패키지 폴더가 없으며,
- 파일의 수정 시간이 최신이어야 한다. 즉, 이 숨겨진 lock 파일은 패키지 트리의 최신 업데이트의 일부로 생성된 경우에만 관련이 있다.

세 번재 조건에 대한 부연 설명이다.  
직접 수동으로 `node_modules/foo/lib/bar.js`라는 파일을 추가하면 `node_modules/foo`의 수정된 시간이 이 변경 사항을 반영하지 않으므로 `node_modules/.package-lock.json`에서 파일을 삭제해야 된다는 말이다.

## 2. `node_modules`의 단점

비효율적인 것은 잘 몰랐으나 node modules를 사용하다 보면 패키지가 깨지는 경우가 많았다.  
비효율성과 패키지 관리를 위해 yarn v2에 출시되었다고 한다.

### 단점 1) 비효율적인 의존성 검색

npm은 파일 시스템을 이용하여 의존성을 관리한다.  
이 때 의존성 검색은 비효율적으로 동작한다.  
node의 `require.resolve.paths()` 함수를 사용하면 npm이 검색하는 디렉토리의 목록을 반환한다.  
이 명령어를 통해 볼 수 있듯이 npm은 패키지를 찾기 위해서 계속 상위 디렉토리의 `node_modules` 폴더를 탐색한다.  
아래 결과는 (토스 블로그에서) `require.resolve.paths('node_modules')`를 사용한 내용이다.

```sh
[
  '/Users/toss/dev/toss-frontend-libraries/repl/`node_modules`',
  '/Users/toss/dev/toss-frontend-libraries/`node_modules`',
  '/Users/toss/`node_modules`',
  '/Users/`node_modules`',
  '/`node_modules`',
  '/Users/toss/.`node_modules`',
  '/Users/toss/.node_libraries',
  '/Users/toss/.nvm/versions/node/v12.16.3/lib/node',
  '/Users/toss/.`node_modules`',
  '/Users/toss/.node_libraries',
  '/Users/toss/.nvm/versions/node/v12.16.3/lib/node'
]
```

따라서 패키지를 바로 찾지 못할수록 `readdir`, `stat`와 같은 느린 I/O호출을 반복하고 경우에 따라서는 호출이 실패하기도 한다.

### 단점 2) 환경에 따라 달라지는 동작

위에서 말한 것처럼 npm은 패키지를 찾지 못할 때 상위 디렉토리의 `node_modules` 폴더를 계속 검색하기 때문에, 어떤 의존성을 찾을 수 있는지는 해당 패키지의 상위 디렉토리 환경에 따라 달라진다.  
이 때문에 다른 버전의 의존성을 잘못 불러올 수 있는 여지도 존재한다.

억지로 만든 환경이긴 하지만 이를 증명하는 방법은 의외로 간단하다.

1. (`react`가 없는)특정 프로젝트를 `package.json`과 `package-lock.json`이 포함된 상태로 깃에 올린다.
2. 해당 프로젝트에서 내에서 `node`를 실행한 후 `require('react')`를 진행하면 에러가 난다.
3. `git clone`으로 해당 프로젝트를 `react`가 설치된 프로젝트 내부에 설치한다.
4. `npm install`을 진행한 후 `require('react')`를 하면 에러가 나지 않고 상위에 설치된 `react`를 잘 찾는다.

### 단점 3) 비효율적인 설치

npm이 구성하는 `node_modules` 디렉토리 구조는 매우 큰 공간을 차지한다.  
이로써 많은 I/O 작업을 요구하고 yarn v1이나 npm은 기본적인 의존성 트리의 유효성까지만 검증하고, 각 패키지의 내용이 올바른지는 확인하지 않는다.

### 단점 4) 유령 의존성

npm 또는 yarn v1은 중복 설치를 방지하기 위해 hoisting 기법을 사용한다.  
hoisting으로 인해 직접 의존하고 있지 않은 라이브러리를 `require`할 수 있지만 이 때문에 유령 의존성이 존재한다.  
`package.json`에 명시하지 않은 라이브러리를 조용히 사용할 수 있지만, `package.json`에서 제거했을 때 의도치 않게 사라질 수 있기 때문에 의존성 관리 시스템이 혼란스러워진다.

## 3. yarn berry의 등장

위의 문제를 해결하기 위해 yarn berry는 pnp(Plug n Play) 전략을 사용한다. 아래 terminal 명령어를 통해 yarn berry를 사용할 수 있다.

```sh
npm install -g yarn
cd ../path/to/some-package
yarn set version berry

# yarn set version berry가 안되면 yarn policies set-version을 사용
```

yarn berry는 `node_modules` 디렉토리를 생성하지 않는다.  
대신 .yarn/cache 폴더에 의존성의 정보가 저장되고, `.pnp.cjs` 파일에 의존성을 찾을 수 있는 정보가 기록된다.  
`.pnp.cjs`를 이용하면 디스크 I/O 없이 어떤 패키지가 어떤 라이브러리에 의존하는지, 각 라이브러리는 어디에 위치하는지를 바로 알 수 있다.

### 장점

#### 1) 의존성 검색의 효율성 증가

더이상 `node_modules` 폴더를 순회할 필요 없이 `.pnp.cjs` 파일이 제공하는 자료구조를 이용하여 바로 의존성의 위치를 찾기 때문에 `require()`에 걸리는 시간이 단축된다.

#### 2) 재현 가능성

패키지의 모든 의존성이 `.pnp.cjs` 파일을 이용해 관리하기 때문에 더 이상 외부 환경에 영향을 받지 않는다.  
이로써 다양한 기기 및 CI 환경에서도 `require()` 또는 `import` 문의 동작이 동일할 것임을 보장할 수 있다.

#### 3) 반복적인 의존성 설치 작업을 단축

npm이 설치하는 것처럼 같은 버전의 패키지가 여러 번 복사되지 않아 설치 시간을 단축할 수 있다.  
이에 더해 zero install을 사용하면 대부분 라이브러리를 설치 없이 사용할 수 있다.

#### 4) 엄격한 의존성 관리

hoisting을 사용하지 않기 때문에, 예기치 못한 버그를 쉽게 일으키던 유령 의존성 현상을 막을 수 있다.

#### 5) 의존성 검증

zip 파일을 이용해 패키지를 관리하기 때문에 빠진 의존성을 찾거나 의존성 파일이 변경되었음을 찾기 쉽다.  
`node_modules`를 이용해 의존성을 관리했을 때는 폴더의 의존성 검증이 어려웠기 때문에 차라리 `node_modules`를 전부 지우고 다시 설치해야 했다.

#### 6) zero-install

의존성도 git을 이용해 버전관리를 한다는 개념이다.  
yarn pnp는 의존성을 압축 파일로 관리하기 때문에 의존성의 용량이 작다.  
이 때문에 의존성도 git으로 관리할 수 있다.

## 4. `.yarn` 구조

### 1) cache

프로젝트의 의존성 패키지를 캐싱하는 데 사용되는 파일들이 저장된다.  
이 캐시는 로컬 머신에서 패키지를 다운로드하거나 패키지를 빌드할 때 사용된다.  
나중에 동일한 패키지를 다시 설치할 때 다시 다운로드할 필요 없이 캐시된 버전을 사용할 수 있다(zero-install).  
기존에 `npm install`을 시행할 때 설치되던 패키지들이 압축된 형태(zip 또는 tarball)로 저장된다.

### 2) releases

yarn berry 자체의 실행 가능한 파일(`yarn-berry.cjs`)이 저장된다.  
즉, yarn berry의 실행 파일이다.

### 3) sdks

eslint, typescript 등 해당 프로젝트를 진행할 때 필요한 sdk가 설치된다.

### 4) unplugged

> A package being unplugged means that instead of being referenced directly through its archive, it will be unpacked at install time in the directory configured via pnpUnpluggedFolder. Note that unpacking packages this way is generally not recommended because it'll make it harder to store your packages within the repository. However, it's a good approach to quickly and safely debug some packages, and can even sometimes be required depending on the context (for example when the package contains shellscripts).

바이너리 종속성이 있는 경우(주로 C/C++로 작성된 패키지) 또는 필수 빌드 의존성이 있는 경우, 즉 컴파일러와 같은 복잡한 의존성이 필요한 패키지의 경우 컴파일된 결과물이 저장된다.  
unplugged를 이용하여 패키지 설치 시간을 단축하고, 의존성을 컴파일하는 데 필요한 빌드 도구의 부담을 줄일 수 있다.

다만 이 때문에 서로 다른 OS에서 개발을 진행할 경우 호환되지 않을 수 있다.  
실제로 나는 Windows11에서 진행하던 프로젝트를 종종 Windows10에서 진행하려고 하니 `next-swc` 같은 패키지가 Windows10 운영 체제에 맞게 컴파일되지 않아 호환되지 않았다.  
그래서 애초에 이 부분은 gitignore하는 게 맞다.

+) 만약 gitignore하지 않았었더라면  
`.yarn/unplugged`를 삭제한 후 `yarn install --immutable`을 통해 `.yarn/unplugged`를 재설치해야 한다.

### 5) install-state.gz

## 5. 사용법

참고: https://velog.io/@altmshfkgudtjr/yarn2%EC%99%80-%ED%95%A8%EA%BB%98-Plug-n-Play%EB%A5%BC-%EC%A0%81%EC%9A%A9%ED%95%B4%EB%B3%B4%EC%9E%90  
npm으로 yarn을 설치한 뒤 `yarn set version berry`를 이미 했다고 가정한 뒤 진행한다.

### 1) 기존 파일 수정

- `.npmrc -> .yarnrc.yml`으로 이름 변경. 만약 pnp 방식을 사용하지 않겠다면 `.yarnrc.yml`에 `nodeLinker: node-modules` 추가 (`.npmrc`는 npm 설정을 저장해두는 파일)
- `package-lock.json`이 존재한다면 제거
- `node_modules` 제거
- `package.json` 내에 eslintConfig는 `.eslintrc.json`으로 별도 생성

### 2) 모듈 설치

- `yarn` 또는 `yarn install`

### 3) .gitignore 설정

```
# Zero-Install 사용
.yarn/*
!.yarn/cache
!.yarn/patches
!.yarn/plugins
!.yarn/releases
!.yarn/sdks
!.yarn/versions

# Zero-Install 사용 x
.yarn/*
!.yarn/patches
!.yarn/releases
!.yarn/plugins
!.yarn/sdks
!.yarn/versions
.pnp.*
```

### 4) typescript 설정

typescript를 사용하는 프로젝트라면 설정을 변경한다.

- jsconfig.json -> tsconfig.json 이름 변경
- `yarn add -D typescript @types/node @types/jest ...나머지는 프레임워크에 따라 추가 설치`

### 5) vscode 설정

- _ZipFS_ 확장자를 VSCode extension에서 설치
- 프로젝트 최상위 경로에서 `yarn dlx @yarnpkg/sdks vscode` 설치. 기존에 있던 sdks가 .yarn 디렉토리 내부에 별도로 설치됨.
- http://daplus.net/typescript-visual-studio-code%EB%8A%94-%EC%96%B4%EB%96%A4-typescript-%EB%B2%84%EC%A0%84%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%98%EB%82%98%EC%9A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%97%85%EB%8D%B0/ 를 참고하여 typescript 버전 선택
