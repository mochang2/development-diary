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

```
import { App } from '@slack/bolt'
import Home from './views/Home'

const APP_PORT = Number(process.env.port) || 3000
const app = new App({
  token: process.env.SLACK_BOT_TOKEN, // dotenv 모듈 사용
  signingSecret: process.env.SLACK_SIGNING_SECRET,
  socketMode: true,
  appToken: process.env.SLACK_APP_TOKEN,
  port: APP_PORT,
})

app.event('app_home_opened', async ({ event, client, logger }) => {
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

```
// main app.ts
...
import appMention from './events/app-mention' // 

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
import { appAddedChannels } from '../memory'  // let으로 선언된 object 타입

const appMention = async (event, client) => {
  if (event.channel_type !== 'channel') return // public channel이 아니면
  if (!Object.keys(appAddedChannels).includes(event.channel)) {
    // 이미 파악된 채널이 아니면
    const { channels } = await client.conversations.list({
      token: process.env.SLACK_BOT_TOKEN,
      limit: 1000,
      exclude_archived: true,
      types: 'public_channel',
    })
    const mutex = new Mutex()

    for (const channel of channels) {
      if (channel.id === event.channel) {
        await mutex.runExclusive(async () => {
          appAddedChannels[event.channel] = channel.name
        })
        break
      }
    }
  }
}

export default appMention
```

결과적으로 나온 화면은 아래다.  
<img width="1100" alt="Screen Shot 2022-04-28 at 20 20 37" src="https://user-images.githubusercontent.com/63287638/165741224-3ec364ce-0969-4c6e-8f12-33c3358f1441.png">
  
++) 수정사항: 멘션 이벤트를 바꿨다. 앱이 멘션이 되면 사용법을 채널에 띄우고, 앱이 포함된 채널에서 채팅이 발생하면 위에 선언한 `appMention` 에서 했던 동작을 수행하게끔 바꿨다.  
앱이 포함된 채널에서 채팅을 하게끔 만드는 web api는 https://api.slack.com/methods/chat.postMessage 에서 확인할 수 있다.

##### about 탭
코드로 구현하는 것이 아니라 slack app 관리 화면에서 'basic information' 탭을 수정하면 자동으로 바뀐다(다만 시간이 좀 걸릴 수 있으니 그냥 기다리면 해결된다).  
  
<img width="1097" alt="Screen Shot 2022-04-28 at 20 20 43" src="https://user-images.githubusercontent.com/63287638/165741329-e8a108c6-5014-4554-8715-0fe924d33018.png">


## 4. 삽질3 - typescript

```
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

`MessageEvent` 타입을 사용하려면 뒤에 선언된 이벤트가 가진 모든 property를 142번째 줄에서 선언한 변수인 `event`가 가지고 있어야 한다.  
일반적으로 그런 경우가 존재하지 않기 때문에 강제적으로 `as`로 타입을 바꾸는 게 최선이라고 한다.  


## 5. DB 테이블
기능이 복잡하지 않기 때문에 되게 단순하다.

(작성중)

## 최종 코드
깃헙 url

