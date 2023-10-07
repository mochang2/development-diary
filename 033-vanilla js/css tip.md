## css tip

1. `position: static`이 아닌 요소는 `display: block`이 자동으로 적용됨.
2. `display: block`인 요소는, `width`(또는 `height`) 옵션과 좌우 `margin: auto`(또는 상하 `margin: auto`)를 주면 가운데 정렬 가능.
3. `img`는 inline 요소이며 inline 요소는 `width`, `height`, `margin`, `padding`의 요소를 가질 수 없음. 이러한 inline 요소(글자를 취급하는 요소)는 baseline 때문에 하단에 공간이 생김.

```css
img {
  display: block;
}
```

식으로 해결 가능.

4. 다음과 같은 경우 부모 요소 `width`의 50% 만큼의 `height`를 부모, 자식 요소가 가짐.

```html
<div class="container">
  <div class="item"></div>
</div>
```

```css
.container {
  width: 300px;
}

.item {
  width: 100%;
  padding-top: 50%;
}
```

5. 자연스러운 이미지 회전.

```html
<div class="medal">
  <div class="front">
    <img src="" alt="" />
  </div>
  <div class="back">
    <img src="" alt="" />
  </div>
</div>
```

```css
.medal .back,
.medal .front {
  transition: 1s;
  backface-visibility: hidden;
}
.medal .front {
  transform: rotateY(
    0deg
  ); /* 구형 브라우저에서 동작하지 않을 수 있으므로 명시 */
}
.medal:hover .front {
  transform: rotateY(180deg);
}
.medal .back {
  transform: rotateY(-180deg);
}
.medal:hover .back {
  transform: rotateY(0deg);
}
```
