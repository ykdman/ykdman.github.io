---
title: JS Promise와 async/await 알아보기
date: 2024-05-30 13:00:00 +09:00
categories: [JS, Javascript]
tags:
  [
    javascript,
    JS,
    비동기,
    async,
    await,
    Promise
  ]
image: ../assets/img/common/JavaScript-logo.png
---

# JS의 Promise와 async/await 알아보기

> Promise는 시간이 소비에 상관없이 `약속` 한 결과를 만들어내는 문법이다.
> 

### 기본적인 Promise 형태

```jsx
const promise = new Promise(function(resolve, reject) {
    // executor    
})
```

- executor는 실행자 함수로, Promise가 만들어질 때 자동으로 실행된다.
- executor의 인수 resolve와  reject는 JS에서 기본으로 제공하는 콜백이다.
    - 상황에 따라 필수로 resolve와 reject중 하나는 꼭 반환해야 한다.
- resolve : 일이 `성공적` 으로 종료된 경우, 결과를 나타내는 Value와 함꼐 호출
- reject : 작업에서 에러가 발새앟ㄹ 경우, 해당 에러의 에러객체를 나타내는 error와 함꼐 호출

### Promise의 State

new Promise로 반환하는 Promise 객체에는 State(상태)와 result가 존재한다.

- state
    - pending : 시작시 발생하는 상태 , 보류중으로 판단.
    - fulfilled : resolve가 호출 되면 적용되는 상태
    - rejected : reject가 호출 되면 적용되는 상태
- result
    - 처음 실행 시 `undefined`
    - resolve(value) 가 호출 되면, `value`
    - reject(error) 가 호출 되면 , `error` 가 된다.
    

## Promise 소비하기

new Promise로 반환된 Promise 객체는 특정 소비 method를  통해 성공, 실패, 무조건 수행의 작업을 진행할 수 있다.

- Promise의 소비자
    - then
    - catch
    - finally

```jsx
const promise = new Promise(function(resolve, reject) {
    // executor    
})
```

위의 코드가 존재한다고 가정할 떄, then의 사용 예시를 알아보자

### 소비자 then

Promise 객체 뒤에 붙는 then은 Promise의 결과에 기반하여(성공 or 실패) 나머지 작업을 진행 시킬 수 있는 method이다.

```jsx
promise.then(
    function(result) {console.log("성공", result)},
    function(error) {console.error(error))}
)
```

then method는 두 개의 callback 함수를 매개 변수로 받는데 각각의 콜백 함수는 성공의 result와 

실패의  error 값을 인수로 받는다.

위의 코드에서 promise의 수행결과가 성공 시, 즉 resolve(value)를 반환했을 때에는

result를 인수로받은 함수가 실행되고, 실패 시에는 error을 인수로 받는 함수가 실행 된다.

중요한것은 인수가 아닌, 콜백이 들어가는 `매개변수 자릿수` 이다.

첫번째 콜백 매개변수는 성공에 대한 콜백 함수를 받으며, 두번째 콜백 매개 변수는 실패에 대한 콜백 함수를 전달해야만 한다.

### 소비자 catch

catch는 Promise의 executor가 실패를 반환 했을 때 작동하는데, 실제 원리는 then(null, callback(err)) 와 같다. (then의 첫번째 인수에 null을 전달하여 성공에 대한 콜백을 수행하지 않는 것과 같다는 뜻

### 소비자 finally

Promise의 결과에 상관없이 마무리로 수행을 하게 되는 method 이다.

Promise의 특징으로는 `인수가 존재하지 않는다.` 

즉, Promise의 결과상태를 알 수 없다.

```jsx
promise.then(
    function(result) {console.log("성공", result)
).catch(function(err) {
	console.log(err)
}).finally(function() {
	console.log("무조건 실행")
}) 
```

## Promise 체이닝

- 순차적으로 처리해야 하는 비동기 작업에 사용하면 유용하다.

```jsx
new Promise(function(resolve, reject) {

  setTimeout(() => resolve(1), 1000); // (*)

}).then(function(result) { // (**)

  console.log(result); // 1
  return result * 2;

}).then(function(result) { // (***)

  console.log(result); // 2
  return result * 2;

}).then(function(result) {

  console.log(result); // 4
  return result * 2;

});
```

위와 같은 코드에서는 첫 Promise의 resolve가 1초뒤에 반환 된 뒤, 이어지는 then들이 순차적으로 실행 됨을 알 수 있다.

이것이 가능한 이유는, promise.then 호출은 결국 `promise` 를 반환하기 때문에 가능하다.

즉, 각각의 연결된 then은 이전의 then에서 return 으로 반환하게 된 promise를 참조할 수 있게 되는 것.

### then으로 Promise 반환(생성)

```jsx
new Promise(function(resolve, reject) {

  setTimeout(() => resolve(1), 1000);

}).then(function(result) {

  alert(result); // 1

  return new Promise((resolve, reject) => { // (*)
    setTimeout(() => resolve(result * 2), 1000);
  });

}).then(function(result) { // (**)

  alert(result); // 2

  return new Promise((resolve, reject) => {
    setTimeout(() => resolve(result * 2), 1000);
  });

}).then(function(result) {

  alert(result); // 4

});
```

위의 코드에서는 연속된 then이 new Promise로 새로운 Promise를 반환하는데, 반환된 Promise들은 모두 다음 then에 result에 값을 전달하는 것을 알 수 있다.

`.then` 또는 `.catch`, `.finally`의 핸들러(어떤 경우도 상관없음)가 프라미스를 반환하면
다음 체인된 핸들러는 이전 Promise가 처리 될 때까지 무조건 기다린다는 것을 알 수 있다.

## Promise API

Promise의 클래스에는 몇가지 static Method가 존재힌다.

- Promise.all
- Promise.allsettled
- Promise.race

### Promise.all

- 여러개의 Promise를 동시에 실행 시켜 모든 Promise가 처리 될 떄 까지 기다린다.

```jsx
let promise = Promise.all([...promises...]);
```

- 전체 요소가 Promise인 배열을 받고 배열 안 Promise가 모두 처리된 결과 값을 담은 배열이 result로 반환된다.
- 각 Promise의 처리순서가 달라도 결과는 배열의 순사대로 저장된다.

- 여러개의 fetch를 수행하는 Promise.all 예시
    
    ```jsx
    let urls = [
      'https://api.github.com/users/iliakan',
      'https://api.github.com/users/Violet-Bora-Lee',
      'https://api.github.com/users/jeresig'
    ];
    
    // fetch를 사용해 url을 프라미스로 매핑합니다.
    let requests = urls.map(url => fetch(url));
    
    // Promise.all은 모든 작업이 이행될 때까지 기다립니다.
    Promise.all(requests)
      .then(responses => responses.forEach(
        response => alert(`${response.url}: ${response.status}`)
      ));
    ```
    
- Promise.all 은 전달되는 Promise 중 하나라도 `reject(거부)` 되면, Promise.all 이 반환하는 Promise는 error와 함께 바로 거부된다.
    - Promise가 하나라도 거부되면, 배열에 저장된 다른 Promise의 결과는 완전히 무시된다.

### Promise.allSettled

- 최근에 추가된 API로 구식 브라우저에서는 `폴리필`이 필요
- Promise.allSettled는 all 과 같이 PRomise 배열을 받아 수행한 결과를 반환한다.
    - 모든 프라미스가 처리 될 떄 까지 기다리는 것 또한 똑같다.
    - Promise.all과 다르게 성공, 실패에 대한 각각의 결과를 다 저장한 result를 반한다.
    
    ```jsx
    [
      {status: 'fulfilled', value: ...응답...}, // 성공 Promise 결과 1
      {status: 'fulfilled', value: ...응답...}, // 성공 Promise 결과 2
      {status: 'rejected', reason: ...에러 객체...} // 실패 Promise 결과 3
    ]
    ```
    

### Promise.race

Promise.all 과 비슷 하나, `가장 먼저 처리되는` Promise의 결과를 반환한다.

```jsx
Promise.race([
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
  new Promise((resolve, reject) => setTimeout(() => reject(new Error("에러 발생!")), 2000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
]).then(alert); // 1
```

- Promise.race는 Promise 경주의 가장 먼저 처리된 승자가 나타나면, 다른 Promise들의 결과는 무시한다.

## async와 await

Promise를 좀 더 쉽게 사용할 수 있는 문법

### async 함수

- async 키워드를 function 앞에 위치시켜 사용한다.
- async함수는 항상 Promise (또는 이행된 Promise)를 반환한다.

```jsx
async function test() {
	return 1;
}
```

### await

- async 함수 내부에서만 동작하는 키워드
- JS는 async 함수 안에서 `await` 키워드를 만나면 해당 Promise 가 처리될때 까지 기다린다.

```jsx
async function f() {

  let promise = new Promise((resolve, reject) => {
    setTimeout(() => resolve("완료!"), 1000)
  });

  let result = await promise; // 프라미스가 이행될 때까지 기다림 (*)

  alert(result); // "완료!"
}

f();
```

- 함수 본분 실행 도중 `(*)` 로 표시된 줄에서 실행이 잠시 중단 되었다가 Promise가 처리되면 실행을 재개 한다.
- await으로 지정한 promise가 반환하는 resolve(value) 의 value가 변수 result에 저장된다.

`await` 은 정확히 Promise가 처리될 때 까지 함수 실행을 기다리게 만든다.

Promise가 처리되면 Promise의 결과와 함꼐 async 함수의 실행이 재개 된다.

Promise가 처리되길 기다리는 동안, 엔진은 다른일을 할 수 있기 때문에, CPU 리소스가 낭비되지 않는 장점이 있다.

### Promise 와 async/await 가독성 비교

- github에서 사용자 아바타를 가져와 띄우는 코드를 각각 Promise 와 async/await문법을 사용하였을 떄의 가독성 비교를 해보자.

- Promise
    
    ```jsx
    function loadJson(url) {
      return fetch(url)
        .then(response => response.json());
    }
    
    function loadGithubUser(name) {
      return fetch(`https://api.github.com/users/${name}`)
        .then(response => response.json());
    }
    
    function showAvatar(githubUser) {
      return new Promise(function(resolve, reject) {
        let img = document.createElement('img');
        img.src = githubUser.avatar_url;
        img.className = "promise-avatar-example";
        document.body.append(img);
    
        setTimeout(() => {
          img.remove();
          resolve(githubUser);
        }, 3000);
      });
    }
    
    // 함수를 이용하여 다시 동일 작업 수행
    loadJson('/article/promise-chaining/user.json')
      .then(user => loadGithubUser(user.name))
      .then(showAvatar)
      .then(githubUser => alert(`Finished showing ${githubUser.name}`));
    ```
    
- async/await
    
    ```jsx
    async function showAvatar() {
    
      // JSON 읽기
      let response = await fetch('/article/promise-chaining/user.json');
      let user = await response.json();
    
      // github 사용자 정보 읽기
      let githubResponse = await fetch(`https://api.github.com/users/${user.name}`);
      let githubUser = await githubResponse.json();
    
      // 아바타 보여주기
      let img = document.createElement('img');
      img.src = githubUser.avatar_url;
      img.className = "promise-avatar-example";
      document.body.append(img);
    
      // 3초 대기
      await new Promise((resolve, reject) => setTimeout(resolve, 3000));
    
      img.remove();
    
      return githubUser;
    }
    
    showAvatar();
    ```
    

async/await 문법이 Promise에 비해 좀 더 간결하고 코드흐름 파악이 원함함을 알 수 있다.

### async/await의 에러처리

- 기본적인 JS 의 `try{...} catch{...}` 로 처리 가능하다.
- await으로 지정한 Promise에서 에러가 발생하면, throw문으로 작성한 것처럼 error 가 발생한다.

```jsx
async function f() {

  try {
    let response = await fetch('http://유효하지-않은-주소');
  } catch(err) {
    alert(err); // TypeError: failed to fetch
  }
}

f().catch(err => alert(err));
```

try에서 사용하는 await Promise에서 Error 발생 했을 때는, 코드 제어 흐름이 catch 블록으로 넘어가게 된다.

때문에 try 블록안에서 발생할 수 있는 에러를 모두 잡기 위해 catch 블록에 에러 처리 로직을 작성하는 것이 일반적이며,

위의 코드에서 처럼 catch 후에도 불안하다면, 호출한 async 함수뒤에 catch 핸들러를 추가하여 에러를 핸들링 할 수 있다.