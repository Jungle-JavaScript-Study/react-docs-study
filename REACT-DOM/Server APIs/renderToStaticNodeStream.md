# renderToStaticNodeStream
- Non-interactive React 트리를 읽기 가능한 Node.js 스트림으로 렌더링한다.

```js
const stream = renderToStaticNodeStream(reactNode, options?)
```

<br/>

# 1. Reference

```js
import { renderToStaticNodeStream } from 'react-dom/server';

const stream = renderToStaticNodeStream(<Page />);
stream.pipe(response);
```

- 반환되는 stream은 React 컴포넌트에 대한 Non-interactive HTML 출력물을 생성한다.

## 1) Parameters
### reactNode
- HTML로 렌더링하려는 React 노드
    - ex: `<App />`과 같은 JSX 엘리먼트

### options (optional)
- 서버 렌더링을 위한 옵션 객체
- identifierPrefix: useId에 의해 생성된 ID에 사용할 문자열 접두사로, 같은 페이지에서 여러 개의 루트를 사용할 때 충돌을 피하는 데 유용하다.

## 2) Returns
- HTML 문자열을 출력하는 읽기 가능한 Node.js 스트림
  - 결과물인 HTML은 클라이언트에서 hydrate 할 수 없다.

## 3) 주의사항
- 이 메서드의 출력은 hydrate 될 수 없다.
- 이 메서드는 모든 Suspense 경계가 완료될 때까지 출력을 반환하지 않고 대기한다.
- React 18부터는 모든 출력을 버퍼링하기 때문에 실제로는 스트리밍의 이점을 누릴 수 없다.
- 반환된 스트림은 utf-8로 인코딩된 바이트 스트림이다.
    - 다른 인코딩의 스트림이 필요한 경우, 텍스트를 변환하는 변환 스트림을 제공하는 iconv-lite와 같은 프로젝트를 사용ㄱㄱ

<br/>

# 2. Usage
## 1) React 트리를 HTML로 읽기 가능한 Node.js 스트림에 렌더링하기

```js
import { renderToStaticNodeStream } from 'react-dom/server';

// route 핸들러 구문은 백엔드 프레임워크에 따라 다름
app.use('/', (request, response) => {
  const stream = renderToStaticNodeStream(<Page />);
  stream.pipe(response);
});
```

- 반환되는 스트림은 React 컴포넌트의 초기 Non-interactive HTML 출력을 생성한다.

<br/>

# 3. Pitfall
- 이 메서드는 hydrate 할 수 없는 Non-interactive HTML을 렌더링한다.
- 이 메서드는 React를 간단한 정적 페이지 생성기로 사용하거나, 이메일과 같이 완전히 정적인 컨텐츠를 렌더링할 때 유용하다.
- interactive 앱은 서버에서 `renderToPipeableStream`을 쓰고 클라이언트에서 `hydrateRoot`를 사용해야 한다.