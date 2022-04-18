## 0. 공부하게 된 계기
DB 시간에 시험범위가 아니라고 가볍게 넘어갔던 것 같다.  
그래서 정규화란 이름을 들어봤다는 사실도 까먹었다...  
이참에 제대로 정리하고 한다.  
다만 사진이 너무 많이 필요하기 때문에 여기에선 말로만 설명하고, 내가 이해하도록 도와준 블로그 사진을 보고 자세히 보는 게 나을 것 같다.
https://velog.io/@bsjp400/Database-DB-%EC%A0%95%EA%B7%9C%ED%99%94-%EB%B9%84%EC%A0%95%EA%B7%9C%ED%99%94%EB%9E%80 와https://mangkyu.tistory.com/110 룰 참고하면 되겠다.  
참고로 지금 내가 쓰고 있는 mongoDB와는 관련 없고 SQL과 관련이 있는 용어다.

## 1. 정규화 vs 비정규화
##### 개념
정규화란 DB 설계에서 중복을 최소화하기 위해 데이터를 구조화하는 프로세스를 말한다.  
이와 반대로 비정규화란 정규화된 엔티티, 속성, 관계를 시스템의 성능 향상 및 개발과 운영의 단순화를 위해 중복, 통합, 분리 등을 수행하는 프로세스를 말한다.

##### 목적
정규화  
* 불필요한 데이터를 제거  
* DB 구조 확장 시 재디자인 최소화
* 무결성 제약조건의 시행을 간단히 하기 위해
* Anomaly를 방지하기 위해(테이블의 구성을 논리적으로 만듦으로써)

비정규화
* 디스크 I/O량이 많아서 조회 시 성능이 저하될 때
* 테이블끼리의 경로가 너무 멀어 조인으로 인한 성능 저하가 예상될 때
* 컬럼을 계산하여 조회할 대 선응이 저하될 것으로 예상될 때

## 2. 제1정규화(1NF)
테이블의 컬럼이 원자값(atomic value), 즉 단 하나의 값만 갖도록 테이블을 분해하는 것이다.  
제1정규화를 만족하면 모든 attribute에 반복되는 그룹이 나타나지 않으며 기본 키를 사용하여 관련 데이터의 각 집합을 고유하게 식별할 수 있게 된다.  
학생이라는 테이블이 있을 때 수강과목이라는 컬럼이 있다고 하자.  
단, 수강과목은 PK가 아닌데, 일반적으로 학생은 여러 과목을 들을 것이다.  
따라서 수강과목 컬럼에 '컴구', '네트워크' 이런 식으로 복수의 객체를 집어넣고자 하는 욕망이 생기겠지만, 그러면 안 된다는 것이다.

```
PK - '컴구'  
PK - '네트워크'  
```

위와 같은 식으로 테이블에 넣어야 된다.

## 3. 제2정규화(2NF)
제1정규화를 진행한 테이블에 대해 완전 함수 종속을 만족하도록 테이블을 분해하는 것이다.  
완전 함수 종속이라는 것은 PK의 부분집합이 다른 컬럼의 값을 결정해서는 안 된다는 것이다.  
만약 ('학생번호', '강좌이름')이 PK이고 같은 테이블에 '강의실', '성적'이라는 컬럼이 있다고 하자.  
성적은 '학생번호', '강좌이름' 두 개에 동시에 종속적으로 결정되지만  
'강의실' 같은 경우는 '강좌이름'만으로도 종속시킬 수 있다.  
이런 경우 테이블을 두 개로 나누어 '학생번호', '강좌이름', '성적'을 가지고 있는 테이블 한 개,  
'강좌이름', '강의실'을 가지고 있는 테이블 한 개를 만들어야 한다는 것이다.

## 4. 제3정규화(3NF)
제2정규화를 진행한 테이블에 대해 '기본키가 아닌 모든 속성이 기본키에 대해' 이행적 종속을 없애도록 테이블을 분해하는 것이다.  
a->b, b->c면 a->c이다를 DB에서 표현한 것이라고 생각하면 된다.  
만약 이러한 관계의 컬럼이 한 테이블에 동시에 존재한다면 이를 두 개의 테이블로 나눠야 한다는 것이다.

## 5. BCNF정규화(strong 3NF)
제3정규화를 진행한 테이블에 대해 모든 결정자(함수에서 독립변수라고 생각해도 좋을 것 같다)가 PK가 되도록 테이블을 분해하는 것이다.  
만약 PK가 1개가 아니라면 제3정규화까진 만족하지만 BCNF 정규화는 만족하지 못 할 수도 있다.  
한 테이블에 '학생이름', '특강이름', '교수' 세 개의 컬럼이 있다고 가정하자.  
PK는 ('학생이름', '특강이름')이며 '교수'는 한 가지의 특강밖에 개설 못 한다고 가정하자.
그렇다면 여기서 문제가 생긴다.  
'학생이름', '특강이름' -> '교수' 이러한 관계가 생김에 동시에  
'교수' -> '특강이름' 이러한 관계가 생긴다.  
따라서 결정자가 PK가 될 수 있도록, 

```
<특강신청 테이블> '학생번호' -> '교수'  
<특강교수 테이블> '특강이름' -> '교수'  
```

위와 같이 테이블을 나눠야 한다는 것이다.

## 6. 정리
제3정규화와 BCNF정규화가 구분이 어렵다면 PK가 두 개지만 한 컬럼이 PK 중 하나를 결정짓는 컬럼을 생각해면 좋을 듯 싶다.  
이에 대해 https://yaboong.github.io/database/2018/03/10/database-normalization-2/ 여기를 참고하자.  
~위에서 서술한 정규화 외에도 더 많은 것 같지만 DBA가 아니면 이 정도만 알아도 괜찮을 것 같다.~