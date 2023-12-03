## 0. 공부하게 된 계기

[재밌는 글](https://velog.io/@superlipbalm/base64-encoding)을 하나 읽고 나서 인코딩/디코딩과 관련하여 간단하게나마 정리하고 싶어졌다.

참고: Chat-GPT

## 1. 정의

- 인코딩: 사용자가 입력한 문자나 기호들을 컴퓨터가 이용할 수 있는 신호로 만드는 것
- 디코딩: 컴퓨터가 이용할 수 있는 신호를 사용자가 이해할 수 있는 문자나 기호로 만드는 것

## 2. 방식

### ASCII(American Standard Code for Information Interchange)

7비트로 표현되는 문자 인코딩 방식이다.  
숫자, 알파벳, 특수문자 등 총 128개의 문자를 표현하고, 각각은 0부터 127까지의 정수 값으로 매핑된다.

```text
A: 01000001
B: 01100001
O: 00110000
```

### UNICODE

전 세계의 모든 문자를 일관되게 표현하기 위한 표준이다.  
대표적으로 UTF-8, UTF-16, UTF-32 등이 있다.

#### UTF-8(Unicode Transformation Format - 8 bit)

가변 길이 문자 인코딩 방식으로 ASCII와 호환된다.  
1 byte ~ 4 byte까지 다양한 길이를 가진다.  
ASCII 문자는 1 byte, 다른 문자는 2 ~ 4 byte이다.

```text
A: 01000001
ä: 11000011 10100100
😊: 11110000 10011111 10010010 10011000
```

```js
// UTF-8 Encoding
const utf8String = '😊';
const utf8Encoded = new TextEncoder().encode(utf8String);
console.log(utf8Encoded); // Uint8Array [ 240, 159, 152, 138 ]

// UTF-8 Decoding
const utf8Decoded = new TextDecoder().decode(utf8Encoded);
console.log(utf8Decoded); // 😊
```

#### UTF-16(Unicode Transformation Format - 16 bit)

기본 다국어 평면(BMP, Basic Multilingual Plane)의 문자는 2 byte로 표현되지만 보충 평면(Supplementary Plane)는 4 byte로 표현된다.

UTF-16을 보다 자세히 이해하기 위해 아래 개념들이 필요하다.

- 코드 포인트(code point)
  - 각 문자에 할당된 고유한 정수값을 나타낸다. 간단히 말해, 코드 포인트는 각 문자에 대한 식별자이다. 예를 들어, 영문 알파벳 "A"는 코드 포인트로 U+0041이라고 표현된다. 여기서 "U+"는 unicode 코드 포인트를 나타내는 접두사이며, 이어지는 네 자리 16진수는 해당 문자의 코드 포인트를 나타낸다. 코드 포인트는 모든 unicode 문자에 대해 고유하게 할당되며, 이것이 unicode 표준을 통해 전 세계의 모든 문자를 일관되게 표현하는 데 사용됩니다. 코드 포인트는 0에서 0x10FFFF까지의 범위에 할당된다.
- 기본 다국어 평면(basic multilingual plane)
  - 유니코드에서 0x0000부터 0xFFFF까지의 코드 포인트 범위를 나타낸다. 즉, U+0000에서 U+FFFF까지의 문자들이 여기에 속한다. 대다수의 일반적인 문자들, 예를 들면, 영문 알파벳, 숫자, 일부 특수 문자 등이 여기에 속한다. 이들은 16비트(2바이트)로 표현된다.
- 보충 평면(supplementary plane)
  - 기본 다국어 평면 이외의 고유 문자들을 포함하는 추가적인 유니코드 평면(planes)을 나타낸다. 0x010000부터 0x10FFFF까지의 코드 포인트가 여기에 속하며, 0x010000부터 0xD7FF, 그리고 0xE000부터 0x10FFFF까지의 범위는 서로게이트 페어(surrogate pair)로 표현된다.
  - 주로 특수한 기호, 이모지, 특정 언어의 글자 등을 포함하고 있어, 전 세계의 모든 문자를 표현하기 위한 확장이 필요한 경우에 사용된다.
- 서로게이트 페어(surrogate pair)
  - UTF-16에서 보충 평면의 문자를 표현하기 위한 방법이다. UTF-16은 기본적으로 16 bit이지만, 보충 평면의 문자를 표현하기 위해 32비트가 필요하다. 이를 위해 두 개의 16 bit 유닛을 조합하여 하나의 보충 평면 문자를 나타내는데, 첫 번째 16 bit는 고위 서로게이트(high surrogate), 두 번째 16 bit는 저위 서로게이트(low surrogate)로 사용된다.
  - 예를 들어, '😊' (U+1F60A)를 UTF-16으로 표현하면 다음과 같습니다: 11011011 10001111 (high surrogate) , 10001001 10000000 (low surrogate)
- 론 서로게이트(lone surrogate)
  - 서로게이트 페어에서 하나만 남은 경우를 가리킨다. 고위 서로게이트가 있고, 저위 서로게이트가 없거나, 그 반대의 경우를 의미한다. 이러한 상황은 올바른 UTF-16 표현이 아니며, 유효한 문자를 나타내지 않는다.
  - 파일 전송이나 텍스트 처리 중에 문제가 발생할 수 있다. unicode 표준은 이러한 상황을 처리하는 규칙을 제공하고 있다.

```text
A: 00000000 01000001
ä: 11000011 10100100
😊: 11011011 10001111 10001001 10000000
```

```js
// UTF-16 Encoding
const utf16String = '😊';
const utf16Encoded = new TextEncoder('utf-16le').encode(utf16String);
console.log(utf16Encoded); // Uint8Array [ 68, 216, 10, 216, 0 ]

// UTF-16 Decoding
const utf16Decoded = new TextDecoder('utf-16le').decode(utf16Encoded);
console.log(utf16Decoded); // 😊
```

#### UTF-32(Unicode Transformation Format - 32 bit)

모든 문자를 고정된 4 byte로 표현하는 방식이다.  
메모리 사용이 많지만 간단하게 인덱싱이 가능하다.

```text
A: 00000000 00000000 00000000 01000001
ä: 00000000 00000000 11000011 10100100
😊: 00000000 00110011 10001111 10001001
```

## 3. JS에서 인코딩, 디코딩

base64 인코딩/디코딩은 바이너리 콘텐츠를 웹에 안전한 텍스트로 표현하기 위해 변환하는 일반적인 형식이다.  
인라인 이미지와 같은 데이터 URL에 일반적으로 사용된다.

### ASCII 문자 인코딩

JS에서 base64 인코딩/디코딩을 위한 핵심 함수는 `btoa`와 `atob`이다. `btoa`는 문자열을 base64로 인코딩된 문자열로 변환하고, `atob`는 반대로 base64로 인코딩 된 문자열을 일반 문자열로 디코딩한다.  
다만 이 기능은 문자열 내의 각 문자가 실제로 하나의 바이트에 매핑된다는 가정으로 동작하기 때문에 ASCII 문자 또는 단일 바이트로 표현할 수 있는 문자가 포함된 문자열에서만 작동한다.  
즉, 유니코드에서는 작동하지 않는다.

실제로 JS에서 'hello⛳❤️🧀' 문자열에 대한 `btoa`를 통한 인코딩은 에러를 발생시킨다.

```js
// '⛳': 단일 16비트 코드 단위.
// '❤️': U+2764와 U+FE0F(하트 및 변형)의 두 가지 16비트 코드 단위.
// '🧀' : 32비트 코드 포인트(U+1F9C0), 또는 두 개의 16비트 코드 단위 '\ud83e\uddc0'의 서로게이트(surrogate) 쌍으로 표현할 수 있음.
const validUTF16String = 'hello⛳❤️🧀';

try {
  const validUTF16StringEncoded = btoa(validUTF16String);
  console.log(`Encoded string: [${validUTF16StringEncoded}]`); // 동작 X
} catch (error) {
  console.log(error); // DOMException: Failed to execute 'btoa' on 'Window': The string to be encoded contains characters outside of the Latin1 range.
}
```

이는 JS가 문자열을 UTF-16 인코딩으로 처리하기 때문에 발생한다.

### 유니코드 문자 인코딩

[MDN](https://developer.mozilla.org/ko/docs/Glossary/Base64)에서 나온 것처럼 유니코드를 통해 이 인코딩 문제를 해결할 수 있다.

```js
function base64ToBytes(base64) {
  const binString = atob(base64);
  return Uint8Array.from(binString, (m) => m.codePointAt(0));
}

function bytesToBase64(bytes) {
  const binString = String.fromCodePoint(...bytes);
  return btoa(binString);
}

const validUTF16String = 'hello⛳❤️🧀';

const validUTF16StringEncoded = bytesToBase64(
  new TextEncoder().encode(validUTF16String)
);
console.log(`Encoded string: [${validUTF16StringEncoded}]`); // Encoded string: [aGVsbG/im7PinaTvuI/wn6eA]

const validUTF16StringDecoded = new TextDecoder().decode(
  base64ToBytes(validUTF16StringEncoded)
);
console.log(`Decoded string: [${validUTF16StringDecoded}]`); // Decoded string: [hello⛳❤️🧀]
```

다만 위 코드는 올바른 유니코드에 대해서만 올바르게 동작한다.  
론 서로게이트에 대해서는 아래와 같은 일이 발생한다.

```js
// ...
// 함수 선언은 동일

const partiallyInvalidUTF16String = 'hello⛳❤️🧀\uDE75';

const partiallyInvalidUTF16StringEncoded = bytesToBase64(
  new TextEncoder().encode(partiallyInvalidUTF16String)
);
console.log(`Encoded string: [${partiallyInvalidUTF16StringEncoded}]`); // Encoded string: [aGVsbG/im7PinaTvuI/wn6eA77+9]

const partiallyInvalidUTF16StringDecoded = new TextDecoder().decode(
  base64ToBytes(partiallyInvalidUTF16StringEncoded)
);
console.log(`Decoded string: [${partiallyInvalidUTF16StringDecoded}]`); // Decoded string: [hello⛳❤️🧀�]
```

### 론 서로게이트 해결하는 방법

최신 브라우저에는 올바른 형식의 문자열인지 확인하기 위한 함수인 [isWellFormed](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/isWellFormed)가 존재한다.  
이를 통해 론 서로게이트로 인해 인코딩/디코딩 과정에서 문자열이 깨지는 것을 방지할 수 있다.

```js
// ...
// 함수 선언은 동일

function isWellFormed(str) {
  if (typeof str.isWellFormed !== 'undefined') {
    return str.isWellFormed();
  } else {
    // 구형 브라우저를 위함
    try {
      encodeURIComponent(str);
      return true;
    } catch (error) {
      return false;
    }
  }
}

const validUTF16String = 'hello⛳❤️🧀';
const partiallyInvalidUTF16String = 'hello⛳❤️🧀\uDE75';

if (isWellFormed(validUTF16String)) {
  const validUTF16StringEncoded = bytesToBase64(
    new TextEncoder().encode(validUTF16String)
  );

  console.log(`Encoded string: [${validUTF16StringEncoded}]`); // Encoded string: [aGVsbG/im7PinaTvuI/wn6eA]

  const validUTF16StringDecoded = new TextDecoder().decode(
    base64ToBytes(validUTF16StringEncoded)
  );
  console.log(`Decoded string: [${validUTF16StringDecoded}]`); // Decoded string: [hello⛳❤️🧀]
} else {
  // ... 처리
}

if (isWellFormed(partiallyInvalidUTF16String)) {
  // ... 처리
} else {
  console.log(
    `Cannot process a string with lone surrogates: [${partiallyInvalidUTF16String}]`
  ); // Cannot process a string with lone surrogates: [hello⛳❤️🧀\uDE75]
}
```
