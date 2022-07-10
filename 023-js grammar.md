# html 상에서 script 위치, option

- **head with no option**

      <head>
        <script type="text/javascript" src="xxx.js"></script>
      </head>

html은 인터프리터처럼 작용해서 위에서부터 읽어내려옴. 그런데 body를 만나기 전에 script태그를 만나서 js를 가져오면 html을 보여주는 속도가 느려짐.

- **body with no option**

      <body>
       <script type="text/javascript" src="xxx.js"></script>
      </body>

html을 먼저 보여줄 수는 있지만 js에 의존적인 사이트라면 좋지 않은 옵션.

- **head with async option**

      <head>
        <script type="text/javascript" src="xxx.js"></script>
        <script async type="text/javascript" src="yyy.js"></script>
      </head>

js를 병렬적으로 다운받음. 먼저 다운로드가 된 js부터 실행(위에서부터 실행하는 것이 아님). 1보다 html을 보여주는 속도는 빠를 수 있지만 동적으로 웹사이트를 만들거나 js간 순서에 의존적이라면 오류가 날 수 있음.

- **head with defer option**

      <head>
        <script type="text/javascript" src="xxx.js"></script>
        <script defer type="text/javascript" src="yyy.js"></script>
      </head>

js를 병렬적으로 다운받지만 다운받자마자 실행하는 것이 아님. html이 전부 로드된 이후에 실행. 제일 효율적이고 안전한 옵션.

---

# use strict

js는 단 1주일만에 만들어진 언어. 그래서 매우 유연하지만 반대로 위험한 언어. 변수 타입을 선언하지 않고

    a=6;

이런 식의 코드만 있어도 오류가 나지 않음. 하지만 js파일 제일 위에

    'use strict';

를 선언하면 a = 6 전에 var a 등의 변수 타입을 선언해주지 않으면 오류가 남. vanila js 등을 배울 때 중요한 요소라고 함.

---

# data type

1. var vs let(read and write)  
   let은 ES6부터 나온 데이터 타입. 과거에는 var를 썼는데, 이는 안전하지 못함. var hoisting(move declaration from bottom to top)이라고 해서 선언이 아래에 있어도 위로 끌어올리면서 일반적인 컴퓨터 언어와 다르게 선언 전에 사용해도 오류가 발생하지 않음. 또한 var는 block scoping이 되지 않아 지역변수, 전역변수의 개념이 없음. 마지막으로 var는 변수 재선언이 가능함. var a = 1;이라고 선언해놓고 뒤에 가서 var a = 'string';으로 선언해도 에러가 나지 않음. 그 외에 var가 지원했던 기능은 let이 모두 대체가 가능하므로 let을 쓰는 게 옳은 방식.

2. const(read only)  
   변경이 불가능한 immutable함. 보안상의 이유로, thread safety를 위해, 휴먼 에러를 줄이기 위해 사용. 참고로 immutable type에는 object.freeze()로 선언한 데이터도 있음. mutable type에 let이 있다면, const는 그 반대. let과 const만으로 아래 나오는 데이터 타입을 전부 사용할 수 있음.

3. variable type</br>
   (primitive type)  
   ![primitive type](https://user-images.githubusercontent.com/63287638/131787747-13e8a747-328a-48b3-9d0c-ca83fced37f6.PNG)

- 숫자: number라는 primitive type이 있는데, let만 써도 동적으로 결정됨. 원래는 +-2^53까지만 표현이 가능했는데, 최근에 bigInt형이 생겼는데

      const bigInt = 1234124235125123321512n // 이렇게 뒤에 n만 붙이면 알아서 인식함
      console.log(`type: ${typeof bigInt}`); // 데이터 타입을 출력해줌

- 문자열: string(char과 구분하지 않음). 파이썬처럼 '+' 기호를 통해서 문자열끼리 연결이 가능.  
  template literals라고 해서 아래와 같은 식으로도 변수를 연결할 수 있음.

      	const helloBob = `hi ${variable}!`;

- boolean: true, false가 있음.
- null: 값이 들어가있지 않다는 것을 명확히 해줌(empty).
- undefined: 변수 선언은 되었지만 값이 할당되지 않음. 값이 빈 것인지 값이 지정되지 않은 건지 명확하지 않음.
- symbol: 주어진 string에 상관없이 고유한 식별값을 가짐. 또는 동시 다발적으로 일어나는 코드에서 우선순위를 주기 위해 사용.

      const symbol1 = Symbol('id');
      const symbol2 = Symbol('id');
      console.log(symbol1 === symbol2);  // false

      const symbol1 = Symbol.for('id');  // string이 같을 때 동일한 심볼을 가지게 하고 싶다면
      const symbol2 = Symbol.for('id');
      console.log(symbol1 === symbol2);  // true
      console.log(symbol1.description);  // 이렇게 출력 안 하면 에러가 남

(object type)  
![obj type](https://user-images.githubusercontent.com/63287638/131787774-a9ebf927-460f-49ae-bcef-032455bbb289.PNG)

- object: c의 struct나 python의 dict와 같이 사용. 다만 pointer 같은 느낌.

      const human = { name: 'cm', age: 20 };
      human.age = 21;  // 에러가 나지 않음

human이 가리키는 객체에 대해서는 변경이 불가능하지만, human.name이나 human.age는 변경이 가능함.

#### shallow copy vs deep copy

shallow copy는 객체의 주소값만 참조하는 것을 말하고, deep copy는 완전히 새로운 객체를 생성하는 것을 말한다.  
어느 포스팅된 글의 표현을 빌리자면 얕은 복사는 한 단계까지만 복사하고, 깊은 복사는 객체에 중첩된 객체까지 모두 복사한다.  
당연히 속도는 deep copy가 훨씬 느리다.  
파이썬을 예시로 들자면 아래와 같다.

```javascript
a = [1,2,3]
b = a # shallow copy
b[0] = 100

print(a) # [100,2,3]

-------
from copy import deepcopy

a = [1,2,3]
b = deepcopy(a) # deep copy
b[0] = 100

print(a) # [1,2,3]
```

js에서 원시값(primitive)은 변경 불가능한 불변의 값을 말한다.  
원시값에는 String, Number, undefined, Boolean, Symbol, BigInt 6종류가 존재하고 이것들은 기본적으로 할당만 해도 deep copy가 된다.

```javascript
let a = 1;
let b = a;
b = 2;
console.log(a); // 1
```

하지만 object, array 등의 객체는 기본적으로 shallow copy를 한다.  
이것들을 deepcopy하기 위한 방법은 크게 3가지가 있다.  
[1](https://roseline.oopy.io/dev/javascript-back-to-the-basic/shallow-copy-deep-copy)과 [2](https://leonkong.cc/posts/js-deep-copy.html)를 참고했다.

```javascript
// 1. JSON 사용. 속도가 가장 느림.
const obj1 = {...}
const obj2 = JSON.parse(JSON.stringify(obj1));


// 2. 커스텀 재귀 함수 사용
function cloneObject(obj) {
  let result = {};
  for (let key in obj) {
    typeof obj[key] == "object" && obj[key] != null
      ? clone[key] = cloneObject(obj[key])
      :clone[key] = obj[key]
  }

  return result;
}


// 3. lodash의 cloneDeep() 사용
import cloneDeep from lodash/cloneDeep;

const originalObj = {...};
const copiedObj = originalObj;
```

4.  Dynamic type(c, java: starting typed language)

        let text = 'hello';
        console.log(text.charAt(0));  // h
        console.log('value: ${text}, type: ${typeof text}');  // hello, string
        text = 1;
        console.log('value: ${text}, type: ${typeof text}');  // 1, number
        text = '7' + 5;
        console.log('value: ${text}, type: ${typeof text}');  // 75, string
        text = '8' / '2';
        console.log('value: ${text}, type: ${typeof text}');  // 4, number
        console.log(text.charAt(0));  // error

이처럼 js는 run time에 데이터 타입이 결정됨.

---

# string literals

작은 따옴표(')가 아닌 `를 사용하여 console.log를 감싸면 작은 따옴표를 그대로 출력할 뿐만 아니라, $를 사용하여 변수를 부를 수 있음.

    console.log(`string literals:
    '''
    1 + 2 = ${1 + 2}`);
    // string literals:
    // '''
    // 1 + 2 = 3

---

# equlity

    const stringFive = '5';
    const numberFive = 5;
    // 1. loose equality, with type conversion
    console.log(stringFive == numberFive); // true

    // 2. strict equality, without type conversion
    console.log(stringFive === numberFive); // false

    // 3. object equality by reference
    const obj1 = { name: "aa" };
    const obj2 = { name: "aa" };
    const obj3 = obj1
    console.log(obj1 == obj2);  // false
    console.log(obj1 === obj2); // false
    console.log(obj1 === obj3); // true

    // example
    console.log(0 == false);   // true
    console.log(0 === false);  // false
    console.log('' == false);  // true
    console.log('' === false); // false
    console.log(null == undefined);  // true
    console.log(null === undefined); // false

---

# function(function is a kind of an object in js)

#### 'fucntion.' 하면 여러 속성값이 나온다는 뜻.

1.  rest parameters. 배열 형태로 인자를 전달.

        function printAll(...args){
        	for (let i = 0; i < args.length; i++){
        		console.log(args[i]);
        	}

        	for (const arg of args) {
        		console.log(arg);  // 위와 결과가 똑같음
        	}

        	args.forEach((arg) => console.log(arg));  // 동일한 결과
        }

        printAll('dream', 'coding', 'ellie');  // 세 개 모두 출력.

_cf forEach 내부에서 await 사용시 조심할 점_
forEach는 async/await으로 선언된 비동기 처리 구문을 기다려주지 않는다.

```javascript
const wantedGenres = ["로맨스", "판타지"];
let genres = [];
wantedGenres.forEach(async (value, index) => {
  const g = await Genre.findOne({ name: value }); // MongoDB에서 오브젝트 조회
  genres.push(g);
  console.log(`Push ${index}: ${genres}`);
});
console.log(`Done: ` + genres);

// 예상 결과
// Push 0: [{...name: '로맨스'...}]
// Push 1: [{...name: '로맨스'...}, {...name: '판타지'...}]
// Done: [{...name: '로맨스'...}, {...name: '판타지'...}]

// 실행 결과
// Done:
// Push 0: [{...name: '로맨스'...}]
// Push 1: [{...name: '로맨스'...}, {...name: '판타지'...}]

// 해결방법 1. for (const/let variable of, in iterableObject) 사용

// 해결방법 2. Promise.all() 사용
let genres = [];
const promises = wantedGenres.map(async (value, index) => {
  const g = await Genre.findOne({ name: value });
  genres.push(g);
});
await Promise.all(promises);
```

2.  return  
    return 을 명시하지 않으면 모든 함수 끝에는 return undefined가 있는 것과 마찬가지. python과 마찬가지로 함수 앞에 어떤 type을 return할 것인지 함수 선언에서 명시하지 않음.

3.  first class function  
    _js에서 함수는 자동으로 hoisting됨. 즉 c에서 함수 선언만 main보다 위에서 한 것과 같은 것처럼, 함수 body가 함수 호출보다 아래 있어도 해당 함수를 사용 가능하다는 말임_  
    함수는 또 다른 변수로 취급됨. 변수에 값을 할당할 수 있으며, 함수의 인자로 전달될 수도 있고, 또 다른 함수의 return으로 사용될 수도 있음.

        	const print = function () { // anonymous function(function 이름이 필요 없음)
        		console.log('print');
        	};
        	print();   // print를 출력
        	const printAgain = print;
        	printAgain();  // print를 출력

4.  Arrow function  
    항상 anonymous function임.

        	const simplePrint = () => console.log('simplePrint!');
        	const simplePrint2 = function() {
        		console.log('simplePrint2!');
        	}

        	const add = (a, b) => a + b;  // 다만 줄이 길어져서 {}로 감싸게 되면 arrow function도 return을 명시해줘야 함
        	const add2 = function(a, b) {
        		return a + b;
        	}

5.  IIFE: Immediately Invoked Function Expression

        (function hello() {
        	console.log('IIFE');
        })(); // 선언함과 동시에 호출

---

# class

1.  getter and setter

        class User {
            constructor(firstName, lastName, age){
        	this.firstName = firstName;
        	this.lastName = lastName;
        	this.age = age;
            }

            get age(){
        	return this._age;
            }

            set age(value){
            	// if (value < 0){
        	// 	throw Error('negative');
        	// }
        	this._age = value < 0 ? 0 : value;
            }
        }

        const user1 = new User("Steve", "Jobs", -1 );
        console.log(user1.age);

누누히 얘기했듯이 js는 interpreter처럼 작동. constructor에서 this.age = age; 라는 부분이 쓰이면 object가 생성되길 기다리는 것이 아니라, 'this.age'는 get을 '= age;'는 set을 부르게 됨.  
만약 getter와 setter에서 this.\_age가 아닌 this.age를 썼다면 다시 this.age = value;는 getter와 setter를 호출하게 되어 결과적으로 callstack이 다 찼다는 에러 메시지가 나옴.

2.  public, private

        class Experiment {
        	publicField = 2;
        	#privateField = 0;
        }
        const experiment = new Experiment();
        console.log(experiment.publicField);  // 2
        console.log(experiment.privateField); // undefined

3.  static(c에서와 같음)

        class Article {
        	static publisher = "aa";
        	constructor(aricleNumber) {
        		this.articleNumber = articleNumber;
        	}

        	static printPublisher() {
        		console.log(Article.publisher);
        	}
        }
        const article1 = new Article(1);
        const article2 = new Article(2);

        console.log(article1.publisher); // undefined
        console.log(Article.publisher);  // aa
        Article.printPublisher();        // aa

4.  상속: extends를 활용
5.  부모 것 call: super 활용
6.  인스턴스인지 확인: 객체 instanceOf 클래스이름 으로 활용

---

# object

1.  computed property를 사용하는 경우

        const user1 = { name : 'cm', age: 25 };

        console.log(user1.name);    // cm
        console.log(user1['name']); // cm

        function printValue1(obj, key){
        	console.log(obj.key);
        }

        function printValue2(obj, key){
        	console.log(obj[key]);
        }

        printValue1(user1, 'name');   // undefined -> user1에는 key라는 key property가 없으므로
        printValue2(user1, 'name');   // cm

2.  property value shorthand

        const person1 = { name: 'bob', age: 2 };
        const person2 = { name: 'steve', age: 3 };
        // 반복하기 귀찮으므로 함수 생성

        function makePerson(name, age){
        	return {
        		name,   // name = name; 해줄 필요가 없음
        		age,
        	}
        }
        const person3 = makePerson('cm', 25);

3.  constructor function

        const person1 = { name: 'bob', age: 2 };
        const person2 = { name: 'steve', age: 3 };
        // 반복하기 귀찮으므로 함수 생성

        function Person(name, age){
        	// this = {}; 생략
        	this.name = name;
        	this.age = age;
        	// return this; 생략
        }
        const person3 = new Person('cm', 25);

4.  in  
    key가 해당 object에 있는지 확인. python에서 쓰는 것과 같음.

5.  for..in vs for..of

        console.clear(); // console 깨끗이 해줌
        const obj = { name: 'blabla', age: 2222 };
        for (key in obj) {  // key in obj
        	console.log(key);
        }

        const array = [1, 2, 4, 5];
        for (value of array) { // value of iterable
        	console.log(value);
        }

6.  assign과 overwrite

        const fruit1 = { color: 'red' };
        const fruit2 = { color: 'blue', size: 'big' };
        const mixed = Object.assign({}, fruit1, fruit2);
        console.log(mixed.color); // blue
        console.log(mixed.size);  // big

---

# array

1.  배열 전체 출력 3가지 방법

        for (let i = 0; i < fruits.length; i++) {
        	console.log(fruits[i]);
        }
        for (let fruit of fruits) {
        	console.log(fruit);
        }
        fruits.forEach((fruit) => console.log(fruit));

        // for Each API
        // forEach(callbackfn: value: T, index: number, array: T[]) => void, thisArg?: any : void;
        // ?는 있어도, 없어도 된다는 의미, array는 보통 인자로 넣어주지 않음

2.  add, delete, copy

        fruits.push('apple', 'orange');  // 배열 뒤에 추가

        fruits.pop(); // 배열 뒤에서 삭제
        fruits.pop();

        // slower than the above APIs
        fruits.unshift('apple', 'orange'); // 배열 앞에 추가

        fruits.shift(); // 배열 앞에서 삭제
        fruits.shift();

        // rm an item by index position
        // splice는 배열 자체를 수정, slice는 배열에서 원하는 부분만 가져옴
        fruits.push('apple, 'orange');
        fruits.splice(2); // 시작 인덱스, 몇 개 지울지(optional) => 앞에 두 개의 인자 빼고 전부 지움
        fruits.splice(1, 1, 'watermelon', 'strawberry'); // 지운 뒤에 수박과 딸기를 추가

        // combine two arrays
        fruits2 = ['mango', 'banana'];
        const newFruits = fruits.concat(fruits2);

3.  search

        console.log(fruits.indexOf('apple'));  // return the first matched fruit's index
        console.log(fruits.includes('apple')); // return boolean
        console.log(fruits.lastIndexOf('apple')); // return the last matched fruit's index

4.  find

        const result = fruits.find((fruit, index) => {
        	return fruit.name === 'apple';  // find는 true를 return 하게 되면 즉시 찾는 것을 멈춤
        };
        console.log(result);   // apple을 포함한 fruit object가 출력

5.  filter

        const result = students.filter((student) => student.enrolled);
        console.log(result);  // enrolled === true인 애들만 출력

6.  map

        const result = students.map((student) => student.score);
        console.log(result);  // students의 점수 하나의 배열로 만들어져서 출력

7.  some, every

        const result = students.some((student) => student.score < 50);
        console.log(result);  // true / false 학생들 중에 점수가 50점 미만인 애가 있으면 true 아니면 false

        const result2 = students.every((student) => student.score < 50);
        console.log(result2)  // true / false 모든 학생들 점수가 50점 미만이면 true 아니면 false

8.  reduce

        const result = students.reduce((prev, curr) => {
        	console.log('----');
        	console.log(prev);
        	console.log(curr);
        	return curr;   // reduce는 curr 값이 prev로 다시 들어가서 호출됨
        });

        const result2 = students.reduce((prev, curr) => {
        	console.log('----');
        	console.log(prev);
        	console.log(curr);
        	return prev + curr.score;
        }, 0);  // prev 초기값 설정
        console.log(result);  // 학생들 점수의 합이 출력됨

        // reduceRight은 배열의 뒤에서부터 시작함

---

# json

1. simplest data interchange format
2. lightweight text-based structure
3. easy to read
4. key-value pairs
5. used for serialization and transmission of data between the network
6. **independent programming language and platform**

object to string: serialization  
string to object: deserializaion

    	// serialization
    	let json = JSON.stringify(true);
    	console.log(json);

    	json = JSON.stringify(['apple', 'banana']);
    	console.log(json);

    	const rabbit = {
    		name: 'tori',
    		color: 'white',
    		size: null,
    		birthDate: new Date(),
    		symbol: Symbol('id'),  // js에만 있는 데이터 타입으로, json 형태로 변형되지 않음
    		jump: () => {  // object에 있는 데이터가 아니기 때문에 json 형태로 변형되지 않음
    			console.log(`${name} can jump!`);
    		},
    	};
    	json = JSON.stringify(rabbit);
    	console.log(json);

    	json = JSON.stringify(rabbit, ['name', color']);
    	console.log(json);  // rabbit object에서 name과 color만 json 형식으로 변형됨

    	json = JSON.stringify(rabbit, (key, value) => {
    		console.log(`key: ${key}, value: ${value}`);
    		return value;
    		// return key === 'name' ? 'ellie' : value;
    	});
    	console.log(json);

    	// deserialization
    	const obj = JSON.parse(json);
    	console.log(obj);

    	rabbit.jump();
    	obj.jump();  // error
    	console.log(rabbit.birthDate.getDate());
    	console.log(obj.birthDate); // type: string

    	obj = JSON.parse(json, (key, value) => {
    		console.log(`key: ${key}, value: ${value}`);
    		return key === 'birthDate' ? new Date(value) : value;
    	});
    	console.log(obj.birthDate.getDate());  //  not error

---

# callback

js는 synchronous한 언어이다. 코드는 hoisting이 된 이후에 작성한 순서대로 실행이 된다는 뜻이다.  
asynchronous(sleep과 같은 것처럼)라면 순서대로 작동하지 않아서 어떤 순서대로 실행될지 감이 안 잡힌다.

    	console.log(1);
    	setTimeout(() => {
    		console.log(2);
    	}, 1000);  // 1000ms
    	console.log(3);
    	// 1 3 2 순으로 출력

콜백함수(callback function): 파라미터로 함수를 전달하는 함수.  
콜백도 synchronous와 asynchronous로 나뉨.

    	function printImmediately(print) {
    		print();
    	}
    	printImmediately(() => console.log('hello'));

    	function printWithDelay(print, timeout){
    		setTimeout(print, timeout);
    	}
    	printWithDelay(() => console.log('hello2'), 2000);

#### callback 지옥 체험

    	class UserStorage{
    	    loginUser(id, password, onSuccess, onError){
    		setTimeout(() => {
    		    if((id === 'ellie' && password === 'dream') ||
    			(id === 'coder' && password === 'academy'))
    			onSuccess(id);
    		    else
    			onError(new Error('not found'));
    		}, 1000);
    	    }

    	    getRoles(user, onSuccess, onError){
    		setTimeout(()=>{
    		    if (user === 'ellie')
    			onSuccess({name: 'ellie', role: 'admin'});
    		    else
    			onError(new Error('no access'));
    		}, 500);
    	    }
    	}

    	const userstorage = new UserStorage();
    	const id = prompt('enter your id');
    	const password = prompt('neter your password');
    	userstorage.loginUser(
    	    id,
    	    password,
    	    (user) => {
    		userstorage.getRoles(
    		    user,
    		    (userWithRole) => {
    			alert(`Hello ${userWithRole.name}, you have a ${userWithRole.role} role`);
    		    },
    		    (error) => {
    			console.log(error);
    		});
    	    },
    	    (error) => {
    		console.log(error);
    	    }
    	);

---

# promise

비동기를 간편하게 처리할 수 있게 도와주는 오브젝트. 정해진 장시간의 기능을 수행하고 나서 정상적으로 수행했다면 성공 메시지와 함께 결과값을 전달. 만약 예상치 못한 문제가 발생했다면 에러 전달.  
promise의 state. pending -> fulfilled or rejected.  
promise의 object. Producer와 Consumer가 있음.

    	// Producer
    	// Promise object가 생성되자마자, executor는 자동으로 동작하기 때문에 이를 항상 염두해야 함
    	const promise = new Promise((resolve, reject) => {
    		// executor라는 콜백 함수는 resolve와 reject 콜백을 전달받음
    		// doing some heavy work(network, read files)
    		console.log('doing something...');
    		setTimeout(() => {
    			resolve('ellie');
    			// reject(new Error('no network'));
    		}, 2000);
    	});


    	// Consumer: then, catch, finally
    	promise
    	.then((value) => {  // promise가 정상적으로 수행된다면 executor 값을 받아올거야
    		console.log(value);
    		})
    	.catch((error) => { // Error 값을 받아옴
    		console.log(error);
    		});
    	.finally(() => {    // 무조건 실행
    		console.log('finally');
    		});


    	// Promise chaining
    	const fetchNumber = new Promise((resolve, reject) => {
    		setTimeout(() => resolve(1), 1000);
    	});

    	fetchNumber
    	.then((num) => num * 2)
    	.then((num) => num * 3)
    	.then((num) => {
    		return new Promise((resolve, reject) => {
    			setTimeout(() => resolve(num - 1), 1000);
    		});
    	})
    	.then((num) => console.log(num));


    	// Error Handling
    	const getHen = () =>
    		new Promise((resolve, reject) => {
    			setTimeout(() => resolve('hen'), 1000);
    		});
    	const getEgg = (hen) =>
    		new Promise((resolve, reject) => {
    			// setTimeout(() => resolve(`${hen} => egg`), 1000);
    			setTimeout(() => reject(new Error(`error! ${hen} => egg`)), 1000);
    		});
    	const cook = (egg) =>
    		new Promise((resolve, reject) => {
    			setTimeout(() => resolve(`${egg} => fried egg`), 1000);
    		});

    	getHen()
    	.then((hen) => getEgg(hen))  // 1가지만 인자로 받을 경우 .then(getEgg) 만 써도 됨
    	.then((egg) => cook(egg))
    	.then((meal) => console.log(meal));
    	.catch(console.log);	     // 마찬가지로 console.log만 써도 됨
    	// catch 위치에 따라 에러가 떠도 최종 결과를 만들어낼 수 있음. catch 위치 조정에 신경

---

# async, await

    	// 1. async
    	async function fetchUser() {
    		return 'ellie'
    	}

    	function fetchUser2() {
    		return new Promise((resolve, reject) => {
    			resolve('ellie');
    		}
    	}
    	// 위 두 함수는 같은 결과를 가짐

    	const user = fetchUser();
    	user.then(console.log);
    	console.log(user);


    	// 2. await
    	// async가 붙은 함수 안에서만 쓸 수 있는 키워드

    	function delay(ms) {
    	    return new Promise((resolve) => setTimeout(resolve, ms));
    	}

    	async function getApple() {
    	    await delay(1000);   // delay가 끝날 때까지 return을 하지 않음
    	    return '🍎';
    	}

    	async function getBanana() {
    	    await delay(1000);
    	    return '🍌';
    	}

    	function getBanana2() {
    	    return delay(1000)
    	    .then(() => '🍌');
    	}

    	function pickFruits(){
    	    return getApple().then((apple) => {
    		return getBanana().then((banana) => `${apple} + ${banana}`);
    	    });
    	}
    	pickFruits().then(console.log);

    	async function pickFruits2() {
    	    const apple = await getApple();
    	    const banana = await getBanana();
    	    return `${apple} + ${banana}`;
    	}
    	pickFruits2().then(console.log);


    	// 3. async를 병렬적으로 처리
    	async function pickFruits3() {
    	    const applePromise = getApple();  // promise obj는 변수를 만드는 순간 바로 실행됨
    	    const bananaPromise = getBanana();
    	    const apple = await applePromise;
    	    const banana = await bananaPromise;
    	    return `${apple} + ${banana}`;
    	}

---
