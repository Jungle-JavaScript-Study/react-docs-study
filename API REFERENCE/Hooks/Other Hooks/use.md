use는 Canary(가장 최신의 개발 버전)와 실험 채널에서만 사용이 가능하다.

# use
use는 Promise나 context와 같은 리소스의 값을 읽을 수 있게 해주는 훅이다.

```js
const value = use(resource);
```

# 1. Reference

```js
import { use } from 'react';

function MessageComponent({ messagePromise }) {
  const message = use(messagePromise);
  const theme = use(ThemeContext);
  // ...
}
```

- use는 다른 모든 React 훅과 달리, 반복문이나 조건문 같은 곳에서도 호출될 수 있다!!
- 단, use를 호출하는 함수는 컴포넌트나 다른 훅이어야 한다.

- Suspense와의 통합: use는 Suspense와 함께 동작한다. 컴포넌트가 데이터를 기다리는 동안 다른 UI를 보여줄 수 있다.
- 컴포넌트 중단: use 훅에 Promise가 전달되면, 이 Promise가 완료될 때까지 use를 호출한 컴포넌트의 렌더링이 중단된다.
- 대체 컨텐츠 표시: 만약 use를 호출하는 컴포넌트가 Suspense 경계 안에 있으면, Promise가 완료될 때까지 대체 컨텐츠(fallback)가 표시된다.
- Promise 해결: Promise가 성공적으로 resolve되면, Suspense 대체 컨텐츠는 use 훅으로부터 반환된 데이터를 사용하여 렌더링된 컴포넌트로 대체된다.
- 에러 처리: use에 전달된 Promise가 reject되면, 가장 가까운 에러 경계의 대체 컨텐츠가 표시된다.

이러한 방식으로 use 훅은 비동기 작업이 진행되는 동안 사용자 인터페이스의 반응성을 유지하고, 데이터 로딩 또는 오류 상황을 처리할 수 있다.


## 1-1. Parameters
### resource
- 읽으려는 데이터의 출처
- Promise 혹은 context

## 1-2. Returns
- 리소스에서 읽은 값
- Promise의 해결된 값이나 context의 값


# 2. 주의사항
- 서버 컴포넌트에서 데이터를 가져올 때는 use 대신 async/await를 사용하는 것이 좋다.
- async/await는 await가 호출된 지점부터 렌더링을 이어가지만, use는 데이터가 resolve된 후 컴포넌트를 **다시 렌더링**한다.
- 컴포넌트에서 Promise를 직접 생성하지 말고, 서버 컴포넌트에서 Promise를 생성해서 클라이언트 컴포넌트로 전달하는 것이 바람직하다.
- 클라이언트 컴포넌트에서 생성된 Promise는 매 렌더링마다 재생성되지만, 서버 컴포넌트에서 전달된 Promise는 렌더링 간에 안정적으로 유지된다.


# 3. 사용법

## 3-1. context 읽기

- useContext 훅과 유사하지만, use는 조건문이나 반복문 내에서 호출할 수 있어 더 유연하다.

```jsx
if (show) {
  const theme = use(ThemeContext);
  return <hr className={theme} />;
}
```

## 3-2. 서버에서 클라이언트로 데이터 스트리밍
### 1) 서버 컴포넌트에서 Promise 생성
- 먼저, 서버 컴포넌트에서 데이터를 가져오는 fetchMessage 같은 함수를 호출하여 Promise를 생성한다.
- 이 Promise는 서버에서 클라이언트로 전달될 데이터를 포함합니다.

```jsx
import { fetchMessage } from './lib.js';
import { Message } from './message.js';

export default function App() {
  const messagePromise = fetchMessage();
  return (
    <Suspense fallback={<p>waiting for message...</p>}>
      <Message messagePromise={messagePromise} />
    </Suspense>
  );
}
```

### 2) 클라이언트 컴포넌트에서 Promise 사용
- 서버 컴포넌트에서 생성된 Promise는 prop으로 클라이언트 컴포넌트에 전달된다. 
- 클라이언트 컴포넌트는 이 Promise를 use 훅에 전달하여 데이터를 읽는다.

```jsx
// message.js
'use client';

import { use } from 'react';

export function Message({ messagePromise }) {
  const messageContent = use(messagePromise);
  return <p>Here is the message: {messageContent}</p>;
}
```

### 3) Suspense와의 통합
- Message 컴포넌트는 Suspense 컴포넌트로 감싸져 있다.
- 따라서, Promise가 해결될 때까지 Suspense의 대체 컨텐츠가 표시된다.

### 4) 데이터 로딩과 UI 업데이트
- Promise가 해결되면, use 훅을 통해 데이터가 읽혀지고, Message 컴포넌트는 Suspense의 대체 컨텐츠를 대신하여 렌더링된다.
- 이때, Message 컴포넌트는 서버에서 가져온 메시지를 표시한다.

```jsx
"use client";

import { use, Suspense } from "react";

function Message({ messagePromise }) {
  const messageContent = use(messagePromise);
  return <p>Here is the message: {messageContent}</p>;
}

export function MessageContainer({ messagePromise }) {
  return (
    <Suspense fallback={<p>⌛Downloading message...</p>}>
      <Message messagePromise={messagePromise} />
    </Suspense>
  );
}
```

### Note!
- 서버에서 클라이언트로 Promise를 전달할 때 Promise가 해결된 값은 서버와 클라이언트 간에 전송될 수 있도록 직렬화 가능한(serializable) 형태여야 한다. 
- 직렬화 가능한 데이터 유형은 주로 문자열, 숫자, 객체, 배열 등 기본적인 데이터 구조
- 함수와 같은 일부 데이터 유형은 직렬화할 수 없다.
- 서버에서 생성된 데이터가 클라이언트로 안전하게 전송되고, 클라이언트에서 올바르게 해석될 수 있도록!

## 3-3. 거부된 Promise를 처리하기
use에 전달한 Promise가 reject되는 경우에 대한 처리 방법을 알아보자!

- use는 try-catch 블록 안에서 호출될 수 없다.
- 대신, 컴포넌트를 에러 경계(Error Boundary)로 감싸거나, Promise의 .catch 메소드를 사용하여 use에 대체 값을 제공해야 한다.

### 1) ErrorBoundary를 사용하여 에러 표시

- ErrorBoundary를 사용하기 위해, use 훅을 호출하는 컴포넌트를 ErrorBoundary로 감싼다. 
- use에 전달된 Promise가 거부되면, ErrorBoundary의 대체 컨텐츠(fallback)가 표시된다.
- ErrorBoundary는 React 컴포넌트의 하위 트리에서 JavaScript 오류를 잡아내고, 오류가 발생했을 때 백업 UI를 렌더링하는 컴포넌트이다.

```jsx
"use client";

import { use, Suspense } from "react";
import { ErrorBoundary } from "react-error-boundary";

export function MessageContainer({ messagePromise }) {
  return (
    <ErrorBoundary fallback={<p>⚠️Something went wrong</p>}>
      <Suspense fallback={<p>⌛Downloading message...</p>}>
        <Message messagePromise={messagePromise} />
      </Suspense>
    </ErrorBoundary>
  );
}

function Message({ messagePromise }) {
  const content = use(messagePromise);
  return <p>Here is the message: {content}</p>;
}
```

### 2) Promise.catch에서 대체값 제공

- catch에 전달된 함수가 반환하는 내용은 Promise의 resolved 값으로 사용된다.

```jsx
import { Message } from './message.js';

export default function App() {
  const messagePromise = new Promise((resolve, reject) => {
    reject(); // 거부됨
  }).catch(() => {
    return "no new message found.";
  });

  return (
    <Suspense fallback={<p>waiting for message...</p>}>
      <Message messagePromise={messagePromise} />
    </Suspense>
  );
}
```
