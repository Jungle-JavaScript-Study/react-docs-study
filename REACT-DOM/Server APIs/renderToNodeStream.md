# renderToNodeStream
- React 트리를 읽기 가능한 Node.js Stream으로 렌더링하는 API
```js
const stream = renderToNodeStream(reactNode, options?)
```

> 🚨 Deprecated
> - 이후에 제거될 예정인 API
> - 대신 `renderToPipeableStream`을 사용하자

<br/>

# 1. Reference 

```js
import { renderToNodeStream } from 'react-dom/server';

const stream = renderToNodeStream(<App />);
stream.pipe(response);
```

- 서버에서 `renderToNodeStream`을 호출해서 연결(pipe into)할 수 있는 '읽기 가능한 Node.js 스트림'을 응답으로 받는다.
- 클라이언트에서는 `hydrateRoot`를 호출해서 서버에서 생성된 HTML을 상호작용 가능하도록 만든다.

## 1) Parameters
### reactNode
- HTML로 렌더링하려는 React 노드
    - ex: `<App />`과 같은 JSX 엘리먼트

### options (optional)
- 서버 렌더링을 위한 옵션 객체
- identifierPrefix: useId에 의해 생성된 ID에 사용할 문자열 접두사로, 같은 페이지에서 여러 개의 루트를 사용할 때 충돌을 피하는 데 유용하다. hydrateRoot에 전달한 접두사와 동일해야 한다.

## 2) Returns
- HTML 문자열을 출력하는 읽기 가능한 Node.js 스트림

## 3) 주의사항
- 이 메서드는 모든 Suspense 경계가 완료될 때까지 출력을 반환하지 않고 대기한다.
- React 18부터는 모든 출력을 버퍼링하기 때문에 실제로는 스트리밍의 이점을 누릴 수 없다.
    - 따라서 `renderToPipeableStream`으로 마이그레이션 하시길
- 반환된 스트림은 utf-8로 인코딩된 바이트 스트림이다.
    - 다른 인코딩의 스트림이 필요한 경우, 텍스트를 변환하는 변환 스트림을 제공하는 iconv-lite와 같은 프로젝트를 사용ㄱㄱ

<br/>

# 2. Usage
## 1) React 트리를 HTML로 읽기 가능한 Node.js 스트림에 렌더링하기

```js
import { renderToNodeStream } from 'react-dom/server';

// route 핸들러 구문은 백엔드 프레임워크에 따라 다름
app.use('/', (request, response) => {
  const stream = renderToNodeStream(<App />);
  stream.pipe(response);
});
```

- 스트림은 React 컴포넌트의 초기 non-interactive HTML 출력을 생성할 것이다.
- 클라이언트에서는 서버에서 생성된 HTML을 hydrate해서 interactive하게 만들기 위해 `hydrateRoot`를 호출해야 한다.