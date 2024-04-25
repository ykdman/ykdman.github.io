---
title: React context 사용해보기
date: 2024-04-25 13:40:00 +09:00
categories: [TIL, React]
tags:
  [
    TIL,
    javascript,
    React,
    context,
    hook
  ]

---



## 참고 공식문서
[Context를 사용해 데이터를 깊게 전달하기 – React](https://ko.react.dev/learn/passing-data-deeply-with-context)

# Context 란

- `prop drilling` 을 방지하기 좋은 방법이다.
- context 로 제공되는 `provider` 컴포넌트 형식으로 자식 태그들을 감싸면, context 의 정보 (데이터) 에 접근 및 사용이 가능하다.
- 깊이가 얼마나 됐든, `context.provider` 로 감싸져 있다면, 자식 컴포넌트는 항사 접근이가능하다.
    - `context.provider` 가 제공하는 `value` 에 접근할 수 있는것

## context 사용 순서

1. context를 생성 한다.
    1. `createContext` api
    2. `export`
2. context가 필요한 컴포넌트에서 만들어진 context를 사용한다.
    1. `useContext`
    2. `context 객체`
3. context 를 제공한다.
    1. `<context.provider value={전달할 값}/>` 
    

### 1. context 생성 하기

- 임의로 실습을 진행하므로 간단하게 진행한다.
- app.jsx 가 React 렌더링을 진행
- 특정 버튼을 누를 때마다, random guid 를 발생 시키고, 발생한 guid를 렌더링하는 실습을 진행해본다.

- context 생성

```jsx
/** GuidContext.jsx */

// createContext api를 호출 한다.
import { createContext } from 'react';

// context 로 사용할 변수 를 선언
// createContext의 인수는 1개인데, 이때 인수는 '기본값' 이 들어가면 된다.

export const GuidContext = createContext('');

```

- react 소스 내부 index.d.ts 에서 작성되어있는 `createContext` 함수
- 코드를 확인해보면 인수가 1개 이며, 그 이름이 `defaultValue` 인 것을 알 수 있다.

```jsx
function createContext<T>(
        // If you thought this should be optional, see
        // https://github.com/DefinitelyTyped/DefinitelyTyped/pull/24509#issuecomment-382213106
        defaultValue: T,
    ): Context<T>;
    

```

## 2. context 사용하기

- app.jsx에서 context의 값을받아와 <li> 를 렌더링하는 Post 컴포넌트를 만들어보자

```jsx
/** Post.jsx */
import { useContext } from "react";
import { GuidContext } from "../store/GuidContext";

export default function Post() {
	// GuidContext 사용 => guidCtx
  const guidCtx = useContext(GuidContext);

  return (
    <>
      <ul>
        <li>{guidCtx}</li>
      </ul>
    </>
  );
}
```

- `useContext` hook 을 사용하여, 프로젝트 내에 있는 `GuidContext`  를 사용한다.
- `GuidContext` 를 사용한다고 선언한 변수 `guidCtx` 는 context의 값을 전부 조회 및 사용이 가능하다.
- 하지만 현재 context를 제공하지 않았기 때문에,  Post를 위와같이 수정하여도 context값이 표현 되지는 않는 상태이다.

## 3. context 제공하기

- 우리는 app.jsx 에서 `GuidContext` 를 호출 한다.
- context를 호출하기 위해서는, 최소 2가지의 준비가 필요하다.
    - `context Object`

이제 app.jsx에서 `GuidContext` 를 호출 해보자

```jsx
/** app.jsx */

import Post from './components/Post';
import { GuidContext } from './store/GuidContext';
import {useState} from 'react';

export default function App() {
const [userGuid, setUserGuid] = useState("");

  return (
    // GuidContext에 접근 가능하도록 자식 컴포넌트들을 모두 감쌌다.
    <GuidContext.Provider>
      {/* button */}
      <button>Generate!</button>
      {/* Post */}
      <Post />
    </GuidContext.Provider>
  );
}

```

- 지금 상태에서는 context가 어떠한 작동도 하지 않을 것이다.
- 왜냐면 context 의 Provider가 접근가능한 `value` 를 열어주지 않았기 때문!

Provider 가 접근 가능한 Value를 열도록 해주자

```jsx
/** app.jsx */

import Post from './components/Post';
import { GuidContext } from './store/GuidContext';

export default function App() {
	 const [userGuid, setUserGuid] = useState("");
		
  return (
	    // Provider 에 value 속성을 사용하여 자식 컴포넌트 들이 guidCtx에 모두 접근 가능해졌다.
      <GuidContext.Provider value={userGuid}>
      {/* button */}
      <button>Generate!</button>
      {/* Post */}
      <Post />
    </GuidContext.Provider>
  );
}

```

- 이제 `GuidContext.Provider` 태그의 자식 컴포넌트들은 Context가 제공하는 value  `userGuid` 에 접근 가능하다.
- 추가적으로 버튼을 누르면 `userGuid` 에 새로운 값을 할당해주는 함수를 만들어 연결해놓자.

```jsx
/** app.jsx */
import Post from "./components/Post";
import { GuidContext } from "./store/GuidContext";
import { useState } from "react";

export default function App() {
  const [userGuid, setUserGuid] = useState("");
  
  // random generate guid 함수
  function generateGuid() {
    const guid = crypto.randomUUID();
    setUserGuid(guid);
  }

  return (
    // GuidContext에 접근 가능하도록 자식 컴포넌트들을 모두 감쌌다.
    <GuidContext.Provider value={userGuid}>
      {/* button */}
      <button onClick={generateGuid}>Generate!</button>
      {/* Post */}
      <Post />
    </GuidContext.Provider>
  );
}
```

이제 context 를 사용하여, Provider 를 통해 자식 컴포넌트들에게 전역 참조가 가능해지게 되었다.

결과적으로, 지금은 Post 컴포넌트 외에 이렇다 할 자식 컴포넌트가 없지만,

실제로는 정말 많은 자식 컴포넌트를 이용한 프로젝트 때에는, context를 사용하는 것이 아니면

Prop Drilling 이 발생할 수 밖에 없을 것 같다.

context를 대체할 수 있는 전역 상태 관리 라이브러리인 redux, recoil, zustand 것들도 있는데,

그에 관해서는 추후 한번 익혀보면서 작성해야겠다.

### context를 익혀보면서…

- 공식문서 기반으로 익히면서, 그 안에있는 예제 까지 가져오고 싶지는 않아서 직접 간단한 예제를 만들어 보면서 했다.
- 실제로context는 이런 간단한 상황이 아닌 useReducer 까지 구현하여 같이 사용하여야 효과가 배가 되는것을 알지만, 우선 context에 집중해서 글을 작성했다.
- 공식문서 짱이다.