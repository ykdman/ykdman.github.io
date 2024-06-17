---
title: Efficieneer 개발일지 - Day1
date: 2024-06-17 20:00:00 +09:00
categories: [Project, Efficineer]
tags:
  [
    React,
    TypeScript,
    Vite,
    Tailwind
  ]
---
# Efficieneer 개발 1 일차

- 2024-06-17 (월)

### 개발 branch

- 현재 Home 페이지를 제작중이므로 “feature/메인페이지” 브랜치에서 개발

## Root Layout Page 만들기

- tailwind.config.js 에 bg 컬러 커스텀 색상 설정 하기
    
    ```jsx
    /** @type {import('tailwindcss').Config} */
    export default {
      content: ["/index.html", "./src/**/*.{html,js,jsx,tsx}"],
      theme: {
        extend: {
          colors: {
            signature: "#4CCD99”,
          },
        },
      },
      plugins: [],
    };
    
    ```
    
    - bg 속성에 대한 커스텀 컬러를 위와 같이 설정하여 tailwind의 bg - “사용자 정의 클래스명” 으로 사용 가능하다.
    - 호기심과 창의력 + 조금은 묵직한 느낌을 주는 색상을 찾아서 적용 시켜 보자
    - 색상 선정 : "#4CCD99”
    

## MainNavigation 만들기

RootLayout의 컴포넌트로 들어갈 MainNavigation 컴포넌트 작성.

- Link 를 이용한 페이지 이동 링크
    - Notes
    - Todos
    - Calender

```tsx
import React, { useState } from "react";
import NavList from "./NavList";
import MainTitle from "./MainTitle";

const navList = [
  { name: "Notes", path: "/notes" },
  { name: "Todos", path: "/todos" },
  { name: "Calender", path: "/calender" },
];

function MainNavigation() {
  const [isNavOpen, setIsNavOpen] = useState(false);

  return (
    <header
      id="root-header"
      className="bg-signature h-24 max-h-24 w-full max-w-full min-w-full flex justify-around"
    >
      <section
        id="root-title"
        className="text-center flex items-center justify-center h-full "
      >
        {/* <h1 className="text-4xl p-0 m-0">Efficieneer</h1> */}
        <MainTitle />
      </section>
      <section id="root-nav-menu" className="flex justify-center items-center">
        <nav className={`${isNavOpen ? "block" : "hidden"} md:block`}>
          <ul id="root-nav-list" className="flex justify-around items-center">
            {navList.map((nav) => (
              <NavList
                key={nav.name + nav.path}
                name={nav.name}
                path={nav.path}
              />
            ))}
          </ul>
        </nav>
        <button
          className={`md:hidden block  text-black group `}
          onClick={() => setIsNavOpen(!isNavOpen)}
        >
          <svg
            className="w-8 h-8 group-hover:text-primary transition-color duration-300"
            fill="none"
            stroke="currentColor"
            viewBox="0 0 24 24"
            xmlns="http://www.w3.org/2000/svg"
          >
            <path
              strokeLinecap="round"
              strokeLinejoin="round"
              strokeWidth="2"
              d="M4 6h16M4 12h16m-7 6h7"
            ></path>
          </svg>
        </button>
      </section>
    </header>
  );
}

export default MainNavigation;

```

### NavList 컴포넌트 작성

- MainNavigation 에서 서비스 목록에 대한 링크를 제공하기 위해 사용 되는 컴포넌트

```tsx
import React from "react";
import { Link } from "react-router-dom";

interface NavListProps {
  name: string;
  path: string;
}

function NavList({ name, path }: NavListProps) {
  return (
    <li className="p-3 mx-2 cursor-pointer ">
      <Link to={path} className="text-lg hover:text-primary transition-colors">
        {name}
      </Link>
    </li>
  );
}

export default NavList;

```

### MainTitle 컴포넌트 작성

- MainNavigation의 홈 버튼으로 사용될 컴포넌트

```tsx

import React from "react";
import { Link } from "react-router-dom";

function MainTitle() {
  return (
    <Link to="/">
      <h1 className="text-4xl p-0 m-0">Efficieneer</h1>
    </Link>
  );
}

export default MainTitle;

```

## 전역 CSS 설정

- Font Family : mono 로 설정

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  /* background: #434242; */
  background: #686d76;
  font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas,
    "Liberation Mono", "Courier New", monospace;
}

```

## tailwind.config.js 오늘의 설정

```jsx
/** @type {import('tailwindcss').Config} */
export default {
  content: ["/index.html", "./src/**/*.{html,js,jsx,tsx}"],
  theme: {
    extend: {
      colors: {
        signature: "#4CCD99", // 프로젝트 시그니처 색상
        primary: "#FBF3D5",
        black: "#000",
        card: "#E8F6EF",
        button: {
          common: "#495e57",
        },
      },
  },
  plugins: [],
};

```

- tailwind 의 장점을 여실히 알 수 있는 부분 이었다.
- 사용자가 커스텀으로 사용하고 싶은 스타일을 지정하여 tailwind 문법으로 호출하여 사용할 수 있다.
- 다만 어려운 점은 `작명` 이 어렵다 ㅎㅎ

## 오늘의 코드 완성

- App.tsx
    
    ```tsx
    import { createBrowserRouter, RouterProvider } from "react-router-dom";
    import Root from "./components/Root";
    import Home from "./pages/Home";
    
    const router = createBrowserRouter([
      {
        path: "/",
        element: <Root />,
        children: [{ index: true, element: <Home /> }],
      },
    ]);
    
    function App() {
      return <RouterProvider router={router} />;
    }
    
    export default App;
    
    ```
    
- Root.tsx
    
    ```tsx
    import React from "react";
    import MainNavigation from "./MainNavigation";
    import { Outlet } from "react-router-dom";
    
    function Root() {
      return (
        <>
          <MainNavigation />
          <main className="h-screen">
            <Outlet />
          </main>
        </>
      );
    }
    
    export default Root;
    
    ```
    
- MainNavigation.tsx
    
    ```tsx
    import { useState } from "react";
    import NavList from "./NavList";
    import MainTitle from "./MainTitle";
    
    const navList = [
      { name: "Notes", path: "/notes" },
      { name: "Todos", path: "/todos" },
      { name: "Calender", path: "/calender" },
    ];
    
    function MainNavigation() {
      const [isNavOpen, setIsNavOpen] = useState(false);
    
      return (
        <header
          id="root-header"
          className="bg-signature h-24 max-h-24 w-full max-w-full min-w-full flex justify-around"
        >
          <section
            id="root-title"
            className="text-center flex items-center justify-center h-full "
          >
            {/* <h1 className="text-4xl p-0 m-0">Efficieneer</h1> */}
            <MainTitle />
          </section>
          <section id="root-nav-menu" className="flex justify-center items-center">
            <nav className={`${isNavOpen ? "block" : "hidden"} md:block`}>
              <ul id="root-nav-list" className="flex justify-around items-center">
                {navList.map((nav) => (
                  <NavList
                    key={nav.name + nav.path}
                    name={nav.name}
                    path={nav.path}
                  />
                ))}
              </ul>
            </nav>
            <button
              className={`md:hidden block  text-black group `}
              onClick={() => setIsNavOpen(!isNavOpen)}
            >
              <svg
                className="w-8 h-8 group-hover:text-primary transition-color duration-300"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
                xmlns="http://www.w3.org/2000/svg"
              >
                <path
                  strokeLinecap="round"
                  strokeLinejoin="round"
                  strokeWidth="2"
                  d="M4 6h16M4 12h16m-7 6h7"
                ></path>
              </svg>
            </button>
          </section>
        </header>
      );
    }
    
    export default MainNavigation;
    
    ```
    
- MainTitle.tsx
    
    ```tsx
    import React from "react";
    import { Link } from "react-router-dom";
    
    function MainTitle() {
      return (
        <Link to="/">
          <h1 className="text-4xl p-0 m-0">Efficieneer</h1>
        </Link>
      );
    }
    
    export default MainTitle;
    
    ```
    
- NavList.tsx
    
    ```tsx
    import React from "react";
    import { Link } from "react-router-dom";
    
    interface NavListProps {
      name: string;
      path: string;
    }
    
    function NavList({ name, path }: NavListProps) {
      return (
        <li className="p-3 mx-2 cursor-pointer ">
          <Link to={path} className="text-lg hover:text-primary transition-colors">
            {name}
          </Link>
        </li>
      );
    }
    
    export default NavList;
    
    ```
    

## 오늘의 개발 소감

일단, 시작하기를 잘했다는 생각이 많이 든다.

UI가 특히나 못봐줄 정도지만, 나름 내가 좋아하는 색을 프로젝트의 시그니처 색으로 지정하는

`패기 ?` 까지 선보였다.

일단 첫 개인 프로젝트이고, 완전 제로부터 시작하니까

천천히 하나씩 더해가는 것에 많은 의의를 두고자 한다.

그리고 tailwind쓰기로 한 건 잘한거 같다.

우연히 Next.js로 개발된 페이지들의 소스들을 보게 되었는데, tailwind로 스타일링을 하는 것을 많이 본 것 같다.

그리고 오늘 가장 크게 느낀건!
`컴포넌트 재사용성` 에 대해 정말 많은 고민과 경험이 필요함을 느꼈다.

컴포넌트 하나를 작성해도, 이것이 나중에 쓰이려면 어떻게 써야할까를 계속 고민하다보니

물론 더 어려워 졌지만, 성취감이 정말 높았다.

그리고 루틴을 정해서 프로젝트에 몰두하는 시간을 정해서 집중하니까

확실히 시간이 훅훅 지나갔다.

개발은 정말 재밌다.