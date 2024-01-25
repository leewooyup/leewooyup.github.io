---
title: "[HTML/CSS] @keyframes룰을 정의해서 타이핑(typing)하는 듯한 효과 구현"
date: 2024-01-25 16:40 +0900
lastmod: 2024-01-25 16:40 +0900
categories: Frontend
tags:
---

## 타이핑 효과
<div style="margin-bottom: 15px;font-size:15px;background-color:#FFD24D;color:black;font-weight:bolder;border-top-left-radius:5px;border-top-right-radius:5px;padding:7px;">
    🪱 @keyframes: 애니메이션 효과를 만들기 위해, 특정 시점에 대한 스타일 정의
</div>

`from속성` or `0%속성`에서 설정한 스타일로 시작해  
`to속성` or `100%속성`까지 설정한 스타일로 바껴가며 <span style='background-color:#ffdce0;font-weight:bold'>애니메이션</span>이 재생

`📑 타이핑텍스트.html `
```html
<div class="fit-wrapper">
  <div class="typing-text">Lorem ipsum dolor sit amet, consectetur adipisicing elit.</div>
</div>
```

`🎉 타이핑효과.css`
```css
.fit-wrapper {
  width: fit-content;
}

.typing-text {
  white-space: nowrap; /* 텍스트가 요소의 너비를 넘어갈 경우에도 한 줄로 유지 */
  overflow: hidden; /* 요소의 내용이 요소의 영역을 벗어날 때, 벗어난 내용 숨김 */
  border-right: 10px solid black; /* 마우스 커서 역할 */
  animation: type-text forwards 3s, cursor-blink 1s infinite;
  /* @keyframes type-text, @keyframes cursor-blink가 정의되어 있어야 한다 (특정 시점 스타일에 대한 규칙 정의) */
  /* forwards 애니매이션이 완료된 후 최종 상태를 유지 */
  animation-timing-function: steps(40);
  /* 애니메이션을 40단계로 나누고, 각 프레임 간 전환이 갑자기 일어남, 끊기는 듯이 진행되므로, 타이핑 치는 역할 */
}

@keyframes type-text { /* 마우스 커서가 움직이는 역할 */
  0% {
    width: 0%
  }
  100% {
    width: 100%
  }
}

@keyframes cursor-blink { /* 마우스 커서가 깜빡이는 역할 */
  0% {
    border-color: transparent;
  }
  100% {
    border-color: black;
  }
}
```

`💻 화면`  
![Alt text](https://i.esdrop.com/d/f/OAHra5CzfD/GMY41oOyhW.gif "Optional title")
