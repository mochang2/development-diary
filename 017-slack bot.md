## 0. 공부하게 된 계기
slack에 Heytaco라는 앱이 있다.  
칭찬문화를 장려하기 위한 앱인데, `@사람이름` 멘션을 통해 칭찬하고 싶은 사람을 지정하고, 해당 문장에 `:taco:`를 통해 타코를 주면 그 사람이 받은 칭찬의 숫자가 올라간다.  
해당 서비스를 쓰기 위해서는 사람 한 명당 3,000 ~ 5,000원 정도 들어서 싸지 않은 비용이기 때문에 직접 개발을 했다.  
이번에는 개발 지식과는 달리 일기처럼 여러 삽질 과정과 그 끝에 얻은 코드들이 정리했다.  
참고로 bolt라는 것을 이용해서 slack app을 만들기로 했다.  
~회사의 앱 이름이 '니콘내콘'이기 때문에 :corn:을 사용했지만 여기서는 편히 칭찬 스티커라고 부르겠다~


## 1. 생각해야 할 점
칭찬 내역을 파일에 저장하려고 했으나 동시적인 접근에 대한 처리가 복잡해질 것 같기 때문에 편히 새로운 DB 테이블을 만들기로 결정했다.  

필요한 기능을 나열하자면,  
1. 앱이 특정 채널에 추가될 때 추가된 채널을 사용자에게 보여준다(이 채널에서는 칭찬을 하면 칭찬 스티커가 소모됨을 알려주기 위해). 앱이 해당 채널에서 삭제되면 해당 정보도 삭제된다.
2. 동시에 해당 채널의 사용자들의 정보를 DB 테이블에 업데이트한다.
3. 앱이 추가된 채널에서 사용자가 조건에 맞는 칭찬 문구를 입력하면 본인이 가진 칭찬 스티커는 감소하고, 칭찬을 들은 사람이 보유한 스티커는 늘어난다.
4. 이 때, 여러 사용자를 동시에 멘션하거나, 칭찬 스티커를 한 번에 여러 개를 줄 수 있으며, 본인이 가지고 있는 칭찬 스티커를 초과하면 더 이상의 칭찬 스티커를 줄 수 없어야 한다.
5. 앱이 추가된 채널에서 사용자를 추가하면 DB 테이블에 해당 사용자 정보가 업데이트된다.
6. 칭찬 스티커를 받은 개수를 한 눈에 볼 수 있는 리더보드가 존재한다.
7. 매일 줄 수 있는 칭찬 스티커는 5개로 제한되어 있기 때문에 매일 12시에 칭찬 스티커의 개수가 5개로 리셋된다.
8. 특정 시기마다(분기별) 칭찬 스티커를 받은 사람의 등수를 csv 파일로 저장한 뒤 DB에서 칭찬을 받은 누적 개수는 0개로 리셋된다.

7, 8번 기능 때문에 다양한 방법을 고민했는데, 속편히 크론잡을 만들기로 했다.  
setTimeout 같은 함수는 오차가 크기 때문에 사용하지 않았다.  


## 2. 삽질1 - app 메인 화면 띄우기
bolt로 slack app을 만들 때는 slack 공식문서를 참조하면 안 된다.  
[bolt-js 슬랙 앱 만들기](https://slack.dev/bolt-js/tutorial/getting-started) 를 참조해야 한다.  
익스프레스 등으로 만드는 방법은 블로그에 많이 나와있지만 https가 포함된 url이 존재하지 않으면 앱 설치를 할 수 없다(사실 할 수 있는 것 같지만 수많은 삽질 끝에 포기했다).  

이후 다른 방법을 찾은 것이 bolt를 이용하는 것인데, socket mode로 앱을 작동시킬 수 있어서 public url이 필요하지 않다.  
위에 적은 공식 문서를 따라하면서 socket mode 설정이나, bot auth, Oauth & Permissions만 잘 설정하면 문제 없이 동작한다.  
설정을 완료한 후 아래와 같은 코드를 작성하면 정상적으로 동작한다.

```typescript
import { App } from '@slack/bolt'
import Home from 'where'

const APP_PORT = Number(process.env.port)
const app = new App({
  token: process.env.SLACK_BOT_TOKEN, // dotenv 모듈 사용
  signingSecret: process.env.SLACK_SIGNING_SECRET,
  socketMode: true,
  appToken: process.env.SLACK_APP_TOKEN,
  port: APP_PORT,
})

app.event('app_home_opened', async ({ event, client, logger }) => {
  // async 없으면 모듈에서 일치하는 함수를 찾지 못해 에러 발생
  try {
    await client.views.publish(Home(event))
  } catch (err) {
    logger.error(err.message)
  }
})
```

Home(view)를 만드는 코드는 https://api.slack.com/tutorials/app-home-with-modal 와 https://api.slack.com/surfaces/tabs/using 를 참고하면 된다.  
다만 여기서 나오는 마크다운은 깃헙에서 사용하는 마크다운 문법과 다르다.  
https://api.slack.com/reference/surfaces/formatting 에서 나오는 문법을 사용해야 원하는 뷰를 만들 수 있다.  


## 3. 삽질2 - 앱이 채널에 설치될 때 해당 채널 파악하기
web api 메소드는 https://api.slack.com/methods 이 문서에서 확인할 수 있고, 모든 이벤트 타입은 https://api.slack.com/events 에서 확인할 수 있다.  
당연히 나는 앱이 추가된 채널을 볼 수 있는 이벤트나 메소드는 하나쯤은 있을 줄 알았다.  
하지만 전혀 없어서 6시간이 넘게 공식문서를 정독하는 삽질을 했다.  
내가 찾은 유일한 방법은 앱이 채널에 추가되는 이벤트를 파악하는 것이 아니라 추가된 이후(또는 멘션으로 앱을 추가할 때) `@봇 이름` 이 불렸을 때 해당 채널을 파악하는 것이었다.  

##### 구현
앱이 설치된 채널 파악하는 것은 사실 앱의 기능을 크게 좌우하는 중요한 정보는 아니다.  
다만 사용자에게 조금 더 편의성을 제공하고자 보여주는 정보였다.  
그래서 이것을 위해서 따로 DB를 파는 것은 아깝다고 생각하고, 데이터도 크지 않기 때문에 메모리에 저장하기로 결정했다.  
코드는 아래와 같다.

```typescript
// main app.ts
...
import appMention from 'where' // 

const app = new App({
  // ...
})

app.event('app_home_opened', async ({ event, client, logger }) => {
  // ...
})
app.event('app_mention', async ({ event, client, logger }) => {
  try {
    await appMention(event, client)
  } catch (err) {
    logger.error(err.message)
  }
})


// appMention.ts
...
import { appAddedChannels } from 'where'

const appMention = async (event, client) => {
  if (event.channel_type !== 'channel') return // public channel이 아니면
  if (!Object.keys(appAddedChannels).includes(event.channel)) {
    // 이미 파악된 채널이 아니면
    // 현재 슬랙의 모든 채널을 가져와서
    // 같은 id를 가진 애를 변수(메모리)에 저장
    // 해당 변수는 다른 곳에서도 접근해야 되므로 mutex 등 필요.
  }
}
...
```

결과적으로 나온 화면은 아래다.  
<img width="1100" alt="Screen Shot 2022-04-28 at 20 20 37" src="https://user-images.githubusercontent.com/63287638/165741224-3ec364ce-0969-4c6e-8f12-33c3358f1441.png">
  
++) 수정사항: 멘션 이벤트를 바꿨다. 앱이 멘션이 되면 사용법을 채널에 띄우고, 앱이 포함된 채널에서 채팅이 발생하면 위에 선언한 `appMention` 에서 했던 동작을 수행하게끔 바꿨다.  
앱이 포함된 채널에서 채팅을 하게끔 만드는 web api는 https://api.slack.com/methods/chat.postMessage 에서 확인할 수 있다.

##### about 탭
코드로 구현하는 것이 아니라 slack app 관리 화면에서 'basic information' 탭을 수정하면 자동으로 바뀐다(다만 시간이 좀 걸릴 수 있으니 그냥 기다리면 해결된다).  
  
<img width="1097" alt="Screen Shot 2022-04-28 at 20 20 43" src="https://user-images.githubusercontent.com/63287638/165741329-e8a108c6-5014-4554-8715-0fe924d33018.png">


## 4. 삽질3 - typescript

```typescript
import { GenericMessageEvent, MessageEvent } from '@slack/bolt'

const messageChannel = async (event: MessageEvent, client) => {
  // ...
  const messageEvent = event as GenericMessageEvent
}
```

`@slack/bolt` 파일을 타고 들어가면 다음과 같이 선언되어 있다.

```
export declare type MessageEvent = GenericMessageEvent | BotMessageEvent | ChannelArchiveMessageEvent | ChannelJoinMessageEvent | ChannelLeaveMessageEvent | ChannelNameMessageEvent | ChannelPostingPermissionsMessageEvent | ChannelPurposeMessageEvent | ChannelTopicMessageEvent | ChannelUnarchiveMessageEvent | EKMAccessDeniedMessageEvent | FileShareMessageEvent | MeMessageEvent | MessageChangedEvent | MessageDeletedEvent | MessageRepliedEvent | ThreadBroadcastMessageEvent;
```

`MessageEvent` 타입을 사용하려면 뒤에 선언된 이벤트가 가진 모든 property를 139번째 줄에서 선언한 변수인 `event`가 가지고 있어야 한다.  
일반적으로 그런 경우가 존재하지 않기 때문에 강제적으로 `as`로 타입을 바꾸는 게 최선이라고 한다.  


## 5. 삽질4 - 앱 유저 구분
기획 의도대로라면 칭찬 프로그램을 만들 때 `@강냉이 칭찬해~ :corn:` (봇 이름이자 앱 유저 이름이 강냉이였다)라고 해도 칭찬 프로그램이 반응하지 않아야 했다.  
따라서 앱 유저 이름과 일반 유저 이름을 멘션할 때의 경우를 구분할 필요가 있었다.  
[삽질2](https://github.com/mochang2/development-diary/edit/main/017-slack%20bot.md#3-%EC%82%BD%EC%A7%882---%EC%95%B1%EC%9D%B4-%EC%B1%84%EB%84%90%EC%97%90-%EC%84%A4%EC%B9%98%EB%90%A0-%EB%95%8C-%ED%95%B4%EB%8B%B9-%EC%B1%84%EB%84%90-%ED%8C%8C%EC%95%85%ED%95%98%EA%B8%B0)에서 말했듯이 앱은 초대된 채널에 대해서만 이벤트가 동작할 수 있다.  
다만 이 때 앱은 한 명의 유저로서 채널에 초대되지만 슬랙으로 직접 채널 구성원을 살펴보면 나오지 않는다.  

![화면 캡처 2022-05-06 235207](https://user-images.githubusercontent.com/63287638/167157937-6bae5ca2-14d9-4a69-9821-4b27ae87733a.png)  
<br/>

하지만 api로 [slack web api](https://api.slack.com/methods)에서 살펴볼 수 있듯이 `conversations.members`을 통해 구성원을 받아오면 슬랙에서 볼 때에 비해 한 명 더 존재한다.  

```
// 결과
"members": ["U023BECGF", "U061F7AUR"]
```

하지만 이렇게 가져오는 정보 외에 앱 유저를 구분하는 방법이 존재하지 않아서(아니면 내가 못 찾은 걸 수도 있다) 결국 앱 유저 id를 코드 일부에 하드코딩 한 뒤에 가져다가 사용했다.  


## 6. 삽질5 - 모달 띄우기
이 부분은 참고할 코드가 있어서 덜 삽질했다.  
view를 만들 때 blocks의 타입에는 section, header, divider, button 등이 있다.  
그냥 button를 만들면 왼쪽 정렬이 되지만, section 내부에 accessory라는 옵션을 통해 button을 만들면  
우측 정렬이 된다.  
UI를 구성하는 일이다 보니 css에서 할 수 있는 기능은 전부 할 수 있을 줄 알고 기대했지만 정렬 그 이상의 이쁨을 바랄 수는 없는 것 같다.  
아래는 예시 코드이다.  

```typescript
const Home = async (
  event: AppHomeOpenedEvent, // import { AppHomeOpenedEvent } from '@slack/bolt'
): Promise<ViewsPublishArguments> => {
  // 어떤 로직

  return {
    user_id: event.user,
    view: {
	...
        {
          type: 'section',
          text: {
            type: 'mrkdwn',
            text: '*마크다운* 문법 이런 식으로'
            verbatim: true,
          },
          accessory: {
            type: 'button',
            action_id: 'action 이름',
            text: {
              type: 'plain_text',
              text: '글글글',
            },
          },
        },
	...
    }
  }
}
```

`action 이름` 이 하는 역할은 [아래](https://github.com/mochang2/development-diary/edit/main/017-slack%20bot.md#csv-%EB%8B%A4%EC%9A%B4%EB%A1%9C%EB%93%9C-%EB%B2%84%ED%8A%BC%EC%9D%84-%ED%81%B4%EB%A6%AD%ED%96%88%EC%9D%84-%EB%95%8C) 예제 코드에 나와 있다.  


## 7. 삽질6 - 개인 DM 보내기
이거는 사실 삽질이라기 보다는 약간 자주 쓰지 않는 코드도 섞여 있어서 시간을 잡아먹었다.  
개인 DM은 두 가지 이벤트 시에 발생한다.  
첫 번째는 `:corn:`을 주고받을 때 '누가' '누구에게' '어느 채널에서' '몇 개의' `:corn:`을 보냈는지 알림을 띄울 때 
두 번째는 [삽질6](https://github.com/mochang2/development-diary/edit/main/017-slack%20bot.md#7-%EC%82%BD%EC%A7%886---%EA%B0%9C%EC%9D%B8-dm-%EB%B3%B4%EB%82%B4%EA%B8%B0)에서 모달을 띄운 뒤 'csv 다운로드' 버튼을 클릭했을 때이다.

##### `:corn:` 알림을 띄울 때   
https://api.slack.com/surfaces/tabs/events 를 보면 앱이 사용자에게 메시지 탭을 통해 개인 DM을 보내기 위한 방법은 총 세가지가 있다.  

* The user and channel IDs found in the app_home_opened event can be used as the channel parameter when calling the chat.postMessage Web API method to publish a message.
* An incoming webhook can be created for the Messages tab conversation.
* Message responses can be created when a valid request_url is received.

코드가 조금 지저분하지만 첫 번째 방식에서 영감을 얻었다.  
두 번째는 리소스가 많이 드는 방식이었고, 세 번째는 요청을 받을 url이 따로 존재해야 개발을 할 수가 있다.  
  
slack api 중 [users.conversations](https://api.slack.com/methods/users.conversations)을 이용해 `types: im`을 설정하면 개인 DM 채널 id를 얻을 수 있다.  
모든 사용자는 해당 타입의 채널이 1개이기 때문에 ~(수 천, 수 만을 테스트해본 것은 아니기 때문에 아닐 수도 있다)~ 그 채널이 바로 앱과 유저가 직접 소통하는 채널이다.  
이 채널에 [chat.postMessage](https://api.slack.com/methods/chat.postMessage) api를 사용해서 칭찬 내역을 보낼 수 있다.

##### 'csv 다운로드' 버튼을 클릭했을 때
~csv의 약어가 Comma Separated Values 인지 처음 알았다. 그래서 csv 파일을 보냈을 때 엑셀과 같은 파일이 아닌 그저 텍스트 파일이 보내졌을 때 에러인지 알고 한참 찾았다.~  
처음에는 바로 사용자의 컴퓨터에게 csv 파일을 다운받는 방법을 생각했다.  
하지만 그렇게 하려면 프로토콜을 하나 만들고, 클라이언트 측에서도 포트가 하나 열려 있게 만들고... 등 생각해야 할 일이 많았다.  
생각해낸 방법은 csv 파일을 앱이 개인에게 보낼 수 있는 DM을 통해 보내는 것이었다.  
  
slack app view에는 button type이 있는데 해당 버튼에 action을 달 수 있다.  

```typescript
// app.ts(또는 진입점)
app.action('액션 이름', 실행할 함수 이름)

// 실행할 함수 선언된 부분
export const sendRankFile = async ({ client, ack, body, logger }) => {
  await ack()

  // 동작
  // csv 파일은 json2csv 모듈을 이용했다.
  // mongoose에서 가져온 결과값이 어차피 object[] 형태이기 때문에
  // json2csv의 parser을 이용하면 손쉽게 가능했다.
  const fields = ['field1', 'field2', 'field3']
  const opts = { fields }
  const json2csvParser = new Parser({ fields, delimiter: '\t' })
  const tsv = json2csvParser.parse(usersInfo)
  // 위와 같은 방식으로 사용하면 default delimiter인 ','에서 '\t'로 바꿀 수 있다.
}
```

++) 수정사항: 기존 방식에서 DM 채널을 찾는 방법이 정상적으로 기능하지 않을 때가 있었다. 그래서 개인에게 DM을 보낼 수 있는 채널을 찾는 방법을 바꿨다. `client.conversations.open` api를 사용한다면 오류없이 동작시킬 수 있다.


## 8. DB
기능이 복잡하지 않기 때문에 되게 단순하다.  
유저를 구분할 수 있는 id, 오늘 남은 칭찬 횟수, 기간 동안 총 받은 칭찬 개수, 기간 동안 총 준 칭찬 개수 정도만 저장하면 됐다.  
DB에 이러한 데이터를 저장하면 편했던 점이 [삽질2](https://github.com/mochang2/development-diary/edit/main/017-slack%20bot.md#3-%EC%82%BD%EC%A7%882---%EC%95%B1%EC%9D%B4-%EC%B1%84%EB%84%90%EC%97%90-%EC%84%A4%EC%B9%98%EB%90%A0-%EB%95%8C-%ED%95%B4%EB%8B%B9-%EC%B1%84%EB%84%90-%ED%8C%8C%EC%95%85%ED%95%98%EA%B8%B0)와 달리 concurrency에 대한 관리를 하지 않아도 됐다.  
처음에 DB 테이블을 초기화하는 방법에 대해서 고민을 했다. 후보군은 다음과 같았다.  

1. 앱 유저가 채널에 추가될 때마다 해당 채널의 모든 유저를 가져오고, DB에 없는 유저라면 새롭게 데이터를 만든다. 만약 채널에 새로운 유저가 들어오거나 나가면 해당 내용을 DB에 업데이트한다.
2. `/invite` 명령어로 앱 유저를 채널에 추가한 뒤 무조건 `@강냉이`를 부름으로써 이벤트를 발생시킨다. 단 이때, 구성원 간의 사용 규칙을 명시해야 한다.
3. 채널에 앱 유저가 초대되든, 추방되든, 채널에 일반 유저가 초대되든, 추방되든 신경쓰지 않는다. 오직 신경쓰는 거는 앱이 초대된 채널에서 message를 보내는 이벤트가 발생했는지 여부이다. 해당 message가 '칭찬 규칙'에 맞는 칭찬 text를 가지고 있으면, 파싱한 뒤 규칙에 맞게 DB를 업데이트한다. 만약 이 때 DB에 없는 유저라면 새롭게 만들고, 그렇지 않다면 바로 업데이트를 진행한다.

결과적으로 선택한 것은 3번이다.  
1번은 너무 경우의 수가 다양해서 전부 커버하기가 힘들었다.  
2번도 규칙이 잘 지켜진다면 괜찮았지만, '앱 멘션' 이벤트를 다른 방식으로 사용할 수 있으며, 굳이 규칙을 세우지 않아도 괜찮은 3번을 선택했다.  


## 9. cronjob
docker가 어차피 리눅스 기반이기 때문에 크론잡의 문법도 동일했다.  
`분 시 일 월 요일`이며 `*`은 모든 분, 모든 시, 모든 일, 모든 월, 모든 요일 등을 가리킨다.  
즉 `* * * * *`는 매일 매시 매분을 의미한다.  
동작시키고 싶은 기능은 `yarn program`에 등록했다.  
commander의 자세한 설명은 [공식문서](https://www.npmjs.com/package/commander)에 있다.

```
// package.json
...
  "scripts": {
    ...
    "program": "NODE_ENV=development ts-node-dev -r dotenv/config ./src/program.ts" //  이렇게 하면 build 없이 실행 가능하다
  },
...

// program.ts
import { Command } from 'commander'

const program = new Command()

program.version('0.0.1')

program.command('cron-daily').action(async () => {
  try {
    // db에 연결하는 등 실행 시킬 함수.

    process.exit()
  } catch (err) {
    console.log(err)
  }
})

program.parse(process.argv)


// cronjob.yaml
apiVersion: 버전 0.1.0 등
kind: CronJob
metadata:
  name: 크론잡 이름
spec:
   // 이후는 배포에 맞는 스펙
```


## 최종 코드
~회사 자산이라 개인 깃헙에 못 올리는게 너무 맘이 아프다... 열심히 만들었는데~  
이외에도 유저가 채널에서 사라질 때 해당 유저에 대한 데이터도 DB에서 지우거나, csv 파일을 Drive에 올리거나, 슬랙 워크 스페이스에서 관리자만 사용할 수 있는 기능을 추가하는 등이 있었지만, 위에 제시했던 slack api method나 slack event subscription만 잘 찾아보면 구현이 가능해서 더 자세히 적지는 않겠다.  

