---
title: Efficieneer 개발일지 - Day 3
date: 2024-06-21 20:30:00 +09:00
categories: [Project, Efficineer]
tags:
  [
    React,
    TypeScript,
    Vite,
    Tailwind
  ]
---
# Efficieneer 개발일지 : 3 일차

### 먼저 느낀점!

Main 페이지의 레이아웃을 구성하다가, Flex와 Grid의 사용해야할 시점에 대해서 아주 조금 깨달음이 왔다.

Flex는 확실히 컴포넌트 단위에서는 강력한 스타일링 기능이다.

그러나, 페이지 하나의 레이아웃을 구성하기에는 축 1개에 대한 방향성만 설정할 수 있는 것이 꽤나 제약이라고 생각이들었는데

꽁꽁 고민하다가 ChatGPT에게 flex없이 중앙정렬 및 사이드바 사용이 가능한 코드로 변환해달라고 요청 했는데,

```tsx
// 이전 main 태그 클래스
"flex justify-center items-baseline pt-[96px] w-full h-full md:h-auto overflow-auto";

// 변경된 main 태그 클래스
"grid place-items-center pt-[96px] w-full h-full md:h-auto overflow-auto"
```

tailwind 코드를 확인해보면, 새로 작성 해준 className이 조금 더 적다는 것을 알 수 있는데,

이걸 보고 또 기대하게 된 점은, 각 서비스 (Notes, Todo, Calender) 페이지 마다, 사이드바 요소를 만드려고 했는데

flex로만 해야한다고 생각이 들었을 때는 조금 막막 했었는데, grid 가 머릿속에 들어오니까 조금 머리가 트였다.

그래서 css 의 grid에 대해 간단히라도 지금 당장 알아 봐야겠다고 생각이 들었다.