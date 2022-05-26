## 0. 공부하게 된 계기
double nc에서 다양한 프로젝트가 있었는데 그 중 일부는 rest api 대신 graphql을 사용했다.  
공부를 하며 한 가지 착각한 것이 있는데 gql은 sql의 대안이 아니라 rest api의 대안이다.  
rest api와 일장일단이 있다고 하지만 현재 graphql은 deprecated라고 한다.  
참고한 블로그는 https://tech.kakao.com/2019/08/01/graphql-basic/ 이다.
  
## 1. graphql이란
sql과 마찬가지로 쿼리 언어로 페이스북에서 만들었다고 한다.  
언어적 구조 차이가 존재하는데, 이는 사용 시기도, 배경도 다르기 때문이다.  
sql은 데이터베이스 시스템에 저장된 데이터를 효율적으로 가져오는 것이 목적이고, gql은 웹 클라이언트가 데이터를 서버로부터 효율적으로 가져오는 것을 목적으로 한다.  
sql은 주로 백엔드에서 작성하고 호출하는 반면, gql은 주로 클라이언트 시스템에서 작성하고 호출한다.
서버사이드 gql 어플리케이션은 gql로 작성된 쿼리를 입력으로 받아 쿼리를 처리한 결과를 다시 클라이언트로 돌려준다.  

## 2. rest api와 비교
REST API는 URL, METHOD등을 조합하기 때문에 다양한 Endpoint가 존재한다.  
반면, gql은 단 하나의 Endpoint가 존재한다.  
또한, gql API에서는 불러오는 데이터의 종류를 쿼리 조합을 통해서 결정한다.  
예를 들면, REST API에서는 각 Endpoint마다 데이터베이스 SQL 쿼리가 달라지는 반면, gql API는 gql 스키마의 타입마다 데이터베이스 SQL 쿼리가 달라진다.  

```
{
  human(id: "1000") {
    name
    height
  }
}

query HeroNameAndFriends($episode: Episode) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}
```

위와 같은 형식으로 생겼으며 쿼리는 주로 select같은, 뮤테이션은 insert, delete, update와 같은 것이라고 생각하면 된다.  
개념적으로 구분은 했지만 내부적으로는 크게 다르지 않다고 한다.

##### 장점 1) overfetching 또는 underfetching이 발생하지 않는다.
필요한 데이터만 달라고 한 뒤 그 데이터만 이용하면 되므로 필요 없는 데이터를 받거나, 필요한 데이터를 받지 못하는 경우가 발생하지 않는다.  
다만 이 때문에 요청(request)가 길어지는 단점이 발생한다.

##### 장점 2) introspection
rest api를 사용하며 겪은 가장 큰 문제는 api 명세서가 실시간으로 관리가 안 된다는 점이었다.  
api가 바뀌면 명세를 동시에 바꿔주는 기능이 없기 때문에 시차가 발생하여 명세서에 쓰인 api가 최신이라는 보장이 없다.  
이러한 REST의 API 명세서 공유와 같은 문제를 해결하는 것이 gql의 인트로스펙션 기능이다.  
gql의 인트로스펙션은 서버 자체에서 현재 서버에 정의된 스키마의 실시간 정보를 공유를 할 수 있게 한다.  
이 스키마 정보만 알고 있으면 클라이언트 사이드에서는 따로 연동규격서를 요청 할 필요가 없다.  
클라이언트 사이드에서는 실시간으로 현재 서버에서 정의하고 있는 스키마를 의심 할 필요 없이 받아들이고, 그에 맞게 쿼리문을 작성하면 된다.


## 3. code example
~다른 것보다 enum 사용이 너무 답답해서 추가했다~  
포스트맨으로 테스트할 때 엄청 삽질했던 것 중에 하나가 graphql은 enum 값을 명시하지 않았을 때 0부터 숫자가 들어가는 것이 아니라  
enum 의 key값이 string으로 들어간다.  
공식 문서에도 명확히 안 써 있어서 수많은 삽질 끝에 터득했다.  
추가적으로 포스트맨에서 graphql enum 선언은 ... (스샷 추가 예정) 이렇게 선언해서 (스샷 추가 예정)  가져다 쓴다.

```typescript
// // BE 
// function => 일반적으로 express를 사용하면 router(resolver)에서 이 함수를 불러서 사용
// db connection 관련 import

export enum option {
  EXCLUDED = 'EXCLUDED',
  REQUIRED = 'REQUIRED',
}

export const Func = async (_, {option, Id1, Id2}) => {
  try {
    const collection = someModel // mongoose model
    const result = await collection.find({'조건 명시'}).select('key1 key2')

    return result
  } catch (err) {
    console.error(err)
  }
}

// resolver
import { Func } from 'where'

export const resolvers = { // Apollo server 생성시 인자로 넣어줌
  Query: {
    Func,
    ...
  },
  Mutation: { }
}

// 진입점
const server = new ApolloServer({
  typeDefs: ,
  resolvers: resolvers,
  schemaDirectives: {}, 
  context: callback func,
  plugins: [],
  engine: {},
  formatError: callback func,
})

"""
~~.graphql. 모든 graphql 형태 선언
"""
type Query {
  Func(option: someOption, Id1: Int!, Id2: Int!): [someType!] 
  """
  !은 non nullable을 의미
  : 뒤는 타입을 명시
  
  Func는 someOption 타입의 option 인자와, non-nullable한 Int 타입의 Id1, Id2를 인자로 받으면,
  non-nullable인 someType을 array 형태로 response를 줌
  """
  ...
}

"""
graphql에서 enum
"""
enum someOption {
  EXCLUDED
  REQUIRED
}



// // FE 부분
"""
쿼리 선언
"""
query queryName($option: someOption, $Id1: Int!, $Id2: Int!) { // 변수 이름: 타입
  aiWords( // BE에 전달하는 내용
    option: $option
    Id1: $Id1
    Id2: $Id2
  ) { // BE에게 받는 내용
    key1
    key2
  }
}

// component.tsx, component.jsx, component.vue 등에서. enum을 위한 변수 선언
buttons: [
  {caption: '제외', option: 'EXCLUDED'},
  {caption: '필수', option: 'REQUIRED'},
],

// 쿼리 사용하는 함수 선언
async fetchSome() {
  const res = await this.$apollo.query({
    query: queryName,
    variables: {
      option: this.buttons[index].option,
      Id1: Number(Id1),
      Id2: Number(Id2),
    },
    fetchPolicy: 'network-only',
  })
  const {Funcs} = res.data
},
```


