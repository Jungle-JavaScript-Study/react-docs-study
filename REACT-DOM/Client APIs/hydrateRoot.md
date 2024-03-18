# hydrateRoot
- `react-dom/server`를 통해 사전에 생성한 HTML 컨텐츠를 가진 브라우저 DOM 노드 안에 React 컴포넌트를 표시할 수 있게 해주는 API

<br/>

# 1. Reference 
```js
import { hydrateRoot } from 'react-dom/client';

const domNode = document.getElementById('root');
const root = hydrateRoot(domNode, reactNode);
```

- 이미 서버 환경에서 React에 의해 렌더링된 기존 HTML에 React를 붙이는(attach) 방식
- React는 DOM 노드 내부에 존재하는 HTML에 붙어 그 안의 DOM을 관리하게 된다. 
- 전부 React로 구성된 앱은 일반적으로 루트 컴포넌트와 함께 hydrateRoot를 한 번만 호출한다.

## 1) Parameters
### domMode
- 서버에서 루트 요소로서 렌더링된 DOM 요소

### reactNode
- 기존 HTML을 렌더링하는 데 사용된 React 노드
- 보통 ReactDOM 서버 메서드인 `renderToPipeableStream(<App />)`으로 렌더링된 JSX 조각인 `<App />`이 된다.

### options (optional)
- 루트에 대한 옵션을 가진 객체
- onRecoverableError: 오류로부터 자동으로 복구될 때 호출될 콜백
- identifierPrefix: useId에 의해 생성된 ID에 사용할 문자열 접두사로, 같은 페이지에서 여러 개의 루트를 사용할 때 충돌을 피하는 데 유용하다. **서버에서 사용된 것과 동일한 접두어여야 한다!**

## 2) Returns
- `render` 메서드와 `unmount` 메서드가 있는 객체를 반환한다.

## 3) 주의사항
- `hydrateRoot`는 렌더링된 컨텐츠가 서버에서 렌더링된 컨텐츠와 동일하다고 예상하므로, 불일치를 버그로 취급하고 고쳐야 한다.
- dev mode에서 React는 hydration 중 일어나는 불일치에 대해 경고해준다.
    - 여기서 불일치는 서버에서 렌더링된 HTML과 클라이언트 사이드에서의 React 컴포넌트 간에 일치하지 않는 부분이 있는 것을 의미한다.
    - 속성이 불일치할 경우 해당 속성이 올바르게 적용되지 않을 수 있다.
    - 대부분의 앱에서 불일치는 드물기 때문에 성능을 위해 모든 markup을 검증하지 않는다.
- App에서 `hydrateRoot`는 단 한 번만 호출하게 될 것이고, 프레임워크를 사용한다면 프레임워크가 대신 해줄걸!
- App을 사전에 렌더링된 HTML 없이 클라이언트에서 직접 렌더링한다면 `hydrateRoot` 말고 `createRoot`를 사용 ㄱㄱ

<br/>

> 이어서, 리턴되는 객체에 담긴 `render` 메서드와 `unmount` 메서드에 대해 알아보자

# 1-1. root.render(reactNode)
```js
root.render(<App />);
```

- hydrate된 React 루트 내부의 컴포넌트를 업데이트하기 위해 `root.render`를 호출한다.
- 리액트는 hydrate된 루트 안에서 `<App />`을 업데이트 한다.

## 1) Parameters
### reactNode
- 업데이트하려는 React 노드
- 일반적으로 `<App />`과 같은 JSX 조각이 되지만, `createElement()`로 구성된 React 엘리먼트, 문자열, 숫자, null, undefined도 ㄱㄴ

## 2) Returns
- undefined

## 3) 주의사항
- root가 hydrating을 마치기 전에 `root.render`를 호출하면, React는 기존의 서버에서 렌더링된 HTML 내용을 지우고 전체 루트를 클라이언트 렌더링으로 전환한다.

# 1-2. root.unmount()
> createRoot와 완전 똑같다!

```js
root.unmount();
```

- React 루트 안에 렌더링된 트리를 파괴한다.
- 온전히 React만으로 작성된 앱에는 보통 `root.unmount`을 호출하지 않는다.
- 이 함수는 React 루트의 DOM 노드 (또는 그 조상 노드)가 다른 코드에 의해 DOM에서 제거될 수 있는 경우에 유용하다.
    - ex: DOM에서 비활성 탭을 제거하는 jQuery 탭 패널은 탭이 제거되면 그 안에 있는 모든 것도 DOM에서 제거될 테니 이때 `root.unmount`를 호출해서 제거된 루트의 컨텐츠 관리를 중지하도록 React에 알려줘야 한다.
    - 그러지 않으면 제거된 루트 내부의 컴포넌트가 구독과 같은 전역 리소스를 정리하고 확보할 수 없다.
- `root.unmount`를 호출하면 루트의 모든 컴포넌트가 unmount되고, 트리의 이벤트 핸들러나 state가 제거되며, 루트 DOM 노드에서 React가 분리된다.
- Parameters와 Returns 없음

## 1) 주의사항
- `root.unmount`를 호출하고 나면 같은 루트에 대해 `root.render`를 다시 호출할 수 없다.
    - unmount된 루트에서 `root.render`를 호출하려고 하면 `Cannot update an unmounted root` 오류가 발생한다.
    - but, 해당 노드의 이전 루트가 unmount 된 후 동일한 DOM 노드에 새로운 루트를 만드는 건 ㄱㄴ (새로 createRoot 해야된다는 뜻)

<br/>

# 2. Usage
## 1) 서버에서 렌더링된 HTML을 hydrate하기
- `react-dom/server`로 앱의 HTML을 만들었으면 클라이언트에서 hydrate 해줘야 한다.

```js
import { hydrateRoot } from 'react-dom/client';

hydrateRoot(document.getElementById('root'), <App />);
```

- 이렇게 하면 서버 HTML을 브라우저 DOM node에서 React 컴포넌트를 이용해 hydrate 해준다.
- (위에서도 설명했지만) 보통 앱을 시작할 때 단 한 번만 실행한다. (프레임워크를 쓴다면 프레임워크가 알아서 해줄걸!)
- 앱을 hydrate 하기 위해서 React는 컴포넌트의 로직을 사전에 서버에서 만들어진 HTML에 붙인다(attach).
- hydration을 통해 서버에서 만들어진 초기의 HTML 스냅샷을 브라우저에서 실행되는 fully interactive한 앱으로 변환한다.

- `hydrateRoot`에 전달한 React 트리는 서버에서 만들었던 것과 동일한 출력을 생성해야 한다.
    - 서버에서 렌더링한 결과와 클라이언트에서 최초로 렌더링한 결과가 같아야 한다는 뜻!
    - WHY? 사용자 경험을 위해서!
    - 앱이 더 빨리 로드되는 환상을 보여주기 위해서 HTML 스냅샷을 JS 코드가 로드되기 전에 보여주는 것인데, 갑자기 다른 내용을 보여주면 그 환상이 와장창.. 깨짐 💥
- 주로 이런 이유들로 인해 hydration 에러가 발생한다.
    - 새 줄(newlines) 같은 추가적인 공백이 React가 생성한 HTML 주변에 있는 경우
    - 렌더링 로직에서 `typeof window !== 'undefined'`와 같은 검사를 하는 경우
    - 렌더링 로직에서 `window.matchMedia`와 같은 브라우저 전용 API를 사용하는 경우
    - 서버와 클라이언트에서 다른 데이터를 렌더링하는 경우
- React는 일부 hydration 오류에서 복구되지만, 다른 버그처럼 마찬가지로 수정해야 하는 문제이다.
- 알아서 잘 복구되면 성능 저하뿐이지만, 최악의 경우 이벤트 핸들러가 잘못된 요소에 연결될 수도 있다!

## 2) Document 전체를 hydrate하기
- 앱 전체가 React인 경우 `<html>` 태그를 포함해 JSX로 된 전체 document를 렌더링할 수 있다.
- 글로벌 변수인 document를 hydrateRoot의 첫번째 인자로 주면 됨

```jsx
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document, <App />);
```

## 3) 불가피한 hydration 불일치 에러 억제하기
- 쩔수 없이 element의 속성이나 텍스트가 서버와 클라이언트에서 서로 다를 수밖에 없다면(ex: 타임스탬프) hydration 불일치 경고를 안 보이게 할 수 있다.
- 해당 element에 `suppressHydrationWarning={true}`를 전달하면 됨
- 적용한 요소에만 작동하며(자식은 ㄴㄴ), 예외적인 escape hatch일 뿐이므로 남용 금지!
- 이제 React는 이것을 수정하려고 시도하지 않을 것이며, 미래의 업데이트까지 일관성을 잃을 수 있다.

```jsx
export default function App() {
  return (
    <h1 suppressHydrationWarning={true}>
      Current Date: {new Date().toLocaleDateString()}
    </h1>
  );
}
```

## 4) 서로 다른 클라이언트와 서버 컨텐츠 다루기
- 의도적으로 다른 내용을 렌더링하고싶다면, 클라이언트와 서버에서 서로 다른 방법으로 렌더링하면 된다.
- 클라이언트에서는 Effect 안에서 true로 할당되는 state를 사용할 수 있다.
- 이렇게 하면 처음에는 서버와 동일한 결과물을 렌더링하므로 불일치 문제를 피할 수 있다.
- But! 이렇게 하면 두 번 렌더링되므로 hydration이 느려진다.

```jsx
// index.html
<div id="root"><h1>Is Server</h1></div>

// App.js
export default function App() {
  const [isClient, setIsClient] = useState(false);

  useEffect(() => {
    setIsClient(true);
  }, []);

  return (
    <h1>
      {isClient ? 'Is Client' : 'Is Server'}
    </h1>
  );
}
```

## 5) hydrate된 root 컴포넌트 업데이트하기
- root의 hydrating이 끝난 이후에 `root.render`를 호출해서 업데이트 할 수 있다.
- `createRoot`와 다르게 HTML로 최초의 컨텐츠가 이미 렌더링 되어 있기 때문에 자주 사용할 필요는 없다.
- hydration 후에 `root.render`를 호출했을 때, 컴포넌트의 트리 구조가 이전에 렌더링한 구조와 일치한다면 React는 그 상태를 유지한다.
- 이것도 createRoot에서와 마찬가지로 이미 hydrate된 root에 `root.render`를 호출하는 것은 흔한 일이 아니다. 컴포넌트 안에서 state를 업데이트 하도록..