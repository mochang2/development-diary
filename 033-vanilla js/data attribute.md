## data-\* 속성

HTML5부터 추가된 개념으로 '사용자 지정 데이터 특성'이라는 특성 클래스를 형성할 수 있다.  
해당 특성을 이용해 page나 application에 private한 custom data를 저장하거나, HTML element에 custom data attirbute를 embed할 수 있다.  
이를 통해 BE에 Ajax 호출 없이 데이터를 보관할 수 있는 방법으로 사용할 수도 있다(다만 크롤링되어야 하거나 secret한 내용을 담기에는 적합하지 않은 속성이다).  
하나의 element에 여러 개의 data-\* 속성이 포함될 수 있다.

다음과 같은 두 가지 규칙을 지켜야 된다.

1. attribute 이름에 문자, 숫자, 대시(-), 점(.), 콜론(:), 언더스코어(\_)는 사용 가능하지만 대문자를 포함할 수 없다.
2. attribute 이름에 접두사 "data-" 뒤에 하나 이상의 문자가 있어야 한다.
3. attribute 이름이 대소문자 관계없이 'xml'로 시작하면 안 된다.
4. attribute 값은 어떠한 string도 가능하다.

```html
<ul>
  <li data-animal-type="bird">Owl</li>
  <li data-animal-type="fish">Salmon</li>
  <li data-animal-type="spider">Tarantula</li>
</ul>
```

html에서는 위와 같은 형식이지만 dataset은 JS이기 때문에 attribute명은 camelCase로 변환된다.  
즉, html과 dataset 간 `data-create-date -> (dateset) createDate`, `(dataset) dataset.monthSalary = '500 -> data-month-salary="500"`로 변환되는 것이다.

### JS에서의 접근

JS에서 dataset을 get/set 하는 방법은 두 가지가 있다.  
`getAttribute`/`setAttribute`를 이용하는 방법과 DOM 객체의 property를 이용하는 방법이다.

```javascript
const div = document.getElementsByTagName('div')[0];
// const div = document.querySelector('div[data-auto="true"]')

div.setAttribute('data-auto', true);
console.log(div.dataset.auto); // typeof div.dataset.auto === 'string'

div.dataset.size = 'big';
console.log(div.getAttribute('data-size'));
```

### CSS에서의 접근

위 예시에서 `querySelector`의 예시에서 나왔듯이 data-\* 속성은 html의 속성이기 때문에 css에서도 속성 선택자를 통해 선별할 수 있다.

```css
/* https://developer.mozilla.org/ko/docs/Learn/HTML/Howto/Use_data_attributes 예시 */

article[data-columns='3'] {
  width: 400px;
}
article[data-columns='4'] {
  width: 600px;
}
```
