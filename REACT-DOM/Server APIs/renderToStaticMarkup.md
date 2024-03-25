# renderToStaticMarkup
- Non-interactive React 트리를 HTML 문자열로 렌더링한다.

```js
const html = renderToStaticMarkup(reactNode, options?)
```

<br/>

# 1. Reference

```js
import { renderToStaticMarkup } from 'react-dom/server';

const html = renderToStaticMarkup(<Page />);
```

- React 컴포넌트에 대한 Non-interactive HTML 출력물을 생성한다.

## 1) Parameters
### reactNode
- HTML로 렌더링하려는 React 노드
    - ex: `<Page />`과 같은 JSX 엘리먼트

### options (optional)
- 서버 렌더링을 위한 옵션 객체
- identifierPrefix: useId에 의해 생성된 ID에 사용할 문자열 접두사로, 같은 페이지에서 여러 개의 루트를 사용할 때 충돌을 피하는 데 유용하다.

## 2) Returns
- HTML 문자열

## 3) 주의사항
- 이 메서드의 출력은 hydrate 될 수 없다.
- Suspense를 제한적으로 지원한다. 컴포넌트가 일시 중단되면 즉시 폴백을 HTML로 전송한다.
- 브라우저에서도 작동은 하지만, 클라이언트 코드에서 사용하는 것은 권장하지 않는다.
    - 브라우저에서 컴포넌트를 HTML로 렌더링해야 하는 경우 DOM 노드 안에 렌더링(`createRoot`)함으로써 HTML을 가져오도록!

<br/>

# 2. Usage
## 1) Non-interactive React 트리를 HTML 문자열로 렌더링하기

```js
import { renderToStaticMarkup } from 'react-dom/server';

// route 핸들러 구문은 백엔드 프레임워크에 따라 다름
app.use('/', (request, response) => {
  const html = renderToStaticMarkup(<Page />);
  response.send(html);
});
```

- React 컴포넌트의 초기 Non-interactive HTML 출력이 생성된다.

<br/>

# 3. Pitfall
- 이 메서드는 hydrate 할 수 없는 Non-interactive HTML을 렌더링한다.
- 이 메서드는 React를 간단한 정적 페이지 생성기로 사용하거나, 이메일과 같이 완전히 정적인 컨텐츠를 렌더링할 때 유용하다.
- interactive 앱은 서버에서는 `renderToString`을 쓰고 클라이언트에서 `hydrateRoot`를 사용해야 한다.