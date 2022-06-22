## 0. 공부하게 된 계기
double nc에서 다양한 프로젝트가 있었는데 그 중 일부는 node modules로 패키지를 관리하지 않고 yarn berry로 패키지를 관리하고 있었다.  
node modules로 관리할 때의 단점을 많이 커버할 수 있다고 한다.  
참고한 블로그는 https://toss.tech/article/node-modules-and-yarn-berry 이다.
  
## 1. node_modules의 단점
비효율적인 것은 잘 몰랐으나 node modules를 사용하다 보면 패키지가 깨지는 경우가 많았다.  
비효율성과 패키지 관리를 위해 yarn v2에 출시되었다고 한다.  

##### 단점 1) 비효율적인 의존성 검색
npm은 파일 시스템을 이용하여 의존성을 관리한다. 이 때 의존성 검색은 비효율적으로 동작한다.  
node의 'require.resolve.path()' 함수를 사용하면 npm이 검색하는 디렉토리의 목록을 반환하는데, 이 명령어를 통해 볼 수 있듯이 npm은 패키지를 찾기 위해서 계속 상위 디렉토리의 node_modules 폴더를 탐색한다.  
아래 결과는 'require.resolve.path()'를 사용한 내용이다.  
```
[
  '/Users/toss/dev/toss-frontend-libraries/repl/node_modules',
  '/Users/toss/dev/toss-frontend-libraries/node_modules',
  '/Users/toss/node_modules',
  '/Users/node_modules',
  '/node_modules',
  '/Users/toss/.node_modules',
  '/Users/toss/.node_libraries',
  '/Users/toss/.nvm/versions/node/v12.16.3/lib/node',
  '/Users/toss/.node_modules',
  '/Users/toss/.node_libraries',
  '/Users/toss/.nvm/versions/node/v12.16.3/lib/node'
]
```
따라서 패키지를 바로 찾지 못할수록 readdir, stat와 같은 느린 I/O호출을 반복하고 경우에 따라서는 호출이 실패하기도 한다.

##### 단점 2) 환경에 따라 달라지는 동작
위에서 말한 것처럼 npm은 패키지를 찾지 못할 때 상위 디렉토리의 node_modules 폴더를 계속 검색하기 때문에, 어떤 의존성을 찾을 수 있는지는 해당 패키지의 상위 디렉토리 환경에 따라 달라진다.  
이 때문에 다른 버전의 의존성을 잘못 불러올 수 있는 여지도 존재한다.  

##### 단점 3) 비효율적인 설치
npm이 구성하는 node_modules 디렉토리 구조는 매우 큰 공간을 차지한다.  
이로써 많은 I/O 작업을 요구하고 yarn v1이나 npm은 기본적인 의존성 트리의 유효성까지만 검증하고, 각 패키지의 내용이 올바른지는 확인하지 않는다.

##### 단점 4) 유령 의존성
npm 또는 yarn v1은 중복 설치를 방지하기 위해 hoisting 기법을 사용한다.  
hoisting으로 인해 직접 의존하고 있지 않은 라이브러리를 require할 수 있지만 이 때문에 유령 의존성이 존재한다.  
package.json에 명시하지 않은 라이브러리를 조용히 사용할 수 있지만, package.json에서 제거했을 때 의도치 않게 사라질 수 있기 때문에 의존성 관리 시스템이 혼란스러워진다.

## 2. yarn berry의 등장
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

##### 장점 1) 의존성 검색의 효율성 증가
더이상 node_modules 폴더를 순회할 필요 없이 .pnp.cjs 파일이 제공하는 자료구조를 이용하여 바로 의존성의 위치를 찾기 때문에 require()에 걸리는 시간이 단축된다.

##### 장점 2) 재현 가능성
패키지의 모든 의존성이 .pnp.cjs 파일을 이용해 관리하기 때문에 더 이상 외부 환경에 영향을 받지 않는다.  
이로써 다양한 기기 및 CI 환경에서도 require() 또는 import 문의 동작이 동일할 것임을 보장할 수 있다.

##### 장점 3) 반복적인 의존성 설치 작업을 단축
npm이 설치하는 것처럼 같은 버전의 패키지가 여러 번 복사되지 않아 설치 시간을 단축할 수 있다.  
이에 더해 zero install을 사용하면 대부분 라이브러리를 설치 없이 사용할 수 있다.

##### 장점 4) 엄격한 의존성 관리
hoisting을 사용하지 않기 때문에, 예기치 못한 버그를 쉽게 일으키던 유령 의존성 현상을 막을 수 있다.

##### 장점 5) 의존성 검증
zip 파일을 이용해 패키지를 관리하기 대문에 빠진 의존성을 찾거나 의존성 파일이 변경되었음을 찾기 쉽다.
  
## 3. 추가사항: zero-install
의존성도 git을 이용해 버전관리를 한다는 개념이다.  
yarn pnp는 의존성을 압축 파일로 관리하기 때문에 의존성의 용량이 작다.  
이 때문에 의존성도 git으로 관리할 수 있다.  

##### 장점 1) 새로 저장소를 복제하거나 브랜치를 바꾸었다고 해서 yarn (install)을 실행하지 않아도 된다.
##### 장점 2) CI에서 의존성 설치하는 시간을 크게 절약할 수 있다.

## 4. 사용법
참고: https://velog.io/@altmshfkgudtjr/yarn2%EC%99%80-%ED%95%A8%EA%BB%98-Plug-n-Play%EB%A5%BC-%EC%A0%81%EC%9A%A9%ED%95%B4%EB%B3%B4%EC%9E%90  
npm으로 yarn을 install 한 뒤 `yarn set version berry`를 이미 했다고 가정한 뒤 진행한다.

##### 1) 기존 파일 수정
* .npmrc -> .yarnrc.yml으로 이름 변경. 만약 pnp 방식을 사용하지 않겠다면 .yarnrc.yml에 `nodeLinker: node-modules` 추가
* package.lock.json이 존재한다면 제거
* node_modules 제거
* package.json 내에 eslintConfig는 .eslintrc.json으로 별도 생성

##### 2) 모듈 설치
* `yarn` 또는 `yarn install`

##### 3) .gitignore 설정
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

##### 4) typescript 설정
typescript를 사용하는 프로젝트라면 설정을 변경한다.  

* jsconfig.json -> tsconfig.json 이름 변경
* `yarn add -D typescript @types/node @typesjest ...나머지는 프레임워크에 따라 추가 설치`

##### 5) vscode 설정

* _ZipFS_ 확장자를 VSCode extension에서 설치
* 프로젝트 최상위 경로에서 `yarn dlx @yarnpkg/sdks vscode` 설치. 기존에 있던 sdks가 .yarn 디렉토리 내부에 별도로 설치됨.
* http://daplus.net/typescript-visual-studio-code%EB%8A%94-%EC%96%B4%EB%96%A4-typescript-%EB%B2%84%EC%A0%84%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%98%EB%82%98%EC%9A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%97%85%EB%8D%B0/ 를 참고하여 typescript 버전 선택

