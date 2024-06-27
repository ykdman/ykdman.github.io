---
title: Efficieneer 개발일지 - Day 5
date: 2024-06-25 20:30:00 +09:00
categories: [Project, Efficineer]
tags:
  [
    React,
    TypeScript,
    Vite,
    Tailwind
  ]
---
# Efficieneer 개발 5일차

## Quill.js 와 Mantine-RichTextEditor 비교

- [Mantine-RichTextEditor](https://mantine.dev/x/tiptap/)
- [Quill.js](https://quilljs.com/docs/quickstart)

하루종일, React **Wysiwyg(워지윅) 라이브러리를 찾아보고 직접 구현한 케이스들도 찾아봤다.**

실제로 현업을 하게 되면, 이런 라이브러리를 찾는 시간들이 오래걸리기보다는 

직접 구현하는 일이 많겠지?

아무튼, 정말 유용한 오픈소스를 찾았다.

Mantine 라는 React Component Library  인데, 이 설명에서 부터 개발자 경험에 집중하여

개발된 React Component 라이브러리 라고 한다.

다양한 Extension을 통해 정말 편리하게 사용할 수 있는 컴포넌트 들을 제공하고 있는 것 같다.

다른 워지윅 라이브러리 관련 글들에는 정말 해당 라이브러리를 찾아 볼수가없었는데,

github를 돌아다니다가 정말 우연히 발견헀다.

(기적이다..)

Document를 어느정도 정독한결과

그냥 폼이 미쳤다…

아직 써보지는 않았고 문서를 읽어보는게 시간이 좀 걸리기도 했고,

해당 라이브러리가 신뢰할만한 라이브러리 인지 확인하는 것도 중요하다고 생각하여 이리저리

찾아보았다.

### 결론

처음에는 Quill.js를 사용하려 했으나, npm 에서는 2년전에 업데이트가 멈춰있기도 했고,

사용하는 환경이 바닐라 JS에 오히려 더 좋은 것 같았다.

반면 Mantine Rici Text Editor는 설명부터 `React Component` 를 못박고 시작했기 때문에

일단 목적 자체가 내가 원하는 바와 부합했고

다양한 추가 Extension 라이브러리가 존재하여, mantine 오픈소스 자체에서 꽤 많은 도움을 받을 수 도 있지 않을 까 싶다.

Mantine 사용자 디스코드도 가입해버렸다.

현재는 새벽 2시,

이따가 해당 라이브러리로 Note 페이지를 개발 진행할것이다.

## TipTap Editor.js 적용

- [TipTap Editor.js](https://tiptap.dev/docs/editor/installation/react)

### 의존성 설치

```bash
npm install @mantine/tiptap @mantine/core @mantine/hooks @tiptap/react @tiptap/pm @tiptap/extension-link @tiptap/starter-kit
```

### Ricih Text Editor 스타일 적용하기

- Project의 Root 인 ‘main.tsx’ 에 스타일 import

```bash
import '@mantine/tiptap/styles.css';
```

### 결론.

Notes 부분은 추후 마지막으로 개발할 기능으로 놔두어야 할 것 같다…

## 오늘 느낀점

완전히 잘못 생각했다.

지금 코스에서 vanilla-extract를 이용해서 스타일링을 진행하고 있는데,

현재 프로젝트에서 tailwind를 써보니,

무언가 불편한 점이 확실히 있다.

지금 아니면 엎기도 애매하다.

얼른 스타일링 라이브러리를 갈아 엎어야 겠다.