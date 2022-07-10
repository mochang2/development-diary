## 1. babel
비효율적인 것은 잘 몰랐으나 node modules를 사용하다 보면 패키지가 깨지는 경우가 많았다.  
비효율성과 패키지 관리를 위해 yarn v2에 출시되었다고 한다.  

##### 단점 1) 비효율적인 의존성 검색

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

##### react.createchild
더이상 node_modules 폴더를 순회할 필요 없이 .pnp.cjs 파일이 제공하는 자료구조를 이용하여 바로 의존성의 위치를 찾기 때문에 require()에 걸리는 시간이 단축된다.


## 3. node
참고: https://velog.io/@altmshfkgudtjr/yarn2%EC%99%80-%ED%95%A8%EA%BB%98-Plug-n-Play%EB%A5%BC-%EC%A0%81%EC%9A%A9%ED%95%B4%EB%B3%B4%EC%9E%90  
npm으로 yarn을 install 한 뒤 `yarn set version berry`를 이미 했다고 가정한 뒤 진행한다.

##### 1) node.js
* .npmrc -> .yarnrc.yml으로 이름 변경. 만약 pnp 방식을 사용하지 않겠다면 .yarnrc.yml에 `nodeLinker: node-modules` 추가
* package.lock.json이 존재한다면 제거
* node_modules 제거
* package.json 내에 eslintConfig는 .eslintrc.json으로 별도 생성

##### 2) npm
* `yarn` 또는 `yarn install`

##### 3) nvm

##### 4) v8 engine

