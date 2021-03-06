## 0. 공부하게 된 계기

BoB할 때 AWS lambda에 대해 배우고 사용도 해봤지만 여전히 무슨 어떤 때 사용하는 것인지 와닿지 않는다.  
내가 인턴하는 곳에서 lambda를 사용한다고 하니 이참에 개념 정도는 정리해둬야 서비스를 이해하는데 도움이 될 거라고 판단이 된다.  
클라우드 엔지니어링 보다 깊게 알지는 못하겠지만 최소한 그들이 얘기하는 것을 이해하는 정도까지는 공부하고자 했다.  
https://usefultoknow.tistory.com/entry/AWS-Lambda%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C 와 https://www.44bits.io/ko/keyword/aws-lambda 를 참고했다.

## 1. 개념

서버를 프로비저닝하거나 관리하지 않고도 코드를 실행할 수 있는 컴퓨팅 서비스이다.  
흔히들 서버리스 서비스라고 하는데, 말 그대로 서버 없이 돌아간다기 보다는 사용자 입장에서 별도의 서버 설정이 필요 없다는 의미이다.  
참고로 프로비저닝이란 사용자의 요구에 맞게 시스템 자원을 할당, 배치, 배포해 두었다가 필요할 때 시스템을 즉시 사용할 수 있는 상태로 준비해두는 것을 말한다.  
종류에는 서버 자원 프로비저닝, OS 프로비저닝, 소프트웨어 프로비저닝, 스토리지 프로비저닝, 계정 프로비저닝 등이 있다.  
다른 Cloud Service의 비슷한 것으로는 MS Azure의 Azure Functions, GCP의 Cloud Functions, Cloud Run 등이 있다.

### 핸들러

람다 함수에게 어디서부터 코드를 실행하는지, 즉 진입점을 알려주는 부분이다.

### labmda에 코드 올리는 방법

1. 인라인 코드
2. zip파일 업로드
3. S3에서 파일 업로드

## 2. 특징

- 장점

  - 필요 시에만 코드를 실행하며 자동 확장(오토 스케일링)이 가능하다.
  - 코드가 실행되지 않을 때는 요금이 부과되지 않는다. 100ms 단위로 요금을 측정한다고 한다.
  - 애플리케이션이나 백엔드 서비스에 대한 코드를 별도의 관리 없이 실행할 수 있다.
  - 다양한 런타임을 지원한다. Node.js, Python, Ruby, Java, Go, .NET(C#), 사용자 지정 런타임이 그 종류이다.
  - 이벤트에 대한 응답으로 코드를 실행할 수 있다. 가능한 이벤트는 후술하겠다.
  - 인프라에 대한 걱정 없이 코드 실행이 가능하다. 즉, NoOps를 실현한다.
  - 코드가 병렬로 실행되고 각 트리거는 개별적으로 처리되어 정확히 워크로드 규모에 맞게 조정된다.

- 단점
  - 비싸다
  - 함수 실행 시간이 최대 15분이다
  - 처음 함수 호출시 Cold Start를 하기 때문에 초기 지연시간이 발생한다.

## 3. 이벤트(트리거)

- 클라우드와치 이벤트(CloudWatch Events)
  - 시간을 사용하는 경우 크론잡과 같이 실행 스케줄을 지정해 람다를 실행할 수 있다.
  - 이벤트 기반을 사용하는 경우 AWS의 다른 서비스와 결합해 람다를 실행할 수 있다.
- S3
  - S3 버킷의 특정한 이벤트를 소스로 받아 람다를 실행할 수 있다.
- 외부 이벤트
  - 데이터독(Datadog), 뉴 렐릭(New Relic), 페이저듀티(PagerDuty), 젠데스크(ZenDesk) 등의 서비스를 통해 람다 함수를 실행할 수 있다.
- API 게이트웨이
  - HTTP 요청을 통해 람다를 실행할 수 있다. 즉, REST API 이용시 백엔드처럼 사용할 수 있다.

## 4. 사용 예시

- 실시간 파일 처리
  - 이미지 썸네일링, 동영상 트랜스코딩, 파일 인덱싱, 로그 처리, 콘텐츠 검증, 데이터 수집 및 필터링
- 실시간 스트림 처리 등
  - 애플리케이션 활동 추적, 트랜잭션 주문 처리, 클릭 스트림 분석, 데이터 정리, 지표 생성, 로그 필터링, 인덱싱, 소셜 미디어 분석 등
- 추출, 변환, 로드
  - DynamoDB 테이블의 모든 데이터 변경에 대한 데이터 검증, 필터링, 정렬 또는 기타 변환 작업을 수행하고 변환된 데이터를 다른 데이터 스토어로 로드
- IoT, 모바일 백엔드, Web Application
