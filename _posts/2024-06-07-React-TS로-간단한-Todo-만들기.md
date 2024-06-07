---
title: React TS를 적용하여 간단한 Todo 앱 만들기
date: 2024-06-07 21:40:00 +09:00
categories: [TIL, React]
tags:
  [
    TIL,
    javascript,
    React,
    TS,
    Typescript,
    contextAPI,
    Hook
  ]
image: ../assets/img/common/react-logo.png
---

# TS + React 프로젝트 만들기

- 프로젝트 github : https://github.com/ykdman/Study/tree/main/React-TS-practice

- [Vite 시작하기](https://ko.vitejs.dev/guide/)

- vite CLI를 이용하여, React + TS + SWC 로 프로젝트를 생성

  ```bash
  npm create vite@latest
  ```

- 옵션 설정
    - React 라이브러리 사용
    - TypeScript + SWC 옵션 사용

- package.json
    
    ```json
    {
      "name": "react-ts-practice",
      "private": true,
      "version": "0.0.0",
      "type": "module",
      "scripts": {
        "dev": "vite",
        "build": "tsc && vite build",
        "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
        "preview": "vite preview"
      },
      "dependencies": {
        "react": "^18.2.0",
        "react-dom": "^18.2.0"
      },
      "devDependencies": {
        "@types/react": "^18.2.66",
        "@types/react-dom": "^18.2.22",
        "@typescript-eslint/eslint-plugin": "^7.2.0",
        "@typescript-eslint/parser": "^7.2.0",
        "@vitejs/plugin-react-swc": "^3.5.0",
        "eslint": "^8.57.0",
        "eslint-plugin-react-hooks": "^4.6.0",
        "eslint-plugin-react-refresh": "^0.4.6",
        "typescript": "^5.2.2",
        "vite": "^5.2.0"
      }
    }
    
    ```
    
    - @types로 지정된 패키지들은 `번역기` 역할을 수행한다.
    - scripts에도 기존 바닐라 JS로 작업할 때 없던 `tsc` 명령어가 포함되어 있는 것을 알 수 있다.
- vite-env-d.ts
    - 개발자가 작성하는 TS 와 바닐라 JS를 연결하는 역할을 한다.
    

# TS + React 로 간단한 ToDo 웹 만들기

- Components 구조로 간단한  ToDo 어플을 만든다.


## Props를 TS로 작업하기

React에서 Props를 TS로 작업하기 위해서는 children의 타입까지 동시에 지정해주어야 올바르다.

이에 대해 불편함이 존재하여, TS의 제네릭을 이용하여 컴포넌트의 props 타입을 지정한다.

- 참고 블로그 (TS로 컴포넌트 작업하기)
    - https://webruden.tistory.com/931
    - https://velog.io/@velopert/create-typescript-react-component
    - [https://fedev-kim.medium.com/react-react-react-fc-함수형-컴포넌트-타입-e20ddb28a95a](https://fedev-kim.medium.com/react-react-react-fc-%ED%95%A8%EC%88%98%ED%98%95-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-%ED%83%80%EC%9E%85-e20ddb28a95a)
- Interface와 Type의 차이점 참고
    - https://yceffort.kr/2021/03/typescript-interface-vs-type
    

### React 함수형 컴포넌트 TS로 만들어보기

- props는 기본적으로 children 속성을 받아 들인다.
- 개발자가 자체적으로 React.FC (함수형 컴포넌트) 명시를 하지 않으면, TS는 해당 함수형 컴포넌트가 함수형 컴포넌트 인지 알 수가 없다.

1. 선언하고자 하는 함수형 컴포넌트에 `React.FC` 제네릭을 지정한다.
2. 위에서 선언한 제네릭 안에 React.PropsWithChildren으로 children을 받아오는 props를 사용함을 제네릭으로 다시 한번 지정한다.
    
    ```tsx
    import React from "react";
    
    // FC는 Function COmponent => 함수형 컴포넌트를 의미한다.
    const Todos: React.FC<React.PropsWithChildren<{ items: string[] }>> = (
      props
    ) => {
      return (
        <ul>
          {props.items.map((item) => (
            <li key={item}>{item}</li>
          ))}
          {props.children}
        </ul>
      );
    };
    
    export default Todos;
    
    ```
    
    - 위의 방법을 type을 이용해서 좀 더 깔끔하게 작성하는 방법
    
    ```tsx
    import React from "react";
    
    // props로 받아야 할 값들을 아래의 type에 지정 (interface도 가능)
    type TodoProps = {
      items: string[];
    };
    
    // propsWith Children 제네릭으로 위의 TodoProps를 받아오는 제네릭 타입 생성
    type PropsWithChildren = React.PropsWithChildren<TodoProps>;
    
    // FC는 Function COmponent => 함수형 컴포넌트를 의미한다.
    // React.FC의 제네릭에 작성한 PropsWithChildren 을 사용하여 children과 TodosProps를 사용가능하게 됐다. 
    const Todos: React.FC<PropsWithChildren> = (props) => {
      return (
        <ul>
          {props.items.map((item) => (
            <li key={item}>{item}</li>
          ))}
          {props.children}
        </ul>
      );
    };
    
    export default Todos;
    
    ```
    
    - 현재 기준 (2024.06.07) children 을 사용하지 못하는 방식
        
        ```tsx
        const Todos: React.FC<{ items: string[] }> = (props) => {
          return (
            <ul>
              {props.items.map((item) => (
                <li key={item}>{item}</li>
              ))}
              {props.children}
            </ul>
          );
        };
        
        export default Todos;
        
        ```
        
        - Todos 컴포넌트를 사용하여 children을 사용할 수 없음이 에러 메세지로 표시된다.

하지만 현재 프로젝트를 진행하면서, React.FC에 대해 조금 알아보니, 해당 타입지정자가 없어도 함수형 컴포넌트를 올바르게 작성할 수 있고, 또한 React.FC 자체가 TS와 충돌을 일으킬 가능성이 있기 때문에, 다른 방법으로 함수형 컴포넌트를 작성하였다.

## CSS Module 을 위한 설정

- vite-end.d.ts 설정파일에 해당 구문을 추가한다.
    
    ```tsx
    /// <reference types="vite/client" />
    // module CSS 사용을 위한 선언
    declare module "*.module.css" {
      const classes: { [key: string]: string };
      export default classes;
    }
    ```
    

## Todo 컴포넌트 작성

- App.tsx 구성
    - NewTodo.tsx
    - Todo.tsx
        - TodoItem.tsx

### 상태 관리

- context API 를 사용하여 Todo 리스트 관리
- todos-context.tsx
    
    ```tsx
    import React, { useState } from "react";
    import Todo from "../models/todo";
    
    /* todo.ts
    class Todo {
      id: string;
      text: string;
      constructor(todoText: string) {
        this.text = todoText;
        this.id = new Date().toISOString();
      }
    }
    
    export default Todo; 
    */
    
    type TodoContextType = {
      items: Todo[];
      addTodo: (todoText: string) => void;
      removeTodo: (id: string) => void;
    };
    
    // 자식 컴포넌트가 사용하게 될 context
    export const TodoContext = React.createContext<TodoContextType>({
      items: [],
      addTodo: () => {},
      removeTodo: (id: string) => {
        id;
      },
    });
    
    // TodoContextProvider 사용
    // React.PropsWithChildren 으로 children 사용 가능
    function TodosContextProvider(props: React.PropsWithChildren) {
      const [todos, setTodos] = useState<Todo[]>([]);
    
      function addTodoItem(todoText: string) {
        setTodos((prevTodos) => {
          return [...prevTodos, new Todo(todoText)];
        });
      }
      function removeTodoItem(id: string) {
        setTodos((prevTodos) => {
          return prevTodos.filter((todo) => todo.id !== id);
        });
      }
    	
    	// 자식 컴포넌트가 사용할 context Value 값
      const ctxValue: TodoContextType = {
        items: todos,
        addTodo: addTodoItem,
        removeTodo: removeTodoItem,
      };
      return (
        <TodoContext.Provider value={ctxValue}>
          {props.children}
        </TodoContext.Provider>
      );
    }
    
    export default TodosContextProvider;
    
    ```
    

### App.tsx

```tsx
import React, { useContext } from "react";
// import Todo from "../models/todo";
import TodoItem from "./TodoItem";
import classes from "./Todos.module.css";
import { TodoContext } from "../store/todos-context.tsx";

// FC는 Function Component => 함수형 컴포넌트를 의미한다.
function Todos(props: React.PropsWithChildren) {
  const TodosCtx = useContext(TodoContext);

  return (
    <ul className={classes.todos}>
      {TodosCtx.items.map((item) => (
        <TodoItem
          key={item.id}
          todoId={item.id}
          removeItemHandler={TodosCtx.removeTodo}
        >
          {item.text}
        </TodoItem>
      ))}
      {props.children}
    </ul>
  );
}

export default Todos;

```

### Todo.tsx

```tsx
import React, { useContext } from "react";
// import Todo from "../models/todo";
import TodoItem from "./TodoItem";
import classes from "./Todos.module.css";
import { TodoContext } from "../store/todos-context.tsx";

// FC는 Function COmponent => 함수형 컴포넌트를 의미한다.
function Todos(props: React.PropsWithChildren) {
	// context 사용
  const TodosCtx = useContext(TodoContext);

  return (
    <ul className={classes.todos}>
      {TodosCtx.items.map((item) => (
        <TodoItem
          key={item.id}
          todoId={item.id}
          removeItemHandler={TodosCtx.removeTodo}
        >
          {item.text}
        </TodoItem>
      ))}
      {props.children}
    </ul>
  );
}

export default Todos;

```

### TodoItem.tsx

```tsx
import React from "react";
import classes from "./TodoItem.module.css";

type TodoItemProps = {
  todoId: string;
  removeItemHandler: (id: string) => void;
};

function TodoItem(props: React.PropsWithChildren<TodoItemProps>) {
  return (
    <li
      onClick={() => props.removeItemHandler(props.todoId)}
      className={classes.item}
    >
      {props.children}
    </li>
  );
}

export default TodoItem;

```

### NewTodo.tsx

```tsx
import React, { useContext, useRef } from "react";
import classes from "./NewTodo.module.css";
import { TodoContext } from "../store/todos-context";

function NewTodo() {
  const TodosCtx = useContext(TodoContext);
  const inputTextRef = useRef<HTMLInputElement>(null);

  function submitHandler(event: React.FormEvent) {
    event.preventDefault();
    const enteredText = inputTextRef.current!.value;

    if (enteredText.trim().length === 0) {
      // error
      return;
    }
    TodosCtx.addTodo(enteredText);
    inputTextRef.current!.value = "";
  }

  return (
    <form onSubmit={submitHandler} className={classes.form}>
      <label htmlFor="text">Todo Text </label>
      <input type="text" id="text" ref={inputTextRef} />
      <button>Add Todo</button>
    </form>
  );
}

export default NewTodo;

```

## Todo 추가 제거 함수 작성

- TodosContext에서 Todo 요소 추가 및 제거 함수를 작성한다.

```tsx

type TodoContextType = {
  items: Todo[];
  addTodo: (todoText: string) => void;
  removeTodo: (id: string) => void;
};

function TodosContextProvider(props: React.PropsWithChildren) {
  const [todos, setTodos] = useState<Todo[]>([]);

  // Todo 추가 함수
  function addTodoItem(todoText: string) {
    setTodos((prevTodos) => {
      return [...prevTodos, new Todo(todoText)];
    });
  }
  // Todo 삭제 함수
  function removeTodoItem(id: string) {
    setTodos((prevTodos) => {
      return prevTodos.filter((todo) => todo.id !== id);
    });
  }

  const ctxValue: TodoContextType = {
    items: todos,
    addTodo: addTodoItem,
    removeTodo: removeTodoItem,
  };
  return (
    <TodoContext.Provider value={ctxValue}>
      {props.children}
    </TodoContext.Provider>
  );
}

```

## tsconfig.json

- target
    - TS 컴파일러가 TS 파일을 변환할 떄 타겟으로 삼는 ES 버전 설정
    - 최신 브라우저 환경에서는 ES6  로 설정함이 좋고, 구식 브라우저 사용시에는 ES5버전 이하로 낮추어서 지정해주어야 할 수 도있다.
    - exnext는 현재 시점 가장 최신의 ES 버전을 지칭한다.
- lib
    - TS파일을 JS로 컴파일 할 때, 포함할 라이브러리 목록
    - dom : HTML Element와 같은 `DOM API`사용을 위해 적용
    - dom.iterable
    - es2015 : async 문법이나 Promise 를 필요로 할 때 사용
- allowJs
    - JS 파일을 컴파일에 포함할 것인지 여부 설정
    - JS에서 TS로 점진적으로 전환할 때 사용하면 유용 한 옵션 이다.
- strict
    - 타입 확인을 더욱 `엄격하게` 하기 위할 떄 True로 놓고 사용한다.
- jsx
    - JS 파일에서 JSX구문을 내보내는 방법을 컨트롤 한다.
- include
    - 변환할 폴더를 지정
- exclude
    - 변환하지 않을 폴더를 지정