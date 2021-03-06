## 0. 공부하게 된 계기

리액트로 프론트 엔드를 공부하던 와중에 application/json 타입으로 통신을 할 때도 있었고,  
multipart/form-data로 통신을 할 때도 있었다.  
영문을 모르고 사용했었지만, api 명세화를 맡은 김에 확실히 두 가지의 차이점을 알아보기로 했다.  
결론부터 말하자면 string 등의 타입만 보낼 때는 json이 편리하지만, 음악/사진/동영상 등은 form data가 편리하다고 생각한다.  
참고한 두 가지 사이트는 https://2ham-s.tistory.com/307 와 https://velog.io/@ryan-son/multipartform-data-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0-vs-applicationjson 이다.

## 1. application/json

JSON이란 JavaScript Object Notation의 줄임말로, 데이터를 전송하거나 저장할 때 많이 사용되는 경량의 데이터 교환 방식을 말한다.  
다음은 장점과, application/json 형식으로 바이너리 데이터를 취급할 때의 단점을 정리했다.

- 장점
  - 파일과 함께 전달되는 데이터의 연관 관계를 표현하기 용이.
  - 아스키 데이터를 전달하기에 편함.
- 단점
  - 바이너리 데이터를 다루기 위해서는 인코딩이 필요한데, 이를 위해서는 33~37%(base64와 같은 경우) 더 용량이 커짐.
  - 서버는 디코딩, 클라이언트는 인코딩을 위한 오버헤드가 추가.
  - 인코딩한 파일 컨텐츠의 문자열이 길어져서 서버측에서 파일을 읽지 못하는 경우가 발생할 수 있음.

## 2. multipart/form-data

이 타입은 MIME 데이터 스트림의 규칙을 준수한다.  
(프론트) `append` 메소드로 key-value 값을 하나씩 추가해주고, 같은 key를 가진 값을 여러 개 넣을 수 있다.  
단, `set`으로 넣으면 덮어쓰인다.  
_숫자를 넣어도 문자열이 되고, 배열을 넣어도 콤마로 구분한 문자열이 된다. 또한 객체를 넣으면 무시가 된다._
위 특징을 잘 참고해서 사용해야 한다.

- 장점
  - application/json에서 발생하는 오버헤드, 용량 등을 감수하지 않아도 됨.
- 단점
  - 파일과 함께 전달된 데이터의 연관 관계 표현이 제한됨.

## 3. 추가사항

데이터를 보내는 각각의 방식에는 장단점이 있다.  
검색을 하다 알게된 사실인데, form-data와 json으로 각각을 동시에 보내는 방법도 있다.  
두 가지의 장점을 살리기 위해서 고안된 방법 같은데, form 안에 json을 넣어서 보냄으로써 구현된다.
