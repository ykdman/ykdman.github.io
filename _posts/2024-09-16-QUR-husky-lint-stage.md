---
title: 궁금증 1 - husky와 lint-stage 설정 (in Nest.js)
date: 2024-09-16 10:20:00 +09:00
categories: [궁금증]
tags:
  [
    husky,
    commit,
    git,
    npm
  ]
---
# Nest.js 에 husky + lint-stage 설정


### husky를 사용해보는 이유

- 개발자들과의 협업 과정에 있어서 설정해서 개발자들이 편해질 수 있는 방법은 가능한 다 시도해보고 싶었다.
- BE 리드를 맡게 된 이상, 세팅은 나의 영역이라는 마음에 husky도 사용해보게 됐다.
- 이전 프로젝트와 지금 진행중인 프로젝트에서 FE 리드 분이 husky를 세팅해주셨는데,
사용해보니 실제로 내가 편하다는 느낌을 받았다.
- 세팅 방법이 궁금했다!

### husky 란?

> Husky enhances your commits and more 🐶 *woof!*
Automatically lint yout commit messages, code, and run tests upon committing or pushing
[- husky github-](https://typicode.github.io/husky/)
> 

husky 공식문서에 따르면, husky는 커밋 또는 푸시에 따른 코드, 커밋 메세지, 테스트를 자동 lint 해주는 역할이다. 라고 쓰여져 있다.

### lint-stage 까지 설정한 이유

- [lint-stage](https://github.com/lint-staged/lint-staged)

실제 lint가 커밋 할 때 마다 모든 코드에 작동하게 되면, 전체 파일을 확인하고 

lint작업을 진행하기 때문에 속도가 느리다.

lint-stage 는 내가 원하는 파일 (변경된 파일) 만 특정해서 lint를 진행 시키는 동작을 하기 때문에

팀원들의 commit에 더 좋은 영향을 주기 위해서 적용하게 되었다.

## 설정 방법

- 모든 과정은 eslint 와 prettier 가 설정되었다는 전제에 수행한다!

### 1. npm 설치

- husky 설치

```bash
# npm
npm install --save-dev husky

# yarn
yarn add --dev husky

# pnpm
pnpm add --save-dev husky
```

- husky init

```bash
# npm
npx husky init

#pnpm
pnpm exec husky init
```

현재 최신 버전의 husky는 기존의 husky 방식 처럼 husky.sh를 이용한 방식을 사용하지 않고,

.husky 폴더 내부의 스크립트 파일들 (pre-commit, commit-msg등) 을 직접 수행하는 방식으로 변경

되었다.

- husky 디렉토리 생성
    
    ```bash
    npx husky install
    ```
    
    - 해당 명령을 수행하면, git hooks 를 해당 프로젝트 디렉토리에서 관리하게 된다.

### 2. package.json 스크립트 설정

- npx husky init, 또는 npx husky install 을 수행하면 Package.json에 postinstall 스크립트가 추가 된다.
    
    ```json
    "scripts" : {
    	"postinstall" : "husky install"
    }
    ```
    

### 3. pre-commit 작성

- git hook 을 이용하여, commit 에 대해 husky를 통한 작업을 수행 할 수 있다.
- pre-commit 파일은 커밋 전, 코드 스타일을 검사하거나 lint를 수행하는 등의 작업을 하는 동작을 적는 스크립트 파일이다.

- 파일 추가방법
    1. npx 로 추가
        
        ```bash
        npx husky add .husky/pre-commit "npm run lint"
        ```
        
    2. 수동 추가
        - 생성된 프로젝트 root 경로의 .husky 폴더에 ‘pre-commit’ 이라는 파일을 생성
        - 내부에 명령어를 작성
            
            ```bash
            #!/bin/sh
            
            npm run lint
            ```
            

- 실제 내가 작성한 pre-commit
    
    ```bash
    #!/bin/sh
    
    FILES=$(git diff --cached --name-only --diff-filter=ACM | grep '\.ts$' | sed 's/\\/\//g')
    
    echo "변경된 파일 목록: $FILES"
    
    if [ -z "$FILES" ]; then
      echo "변경된 TypeScript 파일이 없습니다."
      exit 0
    fi
    
    echo "ESLint 실행 중..."
    RESULT=$(npx eslint $FILES --format stylish --fix)
    
    echo "$RESULT"
    
    if echo "$RESULT" | grep -q 'error'; then
      echo "Lint 에러가 발견되었습니다. 커밋을 중단합니다."
    
      echo "다음은 Lint 에러가 발생한 파일 및 상세 항목입니다:"
      echo "$RESULT" | grep -E 'error|^\s+\d+:\d+'
    
      exit 1
    else
      echo "Lint에 적합합니다. 커밋을 계속 진행합니다."
      exit 0
    fi
    
    ```
    
    1. 변경 된 파일 중 .ts 확장자 인 파일만 FILES 라는 변수에 넣는다. 
        1. 운영체제에 따른 경로 표시가 다르기 때문에 /, \ 둘다 사용 가능하도록 설정
        2. FILES가 비어있으면 해당 과정을 그냥 pass
    2. FILES에 들어간 변경된 파일들을 대상으로 Lint 검사를 수행한다.
        1. Lint 검사 상 위배된 규칙이 없으면 정상 pass ⇒ commit 단계로 간다.
        2. Lint 검사 상 위배된 규칙이 발생하면, Lint에러 메세지를 출력 후 종료 ⇒ commit 취소

### 4. commit-msg 작성

- commit-msg 파일은 커밋을 수행할 때 husky를 통해 수행할 작업에 대한 스크립트를 작성하는 곳
- 커밋 메세지 패턴
    - `[영어 소문자]` : `[영어, 한글, 하이픈 사용가능]`

- 실제 사용 스크립트
    
    ```bash
    #!/bin/sh
    
    commit_msg_file=$1
    commit_msg=$(cat "$commit_msg_file")
    
    PATTERN='^[a-z]+ : [a-z가-힣\- ]+$'
    
    if ! echo "$commit_msg" | awk "/$PATTERN/" > /dev/null; then
      echo "커밋 양식을 지켜주세요'"
      exit 1
    fi
    
    echo "커밋 메시지가 패턴에 맞습니다"
    
    ```
    

### 5. lint-stage 설정

- 설치
    
    ```bash
    npm install lint-staged --save-dev
    ```
    
- package.json 내 스크립트 추가
    
    ```json
    	"lint-staged": {
    		"*.ts": [
    			"eslint --max-warnings=0 --fix",
    			"prettier --write",
    			"git add"
    		]
    	}
    ```
    
    1. eslint 를 수행 후, 발견된 위반 사항이 없을 때 통과
    2. prettier 수행
    3. git add 수행

### 결과

실제 lint-stage 설정과 pre-commit 의 코드 내용을 확인해보면

npx eslint 를 변경된 파일들에 대해서 수행하게 된다.

변경된 파일들 중 Lint 규칙을 위배하는 파일이 없다면,

git add 및 기존 commit 에 대한 작업을 수행하게 되면

이때 commit-msg의 스크립트가 실행된다.

(커밋 메시지 패턴에 맞지 않으면 커밋 취소)

### 요약

- husky 과 lint-staged 를 사용해서 commit 시 변경된 파일에 대해서만 lint및 특정 작업을 수행
- 모든 과정은 chatGPT 만을 이용해서 작업 했다.
    - 블로그 글들이 모두 같은 내용이기에 궁금해서 gpt로만 해봤다.

- [gpt 대화 내용 링크](https://chatgpt.com/share/66e70e90-91f0-8006-9738-fceddb805fd1)