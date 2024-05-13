---
title: React CSS Module로 사용하기
date: 2024-05-13 23:40:00 +09:00
categories: [TIL, React]
tags:
  [
    TIL,
    javascript,
    React,
    CSS,
    CSS-Module
  ]
image: ../assets/img/common/react-logo.png
---


# CSS를 Module로 사용하기

[wikidocs : DRF + React, Next 프로젝트](https://wikidocs.net/197754)

- CSS 모듈은 React 에서 컴포넌트의 지역적 범위 스타일을 지정하는 방법
- 특정 컴포넌트에 대한 로컬한 CSS를 작성할 수 있다.
- CSS 작업시 발생하는 이름 충돌 및 고유성 문제를 해결할 수 있다.
- CSS 모듈은 CSS 파일내의 클래스 이름을 고유 식별자로 변환하여 JS코드에서 사용한다.
    - 스타일이 해당 컴포넌트에만 적용되도록 `보장`

## CSS 모듈을 설정하는 방법

- 하나의 컴포넌트에 대응하는 CSS 모듈 파일이 필요
- 기존의 CSS 파일명인 component.css 에서 component.module.css로 이름을 작성해야 한다.
- JSX를 사용하는 코드 내에서 import 문을 사용하여 CSS 모듈 파일을 불러온다.
    - React 뿐만아니라 vue 와 같은 타 라이브러리에서도 사용된다.
- 아래는 사용 예시

```jsx
import classes from './Button.module.css';

export default function Button () {
	return <button className={classes.button}>Click!</button>
}
```

## CSS 클래스와 모듈의 구성 및 명명

- CSS 클래스를 구성하는 방법론에는 여러가지가 있다.
- 그중 인기있는 `BEM` 방법론을 따르는 것도 좋은 방법이다.

### BEM 방법론

- Block, Element, Modifier
- Block 은 독립적인 개체
    - button, navigation bar 같은 자체적으로 의미를 가지는 것
- Element 는 블록에 `의미적으로 종속` , 자체적인 의미를 가지지는 않는다.
- Modifier 는 블록이나 요소에 대한 Flag로 외관이나 동작을 변경하는데 사용된다.

- BEM 을 적용한 Button.module.css

```css
/* Block */
.button {
	margin : 2rem auto;
}

/* Element */
.button__icon {
	color : #fff
}

/* Modifier */
.button--primary {
	backgroundColor : #ddd;
}
.button--secondary { 
	backgroundColor : #333;
 }
```

- React 에 모듈 CSS 적용

```jsx
import classes from "./Button.module.css";

export default function Button ({variant, children}) {
	const buttonClass = `${classes.button} ${class[`button--${variant}`]}`
	return <button className={buttonClass}>{children}</button>;
}
```

- CSS 모듈을 사용하는 것이 마치 객체를 import하여 사용하는 것과 상당이 유사하다.