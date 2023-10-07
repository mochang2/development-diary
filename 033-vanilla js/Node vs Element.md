## Node vs Element

간단히 말해서, `Node`는 모든 DOM 객체이다.  
`Node`는 DOM 계층 구조의 모든 개체 유형에 대한 generic name이다.  
`Node`는 `document` 또는 `document.body`와 같은 기본 제공 DOM 요소 중 하나일 수 있다.  
또한 `<input>`, `<p>`와 같은 지정된 HTML tag나 text, comment 또한 `Node`이다.

`Element` 이러한 `Node` 중 특정 타입, 즉 `Node.ELEMENT_NODE`이다.  
`<input>`, `<p>`와 같은 HTML tag로 적은 모든 `Node`를 의미한다.
