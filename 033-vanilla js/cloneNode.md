## cloneNode

`Node.cloneNode()` 메서드는 이 메서드를 호출한 Node의 복제된 Node를 반환한다.

`const dupNode = node.cloneNode(option)`과 같은 문법으로 사용된다.

- node: 복제되어야 할 node
- dupNode: 복제된 새로운 node
- option?: node의 children까지 복제할지, 해당 node만 복제할지 여부. default: false

```javascript
const test = document.getElementById('cloneTest');
// test 변수에 복제 할 노드를 지정

const testClone1 = test.cloneNode();
const testClone2 = test.cloneNode();
const testClone3 = test.cloneNode();
// 복사할 개수만큼 복제변수를 생성

body.appendChild(testClone1);
body.appendChild(testClone1);
body.appendChild(testClone1);
```

이 메서드를 사용할 때 주의할 점은 duplicated element ID를 생성할 수 있다는 점이므로, `id` property를 부여한 node라면 지양하는 게 좋을 거 같다.
