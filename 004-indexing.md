## 0. 공부하게 된 계기
나중에 DB 설계할 때 필요할 수 있기 때문에 인덱싱을 공부했다.  
~'인덱싱을 건다'라고 많이 표현하더라~  
DB에는 여러 개의 테이블이 존재하고, 그 테이블에는 다양한 수의 데이터가 쌓인다.  
예상치 못하게 많은 데이터가 쌓여있고, 여러 조건을 조합해서 데이터를 조회할 때, 인덱스를 설정하면 검색 속도가 빨라질 수 있다.  
인덱스를 설정할 때 정답은 없다.  
각각의 데이터에 따라 좋은 인덱싱이 될 수도 있고, 쓸모없는 인덱싱이 될 수도 있다.  
https://yurimkoo.github.io/db/2020/03/14/db-index.html 여기를 참고하여 작성했다.  
_추가) https://www.youtube.com/watch?v=VO9bqmJ0Ns8 인덱싱을 하는 이유와 비선형 데이터를 통한 인덱싱의 간단한 원리_
  
## 1. 개념
### 한 마디로 색인
현재 많이 사용하는 DB들은 주로 해시 테이블, B-tree나 B+ tree 자료 구조를 사용하여 인덱스를 구현한다.  
인덱스는 테이블의 동작 속도를 높여주는 자료 구조이다.  
단, 이는 인덱스가 잘 설정되어있을 경우에만 해당한다.  
다음과 같은 경우에 인덱스를 사용하면 좋다.  

* 규모가 큰 테이블
* 삽입, 수정, 삭제 작업이 자주 발생하지 않는 컬럼
* where나 order by, join 등이 자주 사용되는 컬럼
* 데이터의 중복도가 낮은 컬럼

### where 절에서 효과가 있음
인덱스는 where 절에 사용할 컬럼에 대한 효율화라고 볼 수 있다.  
where 절을 사용하지 않고 인덱스가 걸린 컬럼을 조회하는 것은 성능에 아무런 영향이 없다.  
만약 인덱스가 '학번'에 걸려 있다고 하면

```
1번) SELECT '학번' FROM '학생';
2번) SELECT '전화번호' FROM '학생' WHERE '이름' = "김철수";
3번) SELECT * FROM '학생' WHERE '학번' = 1;
```

위 예에서 3번에서만 성능을 향상시킨다.


## 2. 어디에 설정해야 좋은가
인덱스는 하나 혹은 여러 개의 컬럼에 대해 설정할 수 있다.  
단일 인덱스를 여러 개 생성할 수도, 여러 컬럼을 묶어 복합 인덱스를 설정할 수도 있다.  
그러나 무조건 많이 설정하는게 검색 속도를 향상시켜주지는 않는다.  
인덱스는 데이터베이스 메모리를 사용하여 테이블 형태로 저장되므로 개수와 저장 공간은 비례한다.  
따라서, 인덱스는 조회시 자주 사용하고, 고유한 값 위주로 설정해주는 것이 좋다.

### 카디널리티(cardinality)
카디널리티는 전체 행에 대한 특정 컬럼의 중복 수치를 나타내는 지표이다.  
중복도가 ‘낮으면’ 카디널리티가 ‘높다’고 표현한다.  
중복도가 ‘높으면’ 카디널리티가 ‘낮다’고 표현한다.  
카디널리티는 절대적인 수치보다는 상대적인 수치, 즉 어떤 것보다는 낮고 어떤 것보다는 높다 정도로 이해하는 것이 좋다.  
  
__카디널리티가 높을수록(한 컬럼이 갖고 있는 중복 정도가 낮을수록) 인덱스 설정에 좋다.__  
위에서 말한 것처럼, unique key와 같은 경우는 중복 정도가 낮으므로 카디널리티가 높다.  
반면 나이같은 column이 있다면 unique key에 비해 카디널리티가 낮다.

### 선택도(selectivity)
__선택도가 낮을수록 인덱스에 설정에 좋다.__  

```plain text
= 컬럼의 특정 값(유니크한 값) 의 row 수 / 테이블의 총 row 수 * 100
= 컬럼의 값들의 평균 row 수 / 테이블의 총 row 수 * 10
```

위 식은 선택도를 계산하는 식이다.

### 활용도
__활용도가 높을수록 인덱스 설정에 좋은 컬럼이다.__  
해당 컬럼이 실제 작업에서 얼마나 활용되는지에 대한 값이다.  
수동 쿼리 조회, 로직과 서비스에서 쿼리를 날릴 때 WHERE 절에 자주 활용되는지를 판단하면 된다.  


## 3. mongoDB에서 index
https://ryu-e.tistory.com/1 여기와 [공식문서](https://www.mongodb.com/docs/manual/tutorial/getting-started/)를 참고했다.  
mongoDB에는 두 가지 방법으로 인덱스를 설정할 수 있다.  
하나는 Single Field Index와 Compound Index이다.  

### Single Field Index
하나의 필드만 인덱스를 사용하는 것을 말한다.  
1은 오름차순, -1은 내림차순을 의미하는데 Single Field Index에서는 오름차순인지 내림차순인지는 중요하지 않다.  

```sh
> db.user.createIndex({ field: 1 }) // 1 또는 -1
```

위는 일반적인 mongoDB 명령어이고, 아래는 mongoose 코드이다.

```javascript
import { Schema } from 'mongoose'

const animalSchema = new Schema({
  name: String,
  type: String,
  tags: { type: [String], index: true } // field level
});

animalSchema.index({ tags: 1 }); // schema level
```

![몽고 공식문서 단일 인덱스](https://user-images.githubusercontent.com/63287638/173453518-bc795649-c42b-4cd7-a03a-24a708ac24b7.png)  
  
(출처: https://docs.mongodb.com/manual/indexes/)

### Compound Index
두 개 이상의 필드를 사용하는 인덱스를 말한다.  
아래 명령어는 `field1`을 오름차순으로, `field2`를 내림차순으로 정렬한다.  
~mongoose 명령어는 위에 있는 코드와 크게 다를 것 없으니 생략~  

```sh
> db.user.createIndex({ field1: 1, field2: -1 })
```

![몽고 공식문서 복합 인덱스](https://user-images.githubusercontent.com/63287638/173456140-f3fda7b6-d98e-4e81-ac7b-c7538575f432.png)
  
(출처: https://docs.mongodb.com/manual/indexes/)

  
고려할 점
* 인덱싱 순서
  * Compound Index에서는 Single Field Index와 다르게 키의 순서가 중요하다. `field1-field2`로 인덱스를 생성했다면 `field1-field2` 순서의 정렬은 지원하지만 `field2-field1` 순서의 정렬은 지원하지 않는다.
* 정렬 방향
  * 검색 쿼리에서 Compound Index를 사용하는 경우 지정된 정렬 방향은 index와 일치해야 한다. 아래 예시를 보면 무슨 뜻인지 파악할 수 있다.
```sh
> db.user.createIndex({ field1: 1, field2: -1 })

# 실행 시 문제 없이 진행 
> db.user.find({}).sort({ field1: 1, field2: -1 })
> db.user.find({}).sort({ field1: -1, field2: 1 })

# RAM exceeded error 발생
> db.user.find({}).sort({ field1: 1, field2: 1 })
> db.user.find({}).sort({ field1: -1, field2: -1 })
```

* prefix
  * Compound Index에서 검색 쿼리 시 왼쪽 인덱스부터 적용되는 부분집합 인덱스를 말한다. `{ field1: 1, field2: 1, field3: 1, field4: 1}`로 인덱스를 생성했을 경우, `{ field1: 1 }`, `{ field1: 1, field2: 1 }`, `{ field1: 1, field2: 1, field3: 1 }`, `{ field1: 1, field2: 1, field3: 1, field4: 1}` 이렇게 4가지의 경우의 인덱스를 지원한다. 이 말은 `{ field1: 1, field2: 1, field3: 1, field4: 1}`로 인덱스를 생성했다면 `{ field1: 1, field2: 1 }` 등의 인덱스는 삭제해도 된다는 뜻이다.  

```sh
- 생성된 인덱스:  { field1: 1, field2: 1, field3: 1, field4: 1}

- 지원되는 쿼리: { field1: 1 }
- 지원되는 쿼리: { field1: 1, field2: 1 }
- 지원되는 쿼리: { field1: 1, field2: 1, field3: 1 }

- 지원하지 않는 조회 쿼리: "field1" 필드 없이 "field2" 필드만 존재 혹은 "field3" 필드만 존재 
- 지원하지 않는 조회 쿼리: "field1" 필드 없이 "field2", "field3" 필드만 존재 
```

* non-prefix
  * 다만 sort 연산은 non-prefix를 지원한다. 위 예시처럼 `{ field1: 1, field2: 1, field3: 1, field4: 1}`로 인덱스를 생성했을 경우여도 `sort by field2, field3` 등을 지원한다는 뜻이다.
* index intersection
  * 별개의 인덱스가 교차해서 쿼리에 자동으로 적용되는 것을 말한다.

```sh
- 인덱스 1: { field1: 1 }
- 인덱스 2: { field2: 1 }

> db.orders.find( { field1: "what1", field2: { $ne: "what2" } } )
```

