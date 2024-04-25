---
title: React reduce 사용해보기
date: 2024-04-25 16:40:00 +09:00
categories: [TIL, React]
tags:
  [
    TIL,
    javascript,
    React,
    reduce,
    hook
  ]
image: ../assets/img/common/react-logo.png
---
## 참고 공식문서
[state 로직을 reducer로 작성하기 – React](https://ko.react.dev/learn/extracting-state-logic-into-a-reducer)

# Reduce
- useState의 이벤트 핸들러 함수들이 복잡해 질 때 사용하면좋다.
- 컴포넌트의 state 로직을 통합할 수 있는 장점이 있다.
- state 사용법에 대한 기초지식이 필요하다.

## reducer 를 사용하는 방법

1. state를 사용하고 있다면, action 을 dispatch 함수로 전달하는 것으로 바꾼다.
2. reducer함수 작성
3. 컴포넌트에서 reducer 사용

이제 개별적으로 만든 예제를 통해서 실습해보자!

- App.jsx 와 UserInput.jsx 만 존재하는 컴포넌트를 만들었다.
- 프로젝트는, UserInput 컴포넌트 input에 이름을 작성하고 add 또는 subtract 버튼을 누르면 input 에 작성된 이름의 카운트 수를 증가 , 감소 시키는 간단한 프로젝트 이다.

```jsx
/** app.jsx */
import { useState } from "react";

import UserInput from "./components/UserInput";

function App() {
  const [userCount, setUserCount] = useState([]);

  /**
   * user Count를 증가 시키는 함수
   * @param {string} name
   */
  function handleAddNameCount(name) {
    // 조건에 따라 Count +1 또는 새로운 객체 삽입

    if (userCount.length === 0) {
      // userCount 길이가 0일 때, 새로운 객체 삽입
      setUserCount((prevUsers) => [...prevUsers, { name, count: 1 }]);
    } else {
      // userCount에 이미 타겟 객체가 존재할때 Count + 1 한 객체로 업데이트
      const existingIndex = userCount.findIndex((user) => user.name === name);
      const updateUserCount = {
        name,
        count: userCount[existingIndex].count + 1,
      };
      setUserCount((prevEvent) => {
        return [
          ...prevEvent.filter((user) => user.name !== name),
          { ...updateUserCount },
        ];
      });
    }
  }

  /**
   * user Count를 감소시키는 함수
   * @param {string} name
   */
  function handleSubtractNameCount(name) {
    // 이미 존재하는 User정보 확인
    const existingIndex = userCount.findIndex((user) => user.name === name);

    console.log(existingIndex);
    if (existingIndex === -1) {
      // User가 존재하지 않는 경우
      console.log(`${name} key was not exist in list`);
      return;
    } else if (userCount[existingIndex].count > 0) {
      // userCount에 존재하면서 count > 0 일 때 count - 1 한 객체로 업데이트
      const updateUserCount = {
        name,
        count: userCount[existingIndex].count - 1,
      };
      console.log("update : " + updateUserCount);
      setUserCount((prevUsers) => {
        return [
          ...prevUsers.filter((user) => user.name !== name),
          { ...updateUserCount },
        ];
      });
    } else if (userCount[existingIndex].count === 0) {
      // userCount 에 이미 존재하지만 count 가 0일 때는 삭제
      setUserCount((prevUsers) => {
        return [...prevUsers.filter((user) => user.name !== name)];
      });
    }
  }
  return (
    <>
      <UserInput
        onAddUser={handleAddNameCount}
        onSubtractUser={handleSubtractNameCount}
      />
    </>
  );
}

export default App;
      console.log("update : " + updateUserCount);
      setUserCount((prevUsers) => {
        return [
          ...prevUsers.filter((user) => user.name !== name),
          { ...updateUserCount },
        ];
      });
    } else if (userCount[existingIndex].count === 0) {
      // userCount 에 이미 존재하지만 count 가 0일 때는 삭제
      setUserCount((prevUsers) => {
        return [...prevUsers.filter((user) => user.name !== name)];
      });
    }
  }
  return (
    <>
      <UserInput
        onAddUser={handleAddNameCount}
        onSubtractUser={handleSubtractUser}
      />
    </>
  );
}

export default App;

```

- app.jsx 에서는 userCount 라는 state 와 state 를 업데이트하는 handelAddNameCount, handleSubtractNameCount 함수가 존재한다.

```jsx
/** UserInput.jsx */
import { useRef } from "react";

// eslint-disable-next-line react/prop-types
export default function UserInput({ onAddUser, onSubtractUser }) {
  const inputText = useRef("");
  return (
    <div className="user-input">
      <input type="text" ref={inputText} />
      <button
        onClick={() => {
          onAddUser(inputText.current.value);
        }}
      >
        Add
      </button>
      <button onClick={() => onSubtractUser(inputText.current.value)}>
        Subtract
      </button>
    </div>
  );
}

```

- UserInput.jsx는 props 로 onAddUser와 onSubtractUser를 전달받아 각각의 버튼에 이벤트바인딩을 하였다.
- 이제 위의 state 기반 소스코드를 reducer 기반 소스코드로 변경해보자

## 1. state 의 이벤트 핸들러 함수들의 내용을 모두 지우고. reducer형태에 맞게 dispatch 로 변환한다.

```jsx
  /**
   * user Count를 증가 시키는 함수
   * @param {string} name
   */
  function handleAddNameCount(name) {
    dispatch({
    // action 객체
	    type : 'add',
	    name: name
    })
  }

  /**
   * user Count를 감소시키는 함수
   * @param {string} name
   */
  function handleSubtractNameCount(name) {
   dispatch({
       // action 객체
	   type : 'subtract',
	   name: name
   })
  }
```

- `dispath` 함수에 넣어준 객체를 `action` 이라고 한다.
- reducer에 전달될 `action` 객체에는 반드시 String 타입의 `type` 프로퍼티가 존재해야 한다.

## 2. reducer 함수 작성하기

- 기존의 이벤트 핸들러 함수들에 dispatch 를 적용해 놓았다면, 이제 reducer 함수를 작성해볼 차례다.

```jsx
function UserReducer(state, action) {
	// React 가 설정하게될 다음 state 값을 반환한다.
}
```

- reducer 함수안에서의 return 은 다음 state 의 값이 된다.
- reducer 함수의 인자
    1. state 선언자리
    2. action 객체 선언자리

- reduce 함수에서는 switch  문을 사용하는 것이 정석이다.

```jsx
/** UserReducer.jsx */
export default function UserReducer(state, action) {
  switch (action.type) {
    case "add": {
      if (
        state.length === 0 ||
        state.filter((user) => user.name === action.name).length === 0
      ) {
        // state 길이가 0일 때, 새로운 객체 삽입
        // setstate((prevUsers) => [...prevUsers, { name, count: 1 }]);
        return [...state, { name: action.name, count: 1 }];
      } else {
        // state에 이미 타겟 객체가 존재할때 Count + 1 한 객체로 업데이트
        const existingIndex = state.findIndex(
          (user) => user.name === action.name
        );
        const updateState = {
          name: action.name,
          count: state[existingIndex].count + 1,
        };
        return [
          ...state.filter((user) => user.name !== action.name),
          { ...updateState },
        ];
        // setstate((prevEvent) => {
        //   return [
        //     ...prevEvent.filter((user) => user.name !== name),
        //     { ...updatestate },
        //   ];
        // });
      }
    }

    case "subtract": {
      const existingIndex = state.findIndex(
        (user) => user.name === action.name
      );

      if (existingIndex === -1) {
        // User가 존재하지 않는 경우
        console.log(`${action.name} key was not exist in list`);
        return;
      } else if (state[existingIndex].count > 0) {
        // userCount에 존재하면서 count > 0 일 때 count - 1 한 객체로 업데이트
        const updateUserCount = {
          name: action.name,
          count: state[existingIndex].count - 1,
        };
        console.log("update : " + updateUserCount);
        return [
          ...state.filter((user) => user.name !== action.name),
          { ...updateUserCount },
        ];

        // setUserCount((prevUsers) => {
        //   return [
        //     ...prevUsers.filter((user) => user.name !== name),
        //     { ...updateUserCount },
        //   ];
        // });
      } else if (state[existingIndex].count === 0) {
        // userCount 에 이미 존재하지만 count 가 0일 때는 삭제
        return [...state.filter((user) => user.name !== action.name)];

        // setUserCount((prevUsers) => {
        //   return [...prevUsers.filter((user) => user.name !== name)];
        // });
      }
    }
    // eslint-disable-next-line no-fallthrough
    default: {
      throw Error(`Unknown action type : ${action.type}`);
    }
  }
}
      //   return [
        //     ...prevUsers.filter((user) => user.name !== name),
        //     { ...updateUserCount },
        //   ];
        // });
      } else if (state[existingIndex].count === 0) {
        // userCount 에 이미 존재하지만 count 가 0일 때는 삭제
        return [...state.filter((user) => user.name !== action.name)];

        // setUserCount((prevUsers) => {
        //   return [...prevUsers.filter((user) => user.name !== name)];
        // });
      }
    }
  }
}

```

- 기존에 userCount 의 이벤트 핸들러 함수들을 reduce 형식에 맞게 재작성 하였다.

## 3. 컴포넌트에서 reducer 사용하기

- app.jsx 에서 state 사용을 reducer를 사용하는 방식으로 변경
1. `useRedcue` hook import
2. useReduce 함수를 사용하여, reduce 사용을 선언
- `useReduce` hook 의 인자
    1. reduce 함수
    2. 초기 state 값

```jsx
/** app.jsx */
import { useReducer } from "react";

import UserInput from "./components/UserInput";
import UserReducer from "./store/UserReducer";

function App() {
  // const [userCount, setUserCount] = useState([]);
  const [userCount, dispatch] = useReducer(UserReducer, []);
  /**
   * user Count를 증가 시키는 함수
   * @param {string} name
   */
  function handleAddNameCount(name) {
    dispatch({
      type: "add",
      name: name,
    });
  }

  /**
   * user Count를 감소시키는 함수
   * @param {string} name
   */
  function handleSubtractUser(name) {
    dispatch({
      type: "subtract",
      name: name,
    });
  }
  return (
    <>
      <UserInput
        onAddUser={handleAddNameCount}
        onSubtractUser={handleSubtractUser}
      />
    </>
  );
}

export default App;

```

- 추가적으로 userCount 가 렌더링 되는 것까지 확인하기 위해 간단하게 코드를 추가 작성 하겠다.

```jsx
/** app.jsx */
import { useReducer } from "react";

import UserInput from "./components/UserInput";
import UserReducer from "./store/UserReducer";

function App() {
  // const [userCount, setUserCount] = useState([]);
  const [userCount, dispatch] = useReducer(UserReducer, []);
  /**
   * user Count를 증가 시키는 함수
   * @param {string} name
   */
  function handleAddNameCount(name) {
    dispatch({
      type: "add",
      name: name,
    });
  }

  /**
   * user Count를 감소시키는 함수
   * @param {string} name
   */
  function handleSubtractUser(name) {
    dispatch({
      type: "subtract",
      name: name,
    });
  }
  
  // userCount 기반 렌더링 추
  return (
    <>
      <UserInput
        onAddUser={handleAddNameCount}
        onSubtractUser={handleSubtractUser}
      />
      <ul>
        {userCount.map((user) => (
          <li key={user.name}>{`${user.name} : ${user.count}`}</li>
        ))}
      </ul>
    </>
  );
}

export default App;

```

- 여기까지 useReduce 를 사용하여 state 이벤트  핸들러 함수를 외부에서 참조하는 방식을 알아보았다.

- 다음번엔 context 와 reduce를 같이사용하여 더 복잡한 상태 관리를 하는 방법을 공부해야곘다.