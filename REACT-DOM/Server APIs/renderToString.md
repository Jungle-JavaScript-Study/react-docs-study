# renderToString
- React 트리를 HTML 문자열로 렌더링한다.
- 스트리밍 또는 데이터 대기를 지원하지 않는다.

```js
const html = renderToString(reactNode, options?)
```

<br/>

# 1. Reference

```js
import { renderToString } from 'react-dom/server';

const html = renderToString(<App />);
```

- 서버에서 `renderToString`을 호출하면 앱을 HTML로 렌더링한다.
- 클라이언트에서 `hydrateRoot`를 호출하면 서버에서 생성한 HTML을 interactive하게 만든다.

## 1) Parameters
### reactNode
- HTML로 렌더링하려는 React 노드
    - ex: `<App />`과 같은 JSX 엘리먼트

### options (optional)
- 서버 렌더링을 위한 옵션 객체
- identifierPrefix: useId에 의해 생성된 ID에 사용할 문자열 접두사로, 같은 페이지에서 여러 개의 루트를 사용할 때 충돌을 피하는 데 유용하다. hydrateRoot에 전달한 접두사와 동일해야 한다.

## 2) Returns
- HTML 문자열

## 3) 주의사항
- Suspense를 제한적으로 지원한다. 컴포넌트가 일시 중단되면 즉시 폴백을 HTML로 전송한다.
- 브라우저에서도 작동은 하지만, 클라이언트 코드에서 사용하는 것은 권장하지 않는다.

<br/>

# 2. Usage
## 1) React 트리를 HTML 문자열로 렌더링하기

```js
import { renderToString } from 'react-dom/server';

// route 핸들러 구문은 백엔드 프레임워크에 따라 다름
app.use('/', (request, response) => {
  const html = renderToString(<App />);
  response.send(html);
});
```

- 앱을 서버 응답과 함께 보낼 수 있는 HTML 문자열로 렌더링하기 위해 `renderToString`을 호출한다.
- React 컴포넌트의 초기 non-interactive HTML 출력이 생성된다.
- 클라이언트에서는 서버에서 생성된 HTML을 hydrate해서 interactive하게 만들기 위해 `hydrateRoot`를 호출해야 한다.

<br/>

# 3. Pitfall
- 스트리밍 또는 데이터 대기를 지원하지 않는다.
  - 아래의 대안을 참고하자!

<br/>

# 4. 대안
## 1) 스트리밍 메서드로 마이그레이션하기
- `renderToString`은 즉시 문자열을 반환하므로 스트리밍이나 데이터 대기를 지원하지 않는다.
- 가능하다면, 다음과 같은 완전한 기능을 갖춘 대안을 사용하는 것을 권장한다!!
  - Node.js를 사용한다면, `renderToPipeableStream` 쓰기
  - Deno 또는 Web Streams를 지원하는 modern edge runtime을 사용한다면, `renderToReadableStream` 쓰기
  - 사용하는 서버의 환경이 스트림을 지원하지 않는다면 `renderToString` 그냥 쓰기

## 2) 클라이언트 코드에서 `renderToString` 제거하기
- 클라이언트에서 `react-dom/server`를 가져오면 번들 크기가 불필요하게 증가하므로 피해야 한다!!
- 브라우저에서 일부 컴포넌트를 HTML로 렌더링해야 하는 경우에는 `createRoot`를 사용해서 DOM에서 HTML을 읽어오자

```js
import { createRoot } from 'react-dom/client';
import { flushSync } from 'react-dom';

const div = document.createElement('div');
const root = createRoot(div);
flushSync(() => {
  // innerHTML 속성을 읽기 전에 DOM을 업데이트하기 위해 flushSync 사용
  root.render(<MyIcon />);
});
console.log(div.innerHTML); // For example, "<svg>...</svg>"
```

<br/>

# 5. Troubleshooting
## 1) 컴포넌트에 Suspense를 도입하면 HTML에 항상 폴백만 보이는 경우
- `renderToString`은 Suspense를 완전히 지원하지 않는다.
- 일부 컴포넌트가 중단되는 경우 (ex: lazy로 정의되었거나 데이터를 가져오는 경우), `renderToString`은 이 컨텐츠가 resolve될 때까지 기다리지 않고, 가까운 Suspense 경계를 찾아서 폴백 prop을 HTML에 렌더링함ㅠ
  - 이 컨텐츠는 클라이언트 코드가 로드될 때까지 나타나지 않을 것이다.
- 이 문제를 해결하기 위해 스트리밍 솔루션 중 하나를 사용하자
  - 이 솔루션들은 서버에서 컨텐츠가 해결됨에 따라 내용을 조각으로 스트리밍할 수 있어 사용자가 클라이언트 코드가 로드되기 전에 페이지가 점진적으로 채워지는 것을 볼 수 있다.