## Immutability

**불변성: 메모리 영역에서 값을 변경할 수 없음**

JS에서는 객체 관리를 '불변'하게 한다.  
이 말 뜻은 객체가 생성된 이후 그 상태를 변경할 수 없다는 것이다.  
여기서 상태를 변경한다는 것과 값을 재할당하는 것은 다른 의미이다.

https://sustainable-dev.tistory.com/156 의 말을 빌려보자면 불변성은

> 불변성을 지켜 명시적으로 작성된 코드는 다른 개발자가 코드를 보았을 때도 내가 모르는 어딘가에서 데이터가 변화했을거야! 라는 불필요한 의심없이 코드를 읽는 그대로 흐름을 따라가면서 이해할 수 있도록 돕는다. 따라서 불변성을 지키면서 데이터를 변화시킨다면, 예상가능하고 신뢰할 수 있는 코드가 될 수 있다. 애초에 불변성을 지켜야 한다는 것은 react가 만들어낸 새로운 컨셉이 아니라 불변성이라는 개념을 지켜가면서 state와 props를 이용할 수 있도록 하는 아이디어를 react에 녹여낸 것이다.

JS에서 primitive type들, 즉 Boolean, String, Number, Null, Undefined, Symbol이 불변하는 타입들이다.
이 값들은 메모리 영역 안에서 변경이 불가능하며 변수에 할당할 때 완전히 새로운 값이 만들어져서 할당된다.

```javascript
let str = 'str';
let newStr = str;
str = 'str2';

console.log(str); // 'str2'
console.log(newStr); // 'str'
```

1. 'str'라는 string 타입의 값이 메모리에 생성되고, `str`은 'str' 메모리 값을 가리킨다.
2. `newStr`은 `str`이 가리키고 있는 주소 'str'을 가리킨다.
3. 'str2'라는 string 타입의 값이 메모리에 생성되고, `str`은 'str2' 메모리 값을 가리킨다.

최종적으로 `newStr`은 여전히 'str'를 가리키고 있으며 `str`은 'str2'를 가리키고 있다.

![immutability](https://user-images.githubusercontent.com/63287638/187012810-6c9b6161-c903-4e2a-ba5a-a557cdd9bf8a.png)

### const

`let`과 달리 `const`는 재선언 및 재할당이 불가능하다.  
다만, `const`는 선언한 변수 값이 불변하다는 의미가 아니라, `const`는 값에 대한 '참조'가 한 번 변수에 할당되고 나면 변할 수 없음을 의미하는 것이다.  
`const` 변수가 참조하고 있는 '값'이 불변한다는 것을 의미하지 않는다.

위 예시에서 `let` 대신에 `const`를 선언하면 `str = 'str2'` 구문에서 에러가 발생한다.

### 객체의 불변성?

객체도 다음과 같이 사용하면 위에서 설명한 과정과 똑같은 순서를 거친다.

```javascript
let arr1 = [1, 2, 3];
let arr2 = arr1;
arr1 = [1, 2, 3, 4];

console.log(arr1); // [1, 2, 3, 4]
console.log(arr2); // [1, 2, 3]
```

다만 primitive type과 달리 object type는 '불변하다'고 표현하지 않는다.  
새로운 값이 만들어지지 않고 메모리 영역에서 직접적으로 변경이 가능하기 때문이다.

```javascript
const x = {
  // let이어도 동일한 결과
  name: '123',
};

const y = x;

x.name = '456';

console.log(y.name); // 456
console.log(x === y); // true
```

1. `x`에 새로 만든 객체를 할당한다.
2. `y`가 `x`가 가리키고 있는 객체의 주소를 똑같이 가리킨다.
3. `x`가 가리키고 있는 객체의 `name`에 '123'이라는 string 데이터를 다시 할당한다.
4. `y.name`을 출력하면 `456`이 출력되는데, y는 x가 가리키고 있는 값, 원래는 `{ name: '123' }` 었다가 지금은 `{ name: '456' }` 으로 변화한 데이터의 주소를 똑같이 참조하고 있기 때문이다.
5. 따라서 마지막 구문에서도 `x`와 `y`는 똑같은 데이터를 가리키고 있기 때문에 `true`가 출력된다. ~사실 객체는 얕은 참조라서 `false`가 나올 줄 알았는데 같은 주소값을 가리키므로 `true`가 나오는 게 맞는 것 같다.~

배열도 object이므로 위와 같은 시도를 할 때 동일한 결과가 발생한다.

객체는 불변하지 않다.  
다만 불변성을 지키기 위한 경우가 필요하다.

- 스프레드 문법 사용
- `immer` 라이브러리 사용
- `Object.freeze()` 내장 함수 사용

위 세 가지 방법을 통해 불변성을 지킬 수 있다.