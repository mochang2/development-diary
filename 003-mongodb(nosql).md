## 0. 공부하게 된 계기
double nc에서 다양한 프로젝트가 있었는데 DB 중 하나로 mongoDB를 사용한다.  
내가 직접적으로 다루는 DB는 아니었지만 공부하면 도움이 될 것 같아 정리를 했다.  
주요 내용은 NoSQL과 SQL의 차이이며, mongoDB만의 특징은 그냥 참고용으로 적었다.  
참고한 블로그는 https://kciter.so/posts/about-mongodb 이다.
  
## 1. 사전지식
##### BSON
binary + json을 의미한다. mongodb에서는 bson 형태로 데이터를 저장한다.

##### 샤딩(출처: https://nesoy.github.io/articles/2018-05/Database-Shard)
~강대명 멘토님 시간에 배웠는데 금방 까먹었다.~  

* 같은 테이블 스키마를 가진 데이터를 다수의 데이터베이스에 분산하여 저장하는 방법을 의미한다.
* application level에서도 가능하지만 database level에서도 가능하다.
* Horizontal Partitioning이라고 볼 수 있다.  

샤딩은 프로그래밍의 복잡성이 증가하기 때문에 최대한 피하는 방법을 우선 찾아봐야 된다.  
* Scale-in
  * Hardware Spec이 더 좋은 컴퓨터를 사용한다.
* Read 부하가 크다면?
  * Cache나 Database의 Replication(master, slave로 나누는 것)을 적용하는 것도 하나의 방법이다.
* Table의 일부 컬럼만 자주 사용한다면?
  * Vertically Partition도 하나의 방법이다.
  * Data를 Hot, Warm, Cold Data로 분리하는 것도 가능하다.


## 2. NoSQL
없다의 no가 아니라 Not Only의 줄인말이었다.  
즉, SQL 뿐만 아니라~ 라는 뜻으로 SQL을 사용하는 RDB가 아닌 DB를 의미한다.  
RDB로는 MySQL, Oracle, PostgreSQL 등이 있고, NoSQL에는 mongoDB, Redis, Cassandra, HBase 등이 있다.  

##### vs RDBMS
NoSQL의 가장 큰 특징은 _수평 확장 가능한 분산 시스템, Schema-less, 완화된 ACID_ 이다.  
특히 SNS와 같이 간단하고 엄청나게 많은 데이터를 처리하기 위해, 그리고 기존 RDB로는 표현하기 힘든 다양한 데이터를 처리하기 위해 생겨났다.  
자세한 차이는 아래 표를 보면 알 수 있다.  

|  | RDBMS | NoSQL
|-----|-------|-----|  
|적합한 사용례|데이터 정합성이 보장되어야 하는 은행 시스템|낮은 지연 시간, 가용성이 중요한 SNS 시스템|
|데이터 모델|정규화와 참조 무결성이 보장된 스키마|스키마가 없는 자유로운 데이터 모델|
|트랜잭션|강력한 ACID 지원|완화된 ACID(BASE)|
|확장|하드웨어 강화(scale up)|수평 확장 가능한 분산 아키텍처(scale out)|
|API|SQL 쿼리|객체 기반 API 제공|

위에 표가 전부 맞는 말은 아니다.  
마치 NoSQL은 ACID가 안되는 것처럼 적어놨지만, RDBMS에 비해 부족한 것 뿐이지 가능하다.  
또한 RDBMS도 수평 확장이 불가능한 것처럼 적어놨지만 MySQL Replication 등을 사용하면 수평 확장이 가능하다.  

## 3. mongoDB
* Document(BSON)
* BASE(염기)
* Open Source

위 네 가지 단어가 mongoDB를 나타내주는 가장 큰 특징이다.  
데이터는 document(그냥 json이라고 봐도 무방) 기반으로 구성되어 있고, ACID(산) 대신 BASE를 택하여 성능과 가용성을 우선시하며, open source라 무료이다.  
또한 BSON으로 데이터가 쌓이기 때문에 Array 데이터나 Nested한 데이터를 쉽게 넣을 수 있다.  

##### 표현
|RDBMS|mongoDB|
|------|-------|
|Database|Database|
|Tables|Collections|
|Rows|Documents|
|Columns|Fields|

##### ObjectID
<p align="center">
<img src="https://kciter.so/images/2021-02-25-about-mongodb/bson.png" alt="json(bson)의 형태" />
</p>

위 데이터 구조에서 ObjectId라는 생소한 타입을 볼 수 있다.  
ObjectId는 RDBMS의 Primary Key와 같이 고유한 키를 의미하는데 차이점은 Primary Key는 DBMS가 직접 부여한다면 ObjectId는 클라이언트에서 생성한다는 점이다.  
이는 MongoDB 클러스터에서 Sharding된 데이터를 빠르게 가져오기 위함인데 Router(mongos)는 ObjectId를 보고 데이터가 존재하는 Shard에서 데이터를 요청할 수 있다.  
의아하게도 MongoDB 서버에서 알아서 ObjectId를 부여해서 저장해도 될 것 같은데 딱히 지원해주지 않는다.  
참고로 ObjectId를 넣지않고 저장한다면 데이터가 그대로 저장된다.  

##### BASE
BASE는 ACID와 대립되는 개념으로 다음 세 가지로 이루어진다.  
간단하게 정리하자면 일관성을 어느 정도 포기하고 가용성을 우선시, 즉, 일단 요청이 발생하면 데이터가 일치하지 않더라도 보내주는 것을 우선시한다는 것이다.  
* **B**asically **A**valiable
  * 기본적으로 언제든지 사용할 수 있다는 의미이다.
  * 가용성이 필요하다는 뜻을 나타낸다.
* **S**oft state
  * 외부의 개입이 없어도 정보가 변경될 수 있다는 의미이다.
  * 네트워크 파티션 등 문제가 발생되어 일관성이 유지되지 않는 경우 일관성을 위해 데이터를 자동으로 수정한다.
* **E**ventually consistent
  * 일시적으로 일관적이지 않은 상태가 되어도 일정 시간 후 일관적인 상태가 되어야 한다는 의미이다.
  * 장애 발생시 일관성을 유지하기 위한 이벤트를 발생시킨다.

## 4. 분산시스템
NoSQL이 나온 이유 중 하나는 대규모 데이터를 처리하는데 RDBMS에서는 한계가 있었기 때문이다.  
일관성과 무결성을 조금 포기하더라도 빠른 읽기 성능과 수평 확장이 수월한 DB가 필요했다.  

##### CAP 이론
어떤 분산 시스템도 Consistency(일관성), Availability(가용성), Partition tolerance(분할 내성)을 모두 만족할 수 없다는 이론이다.  
* Consistency
  * 모든 노드가 같은 시간에 같은 데이터를 볼 수 있다는 의미이다.
  * 데이터가 업데이트된 후 다른 노드에 동기화가 필요한데, 이 때 유저는 동기화를 위해 대기해야 한다.
  * 동기화를 위한 대기 시간이 길어질 경우 가용성이 떨어진다.

<p align="center">
<img src="https://kciter.so/images/2021-02-25-about-mongodb/consistency.png" alt="" />
</p>

* Availablility
  * 모든 요청에 성공 혹은 실패 결과를 반환할 수 있다는 의미이다.
  * 하나의 노드가 망가져도 다른 노드를 통해 데이터를 제공할 수 있다면 가용성이 있는 시스템이다.
  * 다만 다시 노드가 살아났을 때 다른 노드와 데이터가 다르다면 일관성 떨어지게 된다

<p align="center">
<img src="https://kciter.so/images/2021-02-25-about-mongodb/availability.png" alt="" />
</p>

* Partition tolerance
  * 통신에 실패해도 시스템이 계속 동작해야 한다는 의미이다.
  * 노드가 망가진 것이 아닌 노드끼리 연결시켜주는 네트워크가 고장나는 경우, 동기화가 불가능하면 일관성이 떨어진다.
  * 만약 통신이 복구되고 동기화되는 것을 기다린다면 가용성이 떨어진다.

<p align="center">
<img src="https://kciter.so/images/2021-02-25-about-mongodb/partition-tolerance.png" alt="" />
</p>

##### PACELC 이론
CAP 이론에서 P 즉, 노드끼리 통신이 원활하지 않는 경우는 100% 방지할 수 없다는 사실에서 출발하여, 네트워크 파티션 상황은 반드시 발생한다고 가정한다.  
PACELC은 다음으로 이루어져 있다.

|구분|구성|
|-----|-------|
|Partition|Availability|
|Partition|Consistency|
|Else|Latency|
|Else|Consistency|

위에 두 개는 Partition이 발생할 경우에는 어떤 것을 취할 것인지, 아래 두 개는 Partition이 발생하지 않을 경우(정상 상태)에는 어떤 것을 취할 것인지를 결정한다.  
PA / EL이라면 네트워크 파티션 상황일 때 가용성을 더 우선시하고 평상시에도 지연 시간을 더 신경쓰므로(일관성을 신경쓰느라 지연 시간이 늦어질수록 가용성이 떨어진다) 가용성을 우선시한다는 뜻이 된다.  
mongoDB는 PA /EC 시스템이므로 네트워크 파티션 상황일 때는 가용성을 더 우선시하고 평상시에는 일관성을 더 우선시한다.  

## 4. mongoDB 패턴
아래부터는 그냥 참고용이다.  
mongoDB는 document라는 방식을 사용하기 때문에 모델링을 해야 하며 이를 위해 패턴을 정리할 필요가 있다.  

##### Model Tree Structure
일반적인 tree를 생각하면 된다.  
기본적인 tree를 그릴 때 항상 문제가 되는 것은 부모를 어떻게 찾냐는 것이다.  
이러한 문제에 따라 reference하는 법을 다르게 해 tree를 짤 수 있다.  

* Parent References
  * 부모 document를 바로 찾아야 할 경우 적합하다.
  * 하위 트리를 모두 찾아야 하는 경우에는 적합하지 않다.
  * 아래와 같은 구조를 가졌다.

```
[
  { _id: "MongoDB", parent: "Databases" },
  { _id: "dbm", parent: "Databases" },
  { _id: "Databases", parent: "Programming" },
  { _id: "Languages", parent: "Programming" },
  { _id: "Programming", parent: "Books" },
  { _id: "Books", parent: null }
]
```

* Child References
  * 자식 document를 바로 찾아야 할 경우 적합하다.
  * 부모 Document를 찾을 수 있지만 Parent References보다 성능이 느리다.
  * 아래와 같은 구조를 가졌다.

```
[
  { _id: "MongoDB", children: [] },
  { _id: "dbm", children: [] },
  { _id: "Databases", children: [ "MongoDB", "dbm" ] },
  { _id: "Languages", children: [] },
  { _id: "Programming", children: [ "Databases", "Languages" ] },
  { _id: "Books", children: [ "Programming" ] }
]
```

* Array of Ancestors
  * 조상 Document와 자식 Document를 모두 찾아야 하는 경우 적합하다.
  * 만약 부모가 여럿일 경우 적합하지 않다.
  * 아래와 같은 구조를 가졌다.

```
[
  { _id: "MongoDB", ancestors: [ "Books", "Programming", "Databases" ], parent: "Databases" },
  { _id: "dbm", ancestors: [ "Books", "Programming", "Databases" ], parent: "Databases" },
  { _id: "Databases", ancestors: [ "Books", "Programming" ], parent: "Programming" },
  { _id: "Languages", ancestors: [ "Books", "Programming" ], parent: "Programming" },
  { _id: "Programming", ancestors: [ "Books" ], parent: "Books" },
  { _id: "Books", ancestors: [ ], parent: null }
]
```

* Materialized Paths
  * Array of Ancestors와 유사하다. Array 타입이 아닌 String 타입을 이용하는데 정규식을 이용하여 하위 항목을 찾을 수 있다. 이때 하위 트리를 찾는데에 Array of Ancestors보다 빠르다.
  * 단, 공통 부모를 찾아야 하는 경우엔 더 느려질 수 있다.
  * 아래와 같은 구조를 가졌다.

```
[
  { _id: "Books", path: null },
  { _id: "Programming", path: "Books" },
  { _id: "Databases", path: "Books,Programming" },
  { _id: "Languages", path: "Books,Programming" },
  { _id: "MongoDB", path: "Books,Programming,Databases" },
  { _id: "dbm", path: "Books,Programming,Databases" }
]
```

* Nested Sets
  * 하위 트리를 찾는데 가장 빠르고 효율적이다.
  * 구조가 변경되는 경우 다시 데이터 번호를 매기는데 비용이 크기 때문에 데이터가 추가, 삭제, 변경되지 않는 정적인 구조에 적합하다.
  * 아래와 같은 구조를 가졌다.

```
[
  { _id: "Books", parent: 0, left: 1, right: 12 },
  { _id: "Programming", parent: "Books", left: 2, right: 11 },
  { _id: "Languages", parent: "Programming", left: 3, right: 4 },
  { _id: "Databases", parent: "Programming", left: 5, right: 10 },
  { _id: "MongoDB", parent: "Databases", left: 6, right: 7 },
  { _id: "dbm", parent: "Databases", left: 8, right: 9 }
]
```

##### 모델링 패턴
글이 너무 길기 때문에 자세한 것은 https://kciter.so/posts/about-mongodb#modeling-pattern 여기서 참조하면 된다.

* Attribute
  * 동일한 필드를 묶어서 인덱싱 수를 줄이는 패턴이다.
* Extended Reference
  * 서로 관계가 있는 Document에서 자주 사용되는 데이터를 저장해두는 패턴이다.
  * mongoDB에서는 성능을 위해 join 대신 쿼리를 두 번 날려 데이터를 불러오는데, 데이터가 많아질수록 비효율적이기 때문에 이러한 패턴을 사용한다.
* Subset
  * 관계가 있는 Document 사이에 자주 사용되는 데이터를 부분적으로 Embed하는 패턴이다.
  * 최신 5개의 리뷰만 빠르게 보여줄 때 등에 사용된다.
  * 참고로 만약 데이터 수정이 발생한다면 양쪽(원본 데이터, subset)을 모두 수정해야한다.
* Computed
  * 미리 통계 수치를 데이터 삽입할 때 계산하는 패턴이다.
  * 집계 합수는 데이터가 많을 수록 성능이 느리기 때문에 조금 오차가 발생해도 괜찮다면 사용된다.
* Bucket
  * 하나의 필드를 기준으로 Document를 묶는 패턴이다.
  * 이 경우 필드 추가, 삭제에도 용이하고 인덱스 크기도 절약이 가능하다.

## 5. 요약
RDBMS와 NoSQL은 나온 배경이 다르다.  
NoSQL은 빅데이터와 클라우드 등이 대두되는 현재, 최대한 단순하면서 많은 데이터를 다루기 위해 나타났다.  
반면 RDBMS는 기존 정적인 데이터를 처리하기 위해 사용되었으며, 복잡하면서 무결성이 중요한 데이터에 적합하다.

## 6. mongoose code example
시퀄라이즈에 비해 공식 문서가 훨씬 잘 되어 있는 편이지만 혹시 모를 나중을 위해 모델 정의 관련된 코드만 정리한다.  
~쿼리문은 공식 문서가 잘 되어 있으니 패스~
참고로 몽구스는 objectId라고 document마다 고유한 값을 가진 값을 자동으로 만들어준다.  
`timestamp: true` 로 설정하면 createdAt과 updatedAt 값이 자동으로 할당되며, 서버 시간에 맞춰서 반영된다.  

```typescript
import { model, Schema } from 'mongoose'

export interface someInterface {
  id: string
  name: string
  property: number
  key: string
}

const someSchema = new Schema<someInterface>(
  {
    id: {
      type: String,
      required: true,
    },
    name: {
      type: String,
      required: true,
    },
    property: {
      type: Number,
      required: true,
    },
    key: {
      type: String,
      required: true,
    },
  },
  {timestamps: true},
)
someSchema.index({id: 1, name: 1, property: 1})

const SomeModel = model<someInterface>(
  'Some',
  someSchema,
)

export default SomeModel
```

