## modulo

아래는 파이썬 코드의 나머지 연산이다.

```python
print(-5 % 10) # 5
```

하지만 JS는 음수의 나머지 연산이 일반적이지 않다.

```javascript
console.log(-5 % 10); // -5
```

index에 대한 나머지 연산이 필요한 경우, 예를 들면 -1번째 인덱스를 원하면 object.length - 1번째 인덱스를 가리켜야 하는 경우 위와 같은 연산은 문제가 된다.  
[스택오버플로우](https://stackoverflow.com/questions/4467539/javascript-modulo-gives-a-negative-result-for-negative-numbers)의 해설에 따르면 다음과 같은 custom modulo 연산을 만들어야 한다.

```javascript
// prototype chaining. 보통 더 느림
Number.prototype.mod = function (n) {
  return ((this % n) + n) % n;
};

// 아래 방법 추천
function mod(n, m) {
  return ((n % m) + m) % m;
}
```
