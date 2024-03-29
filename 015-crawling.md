## 0. 공부하게 된 계기

멤버십과 제휴사에 대한 할인 이벤트를 한 눈에 보여주기 위해서 크롤링 기능을 개발할 일이 생겼다.  
기존에 내 [tensorflow-crawling 레포지토리](https://github.com/mochang2/tensorflow-crawling)에 기록한 것처럼 python의 request 모듈을 이용해서 크롤링 개발을 진행한 적이 있었다.  
그때는 http get request에 대한 응답을 html tag에 따라 파싱한 뒤에 원하는 정보를 빼서 사용했다.  
이번에는 js로 개발을 진행해야 되며, user interaction이 필요한 부분이 있기 때문에, node.js의 `selenium` 라이브러리 위주로 정리할 예정이다.

참고  
https://velog.io/@yesdoing/Node.js-%EC%97%90%EC%84%9C-%EC%9B%B9-%ED%81%AC%EB%A1%A4%EB%A7%81%ED%95%98%EA%B8%B0-wtjugync1m

## 1. 개념

일반적으로 **크롤러** 라는 용어는 명확한 최종 목표나 목표가 없어도 사이트나 네트워크가 제공할 수 있는 것을 끝없이 탐색하면서 스스로 웹 페이지를 탐색할 수 있는 프로그램의 능력을 의미한다.  
반면에 **스크래퍼** 는 특정 데이터를 추출하는 프로세스이다.  
웹 크롤링과 달리 웹 스크래퍼는 특정 웹 사이트 또는 페이지에서 특정 정보를 검색한다.  
기본적으로 웹 크롤링은 기존의 복사본을 만들고, 웹 스크래핑은 분석을 위한 특정 데이터를 추출하거나 새로운 것을 만든다는 차이가 있지만, 웹 스크래핑을 수행하기 위해서는 먼저 필요한 정보를 찾기 위해 웹 크롤링 같은 작업을 수행해야 한다.

**주의할 점**

크롤링을 잘못하면 위법이 될 수 있다.  
자세한 내용은 https://redfox.tistory.com/31 과 https://redfox.tistory.com/30 을 참고하면 알 수 있다.  
간단히 정리하자면 robots.txt 권고안을 따르지 않는 것은 위법이 아니다.  
robots.txt는 말 그대로 "권고안" 이기 때문에 따르지 않는다고 위법은 아니다.  
하지만 크롤링이 DoS 공격처럼 이용되거나 사이트에서 자체적으로 생산 및 배포하는 것으로 이익을 창출해내는 데이터를 임의로 수집한 다음 자신의 이익을 위해 사용하게 되는 경우 불법이라는 판결을 받은 사례가 있다.

## 2. `axios` + `cheerio`를 이용한 크롤링

> **axios** : Promise based HTTP client for the browser and node.js  
> **cheerio** : Fast, flexible & lean implementation of core jQuery designed specifically for the server

크롤링에서 두 라이브러리는 `axios`를 통해 http get 요청을 보낸 뒤, http response를 변수에 저장하고 `cheerio`를 이용해 해당 변수를 jQuery 문법으로 파싱하는 방식으로 이용된다.

~axios는 어렵지 않으니 생략~

### `cheerio`

#### 사용법

- load : `usage: load(content: any, options?: CheerioOptions, isDocument?: boolean): CheerioAPI`
  - 인자로 "html 문자열"을 받아 cheerioAPI 객체를 반환
- $: `usage: $(selector?: BasicAcceptedElems<Node>, context?: BasicAcceptedElems<Node>, root?: BasicAcceptedElems<Document>, options?: CheerioOptions): Cheerio<Node>`
  - 문서 내에서 찾을 selector 또는 새 Cheerio 인스턴스의 내용에 사용
- children : `usage: children(selector?: AcceptedFilters<Element>): Cheerio<Element>`
  - $의 함수로, 인자는 html selector 문자열을 받아 cheerio 객체에서 선택된 html 문자열에서 해당하는 모든 태그들의 배열을 반환
- each : `usage: each(fn: (this: Element, i: number, el: Element) => boolean | void): Cheerio<Element>`
  - $의 함수로, 인자는 콜백 함수를 받아 태그들의 배열을 순회 하면서 콜백함수를 실행
- find : `usage: find(selectorOrHaystack?: string | Element | Cheerio<Element>): Cheerio<Element>`
  - $의 함수로, 인자로 html selector 를 문자열로 받아 해당하는 태그를 반환

### 코드 예시

```javascript
import axios from 'axios'
import cheerio from 'cheerio'
import { myGithubUrl } from './constant/urls/index.js'
;(async function () {
  try {
    const result = await axios.get(myGithubUrl)

    const $ = cheerio.load(result.data)
    const $publicRepoTitles = $('div.pinned-item-list-item-content')
      .children('div')
      .find('a')
      .children('span')

    $publicRepoTitles.each((index, element) => {
      console.log(element.attribs.title) // innerText를 얻을 수 있음
    })
  } catch (err) {
    // handle error
  }
})()

/* 결과
suricata-ruleset
breaking-algorithm
Univ-Maps
cpp-practice
data-structure
tensorflow-crawling
*/
```

```javascript
// 실제 크롤링 코드
;(async function () {
  try {
    // 멤버 등급 정보
    const rankPage = await axios.get(T_MEMBERSHIP_URL.rank)
    const $ = cheerio.load(rankPage.data)
    const $specialRank = $('div.info-txt > p > em:nth-child(2)')
      .text()
      .split(' / ')
    console.log('등급 정보:', $specialRank)
    const $normalRank = $('div.info-txt > p > em:nth-child(4)').text()
    console.log('등급 정보:', $normalRank, '\n')
  } catch (err) {
    // handle error
  }
})()
```

### 문제점

다만 이렇게 HTTP request를 보낸 뒤 response를 파싱하는 방식은 user interaction이 필요한 사이트 또는 CSR하는 사이트에 대해서 크롤링을 진행할 수 없었다.  
내가 원하던 내용 중 하나는 [skt membership 사이트](https://sktmembership.tworld.co.kr/mps/pc-bff/benefitbrand/detail.do?brandId=0)에서 브랜드 별로 혜택을 가져오는 것이었으며 이를 위해서는 user interaction을 자동적으로 흉내내는 방식이 필요했다.  
![메인 화면](https://user-images.githubusercontent.com/63287638/162609628-94c5f6a2-6527-41a1-9825-998a795a38e4.png)  
(skt membership에서 혜택 브랜드를 고를 수 있는 부분)

url을 보면 brandId의 query string 값이 변함에 따라 브랜드가 변하는 것인데, skt membership으로 혜택을 받을 수 있는 브랜드가 매년 바뀔 수 있기 때문에 해당 query string 값을 하드코딩할 수 없었다.

![tag](https://user-images.githubusercontent.com/63287638/162609712-c9436b99-4d24-4f93-bbfc-205e567fde4a.png)  
(두 번째 select tag를 크롬 개발자 도구로 본 모습)

query string을 하드코딩하지 않을 방법을 고민하다가 _[skt membership에서 혜택 브랜드를 고를 수 있는 부분]_ 사진에서 보듯이 첫 번째 select tag를 user interaction 인 것처럼 바꾸면 query string 값을 얻어낼 수 있을 것이라고 생각했다.  
따라서 `axios` 와 `cheerio` 라이브러리 이외의 방식이 필요했다.

## 3. `selenium`을 이용한 크롤링

_참고: https://dreamjy.tistory.com/96_  
`selenium`은 브라우저별로 드라이버를 따로 설치해야 된다.  
나는 개인적으로 크롬 브라우저를 주로 사용하기 때문에 크롬을 예시로 들어서 글을 작성하겠다.  
[공식문서](https://www.npmjs.com/package/selenium-webdriver)에 따르면 Chrome, IE, Edge, Firefox, Opera, Safari에 대해 드라이버를 지원한다.

### 1. chorme-driver 다운로드

chrome 브라우저를 사용하기 위해서는 chrome 버전에 맞는 드라이브를 받아야 한다.  
[chromedriver download link](https://chromedriver.chromium.org/downloads)  
클릭 후 본인 OS에 맞는 driver를 선택한다.

_cf) 본인 chrome version 확인하는 방법_

1. 우측 상단에 ⋮ 또는 ᐧᐧᐧ 버튼을 클릭 후 settings(설정) 클릭(mac의 경우 command + ,)
2. About Chrome 탭 클릭

편의를 위해 해당 zip 파일을 압축해제 한 후 같은 프로젝트 안에 넣어둔다.

### 2. 노드 모듈 selenium-webdriver 설치

`npm install selenium-webdriver`

### 3. 코드 예시

```javascript
import webdriver, { WebDriver } from 'selenium-webdriver'
import chrome from 'selenium-webdriver/chrome.js'
import { T_MEMBERSHIP_URL } from './constant/urls/index.js'
;(async function () {
  let driver: WebDriver | null = null // finally 구문에서 typescript 컴파일 에러가 나지 않게 하기 위해

  try {
    // chorme driver 경로 입력. 이 코드가 없어도 크롬브라우저만 node가 찾을 수 있는 위치에 잘 설치되어 있다면 에러가 나지 않음
    const service = new chrome.ServiceBuilder('./chromedriver').build()
    chrome.setDefaultService(service)

    // chrome 브라우저 빌드
    driver = await new webdriver.Builder().forBrowser('chrome').build()

    // 사이트 열기
    await driver.get(`${T_MEMBERSHIP_URL}0`) // 크롬 브라우저가 guest 권한을 실행

    const brands = await driver.findElements(By.css('#sel-list > li'))

    for (const brand of brands) {
      const brandName = await brand
        .findElement(By.xpath('./child::*'))
        .getAttribute('text') // css의 display: none; 이 설정된 엘리먼트 같은 경우 getText()로 내부의 텍스트를 가져올 수 없다
      // 그럴 때는 .getAttribute("text" | "textContent" | "innerHTML" | "innerText")를 사용하면 된다
      const brandId = await brand.getAttribute('id')
      console.log('브랜드 정보:', brandName, brandId)

      // 브랜드별 혜택
      // selenium을 사용하더라도 axios + cheerio를 같이 이용할 수 있음
      const benefitPage = await axios.get(
        `${T_MEMBERSHIP_URL.benefit}${brandId}`
      )
      const $ = cheerio.load(benefitPage.data)
      const $benefits = $('div.detail-list dl.dl-bnf')
      $benefits.each((_, beneift) => {
        console.log($(beneift).children('dt').text())

        // ...
      })

      sleep(50) // 너무 자주 크롤링하지 않도록 하기 위해
    }
  } catch (err) {
    // handle error
  } finally {
    if (driver) driver.quit() // 중간에 에러가 나면 드라이버가 종료가 되지 않으므로
  }
})()
```

### 4. DOM element

```javascript
// usage
import { By } from 'selenium-webdriver'

// ...
// By.css, By.className, By.id 등 select하는 방법은 다양함
const div = await driver.findElemnt(By.className('class name'))
await start_btn.click() // 버튼 객체 클릭. 해당 객체에게 다양한 명령을 구사할 수 있음
// ...
```

`driver.findElement` 일치하는 하나의 Element를 반환  
`driver.findElements` 일치하는 모든 Element들을 array 형태로 반환  
`${element}.getText()` 해당 element(위에서 div와 같은 변수)의 sub-elements 포함한 모든 텍스트를 return

## 4. 주의사항

[위](https://github.com/mochang2/development-diary/new/main#1-%EA%B0%9C%EB%85%90)에서 이야기 했듯이 크롤링에 대해 제한을 두지 않으면 DoS와 같은 효과가 날 수 있다.  
DoS로 고소를 안 먹으면 다행이지만, 그냥 IP 차단을 당할 수도 있다.  
이를 피하기 위해서 중간에 sleep 함수를 넣는 것이 필요하다.  
다만 node는 비동기 기반의 sleep 함수이기 때문에 setTimeout 등을 동기적으로 처리할 로직이 필요하다.  
처음에는 여러 사이트에 대해 순차적으로 반복문을 돌리려고 했는데, 반복문에 대해서도 비동기적으로 동작해서 동기적 sleep을 할 수 있는 방법을 찾아봤다.  
그 결과, `sleep-synchronously` 모듈을 사용하면 반복문 중간에도 sleep을 할 수 있다는 것을 알았다.

또는

```javascript
const delay = (ms: number) => {
  return new Promise((resolve) => setTimeout(resolve, ms))
}
```

선언 후 `await delay(1000)` 이런 식으로 사용해도 동기 sleep이 가능하다.

## 5. docker

크롤링을 한 번만 하고 끝내지 않을 것이라면 주기적으로 돌릴 필요가 있다.  
현재 회사에서는 k8s cronjob을 쓰기 때문에 gui가 아닌 cli에서 headless로 돌릴 방법이 필요했다.  
아래는 driver에 headless 옵션을 추가하기 위한 방법이다.

```javascript
const driverOption = new chrome.Options()
driverOption.addArguments('--headless')
const driver = await new webdriver.Builder()
  .forBrowser('chrome')
  .setChromeOptions(driverOption)
  .build()
```

아래는 도커파일 예시다.

```dockerfile
FROM selenium/standalone-chrome
# 셀레니움을 돌리기 위한 driver들이 설치되어 있는 gnu/linux 이미지

# seluser 기본 유저로 로그인이 되기 때문에 거의 모든 명령어에 sudo가 필요함
ENV TZ=Asia/Seoul
RUN sudo chmod o+w /etc/timezone \
  && sudo ln -snf /usr/share/zoneinfo/$TZ /etc/localtime \
  && sudo echo $TZ > /etc/timezone

ARG GITHUB_PAT=

WORKDIR /

RUN sudo apt update \
  && sudo apt install curl -y \
  && cd ~ \
  && curl -sL https://deb.nodesource.com/setup_17.x -o nodesource_setup.sh \
  && sudo bash nodesource_setup.sh \
  && sudo apt install nodejs -y \
  && sudo apt install build-essential -y \
  && sudo npm install -g yarn

WORKDIR /${project_name}

COPY . .

RUN sudo yarn config set 'npmScopes[""].npmRegistryServer' "https://npm.pkg.github.com" \
  && sudo yarn config set 'npmScopes[""].npmAlwaysAuth' "true" \
  && sudo yarn config set 'npmScopes[""].npmAuthToken' "$GITHUB_PAT" \
  && sudo yarn install \  # os마다 npm이 달라질 수 있으므로 새롭게 설치
  && yarn build

ENTRYPOINT ["yarn","start"]
```

아래는 `.dockerignore` 파일이다.

```plaintext
Dockerfile
chromedriver  # hostOS 용 chrome driver 이기 때문에
dist/
.yarn/cache/ # yarn berry를 사용한다면
node_modules/ # yarn berry를 사용하지 않는다면
```

이런 다음에 `docker build --tag ${image name}:1.0 .`, `docker run -d -p 4444:4444 --shm-size='2g' --name ${container name} ${image name}:1.0` 명령어로 도커 빌드 후 실행할 수 있다.  
`--shm-size='2g'`는 컨테이너에 자원을 더 할당함으로써 셀레니움을 돌릴 때 문제가 없도록 하기 위한 옵션이다.
