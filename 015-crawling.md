## 0. 공부하게 된 계기
멤버십과 제휴사에 대한 할인 이벤트를 한 눈에 보여주기 위해서 크롤링 기능을 개발할 일이 생겼다.  
기존에 내 tensorflow-crawling 레포지토리에 기록한 것처럼 python의 request 모듈을 이용해서 크롤링 개발을 진행한 적이 있었다.  
그때는 http get request에 대한 응답을 html tag에 따라 파싱한 뒤에 원하는 정보를 빼서 사용했다.  
이번에는 js로 개발을 진행해야 되며, user interaction이 필요한 부분이 있기 때문에, node.js의 __selenium__ 모듈 위주로 정리할 예정이다.  

## 1. 개념
일반적으로 __크롤러__ 라는 용어는 명확한 최종 목표나 목표가 없어도 사이트나 네트워크가 제공할 수 있는 것을 끝없이 탐색하면서 스스로 웹 페이지를 탐색할 수 있는 프로그램의 능력을 의미한다.  
반면에 __스크래퍼__ 는 특정 데이터를 추출하는 프로세스이다.  
웹 크롤링과 달리 웹 스크래퍼는 특정 웹 사이트 또는 페이지에서 특정 정보를 검색한다.  
기본적으로 웹 크롤링은 기존의 복사본을 만들고, 웹 스크래핑은 분석을 위한 특정 데이터를 추출하거나 새로운 것을 만든다는 차이가 있지만, 웹 스크래핑을 수행하기 위해서는 먼저 필요한 정보를 찾기 위해 웹 크롤링 같은 작업을 수행해야 한다.  
<br/>

* 주의할 점

크롤링을 잘못하면 위법이 될 수 있다.  
자세한 내용은 https://redfox.tistory.com/31 과 https://redfox.tistory.com/30 을 참고하면 알 수 있다.  
내용을 간단히 정리하자면 robots.txt 권고안을 따르지 않는 것은 위법이 아니다.  
robots.txt는 말 그대로 "권고안" 이기 때문에 따르지 않는다고 위법은 아니다.  
하지만 크롤링이 DoS 공격처럼 이용되거나 사이트에서 자체적으로 생산 및 배포하는 것으로 이익을 창출해내는 데이터를 임의로 수집한 다음 자신의 이익을 위해 사용하게 되는 경우 불법이라는 판결을 받은 사례가 있다.  


## 2. axios + cheerio를 이용한 크롤링
_참고: https://velog.io/@yesdoing/Node.js-%EC%97%90%EC%84%9C-%EC%9B%B9-%ED%81%AC%EB%A1%A4%EB%A7%81%ED%95%98%EA%B8%B0-wtjugync1m_  
__axios__ : Promise based HTTP client for the browser and node.js  
__cheerio__ : Fast, flexible & lean implementation of core jQuery designed specifically for the server  
위 설명은 npm 공식 문서에 나와 있는 설명이다.  
크롤링에서 두 라이브러리는 axios를 통해 http get 요청을 보낸 뒤, http response를 변수에 저장하고 cheerio를 이용해 해당 변수를 jQuery 문법으로 파싱하는 방식으로 이용된다.  
설치 명령어: `npm install axios cheerio`  
<br/>

~axios는 어렵지 않으니 생략~

* cheerio의 함수
  * load : `usage: load(content: any, options?: CheerioOptions, isDocument?: boolean): CheerioAPI`
    * 인자로 "html 문자열"을 받아 cheerioAPI 객체를 반환
  * $: `usage: $(selector?: BasicAcceptedElems<Node>, context?: BasicAcceptedElems<Node>, root?: BasicAcceptedElems<Document>, options?: CheerioOptions): Cheerio<Node>`
    * 문서 내에서 찾을 selector 또는 새 Cheerio 인스턴스의 내용에 사용
  * children : `usage: children(selector?: AcceptedFilters<Element>): Cheerio<Element>`
    * $의 함수로, 인자는 html selector 문자열을 받아 cheerio 객체에서 선택된 html 문자열에서 해당하는 모든 태그들의 배열을 반환
  * each : `usage: each(fn: (this: Element, i: number, el: Element) => boolean | void): Cheerio<Element>`
    * $의 함수로, 인자는 콜백 함수를 받아 태그들의 배열을 순회 하면서 콜백함수를 실행
  * find : `usage: find(selectorOrHaystack?: string | Element | Cheerio<Element>): Cheerio<Element>`
    * $의 함수로, 인자로 html selector 를 문자열로 받아 해당하는 태그를 반환

* 명령어 예시

```
// 간단히 test를 위한 용도이므로 익명 함수로 선언해서 node `${파일이름.js}로 실행
// 일반적으로 axios API 요청을 위한 용도로 사용하면 전역변수로 선언하지만, 크롤링을 한다면 url이 매번 바뀌기 때문에 그러지 않음
// children은 > 등을 통한 자식에 대한 select를 못하므로 좀 억지스럽지만 아래처럼 사용함
// 참고로 이 예제를 짜다가 ' ' select vs '>' select에 대한 차이도 알게 됐다.
// https://stackoverflow.com/questions/22632786/use-a-space-or-greater-than-sign-in-css-selector

import axios from 'axios';
import cheerio from 'cheerio';
import { myGithubUrl } from './constant/urls/index.js';

(async function () {
  try {
    const result = await axios.get(myGithubUrl);
    
    const $ = cheerio.load(result.data);
    const $publicRepoTitles = $("div.pinned-item-list-item-content").children("div").find("a").children("span");
    
    $publicRepoTitles.each((index, element) => {
      console.log(element.attribs.title); // innerText를 얻을 수 있음
    })
  } catch (err) {
    console.error(err);
  }
})();

/* 결과
suricata-ruleset
breaking-algorithm
Univ-Maps
cpp-practice
data-structure
tensorflow-crawling
*/
```

```
// 실제 크롤링 코드
(async function () {
  try {
    // 멤버 등급 정보
    const rankPage = await axios.get(T_MEMBERSHIP_URL.rank);
    const $ = cheerio.load(rankPage.data);
    const $specialRank = $("div.info-txt > p > em:nth-child(2)")
      .text()
      .split(" / ");
    console.log("등급 정보:", $specialRank);
    const $normalRank = $("div.info-txt > p > em:nth-child(4)").text();
    console.log("등급 정보:", $normalRank, "\n");

    // 멤버십 혜택 정보
    // 크롬 브라우저 빌드
    const service = new chrome.ServiceBuilder("./chromedriver").build();
    chrome.setDefaultService(service);
    const driver = await new webdriver.Builder().forBrowser("chrome").build();

    await driver.get(`${T_MEMBERSHIP_URL.benefit}0`);

    const brands = await driver.findElements(By.css("#sel-list > li"));
    for (const brand of brands) {
      // 브랜드 정보
      const brandName = await brand
        .findElement(By.xpath("./child::*"))
        .getAttribute("text"); // css의 display: none; 이 설정된 엘리먼트 같은 경우 getText()로 내부의 텍스트를 가져올 수 없다
      // 그러한  .getAttribute("text" | "textContent" | "innerHTML" | "innerText")를 사용하면 된다
      const brandId = await brand.getAttribute("id");
      console.log("브랜드 정보:", brandName, brandId);

      // 브랜드별 혜택
      const benefitPage = await axios.get(
        `${T_MEMBERSHIP_URL.benefit}${brandId}`
      );
      const $ = cheerio.load(benefitPage.data);
      const $benefits = $("div.detail-list dl.dl-bnf");
      $benefits.each((_, beneift) => {
        // 할인형 vs 적립형
        console.log($(beneift).children("dt").text());

        const $benefitDetails = $(beneift).find("dd > div.info");
        $benefitDetails.each((_, benefitDetail) => {
          // 혜택 대상 등급
          const $benefitRanks = $(benefitDetail).find("span.badge-list > i");
          $benefitRanks.each((_, benefitRank) => {
            console.log(
              "적용 등급:",
              $(benefitRank).attr().class.split(" ")[1].toUpperCase()
            );
          });
          // 혜택 정보
          console.log(
            "혜택 정보:",
            $(benefitDetail)
              .first()
              .contents()
              .filter(function () {
                return this.type === "text";
              })
              .text()
              .trim()
          );
        });
      });
      console.log();

      sleep(50); // 너무 자주 크롤링하지 않도록 하기 위해
    }

    await driver.quit();
  } catch (err) {
    console.error(err);
  }
})();
```

* 문제점

다만 이러한 방식은 간단한 방식에 대한 크롤링밖에 진행할 수 없고 user interaction이 필요하다면 다른 방법을 찾았어야 한다.  
내가 원하던 내용 중 하나는 skt membership 사이트(https://sktmembership.tworld.co.kr/mps/pc-bff/benefitbrand/detail.do?brandId=0)에서 브랜드 별로 혜택을 가져오는 것이다.  
![메인 화면](https://user-images.githubusercontent.com/63287638/162609628-94c5f6a2-6527-41a1-9825-998a795a38e4.png)  
\[skt membership에서 혜택 브랜드를 고를 수 있는 부분\]
<br/><br/>
url을 보면 brandId의 query string 값이 변함에 따라 브랜드가 변하는 것인데, skt membership으로 혜택을 받을 수 있는 브랜드가 매년 바뀔 수 있기 때문에 해당 query string 값을 하드코딩할 수 없었다.  
<br/>
![tag](https://user-images.githubusercontent.com/63287638/162609712-c9436b99-4d24-4f93-bbfc-205e567fde4a.png)  
\[두 번째 select tag를 크롬 개발자 도구로 본 모습\]
<br/><br/>
query string을 하드코딩하지 않을 방법을 고민하다가 _[skt membership에서 혜택 브랜드를 고를 수 있는 부분]_ 사진에서 보듯이 첫 번째 select tag를 user interaction 인 것처럼 바꾸면 query string 값을 얻어낼 수 있을 것이라고 생각했다.  
하지만 __axios__ 와 __cheerio__ 모듈만으로는 user interaction을 자동화할 수 없었다.   


## 3. selenium을 이용한 크롤링
_참고: https://dreamjy.tistory.com/96_  
selenium은 브라우저별로 드라이버를 따로 설치해야 된다.  
나는 개인적으로 크롬 브라우저를 좋아하기 때문에 크롬을 예시로 들어서 글을 작성하겠다.  
[공식문서](https://www.npmjs.com/package/selenium-webdriver)에 따르면 Chrome, IE, Edge, Firefox, Opera, Safari에 대해 드라이버를 지원한다.  
<br/>
1\) chorme-driver 다운로드
chrome 브라우저를 사용하기 위해서는 chrome 버전에 맞는 드라이브를 받아야 한다.  
[chromedriver download link](https://chromedriver.chromium.org/downloads)  
클릭 후 본인 OS에 맞는 driver를 선택한다.  
_cf) _본인 chrome version 확인하는 방법_  
<ol>
  <li>우측 상단에 ⋮ 또는 ᐧᐧᐧ 버튼을 클릭 후 settings(설정) 클릭(mac의 경우 command + ,)</li>
  <li>About Chrome 탭 클릭</li>
</ol>
편의를 위해 해당 zip 파일을 압축해제 한 후 같은 프로젝트 안에 넣어둔다.
<br/><br/>

2\) 노드 모듈 selenium-webdriver 설치<br/>
`npm install selenium-webdriver`  
<br/>

3\) 기본 설정 코드

```
import webdriver from "selenium-webdriver";
import chrome from 'selenium-webdriver/chrome.js';
import { T_MEMBERSHIP_URL } from "./constant/urls/index.js";

(async function () {
  try {
    // chorme driver 경로 입력
    const service = new chrome.ServiceBuilder('./chromedriver').build();
    chrome.setDefaultService(service)

    // chrome 브라우저 빌드
    const driver = await new webdriver.Builder().forBrowser('chrome').build();

    // 사이트 열기
    await driver.get(`${T_MEMBERSHIP_URL}0`) // 크롬 브라우저가 guest 권한을 실행

  } catch (err) {
    console.error(err);
  }
})();
```

<br/>

4\) DOM element

```
// usage
// ~~
import { By } from 'selenium-webdriver';

// ~~
// By.css, By.className, By.id 등 select하는 방법은 다양함
const div = await driver.findElemnt(By.className('shalla shalla'));
await start_btn.click(); // 버튼 객체 클릭. 해당 객체에게 다양한 명령을 구사할 수 있음
// ~~
```

`driver.findElement` 일치하는 하나의 Element를 반환  
`driver.findElements` 일치하는 모든 Element들을 array 형태로 반환  
`${element}.getText()` 해당 element(위에서 div와 같은 변수)의 sub-elements 포함한 모든 텍스트를 return  


## 4. 주의사항
[위](https://github.com/mochang2/development-diary/new/main#1-%EA%B0%9C%EB%85%90)에서 이야기 했듯이 크롤링에 대한 스스로 제한을 두지 않으면 DoS와 같은 효과가 날 수 있다.  
DoS로 고소를 안 먹으면 다행이지만, 그냥 IP 차단을 당할 수도 있다.  
이를 피하기 위해서 중간에 sleep 함수를 넣는 것이 필요하다.  
다만 node는 비동기 기반의 sleep 함수이기 때문에 setTimeout 등을 동기적으로 처리할 로직이 필요하다.  
처음에는 여러 사이트에 대해 순차적으로 반복문을 돌리려고 했는데, 반복문에 대해서도 비동기적으로 동작해서 동기적 sleep을 할 수 있는 방법을 찾아봤다.  
그 결과, `sleep-synchronously` 모듈을 사용하면 반복문 중간에도 sleep을 할 수 있다는 것을 알았다.  


