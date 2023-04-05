## 0. 공부하게 된 계기

서버에 저장되는 time 및 price와 화면에 보여지는 내용이 같지 않을 때가 많았다.  
화면에 보여주기 위해 항상 한 번은 가공을 해야 했고, 그 가공 과정을 매번 구글에 검색해서 알아내거나 `day.js`, `moment.js`를 이용해서 해결했다.

이번에는 time에 대해서만 먼저 다루고자 한다.  
원리는 이해하지 않으려 했던 모습을 반성하며 Timezone, 그리고 JS에서 time API를 다루는 방법에 대해 정리했다.

참고 및 사진 출처  
https://yozm.wishket.com/magazine/detail/1695/  
https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Intl  
https://yceffort.kr/2020/12/why-moment-has-been-deprecated

## 1. Timezone

### GMT(그리니치 표준시)

~'영국에서 보물찾기'에서 나온 그 그리니치 천문대이다.~

영국에 위치한 그리니치 천문대를 기준으로, 경도를 나눈 시간대이다.  
지구를 평면 원이라고 생각할 때 15도씩 나눠서 24개의 시간대를 만들었기 때문에 *초*는 지구가 한 바퀴 자전하는 시간을 86,400(60 \* 60 \* 24)로 나눈 단위를 의미했다.

태평양 중간 어딘가가 가장 빠른 시간대이며, 우리나라는 영국보다 9시간 빠른 시간대를 사용한다.  
표현은 다음과 같이 사용한다: **GMT+0900**

### UTC(협정 세계시)

과학자들이 자전 속도를 관찰해본 결과 자전 1바퀴는 GMT를 기준으로 하면 정확히 86,400초가 아니라 점차 느려지고 있다는 사실을 알게 되었다(윤년의 개념).  
*초*라는 시간 단위가 1967년 바뀌면서 현재는 GMT를 국제 표준으로 삼고 있지 않다.

그래서 *초*의 개념을 세슘 원자의 진동수를 기준으로 재정립했다.  
그리고 바뀐 초의 정의를 기반으로 만든 시간 체계를 **IAT(International Atomic Time)** 라고 불렀다.

이후 윤초를 추가함으로써 초의 정의를 바꾸지 않고 시간을 보정할 수 있는 UTC의 개념을 정립했다.

UTC에서 시간대를 나누는 기중는 GMT와 동일하므로 UTC 역시 영국의 시간대를 기준으로 삼는다.  
또한 **UTC는 전세계 어디에서 측정해도 같은 시간**을 나타낸다는 특징이 있다.

### KST(한국 표준시)

UTC에 닉네임을 붙인 정도이다.
보통 다음과 같은 표현으로 많이 쓰인다.

- UTC+09
- UTC+09:00
- UTC+0900
- GMT+9

## 2. Time Format

### Unix Timestamp

1970년 1월 1일 0시 0분을 기준으로 지금까지 흐른 시간을 **초** 단위로 표현한 것이다.  
유닉스 에포크(Unix Epoch)라고 불리기도 한다.

JS에서 `Date.now()`를 사용하면 unix timestamp를 반환하는데 다만 단위는 ms이다.

### ISO 8601

국제 날짜와 시간 표기법이다.  
다음과 같은 형식으로 사용할 수 있다.

1. YYYYMMDDTHHmmsssssZ: 기본 형식
2. YYYY-MM-DDTHH:mm:ss:sssZ: 확장 형식

_cf)_  
`T`는 구분자로 생략하거나 공백으로 대체가 가능하다.  
`Z`는 군사 용어의 Zulu time에서 따온 것을 *현지 시각*을 나타낸다.

## 3. `Date` API

JS 스펙에 따르면 JS는 GMT와 UTC를 혼용해서 사용한다.

![JS Time Spec](https://user-images.githubusercontent.com/63287638/205918658-79821292-14f3-484b-82ee-d6f44e9f5230.jpg)

~엥?~

결국 JS에서 `Date` API를 이해하기 위해서 다음 사항을 꼭 기억해야 한다.

1. JS에서는 GMT와 UTC를 동일시한다.
2. UTC는 시간대와 상관없이 항상 같은 값이다.

### 날짜와 시간 입력

`new Date()`은 입력 포맷에 따라 다른 결과를 출력한다.  
'2020년 3월 9일 자정'을 표시하기 위해 예시이다.

```javascript
console.log(new Date('2020-03-09'))
console.log(new Date('2020-03-09T00:00:00Z'))

// Mon Mar 09 2020 09:00:00 GMT+0900를 출력
```

```javascript
console.log(new Date('March 9, 2020'))
console.log(new Date('2020-03-09 00:00:00'))

// Mon Mar 09 2020 00:00:00 GMT+0900를 출력
```

그렇다면 KST로 2022년 3월 9일 자정을 표현하려면 어떻게 해야 할까?  
~겁나 많은 방법이 있으니... 외우지 말고 자주 쓰는 것만 한 두개 기억해야겠다.~

```javascript
// 1. 날짜와 시간 정보를 각 인자로 넣어 생성하기
//   - 현지 시각 기준으로 시간을 넣어야 함
//   - 시, 분, 초, 밀리초는 생략 가능
//   - 월은 0부터 시작하므로 주의
new Date(2022, 2, 9)
new Date(2022, 2, 9, 0, 0, 0, 0)

// 2. 날짜와 시간 정보 문자열
//   - 현지 시각 기준으로 시간을 넣어야 함
//   - ISO 8601 표준이 아닌, RFC 2822 라는 별도의 방식
//   - 표준은 아닌데 관례적으로 지원하는 경우, 따라서 쓰지 않는 게 좋음
new Date('Mar 9 2022')
new Date('March 9 2022')
new Date('Mar-9-2022')
new Date('Mar 9 2022 00:00:00')

// 3. 밀리초 단위의 유닉스 타임
//   - 해당 시간을 나타낼 수 있는 고정 숫자 값
new Date(1646751600000)

// 4. ISO 8601 표준 문자열
//   - Z가 있으면 UTC 기준, 없으면 현지 시각 기준
//   - 중요! Z가 없어도 `YYYY-MM-DD`만 있는 형태(ex: '2022-03-09')면 UTC로 간주함
//   - https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Date/Date
//   - T는 생략 가능
new Date('2022-03-09 00:00:00') // 현지 시각 기준
new Date('2022-03-08T15:00:00Z') // UTC 기준

// 5. 시간대를 명시적으로 표현한 문자열
//   - 시간대를 표현하는 방법도 다양함
new Date('Mar 9 2022 00:00:00 GMT+09:00')
new Date('Mar 9 2022 00:00:00 UTC+09:00')
new Date('Mar 8 2022 15:00:00 +00:00')
new Date('Mar 9 2022 00:00:00 +9')
new Date('Mar 9 2022 00:00:00 +0900')
new Date('2022-03-09 GMT+9')
new Date('2022-03-09 UTC+9')
```

### 날짜와 시간 출력

```javascript
// 한국 시간 기준으로 2022년 3월 9일 0시 0분 0초
// UTC 기준으로는 2022년 3월 8일 15시 0분 0초
const date = new Date('2022-03-09 00:00:00')

// 1. 유닉스 타임을 반환하는 메서드
date.getTime() // 1646751600000
date.valueOf() // 1646751600000

// 2. UTC 기반의 값을 반환하는 메서드
date.getUTCFullYear() // 2022
date.getUTCMonth() // 2
date.getUTCDate() // 8
date.getUTCDay() // 2 (화요일)
date.getUTCHours() // 15
date.getUTCMinutes() // 0
date.getUTCSeconds() // 0
date.getUTCMilliseconds() // 0

date.toISOString() // '2022-03-08T15:00:00.000Z'
date.toJSON() // '2022-03-08T15:00:00.000Z'
date.toGMTString() // 'Tue, 08 Mar 2022 15:00:00 GMT'
date.toUTCString() // 'Tue, 08 Mar 2022 15:00:00 GMT'

// 3. 현지 시각 기반의 값을 반환하는 메서드
date.getFullYear() // 2022
date.getMonth() // 2
date.getDate() // 9
date.getDay() // 3 (수요일)
date.getHours() // 0
date.getMinutes() // 0
date.getSeconds() // 0
date.getMilliseconds() // 0

date.toString() // 'Wed Mar 09 2022 00:00:00 GMT+0900 (대한민국 표준시)'
date.toDateString() // 'Wed Mar 09 2022'
date.toLocaleString() // '2022. 3. 9. 오전 12:00:00'
date.toLocaleDateString() // '2022. 3. 9.'
date.toLocaleTimeString() // '오전 12:00:00'
date.toTimeString() // '00:00:00 GMT+0900 (대한민국 표준시)'
```

## 4. `Intl.DateTimeFormat` API

모던 브라우저에서 대부분 지원되는 API이다.  
포맷 지정도 매번 따로 함수를 선언했었는데, 이렇게 간단한 API가 있었다니...

```javascript
new Intl.DateTimeFormat('ko', { dateStyle: 'full' }).format(new Date()) // '2022년 3월 8일 화요일'
new Intl.DateTimeFormat('ko', { dateStyle: 'long' }).format(new Date()) // '2022년 3월 8일'
new Intl.DateTimeFormat('ko', { dateStyle: 'medium' }).format(new Date()) // '2022. 3. 8.'
new Intl.DateTimeFormat('ko', { dateStyle: 'short' }).format(new Date()) // '22. 3. 8.'

new Intl.DateTimeFormat('ko', { timeStyle: 'full' }).format(new Date()) // '오후 10시 58분 25초 미 동부 표준시'
new Intl.DateTimeFormat('ko', { timeStyle: 'long' }).format(new Date()) // '오후 10시 58분 31초 GMT-5'
new Intl.DateTimeFormat('ko', { timeStyle: 'medium' }).format(new Date()) // '오후 10:58:37'
new Intl.DateTimeFormat('ko', { timeStyle: 'short' }).format(new Date()) // '오후 10:58'
```

```javascript
var date = new Date(Date.UTC(2012, 11, 20, 3, 0, 0))

// 긴 날짜 서식에 더해 요일 요청
var options = {
  weekday: 'long',
  year: 'numeric',
  month: 'long',
  day: 'numeric',
}
console.log(new Intl.DateTimeFormat('de-DE', options).format(date))
// → "Donnerstag, 20. Dezember 2012"

// 어플리케이션이 GMT를 사용해야 하고, 그 점을 명시해야 할 때
options.timeZone = 'UTC'
options.timeZoneName = 'short'
console.log(new Intl.DateTimeFormat('en-US', options).format(date))
// → "Thursday, December 20, 2012, GMT"

// 좀 더 자세한 설정이 필요하면
options = {
  hour: 'numeric',
  minute: 'numeric',
  second: 'numeric',
  timeZone: 'Australia/Sydney',
  timeZoneName: 'short',
}
console.log(new Intl.DateTimeFormat('en-AU', options).format(date))
// → "2:00:00 pm AEDT"

// 미국에서도 24시간제가 필요할 때
options = {
  year: 'numeric',
  month: 'numeric',
  day: 'numeric',
  hour: 'numeric',
  minute: 'numeric',
  second: 'numeric',
  hour12: false,
  timeZone: 'America/Los_Angeles',
}
console.log(new Intl.DateTimeFormat('en-US', options).format(date))
// → "12/19/2012, 19:00:00"

// 옵션을 지정하면서 로케일은 브라우저 기본값을 사용하고 싶을 땐 'default' 지정
console.log(new Intl.DateTimeFormat('default', options).format(date))
// → "2012. 12. 19. 19시 0분 0초"

// 오전/오후 시간 표시가 필요할 때
options = { hour: 'numeric', dayPeriod: 'short' }
console.log(new Intl.DateTimeFormat('en-US', options).format(date))
// → 10 at night
```

## 5. `moment.js`

개인적인 생각으로 시간 표시가 많지 않고 중요하지 않은 웹 페이지면 서드 파티 라이브러리를 사용하지 않는 게 더 좋은 것 같다.  
어플리케이션이 무거워지기 때문이다.

하지만 클린한 프로젝트를 위해서 datetime 관련 라이브러리를 사용해야 한다면 적어도 `moment.js`는 피해야된다.  
[공식 문서에서도 deprecated 된다고 한다](https://momentjs.com/docs/)

사용하지 말아야 할 이유는 다음과 같다.

1. [regex를 내부적으로 사용하고 있어 다른 라이브러리에 비해 느리다](https://raygun.com/blog/moment-js-vs-date-fns/).
2. [무겁다](https://inventi.studio/en/blog/why-you-shouldnt-use-moment-js).
3. [tree shaking이 제공되지 않으며](https://how-to.dev/what-is-the-size-impact-of-importing-momentjs), tree shaking을 하더라도 다른 라이브러리보다 무겁다.
4. [객체를 불변하게 다루지 않는다. 즉, side effect가 발생한다](https://inventi.studio/en/blog/why-you-shouldnt-use-moment-js).

```javascript
const startedAt = moment()
const endedAt = startedAt.add(1, 'year')

console.log(startedAt) // > 2020-02-09T13:39:07+01:00
console.log(endedAt) // > 2020-02-09T13:39:07+01:00
```

현재는 `dayjs`나 `date-fns`가 제일 나은 대안으로 보인다.
