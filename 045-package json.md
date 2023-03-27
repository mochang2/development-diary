## 0. 공부하게 된 계기

[vanilla로 React 만들기](https://github.com/mochang2/vanilla-to-react)를 도전하다 보니 내가 만든 React를 하나의 모듈로 만들고자 하는 욕심이 생겼다.  
그래서 배포하려고 하는데, 그 과정에서 약간 애 먹은 내용도 있었고 `package.json`의 옵션을 잘 몰라 눈치껏 적은 내용도 있었다.  
그래서 해당 내용을 좀 정리하고자 한다.

## 1. 배포하면서 배운 새로운 사실들

### 모듈 번들러 production 버전을 사용할 필요가 없다.

나는 webpack을 사용했는데, productin 버전으로 빌드하면 난독화가 진행된다.  
그래서 해당 내용을 다른 프로젝트에서 install한 뒤 사용하려고 하면 `useState`와 같은 기존 이름이 망가져서 사용하고자 하는 함수를 찾질 못 한다.

production 버전 빌드의 목표는 보통 난독화 또는 용량 최소화이고 사용 중인 `node_modules`도 빌드 산출물에 포함되므로, 내가 만든 모듈을 사용하는 프로젝트를 production 버전으로 빌드하면 된다.  
내가 만든 모듈은 JS로 만들었다면 바벨을 통한 트랜스파일, TS로 만들었다면 tsc를 통한 트랜스파일을 거치면 된다.

나는 TS로 만들었으며 설정 파일은 다음과 같았다.

```json
// package.json
{
  "typings": "dist/types.d.ts"
}
```

```json
// tsconfig.json
{
  "compilerOptions": {
    "outDir": "./dist/",
    "noImplicitAny": true,
    "module": "commonjs",
    "target": "es5",
    "downlevelIteration": true,
    "jsx": "react",
    "moduleResolution": "node",
    "declaration": true,
    "declarationDir": "./dist"
  },
  "include": ["./src/**/*"]
}
```

이렇게 사용하면 es5로 JS 파일로 빌드되며, 타입 파일도 빌드 산출물에 포함된다.

### @를 prefix로 사용하면 돈을 내야 한다.

@는 ['scoped packages'라고 불리는 NPM의 기능](https://github.com/mochang2/development-diary/blob/main/044-Atsign%20meaning%20in%20node_modules.md)이다.  
`@mochang2/react`라고 모듈 이름을 지었다면 나만의 React라고 의미를 부여할 수 있었다.  
하지만 이러한 모듈은 1달에 7달러를 내야 한다.

### 배포를 다시 하려면 반드시 이전 버전과 달라야 한다.

`npm publish`할 때 403 에러가 나서 에러 내용을 자세히 살펴봤다.

> You cannot publish over the previously published versions: X.X.X

간단한 `README.md` 수정이었는데도 버전이 같아서 새로 배포하는 데에 에러가 났다.  
major, minor, patch release를 명확히 구분해야 함을 느꼈다.

## 2. package.json 옵션

### name

모듈의 이름을 나타내며 필수 요소이다.  
버전과 함께 identifier 역할을 한다.

몇 가지 규칙이 있다.

- scope를 포함하여 214글자 이하여야 한다.
- scope이 없다면 .이나 \_로 시작할 수 없다.
- 새로운 패키지는 대문자가 없어야 한다.
- non-URL-safe한 문자를 포함할 수 없다.

몇 가지 팁도 있다.

- 코어 노드 모듈과 같은 이름을 사용하지 않는다.
- "js"나 "node"를 이름에 포함하지 않는다.
- 역시나 이름이기 때문에 의미를 내포하는 이름이 좋다.

### version

모듈의 버전을 나타내며 필수 요소이다.
이름과 함께 identifier 역할을 한다.

[node-semver](https://github.com/npm/node-semver)에 parsing이 가능한 버전을 사용해야 한다.  
유의적 버전에 대한 자세한 의미는 여기서는 다루지 않겠다.

### description

모듈에 대한 설명이다.  
`npm search`에 리스팅되어 모듈을 찾는 데에 도움을 준다.

### keywords

`npm search`에 리스팅되어 모듈을 찾는 데에 도움을 주는 `string[]`이다.

### homepage

프로젝트 홈페이지 URL을 명시한다.

### bugs

이슈를 보고 받을 email 주소나 url이다.  
두 개 모두 명시해도 되고 하나만 사용해도 된다.

### license

사람들이 어떤 용도로 사용할 수 있는지, 그리고 제한이 있는지 등을 명시할 수 있다.  
주로 내가 사용할 것들은 무료 라이센스들(Apache, ISC, BSD, Beerware 등)일 것이다.  
무료 라이센스 관련 자세한 내용은 [다음](https://ko.wikipedia.org/wiki/%EC%9E%90%EC%9C%A0_%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4_%EC%82%AC%EC%9A%A9%EA%B6%8C)을 참조하자.  
다른 사용자에게 어떠한 조건으로도 비공개 또는 미공개 패키지를 사용할 수 있는 권한을 부여하지 않으려는 경우 다음과 같이 사용할 수도 있다.

```json
{
  "license": "UNLICENSED"
}
```

### funding

패키지 유지를 위해 donate 받을 url 등을 명시할 수 있다.  
funding은 `npm fund`라는 subcommand를 사용할 수도 있다.

### files

패키지 install 시 포함될 항목을 설명하는 파일 패턴 배열이다.  
필수 요소는 아니다.  
`.gitignore`랑 문법은 비슷하지만, 전혀 다른 의미를 내포하고 있다.

프로젝트 최상단에 `.npmignore`로 포함하지 않을 파일을 명시할 수 있다.  
다만 `package.json#files`에 포함(include)된 파일들은 `.npmignore`나 `.gitignore`에 의해 exclude될 수 없다.

아래 파일들은 `.npmignore`에 명시해도 반드시 포함되는 파일들이다.

- `package.json`
- `README` (대소문자 구분 x, 확장자 구분 x)
- `LICENSE` (대소문자 구분 x, 확장자 구분 x)
- "main" 필드에 존재하는 파일

반면 반드시 ignore되는 파일들도 있다.

- `.git`
- `CVS`
- `.svn`
- `.hg`
- `.lock-wscript`
- `.wafpickle-N`
- `.*.swp`
- `.DS_Store`
- `._*`
- `npm-debug.log`
- `.npmrc`
- `node_modules`
- `config.gypi`
- `*.orig`
- `package-lock.json` (use `npm-shrinkwrap.json` if you wish it to be published)

### main

프로그램의 entry point를 나타낸다.  
패키지 이름이 `foo`이고, `require("foo")`를 한다면, 패키지 이용자는 main이 export하는 모듈을 사용하게 된다.

패키지 root에서 상대 경로로 명시되어야 한다.

만약 "main"을 명시하지 않는다면 `index.js`가 기본으로 설정된다.

### browser

모듈이 client-side에서 사용될 예정이라면 "main" 대신 "browser" 필드를 사용해야 한다.  
`window`와 같이 Node.js 모듈이 사용 가능하지 않은 것들을, 사용하는 모듈이라는 힌트를 줄 수 있다.

### bin

명령 이름을 로컬 파일 이름에 매핑하는 json이다.  
대부분의 패키지에는 PATH에 설치할 실행 파일이 하나 이상 있다.  
npm을 사용하면 이 기능을 사용하여 "npm" 실행 파일을 설치할 수 있다.

이 패키지를 전체적으로 설치하면 해당 파일이 글로벌 bins 디렉토리 내에 연결되거나 bin 필드에서 지정된 파일을 실행하는 cmd(Windows 명령 파일)가 생성되므로 name 또는 name.cmd(Windows PowerShell에서)로 실행할 수 있다.

다음과 같이 "bin"이 존재한다고 하자.

```json
{
  "bin": {
    "myapp": "./cli.js"
  }
}
```

`myapp`을 설치하면 OS는 `cli.js`에서 `/user/local/bin/mapp` 또는 `C:\Users\{Username}\AppData\Roaming\npm\myapp.cmd`로 symlink를 연결해 `cli.js`를 실행할 수 있게끔 해준다.

### directories.bin

"directories.bin"에 "bin" 디렉토리를 지정하면 해당 폴더의 모든 파일이 추가된다.

다만 bin 지시어가 작동하는 방식 때문에 "bin" 경로와 "directories.bin" 설정을 모두 지정하는 것은 안 된다.  
단일 파일을 지정하려면 "bin"을 사용하고 기존 bin 디렉토리의 모든 파일에 대해 "directories.bin"을 사용한다.

### repository

패키지 코드의 위치를 명시한다(보통 깃헙, 깃랩 등의 주소).  
다만 반드시 URL 형식일 필요는 없다.  
`github:user/repo`와 같이 레포 shortcut으로도 사용할 수 있다.

### config

업그레이드 후에도 유지되는 패키지 스크립트에 사용되는 구성 매개 변수를 설정하는 데에 사용한다.

### peerDependencies

플러그인(라이브러리와 패키지의 호환성을 표현하지만 호스트를 필요로 하지 않는 경우)을 나타내는 데 사용한다.

```json
{
  "name": "tea-latte",
  "version": "1.3.5",
  "peerDependencies": {
    "tea": "2.x"
  }
}
```

위와 같이 선언하면 이렇게 하면 `tea-latte`는 오직 호스트 패키지 `tea`의 2점대 버전과 함께 설치할 수 있다.  
`npm install tea-latte`는 다음과 dependency를 생성한다

```
├── tea-latte@1.3.5
└── tea@2.2.0
```

npm 3~6까지는 "peerDependencies"를 자동으로 설치하지 않았지만 7부터는 자동으로 설치한다.

dependency를 올바르게 확인할 수 없는 경우 요구 사항이 충돌하는 다른 플러그인을 설치하려고 하면 오류가 발생할 수 있다.  
따라서 플러그인 요구 사항이 가능한 광범위하고 특정 패치 버전으로 제한되지 않도록 해야 한다.

### peerDependenciesMeta

사용자가 패키지를 설치할 때 "peerDependencies"에 지정된 패키지가 아직 설치되지 않은 경우 npm이 경고를 발생시킨다.  
"peerDependenciesMeta"는 "peerDependencies"이 사용되는 방법에 대한 npm 추가 정보를 제공하는 역할을 한다.  
특히 어떤 플러그인은 "optional하다"라고 표현해, 필수적으로 설치하지 않아도 에러가 발생하지 않게 할 수 있다.

### bundleDependencies

패키지를 publish할 때 번들로 제공되는 패키지 이름 배열을 정의한다.

npm 패키지를 로컬로 보존하거나 단일 파일 다운로드를 통해 사용할 수 있어야 하는 경우 종속성 번들 배열에 패키지 이름을 지정하고 `npm pack`을 실행하여 tarball 파일로 패키지를 번들할 수 있다.

```json
{
  "name": "awesome-web-framework",
  "version": "1.0.0",
  "bundleDependencies": ["renderized", "super-streams"]
}
```

`npm pack`을 진행하면 `awesome-web-framework-1.0.0.tgz` 파일이 생성된다.  
해당 파일은 dependency인 `renderized`, `super-stream`를 포함하고 있으며, 이 모듈을 설치하고자 하면 `npm install awesome-web-framework-1.0.0.tgz`을 진행하면 명시한 dependency들이 포함된 패키지가 설치된다.  
참고로 "bundleDependencies"에는 버전을 명시하지 않는데 이는 "dependencies"에 명시되기 때문이다.

만약 "bundleDependencies"를 `true`로 설정하면 "dependencies"에 명시된 모든 파일들이 번들에 포함된다.

### overrides

종속성 버전을 알려진 보안 문제로 바꾸거나, 기존 종속성을 포크로 바꾸거나, 모든 위치에서 동일한 버전의 패키지가 사용되는지 확인하는 등 종속성의 종속성을 구체적으로 변경해야 하는 경우 사용한다.  
"override"를 통해 dependency의 패키지를 다른 버전으로 바꾸거나 다른 패키지 전체로 바꿀 수 있다.

`baz`의 자식 모듈인 `bar`, `bar`의 자식 모듈인 `foo`를 1.0.0으로 고정하고 싶다면 다음과 같이 사용할 수 있다.

```json
{
  "overrides": {
    "baz": {
      "bar": {
        "foo": "1.0.0"
      }
    }
  }
}
```

### engines

모듈이 작동할 node 또는 npm 버전을 명시하는 데에 사용한다.

### OS, cpu

모듈이 작동할 OS(`process.platform`), cpu를 `string[]`형태로 명시하는 데에 사용한다.

### private

"private"이 `true`라면 npm은 publish를 못하게 한다.  
private으로 유지되어야 하는 모듈이 publicly하게 공개되는 사고를 미연에 방지하기 위한 옵션이다.  
사적으로 사용하는 레지스트리에만 올라가도록 설정하길 원한다면 "publishConfig"를 같이 사용하면 된다.
