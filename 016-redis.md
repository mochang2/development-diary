## 0. 공부하게 된 계기
사용자에게 맞춤 상품을 보여주는 api(라우터)를 개발할 일이 생겼다.  
해당 사용자의 구매 이력과 전체 사용자의 구매 이력들을 통해 맞춤 상품을 보여줘야 하는데, 해당 사용자의 구매 이력을 보여주기 위해 참조하는 테이블은 이미 너무 많은 api에서 접근하기 때문에 부하가 큰 상황이다.  
심지어 해당 사용자에게 맞춤 상품을 노출하는 곳이 사용자가 많이 들어가는 화면이다.  
db 부하를 최소화하기 위해 캐시 서버인 redis를 사용하게 되어 공부를 시작했다.  
참고: https://wildeveloperetrain.tistory.com/21 , https://aws.amazon.com/ko/elasticache/what-is-redis/ , https://devlog-wjdrbs96.tistory.com/374

## 1. 개념
redis의 특징을 정리하기 전에 in-memory DB와 cache가 무엇인지 알아야 한다.  

##### in-memory DB
in-memory DB는 disk-based DB와 달리 __메모리에 데이터를 저장하는 DB__ 를 말하며 일반적으로 key-value 방식(dictionary / json과 같은 구조)을 많이 사용한다.  
OS 수업 등에서 배웠다시피 디스크에 대한 접근은 메모리 <-> 디스크 간 병목(bottleneck)이 있기 때문에 메모리에 대한 접근보다 훨씬 _느리다_.  
병목 현상이 생기는 이유는 disk-based DB는 데이터를 페이지 단위로 읽어오기 때문에 본인이 원하는 데이터가 지금 메모리에 있는 페이지에 없다면 다른 페이지를 요청해야되기 때문이다.  
하지만 in-memory DB에도 단점은 있다.  
일반적으로 생각하는 DB와 달리 _휘발성_ 이라는 것이다.  
에러가 나서 갑자기 프로세스가 죽는다면 데이터가 전부 사라진다.  
또한 메모리에 데이터를 저장하기 때문에 _저장 공간이 상대적으로 매우 작다_.  

##### cache
한번 읽은 데이터를 임의의 공간에 저장하여 다음에 읽을 때는 빠르게 결과를 받을 수 있도록 도와주는 저장공간이다.  
캐시 서버에는 Look aside cache, Write Back 두 가지 패턴이 존재한다.

* Look aside cache
  * 클라이언트가 데이터를 요청
  * 웹 서버는 데이터가 존재하는지 Cache 서버에 먼저 확인
  * Cache 서버에 데이터가 있으면 DB에 데이터를 조회하지 않고 Cache 서버에 있는 결과값을 클라이언트에게 바로 반환(Cache hit)
  * Cache 서버에 데이터가 없으면 DB를 조회하여 Cache 서버에 저장하고 결과값을 클라이언트에게 반환(Cache miss)
* Write Back
  * 웹서버는 모든 데이터를 특정 시간 동안 Cache 서버에 저장
  * Cache 서버에 있는 데이터를 DB에 저장
  * DB에 저장된 Cache 서버의 데이터를 삭제
  * insert 쿼리를 여러 개 각각 날리는 것보다 insert 쿼리 한 번에 날리는 게 낫다는 원리에서 비롯됨. 이렇게 하면 DB connection 오버헤드를 줄일 수 있음
  * DB에 저장하기 전에 Cache 서버가 장애발생 시 데이터가 손실될 수 있음

##### redis(REmote DIctionary Server)
memcached와 비슷한 '캐시 시스템'으로 in-memory DB의 하나로 NoSQL이다.  
아마존 공식 문서에 따르면 redis는 다음과 같은 특징을 가지고 있다.  

* 오픈소스
* 빠른 성능(in-memory DB)
* 다양성과 사용 편의성
  * Pub/Sub는 메시지를 채널에 게시하여 채널에서 구독자에게 전달됨. 채팅과 메시징 시스템에 적합
  * TTL을 통해 해당 기간 후에는 스스로를 삭제하게 지정할 수 있음
  * Atomicity Counter는 일관성 없는 결과를 생성하지 않도록 함
  * Lua라는 강력하며 간단한 스크립팅 언어 제공
* 복제 및 지속성
  * matser-slave 구조를 통해 비동기식 복제를 지원
  * 스냅샷(RDB) 및 Append Only File(AOF) 생성을 모두 지원
    * 스냅샷: 순간적으로 메모리에 있는 내용 전체를 disk에 옮겨 담는 방식
    * Append Only File: Reids의 모든 write/update 연산 자체를 log 파일에 기록
* Java, Python, PHP, C, C++, C#, JS, Node, Ruby, R, Go 등 다양한 언어 지원
* 읽기 성능 증대를 위한 서버 측 복제를 지원
* 쓰기 성능 증대를 위한 클라이언트 측 샤딩(Sharding) 지원

하지만 이를 넘어서는 가장 큰 특징은 아래 사진과 같이 String, Bitmap, Bit field 등 다양한 자료구조를 지원한다는 것이다.  
<img src="https://miro.medium.com/max/700/1*tMiZs3RCrmxLGiFZgWRP6g.png" alt="redis DS" />
<br/>
이렇게 다양한 자료구조를 지원하면 __개발의 편의성이 높아지고 난이도는 낮아지는__ 장점이 있다.  
예를 들어, 어떤 데이터를 정렬할 때 DBMS를 이용한다면 DB를 데이터에 저장하고, 저장하는 데이터를 정렬하여 다시 읽어오는 과정에서 오버헤드가 더 들 수 있다.  
하지만 redis의 Sorted-Set을 이용하면 빠르고 간단하게 데이터를 정렬할 수 있다.  
AWS에서는 Amazon ElasticCache라는 이름으로 사용된다.  


## 2. server 설치
MacOS: `brew install redis`  
Linux: `sudo apt install redis-server`  
현재는 이미 있는 서버를 사용하므로 클라이언트 측에 대한 지식만 있으면 된다. 추후에 서버를 사용할 일이 있다면 추가할 예정이다.

## 3. Node.js로 client 사용
Node.js에서 가장 많이 사용하는 클라이언트는 ioredis이다.  
[공식문서](https://www.npmjs.com/package/ioredis) 에 따르면 다음과 같은 특징을 가지고 있다.  
* Cluster, Sentinel, Streams, Pipelining, Lua scripting, Redis Functions, Pub/Sub 등의 기능 지원
* 높은 성능
* 잘 정리된 API
* Lua scripting 추상화 제공
* binary data, TLS, offline queue, ready checking, ES6 types(Map, Set), NAT mapping, auto pipelining 제공
설치: `npm install ioredis && npm install -D @types/node(for typescript)`  

```
// 간단한 사용법(공식 문서의 코드)
// Import ioredis.
// You can also use `import Redis from "ioredis"`
// if your project is an ESM module or a TypeScript project.
const Redis = require("ioredis");

// Create a Redis instance.
// By default, it will connect to localhost:6379.
// We are going to cover how to specify connection options soon.
const redis = new Redis();

redis.set("mykey", "value"); // Returns a promise which resolves to "OK" when the command succeeds.

// ioredis supports the node.js callback style
redis.get("mykey", (err, result) => {
  if (err) {
    console.error(err);
  } else {
    console.log(result); // Prints "value"
  }
});

// Or ioredis returns a promise if the last argument isn't a function
redis.get("mykey").then((result) => {
  console.log(result); // Prints "value"
});

redis.zadd("sortedSet", 1, "one", 2, "dos", 4, "quatro", 3, "three");
redis.zrange("sortedSet", 0, 2, "WITHSCORES").then((elements) => {
  // ["one", "1", "dos", "2", "three", "3"] as if the command was `redis> ZRANGE sortedSet 0 2 WITHSCORES`
  console.log(elements);
});

// All arguments are passed directly to the redis server,
// so technically ioredis supports all Redis commands.
// The format is: redis[SOME_REDIS_COMMAND_IN_LOWERCASE](ARGUMENTS_ARE_JOINED_INTO_COMMAND_STRING)
// so the following statement is equivalent to the CLI: `redis> SET mykey hello EX 10`
redis.set("mykey", "hello", "EX", 10);
```

```
// 실제 내가 사용한 
```
작성중

